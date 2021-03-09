---
title: k8s
date: 2088-08-08 08:08:08
tags:
---

# 1. 准备

## 1.1 软件版本及系统配置

### system

|hostname|platform|ip|configuration|
|---|---|---|---|
|k8s-etcd01|centos7.7|10.55.3.51|2c4g16g|
|k8s-etcd02|centos7.7|10.55.3.52|2c4g16g|
|k8s-etcd03|centos7.7|10.55.3.53|2c4g16g|
|k8s-master01|centos7.7|10.55.3.54|2c4g16g|
|k8s-master02|centos7.7|10.55.3.55|2c4g16g|
|k8s-master03|centos7.7|10.55.3.56|2c4g16g|
|k8s-node01|centos7.7|10.55.3.57|2c4g16g|
|k8s-node02|centos7.7|10.55.3.58|2c4g16g|

### k8s component

|name|version|
|---|---|
|etcd|3.4.13-0|
|docker|19.03.13|
|kubernetes|1.19.3-0|
|pause|3.2|
|keepalived|1.3.5|
|haproxy|1.5.18|


### certificates 

|location|Issuer|Owner|host|type|
|---|---|---|---|---|
|ca.crt|kubernetes|kubernetes|apiserver|ca|
|apiserver.crt|kubernetes|kube-apiserver|Master|server|
|apiserver-kubelet-client.crt|kubernetes|kube-apiserver-kubelet-client|Master|client|
|front-proxy-ca.crt|front-proxy-ca|front-proxy-ca|Master|ca|
|front-proxy-client.crt|front-proxy-ca|front-proxy-client|Master|client|
|apiserver-etcd-client.crt|etcd-ca|kube-apiserver-etcd-client|apiserver|client|
|./etcd/ca.crt|etcd-ca|etcd-ca|etcd server|ca|
|./etcd/healthcheck-client.crt|etcd-ca|kube-etcd-healthcheck-client|etcd server|client|
|./etcd/server.crt|etcd-ca|kube-node1|etcd server|server, client|
|./etcd/peer.crt|etcd-ca|kube-node1|etcd server|server, client|

https://kubernetes.io/docs/setup/best-practices/certificates/
## 1.2 环境配置

```
关闭防火墙：
$ systemctl stop firewalld
$ systemctl disable firewalld

关闭selinux：
$ sed -i 's/enforcing/disabled/' /etc/selinux/config  # 永久
$ setenforce 0  # 临时

关闭swap：
$ swapoff -a  # 临时
$ vim /etc/fstab  # 永久

设置主机名：
$ hostnamectl set-hostname <hostname>

在所有集群节点添加hosts：
$ cat >> /etc/hosts << EOF
10.55.3.54 k8s-master01
10.55.3.55 k8s-master02
10.55.3.56 k8s-master03
10.55.3.57 k8s-node01
10.55.3.58 k8s-node02
10.55.3.59 k8s-node03
EOF

将桥接的IPv4流量传递到iptables的链：
$ cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
$ sysctl --system  # 生效
sysctl -p /etc/sysctl.d/k8s.conf

时间同步：
$ yum install ntpdate -y
$ ntpdate ntp.aliyun.com

```

## 1.3 docker 安装

```
安装yum工具
yum install -y yum-utils

添加docker-ce yum源
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum makecache fast

安装docker-ce
yum -y install docker-ce

#docekr配置文件
配置镜像加速
kubelet配置中cgroup-driver需要与docker一致，不然会导致kubelet无法启动，k8s推荐使用systemd.
cat > /etc/docker/daemon.json << EOF
{
  "registry-mirrors": ["https://b9pmyelo.mirror.aliyuncs.com"],
  "exec-opts": ["native.cgroupdriver=systemd"]
}
EOF

#启动服务
systemctl enable docker-ce --now

#查看信息
docker info
```

## 1.4 安装kubeadm, kubelet和kubectl

