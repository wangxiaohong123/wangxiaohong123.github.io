---
title: JVM参数
tags:
  - JVM
categories: 基础
copyright: true
---


```shell
# 初始堆内存
-Xms512M
# 最大堆内存
-Xmx512M
# 初始年轻代内存  1.8:-XX:NewSize
-Xmn256M
# 1.8最大年轻代内存
-XX:MaxNewSize=256M
# 虚拟机栈内存
-Xss1M
# 初始元空间内存
-XX:MetaspaceSize=128M
# 最大元空间内存
-XX:MaxMetaspaceSize=128M
# 设置Eden区和两个survivor区比例，当前意思是7:1:1默认8:1:1
-XX:SurvivorRatio=7
# 配置年轻代对象躲过多少次minorGC后进入老年代，默认15次
-XX:MaxTenuringThreshold=10
# 超过多少字节的对象直接进入老年代
-XX:PretenureSizeThreshold=1048576
# 当年轻代所有对象大于老年代剩余空间时，判断之前每次进入老年代的对象大小如果都比老年代剩余内存小，就不进行老年代垃圾回收（fullGC），否则进行老年代垃圾回收   JDk1.6之后默认开启
-XX:-HandlePromotionFailure=true
# 设置jvm对新生代GC的垃圾回收器为ParNew
-XX:+UseParNewGC
# 设置jvm对老年代GC的垃圾回收期为CMS
-XX:+UseConcMarkSweepGC
# cms初始标记阶段开启多线程执行
-XX:+CMSParallelInitialMarkEnabled
# 在cms重新标记阶段之前执行一个yongGC
-XX:+CMSScavengeBeforeRemark
# 调节ParNew回收时使用的线程数（默认与CPU核数相同）
-XX:ParallelGCThreads=4
# 当老年代剩余可用空间不足92%时，进行fullGC
-XX:CMSInitiatingOccupancyFraction=92
# fullGC之后停止系统所有线程，进行老年代空间整理
-XX:+UseCMSCompactAtFullCollection
# 每多少次fullGC之后进行一次空间整理
-XX:CMSFullGCsBeforeCompaction=5

##################################G1相关##################################
# 使用G1回收器
-XX:+UseG1GC
# 设置为G1回收器时，新生代默认初始堆内存占比
-XX:G1NewSizePercent
# 
-XX:G1HeapRegionSize
# 新生代占比最多为
-XX:G1MaxNewSizePercent
# G1回收时最多消耗时间
-XX:MaxGCPauseMills
# 默认是45%，当老年代的region数超过总region数的45%，进行年轻代和老年代和大对象的混合垃圾回收
-XX:InitiatingHeapOccupancyPercent
# 配置混合回收次数，默认8次
-XX:G1MixedGCCountTarget=8
# 
-XX:G1HeapWastePercent
# region中的存活对象低于多少时才可以回收，默认是85%
-XX:G1MixedGCLiveThresholdPercent
########################################################################

# 打印详细的gc日志
-XX:+PrintGCDetils
# 打印出来每次GC发生的时间
-XX:+PrintGCTimeStamps
# 设置将gc日志写入一个磁盘文件
-Xloggc:gc.log
# 禁止显示执行GC System.gc()
-XX:+DisableExplicitGC
# 在OOM时生成快照文件
-XX:+HeapDumpOnOutOfMemoryError
# 生成的OOM快照位置
-XX:HeapDumpPath=/usr/local/app/oom
# 关闭锁粗化
-XX:-EliminateLocks
# 堆外内存大小
-XX:MaxDirectMemorySize
```

查看服务启动参数：jps -lv 

模板该有参数

```shell
-Xms512M -Xmx512M -Xmn256M -Xss1M  -XX:MetaspaceSize=128M -XX:MaxMetaspaceSize=128M -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -XX:MaxTenuringThreshold=10 -XX:SurvivorRatio=8 -XX:CMSInitiatingOccupancyFaction=92 -XX:+UseCMSCompactAtFullCollection -XX:CMSFullGCsBeforeCompaction=5 -XX:+CMSParallelInitialMarkEnabled -XX:+CMSScavengeBeforeRemark -XX:+DisableExplicitGC -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -Xloggc:gc.log -XX:+HeapDumpOnOutOfMemoryError  -XX:HeapDumpPath=/usr/local/app/oom 
```

