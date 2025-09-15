---
title: LVS
date: 2021-07-12 06:27:35
tags:
  - 秒杀
categories: 秒杀
copyright: true
---

在我第一个压测的nginx是使用的4c8g的服务器+openresty搭建的nginx代理服务器，nginx使用4个worker绑核，返回的一段静态页面，发现在QPS达到2k后CPU都跑到了2000+，100M的带宽也被占满，开始以为想要接收高并发，nginx是第一个瓶颈，没想到是带宽，但是反过来推算4c8g的nginx也是扛不住200M的带宽的，所以要想抗住高并发，高带宽和高配置的nginx是必须的，但是配置再高也会有上限，还不如搞一个nginx集群，这样就可以无限的横向扩容了。

LVS（Linux Virtual Server）：Linux虚拟机服务器，就是一种负载均衡的技术，负载均衡的缩写是SLB（server load balance），它运行在linux内核层面，4c8g的服务器+LVS每秒处理几十万的请求不是问题，他是建立在4层网络协议基础上的，像nginx就是基于7层网络协议的负载均衡服务器。目前主流的负载均衡架构都是LVS+keepalive+nginx。

### 原理

首先当LVS收到客户端的报文（建立连接、断开连接、请求数据都是报文）之后，会在本地的hash表中查看是否有这个链接对应的后台服务器，如果没有就从后端的服务器地址中通过负载均衡算法拿到一台服务器，然后通过NAT（Network Address Translation，网络地址转换）技术把报文的请求地址换成后端服务器的地址，把请求发送到后端服务器，在hash表中记录这个链接对应的后端服务器，收到后台服务器的报文之后也会通过NAT技术把报文重新定位到客户端，在LVS中是没有HTTP协议这种说法的。

可以想一下，因为大部分的请求都是请求报文比响应报文小很多，如果响应也通过LVS的话会占用很多带宽，严重降低LVS的吞吐量，所以可以使用ip隧道把响应直接从nginx返回给客户端。

![](https://tva1.sinaimg.cn/large/008i3skNly1gskwsola92j31lz0u041p.jpg)