```
#添加k8s的yum源
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

#查看可选版本
yum list kubeadm --showduplicates

#安装指定版本
yum install -y kubelet-1.19.3-0 kubeadm-1.19.3-0 kubectl-1.19.3-0

#kubectl命令行补全
source <(kubectl completion bash)
```

# 2. k8s集群安装

## 2.1 kubeadm单主模式

### 2.1.1 部署Kubernetes Master
https://kubernetes.io/zh/docs/reference/setup-tools/kubeadm/kubeadm-init/#config-file
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#initializing-your-control-plane-node
#在master(10.55.3.54)主机上执行
```
$ kubeadm init \
  --apiserver-advertise-address=10.55.3.54 \
  --image-repository registry.aliyuncs.com/google_containers \
  --kubernetes-version v1.19.3 \
  --service-cidr=10.96.0.0/12 \
  --pod-network-cidr=10.244.0.0/16 \
  --ignore-preflight-errors=all
```

- --apiserver-advertise-address 集群通告地址
- --image-repository  由于默认拉取镜像地址k8s.gcr.io国内无法访问，这里指定阿里云镜像仓库地址
- --kubernetes-version K8s版本，与上面安装的一致
- --service-cidr 集群内部虚拟网络，Pod统一访问入口
- --pod-network-cidr Pod网络，，与下面部署的CNI网络组件yaml中保持一致

或者使用配置文件引导：

```
$ vi kubeadm.conf
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
kubernetesVersion: v1.18.0
imageRepository: registry.aliyuncs.com/google_containers
networking:
  podSubnet: 10.244.0.0/16
  serviceSubnet: 10.96.0.0/12

$ kubeadm init --config kubeadm.conf --ignore-preflight-errors=all
```

### 2.1.2 加入 kubernetes Node
在192.168.31.62/63（Node）执行。

向集群添加新节点，执行在kubeadm init输出的kubeadm join命令：

```
$ kubeadm join 192.168.31.61:6443 --token esce21.q6hetwm8si29qxwn \
    --discovery-token-ca-cert-hash sha256:00603a05805807501d7181c3d60b478788408cfe6cedefedb1f97569708be9c5
```

默认token有效期为24小时，当过期之后，该token就不可用了。这时就需要重新创建token，操作如下：

```
$ kubeadm token create
$ kubeadm token list
$ openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
63bca849e0e01691ae14eab449570284f0c3ddeea590f8da988c07fe2729e924

$ kubeadm join 192.168.31.61:6443 --token nuja6n.o3jrhsffiqs9swnu --discovery-token-ca-cert-hash sha256:63bca849e0e01691ae14eab449570284f0c3ddeea590f8da988c07fe2729e924
```

或者直接命令快捷生成
```
kubeadm token create --print-join-command
```
https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-join/
## 2.2 kubedam多主模式(stacked etcd topology)

不推荐! 果一个节点宕机，etcd成员和控制平面实例都将丢失，冗余性也会受到影响.

## 2.3 kubedam多主模式(External etcd topology)
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/setup-ha-etcd-with-kubeadm/
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/
### 2.3.1 External etcd nodes

1.etcd以kubelet静态pod的方式启动，覆盖原始优先级，写入service配置文件, 使用阿里云pause镜像，
```
cat << EOF > /etc/systemd/system/kubelet.service.d/20-etcd-service-manager.conf
[Service]
ExecStart=
#Replace "systemd" with the cgroup driver of your container runtime. The default value in the kubelet is "cgroupfs".
ExecStart=/usr/bin/kubelet --address=127.0.0.1 --pod-manifest-path=/etc/kubernetes/manifests --cgroup-driver=systemd --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.2
Restart=always
EOF

systemctl daemon-reload
systemctl restart kubelet
```


