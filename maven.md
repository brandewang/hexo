---
title: MAVEN
date: 2088-08-08 08:08:08
tags:
---
**我们都知道Maven本质上是一个插件框架，它的核心并不执行任何具体的构建任务，所有这些任务都交给插件来完成，例如编译源代码是由maven- compiler-plugin完成的**

Maven有三套相互独立的生命周期，分别是clean、default和site。每个生命周期包含一些阶段（phase），阶段是有顺序的，后面的阶段依赖于前面的阶段。

1、clean生命周期：清理项目，包含三个phase。

1）pre-clean：执行清理前需要完成的工作

2）clean：清理上一次构建生成的文件

3）post-clean：执行清理后需要完成的工作

2、default生命周期：构建项目，重要的phase如下。

1）validate：验证工程是否正确，所有需要的资源是否可用。
2）compile：编译项目的源代码。  
3）test：使用合适的单元测试框架来测试已编译的源代码。这些测试不需要已打包和布署。
4）Package：把已编译的代码打包成可发布的格式，比如jar。
5）integration-test：如有需要，将包处理和发布到一个能够进行集成测试的环境。
6）verify：运行所有检查，验证包是否有效且达到质量标准。
7）install：把包安装到maven本地仓库，可以被其他工程作为依赖来使用。
8）Deploy：在集成或者发布环境下执行，将最终版本的包拷贝到远程的repository，使得其他的开发者或者工程可以共享。

3、site生命周期：建立和发布项目站点，phase如下

1）pre-site：生成项目站点之前需要完成的工作

2）site：生成项目站点文档

3）post-site：生成项目站点之后需要完成的工作

4）site-deploy：将项目站点发布到服务器

举例如下：

1、mvn clean

调用clean生命周期的clean阶段，实际执行pre-clean和clean阶段

2、mvn test

调用default生命周期的test阶段，实际执行test以及之前所有阶段

3、mvn clean install

调用clean生命周期的clean阶段和default的install阶段，实际执行pre-clean和clean，install以及之前所有阶段

#MAVEN 坐标GAV
groupId
artifactId
version

##MVN Command
```bash
#生成mvn项目
mvn archetype:generate -DgroupId=com.fruitday -DartifactId=helloworld -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false -s ~/.m2/brande-settings.xml

#生成mvn web项目
mvn archetype:generate -DgroupId=com.fruitday -DartifactId=helloworld -DarchetypeArtifactId=maven-archetype-webapp -DinteractiveMode=false -s ~/.m2/brande-settings.xml

#基本mvn编译指令
mvn clean compile package -am -P profile_name -pl project_name -s ~/.m2/my-settings.xml
#clean 
使用清理插件：maven-clean-plugin:2.x执行清理删除已有target目录
#compile  
使用资源插件：maven-resources-plugin:2.6执行资源文件的复制等
使用编译插件：maven-compiler-plugin:3.1编译所有源文件生成class文件至target\classes目录下
#package
根据pom.xml中的packing打包成对应的war包或jar包
maven-resources-plugin:2.6:resources
maven-compiler-plugin:3.1:compile
maven-resources-plugin:2.6:testResources  测试resources
maven-compiler-plugin:3.1:testCompile   测试编译
maven-surefire-plugin:2.12.4:test  测试区块
maven-war-plugin  打包操作
# -am
多模块子项目时， 表示同时处理选定模块所依赖的模块
# -P
指定生效的profile区块
# -pl
指定子project(poml.xml中module)
# -s
指定配置文件，默认为/usr/local/apache-maven-3.5.2/conf/settings.xml
配置文件主要用来定义远程仓库或镜像
```

#settingx.xml解析
```bash
<settings>
 <localRepository>${user.home}/.m2/brande-repository</localRepository>
 <servers>
  <server>
    <id>releases</id>
  </server>
  <server>
    <id>snapshots</id>
  </server>
  <server>
      <id>nexus</id>
      <username>ops</username>
      <password>wtRw68U23J0m4784frWAd3H3JpAmkn7</password>
  </server>
 </servers>
 <proxies></proxies>
  <mirrors>
    <mirror>
      <name>nexus</name>
      <id>nexus</id>
      <mirrorOf>*</mirrorOf>
            <url>http://10.28.20.102:8081/repository/maven-public/</url>
    </mirror>
  </mirrors>
  <profiles>
  </profiles>
</settings>
```

#mvn 补充
```bash
#参数-DskipTests 与 -Dmaven.test.skip=true 区别
-DskipTests，不执行测试用例，但编译测试用例类生成相应的class文件至target/test-classes下。
-Dmaven.test.skip=true，不执行测试用例，也不编译测试用例类。
```
