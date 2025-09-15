---
title: ribbon
tags:
  - cloud
categories: 框架
copyright: true
---



Ribbon三剑客：ILoadbalancer、IRule、IPing。

ILoadBalancer就是RestClient底层的负载均衡器，基于BaseBalance实现，负载均衡的规则都是基于IRule。

@LoadBalanced，这个注解的意思就是把RestTemplate标志为底层采用LoadBalanceClient来执行http请求，支持负载均衡。目前这个类里啥都没有，可以去看看这个注解所在包下有什么。看看有没有LoadBalanceAutoConfiguration，真的有。但是有两个：AsyncLoadBalancerAutoConfiguration和LoadBalancerAutoConfiguration。能看出来他是支持异步负载均衡请求的。

### LoadBalancerAutoConfiguration

```java
/**
 * Auto configuration for Ribbon (client side load balancing).
 
 * 看这注释，专门为ribbon的Auto configuration
 
 * @author Spencer Gibb
 * @author Dave Syer
 * @author Will Tran
 * @author Gang Li
 */
```

```java
// 这块可能是我在controller或者service定义的RestTemplate，会被放到这个集合里
@LoadBalanced
@Autowired(required = false)
private List<RestTemplate> restTemplates = Collections.emptyList();

// 这就是最后执行完来处理一些东西
@Bean
public SmartInitializingSingleton loadBalancedRestTemplateInitializer(
    final List<RestTemplateCustomizer> customizers) {
    return new SmartInitializingSingleton() {
        @Override
        public void afterSingletonsInstantiated() {
            // 然后遍历RestTemplate在定制化什么的，先猜成这样
            for (RestTemplate restTemplate : LoadBalancerAutoConfiguration.this.restTemplates) {
                for (RestTemplateCustomizer customizer : customizers) {
                    customizer.customize(restTemplate);
                }
            }
        }
    };
}
// 下面有一个类restTemplateCustomizer
// 能看出来就是给restTemplate设置了拦截器
// 拦截器就是上面的LoadBalancerInterceptor
@Bean
@ConditionalOnMissingBean
public RestTemplateCustomizer restTemplateCustomizer(
    final LoadBalancerInterceptor loadBalancerInterceptor) {
    return new RestTemplateCustomizer() {
        @Override
        public void customize(RestTemplate restTemplate) {
            List<ClientHttpRequestInterceptor> list = new ArrayList<>(
                restTemplate.getInterceptors());
            list.add(loadBalancerInterceptor);
            restTemplate.setInterceptors(list);
        }
    };
}
// 下面就是retry相关的代码，先不看
```

### LoadBalancerInterceptor

LoadBalanceInterceptor继承了ClientHttpRequestInterceptor

```java
@Override
public ClientHttpResponse intercept(final HttpRequest request, final byte[] body,
                                    final ClientHttpRequestExecution execution) throws IOException {
    // 获取uri:http://serviceA/hello
    final URI originalUri = request.getURI();
    // 获取serviceA
    String serviceName = originalUri.getHost();
    Assert.state(serviceName != null, "Request URI does not contain a valid hostname: " + originalUri);
    // 这行代码就是把请求封装了
    return this.loadBalancer.execute(serviceName, requestFactory.createRequest(request, body, execution));
}
```

### RibbonLoadBalanceClient

在声明LoadBalanceInterceptor的时候把RibbonLoadBalanceClient给注入进去了，但是在LoadBalancerAutoConfiguration里没看到RibbonLoadBalanceClient，所以猜测RibbonLoadBalanceClient是在别的地方被声明成了一个bean，先找到RibbonLoadBalanceClient所在的项目包，猜测是在哪个autoConfiguration或者Configuration里面，在同目录下找到了一个RibbonAutoConfiguration，里面看到了初始化顺序：

```java
// 可以看出来RibbonAutoConfiguration要在EurekaClientAutoConfiguration初始化之后，LoadBalancerAutoConfiguration初始化之前
@AutoConfigureAfter(name = "org.springframework.cloud.netflix.eureka.EurekaClientAutoConfiguration")
@AutoConfigureBefore({LoadBalancerAutoConfiguration.class, AsyncLoadBalancerAutoConfiguration.class})
```

然后在下面看到了LoadBalancerClient的声明：