2.在k8s-etcd0中为etcd集群各主机生成kubeadm配置文件
```
# Update HOST0, HOST1, and HOST2 with the IPs or resolvable names of your hosts
export HOST0=10.55.3.51
export HOST1=10.55.3.52
export HOST2=10.55.3.53

# Create temp directories to store files that will end up on other hosts.
mkdir -p /tmp/${HOST0}/ /tmp/${HOST1}/ /tmp/${HOST2}/

ETCDHOSTS=(${HOST0} ${HOST1} ${HOST2})
NAMES=("k8s-etcd01" "k8s-etcd02" "k8s-etcd03")

for i in "${!ETCDHOSTS[@]}"; do
HOST=${ETCDHOSTS[$i]}
NAME=${NAMES[$i]}
cat << EOF > /tmp/${HOST}/kubeadmcfg.yaml
apiVersion: "kubeadm.k8s.io/v1beta2"
kind: ClusterConfiguration
etcd:
    local:
        serverCertSANs:
        - "${HOST}"
        peerCertSANs:
        - "${HOST}"
        extraArgs:
            initial-cluster: ${NAMES[0]}=https://${ETCDHOSTS[0]}:2380,${NAMES[1]}=https://${ETCDHOSTS[1]}:2380,${NAMES[2]}=https://${ETCDHOSTS[2]}:2380
            initial-cluster-state: new
            name: ${NAME}
            listen-peer-urls: https://${HOST}:2380
            listen-client-urls: https://${HOST}:2379
            advertise-client-urls: https://${HOST}:2379
            initial-advertise-peer-urls: https://${HOST}:2380
EOF
done

```
3.生成etcd ca证书
```
kubeadm init phase certs etcd-ca
```
This creates two files

- /etc/kubernetes/pki/etcd/ca.crt
- /etc/kubernetes/pki/etcd/ca.key

