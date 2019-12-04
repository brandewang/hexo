---
title: java
date: 2088-08-08 08:08:08
tags:
---

``` bash
##jvm
-Xms 初始化内存 
-Xmx 最大内存



##进程诊断
#查看高占cpu线程
ps -mp pid -o THREAD,tid,time
#转换线程为16进制
printf "%x\n" tid
#打印线程堆栈信息
jstack pid |grep tid -A 30
```
