---
title: SpringBoot
date: 2088-08-08 08:08:08
tags:
---
#手动创建一个最简单的maven-springboot项目
##创建maven目录结构
1.新建一个项目文件夹springboot_test;
2.在项目根目录下新建代码目录src\main\java\com\yihengliu\sprintboottest\controller;
3.在项目根目录下新建资源目录src\main\resources;
结构目录：

```bahs
.
├── pom.xml
└── src
    └── main
        ├── java
        │   └── com
        │       └── yihengliu
        │           └── springboottest
        │               ├── controller
        └── resources
```
##编写pom.xml文件
在项目根目录下新建pom.xml文件，内容如下：
```bash
<?xml version="1.0" encoding="UTF-8"?>
<project>
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.yihengliu</groupId>
    <artifactId>springboot_test</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.9.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
   </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```
##编写运行类和Controller类
1.目录src/main/java/com/yihengliu/springboottest/下编写运行类SpringbootTestApplication.java
```bash
package com.yihengliu.springboottest;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringbootTestApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringbootTestApplication.class, args);
    }
}
```
2.目录src/main/java/com/yihengliu/springboottest/controller/下编写Controller类
```bash
package com.yihengliu.springboottest.controller;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * 测试controller
 *
 * @author liucheng
 **/
@RestController
public class TestController {
    @RequestMapping("/index")
    public String method() {
        return "return request";
    }
}
```

##启动测试
启动项目，在目录根目录运行mvn spring-boot:run；

运行访问 localhost:8080/index
