---
title: docker
date: 2088-08-08 08:08:08
tags:
---

## 镜像比较

``` bash
1.alpine
优点: 体积小,工具较全面
缺点: musl/libc, 原生没有glibc,没有中文,无法支持jdk

2.centos
优点: 稳定,没有明显缺点
缺点: 体积较大

PS:由于alpine需要重新构建内容较多,暂偏向于使用centos
```

## 镜像制作

``` bash
1.时区设置
2.语言环境设置
3.ssh设置(容器内容提取,比较重要)
4.ENTRYPOINT 优先使用入口点脚本
ADD entrypoint.sh entrypoint.sh
ENTRYPOINT ["./entrypoint.sh"]
5.tini(建议使用配合entrypoint启动)
ENTRYPOINT ["/tini", "--", "/entrypoint.sh"]
6.supervisor  需要安装python及相关程序较占空间, 非必要情况不建议使用
```


## 应用启动
``` bash
#hexo
docker run  -d --name hexo -p 4000:4000  -v /opt/hexo/:/opt/hexo/ipple1986/source/_posts/ ipple1986/hexo

#gitlab

```

