---
title: harbor
date: 2088-08-08 08:08:08
tags:
---
# harbor docker-compose部署
``` bash
cd /usr/local/src
curl -O https://storage.googleapis.com/harbor-releases/harbor-offline-installer-v1.5.4.tgz
tar xvf harbor-offline-installer-v1.5.4.tgz
cd /usr/local/src/harbor
##modify harbor.cfg
ui_url_protocol = https
hostname = 10.28.50.6 #填写实际入口地址
#使用cfssl先生成完整项目ca,用ca生成harbor证书(参考cfssl)
#设置证书
ssl_cert = /data/cert/harbor.pem
ssl_cert_key = /data/cert/harbor-key.pem
#环境配置&启动
./install.sh
#启动停止删除创建docker-compose 镜像
docker-compose -f docker-compse.yml stop/start/down/up

#清空数据
rm -rf /data/registry/ /data/database/

#docker 客户端添加ca信任
/data/cert/ca.pem >> /etc/pki/tls/certs/ca-bundle.crt
#添加ca信任后必须重启docker守护进程,重新加载信任
systemd restart docker
#docekr login 生成 ~/.docker/config.json登陆信息
docker login -u admin -p Harbor123451 10.28.50.6
#推送镜像至harbor(镜像创建时docker build -t 需满足命名规则)
docker push 10.28.50.6/oms/app01:latest #10.28.50.6为harbor oms为仓库名 app01:latest为项目名:short_id(commit)
#k8s登录信息设置
#从docker客户端提取
cat > config <<EOF
{
	"auths": {
		"10.28.50.20": {
			"auth": "aGFyYm9yOkhhcmJvckAxMjM="
		}
	}
}
EOF
#获取base64码
cat config.json |base64 -w 0
#创建yaml文件
cat > docker-secret.yaml << EOF
apiVersion: v1
kind: Secret
metadata:
  name: docker-secret
  namespace: java-test
data:
  .dockerconfigjson: ewoJImF1dGhzIjogewoJCSIxMC4yOC41MC4yMCI6IHsKCQkJImF1dGgiOiAiYUdGeVltOXlPa2hoY21KdmNrQXhNak09IgoJCX0KCX0KfQo=
type: kubernetes.io/dockerconfigjson
EOF
```