```java
@Bean
@ConditionalOnMissingBean(LoadBalancerClient.class)
public LoadBalancerClient loadBalancerClient() {
    return new RibbonLoadBalancerClient(springClientFactory());
}
```

看看execute代码：

```java
@Override
public <T> T execute(String serviceId, LoadBalancerRequest<T> request) throws IOException {
    // ILoadBalancer
    ILoadBalancer loadBalancer = getLoadBalancer(serviceId);
    // 根据ILoadBalancer获取所有服务id的ip和端口然后在选择一个
    Server server = getServer(loadBalancer);
    if (server == null) {
        throw new IllegalStateException("No instances available for " + serviceId);
    }
    RibbonServer ribbonServer = new RibbonServer(serviceId, server, isSecure(server, serviceId), serverIntrospector(serviceId).getMetadata(server));

    return execute(serviceId, ribbonServer, request);
}
```

在往下跟，通过NamedContextFactory的getContext来获取AnnotationConfigApplicationContext，可以看到每个服务实例的name对应一个AnnotationConfigApplicationContext，在根据context.getBean(type)，根据类取货bean的实例然后返回ILoadBalancer的实例。

```java
private Map<String, AnnotationConfigApplicationContext> contexts = new ConcurrentHashMap<>();
```

返回的ILoadBalancer肯定也是Spring的bean，在RibbonClientConfiguration里声明的@Bean，返回的是ZoneAwareLoadBalancer，他的父类是DynamicServerListLoadBalancer，DynamicServerListLoadBalancer的父类是BaseLoadBalance，明朗了啊。

在DynamicServerListLoadBalancer类中的初始化方法看到了两个方法：

```java
void restOfInit(IClientConfig clientConfig) {
    boolean primeConnection = this.isEnablePrimingConnections();
    // turn this off to avoid duplicated asynchronous priming done in BaseLoadBalancer.setServerList()
    this.setEnablePrimingConnections(false);
    // 定时更新servletList
    enableAndInitLearnNewServersFeatrue();
	// 更新服务列表，但是这里并没有获取servletList的代码
    updateListOfServers();
    if (primeConnection && this.getPrimeConnections() != null) {
        this.getPrimeConnections()
            .primeConnections(getReachableServers());
    }
    this.setEnablePrimingConnections(primeConnection);
    LOGGER.info("DynamicServerListLoadBalancer for client {} initialized: {}", clientConfig.getClientName(), this.toString());
}
```

updateListOfServers()方法没有拉取注册表，真实的获取注册表的代码在DiscoveryEnabledNIWSServerList中的getUpdatedListOfServers的方法中。

### ping机制

ping服务的bean是

```java
@Bean
@ConditionalOnMissingBean
public IPing ribbonPing(IClientConfig config) {
    if (this.propertiesFactory.isSet(IPing.class, name)) {
        return this.propertiesFactory.get(IPing.class, config, name);
    }
    return new DummyPing();
}
```

但是DummyPing里啥都没有，直接返回true。可能不是在这里，去ribbon.eureka里找找，找到了一个EurekaRibbonClientConfiguration，EurekaRibbonClientConfiguration里的NIWSDiscoveryPing是检查client的bean。这里获取了一个服务状态，如果是InstanceStatus.UP就返回true。在BaseLoadBalancer的setupPingTask方法里，每10sping一下。

### 负载均衡算法

1.  默认是PredicateBasedRule（ZoneAvoidanceRule）下的choose，轮询算法：

```java
private int incrementAndGetModulo(int modulo) {
    for (;;) {
        int current = nextIndex.get();
        // modulo是当前的机器数
        int next = (current + 1) % modulo;
        if (nextIndex.compareAndSet(current, next) && current < modulo)
            return current;
    }
}
```

2.  AvailabilityFilteringRule：先用roundRobinRule获取一个，如果失败就在获取一个，最多10次。
3.  WeightedResponseTimeRule：带权重的随机访问。
4.  BestAvailableRule：先统计，找请求次数最少的服务器。
5.  RandomRule：纯随机。

只是用ribbon的负载均衡可能会导致多次请求都打到宕机的服务上，因为ping机制只是去eureka上看服务的状态，eureka对服务宕机的感知最少需要180s呢，要保证负载均衡的可用需要使用hystrix。