4.为etcd各成员生成证书
```
kubeadm init phase certs etcd-server --config=/tmp/${HOST2}/kubeadmcfg.yaml
kubeadm init phase certs etcd-peer --config=/tmp/${HOST2}/kubeadmcfg.yaml
kubeadm init phase certs etcd-healthcheck-client --config=/tmp/${HOST2}/kubeadmcfg.yaml
kubeadm init phase certs apiserver-etcd-client --config=/tmp/${HOST2}/kubeadmcfg.yaml
cp -R /etc/kubernetes/pki /tmp/${HOST2}/
# cleanup non-reusable certificates
find /etc/kubernetes/pki -not -name ca.crt -not -name ca.key -type f -delete

kubeadm init phase certs etcd-server --config=/tmp/${HOST1}/kubeadmcfg.yaml
kubeadm init phase certs etcd-peer --config=/tmp/${HOST1}/kubeadmcfg.yaml
kubeadm init phase certs etcd-healthcheck-client --config=/tmp/${HOST1}/kubeadmcfg.yaml
kubeadm init phase certs apiserver-etcd-client --config=/tmp/${HOST1}/kubeadmcfg.yaml
cp -R /etc/kubernetes/pki /tmp/${HOST1}/
find /etc/kubernetes/pki -not -name ca.crt -not -name ca.key -type f -delete

kubeadm init phase certs etcd-server --config=/tmp/${HOST0}/kubeadmcfg.yaml
kubeadm init phase certs etcd-peer --config=/tmp/${HOST0}/kubeadmcfg.yaml
kubeadm init phase certs etcd-healthcheck-client --config=/tmp/${HOST0}/kubeadmcfg.yaml
kubeadm init phase certs apiserver-etcd-client --config=/tmp/${HOST0}/kubeadmcfg.yaml
# No need to move the certs because they are for HOST0

# clean up certs that should not be copied off this host
find /tmp/${HOST2} -name ca.key -type f -delete
find /tmp/${HOST1} -name ca.key -type f -delete
```
5. 复制证书及kubeadm配置文件到各节点
```
USER=root
HOST=${HOST1}
scp -r /tmp/${HOST}/* ${USER}@${HOST}:
ssh ${USER}@${HOST}
USER@HOST $ sudo -Es
root@HOST $ chown -R root:root pki
root@HOST $ mv pki /etc/kubernetes/
```
6.确认预期文件存在
The complete list of required files on $HOST0 is:
```
/tmp/${HOST0}
└── kubeadmcfg.yaml
---
/etc/kubernetes/pki
├── apiserver-etcd-client.crt
├── apiserver-etcd-client.key
└── etcd
    ├── ca.crt
    ├── ca.key
    ├── healthcheck-client.crt
    ├── healthcheck-client.key
    ├── peer.crt
    ├── peer.key
    ├── server.crt
    └── server.key
```
On $HOST1:
```
$HOME
└── kubeadmcfg.yaml
---
/etc/kubernetes/pki
├── apiserver-etcd-client.crt
├── apiserver-etcd-client.key
└── etcd
    ├── ca.crt
    ├── healthcheck-client.crt
    ├── healthcheck-client.key
    ├── peer.crt
    ├── peer.key
    ├── server.crt
    └── server.key
```
On $HOST2
```
$HOME
└── kubeadmcfg.yaml
---
/etc/kubernetes/pki
├── apiserver-etcd-client.crt
├── apiserver-etcd-client.key
└── etcd
    ├── ca.crt
    ├── healthcheck-client.crt
    ├── healthcheck-client.key
    ├── peer.crt
    ├── peer.key
    ├── server.crt
    └── server.key
```
7.在集群各主机创建etcd静态pod清单
```
root@HOST0 $ kubeadm init phase etcd local --config=/tmp/${HOST0}/kubeadmcfg.yaml
root@HOST1 $ kubeadm init phase etcd local --config=/home/ubuntu/kubeadmcfg.yaml
root@HOST2 $ kubeadm init phase etcd local --config=/home/ubuntu/kubeadmcfg.yam
```
8.检查集群状态
```
docker exec -it k8s_etcd_etcd-k8s-etcd01_kube-system_5b4fa47f9dc248abb9e63f9ee8de2378_0 etcdctl --cert /etc/kubernetes/pki/etcd/peer.crt --key /etc/kubernetes/pki/etcd/peer.key --cacert /etc/kubernetes/pki/etcd/ca.crt --endpoints https://10.55.3.52:2379 endpoint status --cluster
docker exec -it k8s_etcd_etcd-k8s-etcd01_kube-system_5b4fa47f9dc248abb9e63f9ee8de2378_0 etcdctl --cert /etc/kubernetes/pki/etcd/peer.crt --key /etc/kubernetes/pki/etcd/peer.key --cacert /etc/kubernetes/pki/etcd/ca.crt --endpoints https://10.55.3.52:2379 endpoint health --cluster
```

### 2.3.2 Load Balancing

1.在master节点安装keepalived
```
yum install keepalived -y

#master
cat > /etc/keepalived/keepalived.conf << EOF
global_defs {
    router_id LVS_DEVEL
}
vrrp_script check_apiserver {
  script "/etc/keepalived/check_apiserver.sh"
  interval 3
  weight -2
  fall 10
  rise 2
}

vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 50
    priority 101
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        10.55.3.50
    }
    track_script {
        check_apiserver
    }
}
EOF

#backup
cat > /etc/keepalived/keepalived.conf << EOF
global_defs {
    router_id LVS_DEVEL
}
vrrp_script check_apiserver {
  script "/etc/keepalived/check_apiserver.sh"
  interval 3
  weight -2
  fall 10
  rise 2
}

vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    virtual_router_id 50
    priority 100
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        10.55.3.50
    }
    track_script {
        check_apiserver
    }
}

systemctl enable keepalived --now
```

