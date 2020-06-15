---
title: spring
date: 2088-08-08 08:08:08
tags:
---

## sprintboot

``` bash
项目生成
1.idea
2.https://start.spring.io

#dependencies
web:Spring Web

#controller
1.入口类同级创建文件夹controller
2.controller创建文件与文件内类名一致

package com.gihtg.configserver.controller;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloWorld {
    @RequestMapping("/")
    public String method() {
        return "Hello World! 呵呵";
    }
    @RequestMapping("/api")
    public String method2() {
        return "api! api!";
    }
}
```


## springcloudconfig

``` bash
#dependencies
web:Spring Web
SpringCloudConfig:Config Server


#入口类修改

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;


@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }

}

```
