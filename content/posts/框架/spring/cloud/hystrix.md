---
title: hystrix配置
tags:
  - cloud
categories: 框架
copyright: true
---



### 默认配置

```java
// 是否允许动态扩容
hystrix.threadpool.ServiceA.allowMaximumSizeToDivergeFromCoreSize = false;
// 扩容出来的线程存活时间
hystrix.threadpool.ServiceA.keepAliveTimeMinutes = 1;
// 最大线程数
hystrix.threadpool.ServiceA.maximumSize = 10;
// 核心线程数
hystrix.threadpool.ServiceA.coreSize = 10;
// 队列大小，设置成-1使用SynchronousQueuS，就是没有队列
hystrix.threadpool.ServiceA.maxQueueSize = -1;
// 队列最多存的任务数，如果队列是10，但是这个参数是5，如果队列的任务数超过5个也会拒绝
hystrix.threadpool.ServiceA.queueSizeRejectionThreshold = 5;
// 统计请求次数的时间窗口
hystrix.threadpool.ServiceA.metrics.rollingStats.numBuckets = 10;
// 每隔10s统计一下线程池信息
hystrix.threadpool.ServiceA.metrics.rollingStats.timeInMilliseconds = 10000;
// 在numBuckets时间内超过多少请求判断是否熔断
circuitBreaker.requestVolumeThreshold = 20;
// 超过多少百分比的请求失败会打开熔断开关
circuitBreaker.errorThresholdPercentage = 50;
// 熔断器打开多久会尝试请求
circuitBreaker.sleepWindowInMilliseconds = 5000;
```