2.在master节点安装haproxy
```
yum install -y haproxy

cat > /etc/haproxy/haproxy.cfg << EOF
#---------------------------------------------------------------------
# Global settings
#---------------------------------------------------------------------
global
    log /dev/log local0
    log /dev/log local1 notice
    daemon

#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#---------------------------------------------------------------------
defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option http-server-close
    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 1
    timeout http-request    10s
    timeout queue           20s
    timeout connect         5s
    timeout client          20s
    timeout server          20s
    timeout http-keep-alive 10s
    timeout check           10s


#admin web
listen  admin_stats
    bind 0.0.0.0:10080
    mode http
    log 127.0.0.1 local0 err
    stats refresh 30s
    stats uri /status
    stats realm welcome login\ Haproxy
    stats auth admin:123456
    stats hide-version
    stats admin if TRUE

#---------------------------------------------------------------------
# apiserver frontend which proxys to the masters
#---------------------------------------------------------------------
frontend apiserver
    bind *:8443
    mode tcp
    option tcplog
    default_backend apiserver

#---------------------------------------------------------------------
# round robin balancing for apiserver
#---------------------------------------------------------------------
backend apiserver
    option httpchk GET /healthz
    http-check expect status 200
    mode tcp
    option ssl-hello-chk
    balance     roundrobin
        server k8s-master01 10.55.3.54:6443 check
        server k8s-master02 10.55.3.55:6443 check
        server k8s-master03 10.55.3.56:6443 check
EOF

systemctl enable haproxy --now
```

### 2.3.3 Set up the first control plane node
从etcd节点复制证书
```
/etc/kubernetes/pki/etcd/ca.crt
/etc/kubernetes/pki/apiserver-etcd-client.crt
/etc/kubernetes/pki/apiserver-etcd-client.key
```
创建配置文件，并初始化k8s集群
```
cat > kubeadm-config.yaml << EOF
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
kubernetesVersion: v1.19.3
imageRepository: registry.aliyuncs.com/google_containers
networking:
  podSubnet: 10.244.0.0/16
  serviceSubnet: 10.96.0.0/12
controlPlaneEndpoint: "k8s-apiserver-lb:8443"
etcd:
    external:
        endpoints:
        - https://10.55.3.51:2379
        - https://10.55.3.52:2379
        - https://10.55.3.53:2379
        caFile: /etc/kubernetes/pki/etcd/ca.crt
        certFile: /etc/kubernetes/pki/apiserver-etcd-client.crt
        keyFile: /etc/kubernetes/pki/apiserver-etcd-client.key
EOF

kubeadm init --config kubeadm-config.yaml --upload-certs
```

### 2.3.4 Steps for the rest of the control plane nodes 

加入剩余节点，完成该步骤所有主节点初始化完毕后，再执行后续操作

```
kubeadm join k8s-apiserver-lb:8443 --token g7yiu8.zh6xe28pgr4hhsi2     --discovery-token-ca-cert-hash sha256:0cf1b896ae90170efce09198532e52fabfdb4334918cb07b0924033938896255     --control-plane --certificate-key 7e943e9ad8ec0b33dbaa4307569e713f20c8754521b68725780ed54147d32cb
```

### 2.3.5 Install workers 

```
kubeadm join k8s-apiserver-lb:8443 --token u7gdni.ss8dr3knhzmd58l6     --discovery-token-ca-cert-hash sha256:8e09ca30cbeaa2c043b8e1193e7c2ba64eb47a7e89a361d7f511611668e274b5
```

### 2.3.6 kube-controller-manager  kubelet自动续签设置
/etc/kubernetes/manifests/kube-controller-manager.yaml
    - --cluster-signing-duration=87600h0m0s
    - --feature-gates=RotateKubeletServerCertificate=true

/var/lib/kubelet/config.yaml
rotateCertificates: true

# 3. Addon

## k8s addons
|name|version|
|---|---|
|fannel|v0.13.1-rc1|
|calico|v3.17.1|
|helm|v3.4.2|
|coredns|1.8.0|
|dashboard|v2.1.0|
|metrics-server|v0.3.7|
|ingress-nginx|0.30.0|
|traefik|v2.0.7|


## 3.1 部署容器网络(CNI)
CNI (Container Network Interface, 容器网络接口): 是一个容器网络规范，Kubernetes网络采用的就是这个CNI规范，负责初始化infra容器的网络设备。

