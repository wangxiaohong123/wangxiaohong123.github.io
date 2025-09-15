---
title: 4.put流程
date: 2021-06-14 06:27:35
tags:
  - HBase
categories: 数据库
copyright: true
---

#### hbase:meta

put就是找到对应的regision，然后把数据写到HLog和对应的store里去，所以第一步就是要先拿到region信息，这个信息存到了一张meta表里，所属空间是hbase，当client端没找到region信息的时候就回去这个表里查出来region的信息，然后缓存在本地，下次直接读取本地的就可以了，但是hbase:meta也是一张表，client怎么知道这个表在那个region上呢？在zk中，zk里维护了一份meta表的信息，client先去zk上拿到meta在那，然后把数据读取出来，拿到了将要put的表的region信息，就可以进行put操作了。

hbase:meta只有一个列族，叫info，他的rowkey结构是表名+起始rowkey+region创建时间戳+这3个字段的md5，表里面有4个列，分别是info:regioninfo(包括md5值，region名称，起始rowkey，结束rowkey)、info:seqnumDuringOpen(region打开时的sequenceId)、info:server(存储Region在哪个RegionServer上)、info:serverstartcode(RegionServer的启动timestamp)。

### 开始put

找到region信息之前还需要判断是否开启了autoFlush，如果autoFlush=false(默认是true)，这个时候是不会想region发送数据的，他会在客户端缓存数据，当数据达到2M时一起推送到region，一般没人开这个，因为开了这个数据很容易就丢失。

拿到region信息之后就要把哪一行数据加锁，然后把这次的操作封装成一个**FSWALEntry**对象写入WAL的队列中，这是一个基于disruptor的无锁有界队列，消费者拿到这条数据后会把WAL日志写到缓存中，修改完成之后会释放锁，并且把结果封装成**SyncFutrue**对象发送到disruptor中，消费者拿到这条数据后就可以把WAL日志刷到hdfs里去了，对于HLog的持久化有5种机制，在调用JavaAPI的时候可以通过**put.setDurability(Durability.SYNC_WAL)**来设置：

*   SKIP_WAL：不写WAL日志，把数据写到memStore里就算成功，但是memStore里的数据可能还没写到HFile中，数据丢失的概率也非常大；
*   ASYNC_WAL：这个也是把数据写到memStore里就算成功，然后异步去写WAL；
*   SYNC_WAL：要把WAl日志写到hdfs里，但是也只是存在在hdfs的os cache中；
*   FSYNC_WAL：强制刷到hdfs的磁盘里才算成功；
*   USER_DEFAULT：默认级别，就是SYNC_WAL；

![](https://tva1.sinaimg.cn/large/008i3skNly1grticuek0sj31980s441k.jpg)

### memStore频繁full gc问题

大量的put操作会让memStore年青代频繁gc，短时间发生多次gc就会有大量的对象躲过gc并进入老年代，这样很快老年代也会满掉，频繁的发生full gc，HBase通过chunk数组的方式，每一个memStore会创建一个MemStoreLAB对象，对象里是一个2M的chunk数组，这里的2M可以和autoFlush的2M对应起来，当一个chunk数组写满后，就会申请一个新的chunk数组，这样就算老年代发生了full gc，但是使用了数组的数据结构，内存都是连续的，减少了内存碎片，第一gc的频率会比之前低很多，第二减少了gc之后的整理内存时间。但是还是会频繁的young gc，所以他把chunk放到了一个chunk池里反复利用。

### memStore生成HFile的时机

*   当memStore中的数据达到128M的时候，使用**hbase.hregion.memstore.flush.size**控制这个数值；
*   当region中的所有memStore总和超过了**hbase.hregion.memstore.block.multiplier * hbase.hregion.memstore.flush.size**时；
*   当所有memStore的总和超过了**hbase.regionserver.global.memstore.size.lower.limit * hbase.regionserver.global.memstore.size**的时候，从大到小刷，刷到不满足为止；
*   当HLog数量超过了**hbase.regionserver.maxlogs**时，选择最早的HLog来刷；
*   默认一小时刷一次；
*   通过flush命令；