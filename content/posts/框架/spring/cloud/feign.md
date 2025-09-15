---
title: feign
tags:
  - cloud
categories: 框架
copyright: true
password: wang123
message: 这里需要密码.
---



feign的参数配置：

```yml
spring:
  cloud:
    loadbalancer:
      retry:
        # 这个参数不影响下面的重试
        # 不加这个配置会多重试5次
        enable: true
ribbon:
  ConnectTimeOut: 1000
  ReadTimeOut: 1000
  # 无论请求超时还是报错都会走重试机制
  OkToRetryOnAllOperations: true
  # 失败后重试1次
  MaxAutoReties: 1
  # 失败后重试访问几次
  MaxAutoRetriesNextServer: 3
feign:
  client:
    config:
      # 单独配置
      ServiceA:
        connectTimeout: 5000
        readTimeout: 5000
        loggerLevel: full
        decode404: false
  compression:
    request:
      # 启用压缩
      enabled: true
      # 那些请求要压缩
      mime-types: text/xml,application/xml,application/json
      # 超过多少字节压缩
      min-request-size: 2048
    response:
      enabled: true

feign:
  client:
    config:
      default:
        connectTimeout: 5000
        readTimeout: 5000
        loggerLevel: full
```

核心组件：编码器和解码器、logger、contract：解释springMVC注解的、FeignClient：核心入口。

拦截器，实现RequestInterceptor接口。不是拦截发不发送请求，是修改请求参数的。

### 源码

找feign核心机制入口：

点击@EnableFeignClients注解，在点@import的class，OK看到了implements ImportBeanDefinitionRegistrar，那么重写registerBeanDefinitions这个方法就是注册bean的，那么猜测就是在这个方法里扫描所有@FeignClient注解的接口然后生成代理bean。

```java
@Override
public void registerBeanDefinitions(AnnotationMetadata metadata,
                                    BeanDefinitionRegistry registry) {
    registerDefaultConfiguration(metadata, registry);
    registerFeignClients(metadata, registry);
}
```

#### registerDefaultConfiguration()

```java
private void registerDefaultConfiguration(AnnotationMetadata metadata,
      BeanDefinitionRegistry registry) {
    // 获取EnableFeignClients所有属性的值
   Map<String, Object> defaultAttrs = metadata
         .getAnnotationAttributes(EnableFeignClients.class.getName(), true);

   if (defaultAttrs != null && defaultAttrs.containsKey("defaultConfiguration")) {
      String name;
       // 把启动类拼上"default."
      if (metadata.hasEnclosingClass()) {
         name = "default." + metadata.getEnclosingClassName();
      }
      else {
         name = "default." + metadata.getClassName();
      }
       // 使用构造函数生成BeanDefinition
      registerClientConfiguration(registry, name,
            defaultAttrs.get("defaultConfiguration"));
   }
}
```

### registerFeignClients()

首先获取了一个scanner，ClassPathScanningCandidateComponentProvider，猜想这个类就是扫描指定条件下的bean。

```java
Set<String> basePackages;

// 获取EnableFeignClients的属性，因为这个注解里是可以配置扫描路径的
Map<String, Object> attrs = metadata.getAnnotationAttributes(EnableFeignClients.class.getName());
// 根据注解的类型进行过滤的过滤器
AnnotationTypeFilter annotationTypeFilter = new AnnotationTypeFilter(FeignClient.class);
// 判断获取EnableFeignClients的clients属性是不是空
final Class<?>[] clients = attrs == null ? null : (Class<?>[]) attrs.get("clients");
if (clients == null || clients.length == 0) {
    // 把过滤器添加到扫描器中
    scanner.addIncludeFilter(annotationTypeFilter);
    // 自动生成basePackages
    // 如果没有配置value、basePackages、basePackageClasses就获取启动类所在的包路径
    basePackages = getBasePackages(metadata);
}
// 如果配置了clients属性，就走下面的，一般没有配置的
else {
    final Set<String> clientClasses = new HashSet<>();
    basePackages = new HashSet<>();
    for (Class<?> clazz : clients) {
        basePackages.add(ClassUtils.getPackageName(clazz));
        clientClasses.add(clazz.getCanonicalName());
    }
    AbstractClassTestingTypeFilter filter = new AbstractClassTestingTypeFilter() {
        @Override
        protected boolean match(ClassMetadata metadata) {
            String cleaned = metadata.getClassName().replaceAll("\\$", ".");
            return clientClasses.contains(cleaned);
        }
    };
    scanner.addIncludeFilter(
        new AllTypeFilter(Arrays.asList(filter, annotationTypeFilter)));
}
```