CNI 二进制程序默认路径： /opt/cni/bin/
项目地址: https://github.com/containernetworking/plugins/releases

CNI 配置文件默认路径：/etc/cni/net.d

### flannel
- 项目地址：https://github.com/coreos/flannel
- k8s-v1.17+配置清单: https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
- 网络模式: hostgateway、vxlan

#更改相关配置
```
  net-conf.json: |
    {
      "Network": "172.8.0.0/16",
      "Backend": {
        "Type": "vxlan"
      }
    }
```
#应用配置文件 (flannel不会自动生成cni二进制，需要提前准备)

```
kubectl apply -f kube-flannel.yml
```

### calico
项目地址：https://github.com/projectcalico/calico

calico包括如下重要组件：calico/node，Typha，Felix，etcd，BGP Client，BGP Route Reflector。

- calico/node：把Felix，calico client， confd，Bird封装成统一的组件作为统一入口，同时负责给其他的组件做环境的初始化和条件准备。

- Felix：主要负责路由配置以及ACLS规则的配置以及下发，它存在在每个node节点上。

- etcd：存储各个节点分配的子网信息，可以与kubernetes共用；

- BGPClient(BIRD), 主要负责把 Felix写入 kernel的路由信息分发到当前 Calico网络，确保 workload间的通信的有效性；

- BGPRoute Reflector(BIRD), 大规模部署时使用，在各个节点之间不是mesh模式，通过一个或者多个 BGPRoute Reflector 来完成集中式的路由分发；当etcd中有新的规则加入时，Route Reflector 就会将新的记录同步。
BIRD Route Reflector负责将所有的Route Reflector构建成一个完成的网络，当增减Route Reflector实例时，所有的Route Reflector监听到新的Route Reflector并与之同步交换对等的路由信息.

- Typha：在节点数比较多的情况下，Felix可通过Typha直接和Etcd进行数据交互，不通过kube-apiserver，既降低其压力。生产环境中实例数建议在3~20之间，随着节点数的增加，按照每个Typha对应200节点计算。
如果由kube-apiserver转为Typha。需要将yaml中typha_service_name 修改calico-typha，同时replicas不能为0 ，否则找不到Typha实例会报连接失败。

#部署
https://docs.projectcalico.org/getting-started/kubernetes/self-managed-onprem/onpremises

#Install Calico with Kubernetes API datastore, 50 nodes or less
1.Download the Calico networking manifest for the Kubernetes API datastore.
curl https://docs.projectcalico.org/manifests/calico.yaml -O
2.change CALICO_IPV4POOL_CIDR in calico.yaml
3.Customize the manifest if desired.
4.kubectl apply -f calico.yaml

#Install Calico with Kubernetes API datastore, more than 50 nodes
1.Download the Calico networking manifest for the Kubernetes API datastore.
curl https://docs.projectcalico.org/manifests/calico-typha.yaml -o calico.yaml
2.change CALICO_IPV4POOL_CIDR in calico.yaml
3.Modify the replica count to the desired number in the Deployment named, calico-typha.
``` bash
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: calico-typha
  ...
spec:
  ...
  replicas: <number of replicas>
```
4.Customize the manifest if desired.
5.kubectl apply -f calico.yaml

#Install Calico with etcd datastore
1.Download the Calico networking manifest for etcd.
curl https://docs.projectcalico.org/manifests/calico-etcd.yaml -o calico.yaml
2.change CALICO_IPV4POOL_CIDR in calico.yaml
3.In the ConfigMap named, calico-config, set the value of etcd_endpoints to the IP address and port of your etcd server.
4.Customize the manifest if desired.
5.kubectl apply -f calico.yaml


## calicoctl
https://github.com/projectcalico/calicoctl

#优化网络模式，调整为BGP模式，为降低局域网网络连接数量，选取两个node作为route reflector

