---
title: jenkins
date: 2088-08-08 08:08:08
tags:
---

#install jenkins
docker run -d -e JAVA_OPTS=-Duser.timezone=Asia/Shanghai -v jenkins_home:/var/jenkins_home -p 8080:8080 -p 50000:50000  --name jenkins jenkins/jenkins:lts

#plugins
- Git: 拉取代码
- Git Parameter: Git参数化构建
- Pipeline: 流水线
- Kubernetes: 连接Kubernetes动态创建Slave代理
- Config File Provider: 存储配置文件
- Extended Choice Parameter: 扩展选择框参数, 支持多选
- build-name-setter: 可自定义构建完成后的名称
- Role-based Authorization Strategy: 权限管理
- folder: 创建文件夹 按文件夹管理权限
- Git plugin #git集成插件
- Maven Intergration plugin #可创建maven项目
- Timestamper #构建输出时添加时间戳
- Pipeline Utility Steps #流水线里可使用jdk提供的方法

#agent(slave)
- 静态agent：手动添加节点，并在节点运行agent注册至master
- 动态agent: 通过构建，动态在k8s或openstack中生成agent完成构建，并于构建结束时销毁

#docker image (jenkins-agent) 用于动态agent
#根据需求配置环境，其中jenkins-salve启动脚本可以从默认agent中提取，agent.jar可以在添加节点时从jenkins中生成的下载链接下载
FROM centos:7
LABEL maintainer brande

RUN yum install -y java-1.8.0-openjdk maven curl git libtool-ltdl-devel && \
    yum clean all && \
    rm -rf /var/cache/yum/* && \
    mkdir -p /usr/share/jenkins

COPY agent.jar /usr/share/jenkins/slave.jar
COPY jenkins-slave /usr/bin/jenkins-slave
COPY settings.xml /etc/maven/settings.xml
RUN chmod +x /usr/bin/jenkins-slave
COPY helm kubectl /usr/bin/

ENTRYPOINT ["jenkins-slave"]


#视图&文件夹创建
视图:dev,test,prd,common,other
文件夹:ms-dev,ms-test,ms-prd (主要用于存放同一个项目的服务，以及根据发布环境区分权限)

#权限配置
builder(global roles): overall(read),task(build),view(read)
ms-dev(item role partten:'ms-dev.*'):  task(build,read,workspace),SCM(tag)

#项目配置

