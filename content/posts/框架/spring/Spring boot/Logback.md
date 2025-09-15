---
title: spring-boot-1.Logback
date: 2021-01-19 06:27:35
tags:
  - spring
categories: 框架
copyright: true
---

核心API

*   日志对象：ch.qos.logback.classic.Logger
*   日志级别：ch.qos.logback.classic.Level
*   日志管理上下文：ch.qos.logback.classic.LoggerContext
*   日志附加器：ch.qos.logback.core.Appender
*   日志过滤器：ch.qos.logback.core.filter.Filter
*   日志格式布局：ch.qos.logback.core.Layout
*   日志事件：ch.qos.logback.classic.spi.LoggingEvent
*   日志配置器：ch.qos.logback.classic.spi.Configurator
*   日志诊断上下文：org.slf4j.MDC

api级实现logback的demo

```java
public static void main(String[] args) {
    LoggerContext loggerContext = new LoggerContext();

    // 会从一个cache里取，如果不存在就创建一个新的
    Logger logger = loggerContext.getLogger(LogbackDemo.class);

    // 声明配置器
    // 配置器中会声明默认的附加器和日志格式
    BasicConfigurator configurator = new BasicConfigurator();

    configurator.configure(loggerContext);

    logger.info("info测试");
}
```

不管是log4j还是logback，他们都有自己独立的api，所以开发的时候可能由于个人喜好出现多种日志框架，所以需要一个日志的API框架，比如apache的commons-loggin，slf4j，apache的几年前已经不维护了，现在使用最多的就是slf4j。slf4j会有很多适配包，根据我们引入的不同依赖去适配。如果按照logback的类编程，在以后需要换日志框架的时候很麻烦，如果按照slf4j的接口编程，以后换日志框架可能直接改个依赖就可以了。