#调整calico模式为BGP
``` 
cat << EOF | calicoctl apply -f -
apiVersion: projectcalico.org/v3
kind: IPPool
metadata:
  name: default-ipv4-ippool
spec:
  blockSize: 26
  cidr: 172.8.0.0/16
  ipipMode: Never
  natOutgoing: true
EOF
```

#禁用全局Full-mesh
``` bash
cat << EOF | calicoctl apply -f -
apiVersion: projectcalico.org/v3
kind: BGPConfiguration
metadata:
  name: default
spec:
 logSeverityScreen: Info
 nodeToNodeMeshEnabled: false
 asNumber: 64512
EOF
```

#导出节点1和几点2的配置并修改
``` bash
kubectl label node gihtg-k8s-master01 route-reflector=true

calicoctl get node gihtg-k8s-master01 -o yaml > rr01.yaml

apiVersion: projectcalico.org/v3
kind: Node
metadata:
  creationTimestamp: null
  name: gihtg-k8s-master01
  labels:
    # 增加标签，将rr标签置为true 。建议在kubectl label 中添加，可以保持一致以免发生歧义。
    i-am-a-route-reflector: true
spec:
  bgp:
    ipv4Address: 192.168.2.1/24
    # 增加标签，确保同一个反射簇配置ID一致，即rr01与rr02一致，用于冗余和防环
    routeReflectorClusterID: 224.0.0.1
  orchRefs:
  - nodeName: 192.168.2.1
    orchestrator: k8s

calicoctl apply -f rr01.yaml
```
#建立BGP node 与 Route Reflector 的连接规则
``` 
$ cat << EOF | calicoctl create -f -
kind: BGPPeer
apiVersion: projectcalico.org/v3
metadata:
  name: peer-to-rrs
spec:
  # 规则1：普通 bgp node 与 rr 建立连接
  nodeSelector: !has(i-am-a-route-reflector)
  peerSelector: has(i-am-a-route-reflector)

---
kind: BGPPeer
apiVersion: projectcalico.org/v3
metadata:
  name: rr-mesh
spec:
  # 规则2：route reflectors 之间也建立连接
  nodeSelector: has(i-am-a-route-reflector)
  peerSelector: has(i-am-a-route-reflector)
EOF
```
#查看bgp连接，及各个节点的路由
```
calicoctl node status
route -n
```

## 3.2 Helm
项目地址: https://github.com/helm/helm
下载helm客户端: https://get.helm.sh/helm-v3.4.2-linux-amd64.tar.gz
#安装helm-push plugin
helm plugin install https://github.com/chartmuseum/helm-push
#helm env 系统环境变量，包括设置最大revision history
https://helm.sh/docs/helm/helm/

## 3.3 CoreDNS
helm repo add coredns https://coredns.github.io/helm
helm pull coredns/coredns
#安装前修改values.yaml中的副本数、clusterip、容忍节点等参数.
helm install coredns coredns/ -n kube-system

## 3.4 Kubernetes Dashboard

```
wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.4/aio/deploy/recommended.yaml
```

默认Dashboard只能集群内部访问，修改Service为NodePort类型，暴露到外部：

$ vi recommended.yaml
```
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  ports:
    - port: 443
      targetPort: 8443
      nodePort: 30001
  selector:
    k8s-app: kubernetes-dashboard
  type: NodePort
$ kubectl apply -f recommended.yaml
$ kubectl get pods -n kubernetes-dashboard
NAME                                         READY   STATUS    RESTARTS   AGE
dashboard-metrics-scraper-6b4884c9d5-gl8nr   1/1     Running   0          13m
kubernetes-dashboard-7f99b75bf4-89cds        1/1     Running   0          13m
```
访问地址：https://NodeIP:30001

