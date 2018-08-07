---
title: tomcat
date: 2088-08-08 08:08:08
tags:
---
# tomcat 日志调试
``` bash
原因:

    我的电脑同时使用两个jdk版本，默认1.7,eclipse使用的是1.8，,由于项目启动时有加载类需要jdk1.8的包，1.7不支持。所以导致项目在eclipse直接能够跑，而在外面的tomcat跑是就出现startup failed due to previous errors的错误.

    但是这样的提示信息问题还是表达比较含糊,下面我们开始重新理思绪,通过查看日志来分析原因。

 为了调试，我们要获得更详细的日志。可以在WEB-INF/classes目录下新建一个文件叫logging.properties


handlers = org.apache.juli.FileHandler, java.util.logging.ConsoleHandler  
  
############################################################  
# Handler specific properties.  
# Describes specific configuration info for Handlers.  
############################################################  
  
org.apache.juli.FileHandler.level = FINE  
org.apache.juli.FileHandler.directory = ${catalina.base}/logs  
org.apache.juli.FileHandler.prefix = error-debug.  
  
java.util.logging.ConsoleHandler.level = FINE  
java.util.logging.ConsoleHandler.formatter = java.util.logging.SimpleFormatter 
```
