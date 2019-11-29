---
layout: post
title: 几条常用的性能调试的命令
date: 2017-10-27
categories: blog
tags: [jvm]
description: 性能调试第一步
---

#### jps:虚拟机进程状况工具
> 列出虚拟机正在运行的进程,显示出主类名和本地虚拟机唯一ID（LVMID）

jps -l 

    zhuhaos-MacBook-Pro:Shanbei zhuhao$ jps -l
    6003 sun.tools.jps.Jps
    5750

jps -v 输出进程启动是jvm参数

    zhuhaos-MacBook-Pro:Shanbei zhuhao$ jps -v
    6005 Jps -Dapplication.home=/Library/Java/JavaVirtualMachines/jdk1.8.0_144.jdk/Contents/Home -Xms8m
    5750  -Dfile.encoding=UTF-8 -XX:+UseConcMarkSweepGC -XX:SoftRefLRUPolicyMSPerMB=50 -da -Djna.nosys=true -Djna.boot.library.path=  -Djna.debug_load=true -Djna.debug_load.jna=true -Dsun.io.useCanonCaches=false -Djava.net.preferIPv4Stack=true -Xverify:none -XX:ErrorFile=/Users/zhuhao/java_error_in_studio_%p.log -XX:HeapDumpPath=/Users/zhuhao/java_error_in_studio.hprof -Xbootclasspath/a:../lib/boot.jar -Xms256m -Xmx1280m -XX:ReservedCodeCacheSize=240m -XX:+UseCompressedOops -Djb.vmOptionsFile=/Applications/Android Studio.app/Contents/bin/studio.vmoptions -Didea.java.redist=Bundled -Didea.home.path=/Applications/Android Studio.app/Contents -Didea.executable=studio -Didea.platform.prefix=AndroidStudio -Didea.paths.selector=AndroidStudio2.3

#### jstat:虚拟机统计信息监视工具

> 显示虚拟机各种运行状态信息，可以显示本地活着远程虚拟机进程中的类装载、内存、垃圾收集、JIT编译等运行时数据。

jstat -gc 监视java堆信息

    zhuhaos-MacBook-Pro:Shanbei zhuhao$ jstat -gc 5750 250 10
    S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT   
    16192.0 16192.0  0.0   3019.2 129728.0 45060.7   496976.0   427598.8  205560.0 196369.5 27952.0 26091.6    295    2.960  23      1.250    4.210
    16192.0 16192.0  0.0   3019.2 129728.0 45271.5   496976.0   427598.8  205560.0 196369.5 27952.0 26091.6    295    2.960  23      1.250    4.210
    16192.0 16192.0  0.0   3019.2 129728.0 45300.9   496976.0   427598.8  205560.0 196369.5 27952.0 26091.6    295    2.960  23      1.250    4.210
    16192.0 16192.0  0.0   3019.2 129728.0 45323.5   496976.0   427598.8  205560.0 196369.5 27952.0 26091.6    295    2.960  23      1.250    4.210
    16192.0 16192.0  0.0   3019.2 129728.0 45346.2   496976.0   427598.8  205560.0 196369.5 27952.0 26091.6    295    2.960  23      1.250    4.210
    16192.0 16192.0  0.0   3019.2 129728.0 47362.9   496976.0   427598.8  205560.0 196369.5 27952.0 26091.6    295    2.960  23      1.250    4.210
    16192.0 16192.0  0.0   3019.2 129728.0 47385.3   496976.0   427598.8  205560.0 196369.5 27952.0 26091.6    295    2.960  23      1.250    4.210
    16192.0 16192.0  0.0   3019.2 129728.0 47393.3   496976.0   427598.8  205560.0 196369.5 27952.0 26091.6    295    2.960  23      1.250    4.210
    16192.0 16192.0  0.0   3019.2 129728.0 47416.0   496976.0   427598.8  205560.0 196369.5 27952.0 26091.6    295    2.960  23      1.250    4.210
    16192.0 16192.0  0.0   3019.2 129728.0 47438.3   496976.0   427598.8  205560.0 196369.5 27952.0 26091.6    295    2.960  23      1.250    4.210

#### jmap:制作java内存映射dump的工具

> 用来生成堆转储快照（heapdump或者dump文件），还可以查询finalize执行队列、java堆和永久代的详细信息（空间使用率和使用的收集器等）。

java [option] vmid

#### jhat:配合jmap分析dump文件

> jhat内置了一个微型的HTTP/HTML服务器，生成dump文件的分析结果可以在浏览器中查看。

一般不会直接使用这个命令，因为这是一个耗时而且消耗硬件的过程，并且分析功能比较简陋，有更好的工具。

#### jstack:堆栈跟踪

> 用来生成虚拟机当前的线程快照（threaddump或者javacore）

县城快照指的是每一条线程正在执行的方法的堆栈的集合。它的目的是为了分析线程出现长时间停顿的原因。

