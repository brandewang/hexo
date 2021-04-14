---
title: docker
date: 2088-08-08 08:08:08
tags:
---

## image
#镜像比较
1.alpine
优点: 体积小,工具较全面
缺点: musl/libc, 原生没有glibc,没有中文,无法支持jdk

2.centos
优点: 稳定,没有明显缺点
缺点: 体积较大

PS:由于alpine需要重新构建内容较多,暂偏向于使用centos

#获取镜像中的文件
1. docker save -o k8s-broker-postgresql.tar registry.kube.com/broker/k8s-broker-postgresql:1.0-SNAPSHOT
2. tar -xvf k8s-broker-postgresql.tar */*.tar
3. mkdir target
4. for layer in */layer.tar; do tar -xvf $layer -C target/; done;
5. ll target/



#镜像制作
1.时区设置
2.语言环境设置
3.ssh设置(容器内容提取,比较重要) docker cp可取代
4.ENTRYPOINT 优先使用入口点脚本
ADD entrypoint.sh entrypoint.sh
ENTRYPOINT ["./entrypoint.sh"]
5.tini(建议使用配合entrypoint启动)
ENTRYPOINT ["/tini", "--", "/entrypoint.sh"]
6.supervisor  需要安装python及相关程序较占空间, 非必要情况不建议使用

## docker

#相关配置
#/etc/docker/daemon.json
{
    "registry-mirrors": ["https://hub-mirror.c.163.com", "https://reg-mirror.qiniu.com"],
    "insecure-registries": ["my-harbor:5000"],
    "max-concurrent-downloads": 20,
    "max-concurrent-uploads": 10,
    "debug": true,
    "log-opts": {
      "max-size": "100m",
      "max-file": "5"
    }
}


#systemd 开启tcp接口
ExecStart=/usr/local/bin/dockerd --log-level=error -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock


#概念
docker-compose: docker单节点服务管理,docker-compose.yml为配置文件
docker-stack: docker swarm集群服务管理,docker-compose.yml为配置文件
docker swarm: docker集群管理,分为管理节点和工作节点,可直接使用命令行创建服务
overlay类型网络：跨主机实现容器通信
service: 容器内可直接解析名称，达到配置文件中目标主机地址可预见的目的


#swarm
##创建ingress用于service slb
docker network create --ingress --driver overlay ingress

#docker-compose
#Install Latest Stable Docker Compose Release
COMPOSEVERSION=$(curl -s https://github.com/docker/compose/releases/latest/download 2>&1 | grep -Po [0-9]+\.[0-9]+\.[0-9]+)
curl -L "https://github.com/docker/compose/releases/download/$COMPOSEVERSION/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
echo "Docker Compose Installation done"
