---
title: eureka
tags:
  - cloud
categories: 框架
copyright: true
password: wang123
message: 这里需要密码.
---



### 为什么要有注册发现中心

比如说两个服务都只部署了一台机器，这个时候随便一个http client就可以访问，但是如果有一个服务加了一台机器，还要去修改代码，自己写轮询算法，如果在加机器呢？有了注册发现中心就不会出现这些问题。

### eureka集群

比如说两台eureka服务：

```yml
# 8761向8762注册
server:
  port: 8761
eureka:
  instance:
    hostname: peer1
  client:
    serviceUrl:
      defaultZone: http://peer2:8762/eureka/
```

```yml
# 8762向8761注册
server:
  port: 8762
eureka:
  instance:
    hostname: peer2
  client:
    serviceUrl:
      defaultZone: http://peer1:8761/eureka/
```

正常服务也可以配置eureka集群参数：

```yml
eureka:
  client:
    serviceUrl:
      defaultZone: http://peer1:8761/eureka/,http://peer2:8762/eureka/
```

### 其他配置

```yml
eureka:
  instance:
    # 隔多长时间发送一次心跳，默认30s
    leaseRenewallIntervalInSeconds
    # 多长时间没收到心跳就判断下线，默认90s
    leaseExpirationDurationInSeconds
  client:
    # 其他服务多长时间去eureka上抓取注册信息，默认30s
    registryFetchIntervalSeconds
  server:
  	# 自我保护机制，eureka默认1分钟内收到的心跳低于85%就会开启自我保护模式，开启之后不会把没发心跳的服务下线
    enable-self-preservation: false
```

### 源码

查看eureka源码需要看网飞的源码，cloud的eureka server和client只是对netflex的eureka封装，加了一些注解，提供springboot的支持。

Eureka使用gradle2.10构建，安的我脑瓜子疼，改版本，删依赖才build完。打开之后看到没有几个模块。

##### eureka-server

先看看依赖-》build.gradle：

*   引入了eureka-client服务，因为集群的时候eureka-server也需要充当client注册到其他服务上去；
*   eureka-core，核心逻辑都在这；
*   jersey框架，和springMVC差不多，所以可以看出来eureka是一个web应用；
*   jetty，因为他是web应用，所以在测试时需要把服务放在web容器中跑，正常的话打成war包放在tomcat里也是一样的；
*   有一个war，里面出现了eureka-resources，意思是打包的时候把这个模块打进去，resources模块里是几个jsp，就是eureka启动后的控制台；

在看看他的web.xml，因为web应用的一些配置都在这里：