```java
for (String basePackage : basePackages) {
   Set<BeanDefinition> candidateComponents = scanner
         .findCandidateComponents(basePackage);
    // 遍历所有扫描到的BeanDefinition
   for (BeanDefinition candidateComponent : candidateComponents) {
      if (candidateComponent instanceof AnnotatedBeanDefinition) {
         // 验证只能是接口
         AnnotatedBeanDefinition beanDefinition = (AnnotatedBeanDefinition) candidateComponent;
         AnnotationMetadata annotationMetadata = beanDefinition.getMetadata();
         Assert.isTrue(annotationMetadata.isInterface(),
               "@FeignClient can only be specified on an interface");

         Map<String, Object> attributes = annotationMetadata
               .getAnnotationAttributes(
                     FeignClient.class.getCanonicalName());
		 // 获取服务名称
         String name = getClientName(attributes);
         registerClientConfiguration(registry, name, attributes.get("configuration"));
		 // 注册
         registerFeignClient(registry, annotationMetadata, attributes);
      }
   }
}
```

### registerFeignClient()

```java
private void registerFeignClient(BeanDefinitionRegistry registry,
      AnnotationMetadata annotationMetadata, Map<String, Object> attributes) {
   String className = annotationMetadata.getClassName();
   // 动态代理了
   BeanDefinitionBuilder definition = BeanDefinitionBuilder
         .genericBeanDefinition(FeignClientFactoryBean.class);
   validate(attributes);
   definition.addPropertyValue("url", getUrl(attributes));
   definition.addPropertyValue("path", getPath(attributes));
   String name = getName(attributes);
   definition.addPropertyValue("name", name);
   definition.addPropertyValue("type", className);
   definition.addPropertyValue("decode404", attributes.get("decode404"));
   definition.addPropertyValue("fallback", attributes.get("fallback"));
   definition.addPropertyValue("fallbackFactory", attributes.get("fallbackFactory"));
   definition.setAutowireMode(AbstractBeanDefinition.AUTOWIRE_BY_TYPE);

   String alias = name + "FeignClient";
   AbstractBeanDefinition beanDefinition = definition.getBeanDefinition();

   boolean primary = (Boolean)attributes.get("primary"); // has a default, won't be null

   beanDefinition.setPrimary(primary);

   String qualifier = getQualifier(attributes);
   if (StringUtils.hasText(qualifier)) {
      alias = qualifier;
   }

   BeanDefinitionHolder holder = new BeanDefinitionHolder(beanDefinition, className,
         new String[] { alias });
   BeanDefinitionReaderUtils.registerBeanDefinition(holder, registry);
}
```

### genericBeanDefinition(FeignClientFactoryBean.class)生成动态代理

```java
@Override
public Object getObject() throws Exception {
   FeignContext context = applicationContext.getBean(FeignContext.class);
    // 创建Feign.Builder
   Feign.Builder builder = feign(context);

    // 如果没有配置url
   if (!StringUtils.hasText(this.url)) {
      String url;
      if (!this.name.startsWith("http")) {
         url = "http://" + this.name;
      }
      else {
         url = this.name;
      }
      // 配置了path属性，就会把path属性拼到url上面去
      url += cleanPath();
      // 生成ribbon的动态代理，最最最关键的代码
      return loadBalance(builder, context, new HardCodedTarget<>(this.type,
            this.name, url));
   }
   if (StringUtils.hasText(this.url) && !this.url.startsWith("http")) {
      this.url = "http://" + this.url;
   }
   String url = this.url + cleanPath();
   Client client = getOptional(context, Client.class);
   if (client != null) {
      if (client instanceof LoadBalancerFeignClient) {
         // not lod balancing because we have a url,
         // but ribbon is on the classpath, so unwrap
         client = ((LoadBalancerFeignClient)client).getDelegate();
      }
      builder.client(client);
   }
   Targeter targeter = get(context, Targeter.class);
   return targeter.target(this, builder, context, new HardCodedTarget<>(
         this.type, this.name, url));
}
```

### loadBalance()

```java
protected <T> T loadBalance(Feign.Builder builder, FeignContext context,
      HardCodedTarget<T> target) {
   // 获取FeignClient
   Client client = getOptional(context, Client.class);
   if (client != null) {
      // 添加到builder中
      builder.client(client);
      // 获取target组件，负责生成动态代理的
      Targeter targeter = get(context, Targeter.class);
      return targeter.target(this, builder, context, target);
   }

   throw new IllegalStateException(
         "No Feign Client for loadBalancing defined. Did you forget to include spring-cloud-starter-netflix-ribbon?");
}
```

