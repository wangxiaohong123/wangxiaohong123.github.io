---
title: RocketMQ配置
tags:
  - RocketMQ
categories: 中间件
copyright: true
---
```shell
# 这个是集群的名称，你整个broker集群都可以用这个名称
brokerClusterName = RaftCluster

# 这是Broker的名称，比如你有一个Master和两个Slave，那么他们的Broker名称必须是一样的，因为他们三个是一个分组，如果你有另外一组Master和两个Slave，你可以给他们起个别的名字，比如说RaftNode01
brokerName=broker-a

#0 表示 Master，>0 表示 Slave
brokerId=0

# 这个就是你的Broker监听的端口号，如果每台机器上就部署一个Broker，可以考虑就用这个端口号，不用修改
listenPort=30911

# 这里是配置NameServer的地址，如果你有很多个NameServer的话，可以在这里写入多个NameServer的地址
namesrvAddr=192.168.0.3:9876

# 下面两个目录是存放Broker数据的地方，你可以换成别的目录，类似于是/usr/local/rocketmq/node00之类的
storePathRootDir=/usr/local/rocketmq/store/
# commitLog存储路径
storePathCommitLog=/usr/local/rocketmq/store/commitlog
# 消费队列存储路径
storePathConsumeQueue=/usr/local/rocketmq/store/consumequeue
# 消息索引存储路径
storePathIndex=/usr/local/rocketmq/store/index
# checkpoint 文件存储路径
storeCheckpoint=/usr/local/rocketmq/store/checkpoint
# abort 文件存储路径
abortFile=/usr/local/rocketmq/store/abort

# 这个是非常关键的一个配置，就是是否启用DLeger技术，这个必须是true
enableDLegerCommitLog=true

# 这个一般建议和Broker名字保持一致，一个Master加两个Slave会组成一个Group
dLegerGroup=broker-a

# 这个很关键，对于每一组Broker，你得保证他们的这个配置是一样的，在这里要写出来一个组里有哪几个Broker，比如在这里假设有三台机器部署了Broker，要让他们作为一个组，那么在这里就得写入他们三个的ip地址和监听的端口号
dLegerPeers=n0-192.168.0.3:40911;n1-192.168.0.5:40912;n2-192.168.0.6:40913

# 这个是代表了一个Broker在组里的id，一般就是n0、n1、n2之类的，这个你得跟上面的dLegerPeers中的n0、n1、n2相匹配
dLegerSelfId=n0

# 删除文件时间点，默认是凌晨4点
deleteWhen=04

# 文件保留时间，默认48小时
fileReservedTime=120

# 这个是发送消息的线程数量，一般建议你配置成跟你的CPU核数一样，比如我们的机器假设是24核的，那么这里就修改成24核
sendMessageThreadPoolNums=4

#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数。由于是4个broker节点，所以设置为4
defaultTopicQueueNums=4

#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true

#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true

# commitLog每个文件的大小，默认是1G
mapedFileSizeCommitLog=1073741824

# ConsumeQueue每个文件默认存30W条，根据业务情况调整
mapedFileSizeConsumeQueue=300000

# destroyMapedFileIntervalForcibly=120000

# redeleteHangedFileInterval=120000

# 检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88

# 限制的消息大小
maxMessageSize=65536
flushCommitLogLeastPages=4
flushConsumeQueueLeastPages=2
flushCommitLogThoroughInterval=10000
flushConsumeQueueThoroughInterval=60000

# Broker 的角色
# ASYNC_MASTER 异步复制Master
# SYNC_MASTER 同步双写Master
# SLAVE
brokerRote=ASYNC_MASTER

#刷盘方式
#- ASYNC_FLUSH 异步刷盘
#- SYNC_FLUSH 同步刷盘
flushDiskType=ASYNC_FLUSH

# checkTransactionMessageEnable=false

# 重试时间间隔
messageDelayLevel=1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h
```