-   listener里面是一个EurekaBootStrap，这个应该就是监听服务启动做一些初始化的操作了；
-   statusFilter：状态管理的filter，filter-mapping：/*；
-   requestAuthFilter：权限相关，filter-mapping：/*；
-   rateLimitingFilter：限流用的，但是下面的filter-mapping被注掉了，说是默认关闭，如果想用的话只能修改源码，把注释拿掉了；
-   gzipEncodingEnforcingFilter：压缩编码相关，filter-mapping：/v2/apps下面的接口；
-   还有一些Jersey相关的filter和启动页；

所以可以看出来eureka就是一个restful风格的web应用。

###### EurekaBootStrap

他继承了ServletContextListener，重写contextInitialized方法，web应用的常规写法，在这个方法里主要是两个操作，初始化eureka环境和初始化eureka server上下文。

初始化配置管理器使用的是双重检查的单例。

通过ConfigurationManager拿到当前环境，使用EUREKA_PROPS_FILE加载获取文件名，然后调用loadCascadedPropertiesFromResources加载配置，默认是eureka-server.properties中，然后根据环境还会去找eureka-server-环境.properties文件中的配置去覆盖，把配置都放到ConfigurationManager去管理，通过DynamicPropertyFactory来获取。

因为eureka-server启动中，会根据配置判断，选择把自己当成client去启动，所有eureka-client启动的过程就被包括在了eureka-server中。

但是启动的时候没看到在哪注册的。太垃圾了了，在initScheduledTasks()里的instanceInfoReplicator。

core包下的resources里面的文件是接收http请求的。相当于MVC的controller。

AbstractInstanceRegistry里面有个ConcurrentHashMap<String, Map<String, Lease<InstanceInfo>>>，存的就是注册表，操作注册表的时候会加一个ReentrantReadWriteLock。注册的时候使用读锁，这样可以控制并发注册。

server端获取全量注册表在ApplicationsResource的getContainers方法。

### 多级缓存

在ResponseCacheImpl中，声明了两个缓存，一个是ConcurrentMap<Key, Value> readOnlyCacheMap只读缓存，一个是LoadingCache<Key, Value> readWriteCacheMap读写缓存，在获取注册表信息的时候先从只读缓存中找，如果没有就从读写缓存中找，读写缓存在没有就去AbstractInstanceRegistry中获取。

注册的时候会把readWriteCacheMap过期，缓存自动180s过期。每隔30s比较readOnlyCacheMap和readWriteCacheMap的值，如果不相等就把readWriteCacheMap的值给readOnlyCacheMap。

AbstractInstanceRegistry在初始化的时候会创建一个定时任务，30s一次，看一下服务实例的变更记录是否在队列停留了超过180s，如果超过了就把他从队列里删掉。

### 关闭client

关闭client的方法在DiscoveryClient的shutdown方法，就是关闭一些资源，最后会走到AbstractInstanceRegistry的internalCancel方法中，在这里先把注册表的实例删掉，在把服务实例加入最近下线的queue中，然后保存服务下线时间戳，在把实例加入最近改变的队列中，最后过期注册表缓存，这样等到30到那个线程获取注册表的时候就会看到没有缓存，然后从本地注册表抓取。

### 自我保护机制

他有一个期望值numberOfRenewsPerMinThreshold，期望上一分钟应该接收到多少次心跳，如果上一分钟实际的心跳大于期望值，还是会把失约的服务下线，在自动故障感知的方法PeerAwareInstanceRegistryImpl.openForTraffic中，计算出来的期望心跳次数。每15分钟更新一下期望值。

### 线程池

使用同一个调度线程池执行心跳和缓存刷新，默认是30s；每一分钟检测一次服务续约情况；

### 槽点：

源码里的部署环境判断使用if什么aws（亚马逊的云服务），else default，这种写法很不好扩展，应该使用策略模式；

client端注册的代码贼难找而且很绕；

代码中还有很多魔法值；

判断全量抓取注册表的时候写了一大坨if条件；

默认90s没法心跳就判定为服务下线，但是源码的bug多加了一个duration时间，也就是两个90s才会判定失效，再加上每30s才会去同步注册表，所以感知服务变更可能需要210s。

设置期望心跳值时使用的硬编码，写死的实例数*2，因为默认30s一次心跳，这样一分钟就会发两次，但是如果我把发送心跳间隔改成15s了怎么办？而且只有注册和下线更新了期望值。只有一个定时更新期望值的调度任务，默认15分钟。

### cloud对eureka的封装

使用@EnableEurekaServer注解就可以把eureka-server在springboot内嵌的网络容器里启动了，注解基于springboot的auto configuration机制。

EurekaServerAutoConfiguration中几乎把EurekaBootStrap中的启动代码和相关类中的代码全部拷贝过来去执行或者自己模仿EurekaBootStrap写的逻辑，比如说默认的配置就是单独创建了一个bean，然后去实现EurekaServerConfig，这就变成了spring boot风格的配置。主要都是用代理模式去封装eureka的代码。



@EnableEurekaClient和server一样的，简单的重构一下。因为eureka-client的代码写的一般，我只看到了cloud把注册的逻辑重构了。在cloud中客户端已启动就回去注册。