#dashboard (master01 上执行), 自签证书。
```
cd /opt/cert/
openssl genrsa -out dashboard.key 2048
openssl req -days 36000 -new -out dashboard.csr -key dashboard.key -subj '/CN=k8s-dashboard.corp.gihtg.com'
openssl x509 -req -in dashboard.csr -signkey dashboard.key -out dashboard.crt
kubectl create secret generic kubernetes-dashboard-certs --from-file=dashboard.key --from-file=dashboard.crt -n kubernetes-dashboard
```

创建service account并绑定默认cluster-admin管理员集群角色：

```
# 创建用户
$ kubectl create serviceaccount dashboard-admin -n kube-system
# 用户授权
$ kubectl create clusterrolebinding dashboard-admin --clusterrole=cluster-admin --serviceaccount=kube-system:dashboard-admin
# 获取用户Token
$ kubectl describe secrets -n kube-system $(kubectl -n kube-system get secret | awk '/dashboard-admin/{print $1}')
```
使用输出的token登录Dashboard。

创建登陆dashboard的配置文件
```
kubectl create sa dashboard-admin -n kube-system
kubectl create clusterrolebinding dashboard-admin --clusterrole=cluster-admin --serviceaccount=kube-system:dashboard-admin
ADMIN_SECRET=$(kubectl get secrets -n kube-system | grep dashboard-admin | awk '{print $1}')
DASHBOARD_LOGIN_TOKEN=$(kubectl describe secret -n kube-system ${ADMIN_SECRET} | grep -E '^token' | awk '{print $2}')
echo ${DASHBOARD_LOGIN_TOKEN}

kubectl config set-cluster kubernetes \
  --certificate-authority=/opt/cert/ca.pem \
  --embed-certs=true \
  --server=https://{{ k8s_master_vip }}:8443 \
  --kubeconfig=dashboard.kubeconfig

kubectl config set-credentials dashboard_user \
  --token=${DASHBOARD_LOGIN_TOKEN} \
  --kubeconfig=dashboard.kubeconfig

kubectl config set-context default \
  --cluster=kubernetes \
  --user=dashboard_user \
  --kubeconfig=dashboard.kubeconfig

kubectl config use-context default --kubeconfig=dashboard.kubeconfig
```

## 3.5 Metrics Server
https://github.com/kubernetes-sigs/metrics-server
```
#下载manifests
wget https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.3.7/components.yaml

#修改为国内镜像资源，添加对应args
      containers:
      - name: metrics-server
        image: lizhenliang/metrics-server:v0.3.7
        imagePullPolicy: IfNotPresent
        args:
          - --cert-dir=/tmp
          - --secure-port=4443
          - --kubelet-insecure-tls
          - --kubelet-preferred-address-types=InternalIP

#创建资源
kubectl apply -f components.yaml
```

# 4. Ingress

## 4.1 ingress-nginx
github: https://github.com/kubernetes/ingress-nginx
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.30.0/deploy/static/mandatory.yaml

- 修改国内镜像地址: lizhenliang/nginx-ingress-controller:0.30.0
- 建议直接宿主机网络暴露: hostNetwork: true
- 修改Deployment 为 DaemonSet
- 删除replicas

## 4.2 traefik
#添加helm 仓库
helm repo add traefik https://helm.traefik.io/traefik
#下载traefik
helm pull traefik/traefik
#修改values
- Daemonset方式部署
- 禁用Service
- 将hostnetwork设置为true
- 设置nodeSelector: traefik: "true"
#安装traefik
helm install traefik traefik/ -n traefik
#将指定k8s node 添加traefik为true的标签
kubectl label node gihtg-k8s-node01 traefik-true


# 5. Monitor

## 5.1 kube-state-mertics
将k8s各类资源对象暴露为prometheus指标的简单服务
- git: https://github.com/kubernetes/kube-state-metrics.git

#部署
```
git clone https://github.com/kubernetes/kube-state-metrics.git
cd kube-state-metrics
git checkout v1.9.8
cd examples/standard
#change manifests
kubectl apply -f .
```

## 5.1 kube-prometheus
https://github.com/prometheus-operator/kube-prometheus


