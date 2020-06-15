---
title: harbor
date: 2088-08-08 08:08:08
tags:
---

# install
``` bash
#安装docker-compose
curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker
-compose
chmod +x /usr/local/bin/docker

#安装harbor
wget https://github.com/goharbor/harbor/releases/download/v1.10.3/harbor-offline-installer-v1.10.3.tgz
tar xvf harbor-offline-installer-v1.10.3.tgz -C /opt/
#修改配置文件 harbor.yml, 并申请证书至指定目录
hostname: harbor.corp.gihtg.com
harbor_admin_password: Gihtg@123
certificate: /opt/harbor/cert/harbor.pem
private_key: /opt/harbor/cert/harbor.key
#首次启动
./install.sh


#启停
docker-compose down
docker-compose up -d

#修改配置生效
docker-compose down
./prepare
docker-compose up -d

#清空数据
rm -rf /data/registry/ /data/database/


#自签ca 需要docker 客户端添加ca信任
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
