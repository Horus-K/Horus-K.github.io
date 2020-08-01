---
title: k8s17.3二进制部署-test
date: 2020-05-04 12:01:10
tags:
categories: k8s
---

K8s简介

Kubernetes（简称k8s）是Google在2014年6月开源的一个容器集群管理系统，使用Go语言开发，用于管理云平台中多个主机上的容器化的应用，Kubernetes的目标是让部署容器化的应用简单并且高效,Kubernetes提供了资源调度、部署管理、服务发现、扩容缩容、监控，维护等一整套功能。，努力成为跨主机集群的自动部署、扩展以及运行应用程序容器的平台。 它支持一系列容器工具, 包括Docker等。

\# kubernetes

<!--more-->

# 注意 本文所有操作均在所有节点执行

本文环境

|         名称         |     配置      |
| :------------------: | :-----------: |
|    Kubernetes版本    |    v1.17.3    |
|   Cluster Pod CIDR   | 10.244.0.0/16 |
| Cluster Service CIDR | 10.250.0.0/24 |
|  kubernetes service  |  10.250.0.1   |
|     dns service      |  10.250.0.10  |

本文档主要介绍在裸机环境下以二进制的方式安装部署kubernetes

本次文档主要是对于kubernetes 1.17.3版本

内容主要有环境准备安装配置docker 升级内核 调整系统参数

# 环境准备

|   主机名    |       IP        |                        组件                        |  配置  |
| :---------: | :-------------: | :------------------------------------------------: | :----: |
| k8s-master1 | 192.168.220.101 | apiserver、controller-manager、schedule、docker-ce | 2核 4G |
| k8s-master2 | 192.168.220.102 | apiserver、controller-manager、schedule、docker-ce | 2核 4G |
|  k8s-node1  | 192.168.220.103 |        kube-proxy、kubelet、docker-ce,etcd         | 2核 4G |
|  k8s-node1  | 192.168.220.104 |        kube-proxy、kubelet、docker-ce,etcd         | 2核 4G |
|  k8s-node1  | 192.168.220.105 |        kube-proxy、kubelet、docker-ce,etcd         | 2核 4G |

以下操作除非具体说明的步骤 其他的均在所有节点执行

### ***环境变量***

此环境变量为主节点（控制节点）的主机名和IP地址

```sh
export VIP=192.168.220.100
export MASTER1_HOSTNAME=k8s-master1
export MASTER1_IP=192.168.220.101
export MASTER2_HOSTNAME=k8s-master2
export MASTER2_IP=192.168.220.102
export NODE1_HOSTNAME=k8s-node1
export NODE1_IP=192.168.220.103
export NODE1_HOSTNAME=k8s-node2
export NODE1_IP=192.168.220.104
export NODE1_HOSTNAME=k8s-node3
export NODE1_IP=192.168.220.105
```

### ***SSH免密/时间同步/主机名修改***

· SSH免密

· NTP时间同步

· 主机名修改

· 环境变量生成

· Host 解析

这里需要说一下，所有的密钥分发以及后期拷贝等都在master1上操作，因为master1做免密了

k8s集群所有的机器都需要进行host解析

```
cat >> /etc/hosts << EOF
192.168.220.101 k8s-master1
192.168.220.102 k8s-master2
192.168.220.103 k8s-node1
192.168.220.104 k8s-node2
192.168.220.105 k8s-node3
EOF
```

批量免密

\# 做免密前请修改好主机名对应的host

```
yum -y install expect
ssh-keygen -t rsa -P "" -f /root/.ssh/id_rsa
for i in 192.168.220.101 192.168.220.102 192.168.220.103 192.168.220.104 192.168.220.105;do
expect -c "
spawn ssh-copy-id -i /root/.ssh/id_rsa.pub root@$i
    expect {
        \"*yes/no*\" {send \"yes\r\"; exp_continue}
        \"*password*\" {send \"quhuixx\r\"; exp_continue}
        \"*Password*\" {send \"quhuixx\r\";}
    } "
done 
```

切记所有机器需要自行设定ntp,否则不只HA下apiserver通信有问题,各种千奇百怪的问题。

```
yum -y install ntp
systemctl enable ntpd
systemctl start ntpd
ntpdate -u time1.aliyun.com
hwclock --systohc
timedatectl set-timezone Asia/Shanghai
```

测试通信

```
for i in  k8s-master1 k8s-master2 k8s-node1 k8s-node2 k8s-node3; do ssh root@$i "hostname";done
```

### 升级内核

安装内核语言编译器（内核是用perl语言编写的）

```
yum -y install perl
```

下载密钥和yum源

导入密钥

```
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
```

安装7版本的yum源文件

```
yum -y install https://www.elrepo.org/elrepo-release-7.0-4.el7.elrepo.noarch.rpm
```

安装 ml版本 5版本的内核名字叫ml

```
yum  --enablerepo="elrepo-kernel"  -y install kernel-ml.x86_64
```

附：4.4版本内核安装方法

```
yum   --enablerepo="elrepo-kernel" -y install kernel-lt.x86_64
```

然后配置从新的内核启动

```
grub2-set-default 0
```

然后重新生成grub2.cfg 使虚拟机使用新内核启动

```
grub2-mkconfig -o /boot/grub2/grub.cfg
```

### 调整系统内核参数

```shell
cat > /etc/sysctl.conf <<EOF
net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-ip6tables=1
net.ipv4.ip_forward=1
vm.swappiness=0
vm.overcommit_memory=1
vm.panic_on_oom=0
fs.inotify.max_user_instances=8192
fs.inotify.max_user_watches=1048576
fs.file-max=52706963
fs.nr_open=52706963
net.ipv6.conf.all.disable_ipv6=1
net.netfilter.nf_conntrack_max=2310720
EOF
```

然后执行sysctl -p 使配置的内核响应参数生效

```
sysctl -p
```

### 关闭防火墙、selinux、swap、NetworkManager

```shell
systemctl stop firewalld
systemctl disable firewalld
iptables -F && iptables -X && iptables -F -t nat && iptables -X -t nat
iptables -P FORWARD ACCEPT
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
setenforce 0
sed -i 's/^SELINUX=.*/SELINUX=disabled/' /etc/selinux/config
systemctl stop NetworkManager
systemctl disable NetworkManager
```

### 修改资源限制

```shell
echo "* soft nofile 65536" >> /etc/security/limits.conf
echo "* hard nofile 65536" >> /etc/security/limits.conf
echo "* soft nproc 65536"  >> /etc/security/limits.conf
echo "* hard nproc 65536"  >> /etc/security/limits.conf
echo "* soft memlock unlimited"  >> /etc/security/limits.conf
echo "* hard memlock unlimited"  >> /etc/security/limits.conf
```

### 常用软件安装

```shell
yum -y install bridge-utils chrony ipvsadm ipset sysstat conntrack libseccomp wget tcpdump screen vim nfs-utils bind-utils wget socat telnet sshpass net-tools sysstat lrzsz yum-utils device-mapper-persistent-data lvm2 tree nc lsof strace nmon iptraf iftop rpcbind mlocate ipvsadm
```

### 加载内核ipvs模块

```shell
cat > /etc/sysconfig/modules/ipvs.modules <<EOF

#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4

EOF
chmod 755 /etc/sysconfig/modules/ipvs.modules
bash /etc/sysconfig/modules/ipvs.modules
lsmod | grep -e ip_vs -e nf_conntrack_ipv4
```

### 安装docker-ce：

添加yun源

```
tee /etc/yum.repos.d/docker-ce.repo <<-'EOF'
[aliyun-docker-ce]
name=aliyun-docker-ce
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/x86_64/stable/
enable=1
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg
EOF
```

安装docker-ce

```
yum -y install docker-ce
```

重启docker并设置为开机自启

```
systemctl daemon-reload
systemctl restart docker
systemctl enable docker
```

### 然后重启系统 验证内核是否升级成功

# 下载命令

### 下载cfssl以及cfssljson命令

此举目的是下载创建证书以及配置文件需要的命令

**PKI基础概念**

**什么是PKI？**

公开密钥基础建设（英语：Public Key Infrastructure，缩写：PKI），又称公开密钥基础架构、公钥基础建设、公钥基础设施、公开密码匙基础建设或公钥基础架构，是一组由硬件、软件、参与者、管理政策与流程组成的基础架构，其目的在于创造、管理、分配、使用、存储以及撤销数字证书。（节选维基百科）

PKI是借助CA（权威数字证书颁发/认证机构）将用户的个人身份跟公开密钥链接在一起，它能够确保每个用户身份的唯一性，这种链接关系是通过注册和发布过程实现，并且根据担保级别，链接关系可能由CA和各种软件或在人为监督下完成。PKI用来确定链接关系的这一角色称为RA（Registration Authority, 注册管理中心），RA能够确保公开密钥和个人身份链接，可以防抵赖，防篡改。在微软的公钥基础建设下，RA又被称为CA，目前大多数称为CA。

**PKI组成要素**

从上面可以得知PKI的几个主要组成要素，用户（使用PKI的人或机构），认证机构（CA，颁发证书的人或机构），仓库（保存证书的数据库）等。

**非对称加密**

本文提到的密钥均为非对称加密，有公钥和私钥之分，并且他们总是成对出现，它们的特点就是其中一个加密的数据，只能使用另一个解密，即使它本身也无法解密，也就是说公钥加密的，私钥可以解密，私钥加密的，公钥可以解密。

**证书签名请求CSR**

它是向CA机构申请数字证书时使用的请求文件，这里的CSR不是证书，而向权威证书颁发机构获得签名证书的申请，当CA机构颁发的证书过期时，你可以使用相同的CSR来申请新的证书，此时key不变。

**数字签名**

数字签名就是“非对称加密+摘要算法”，其目的不是为了加密，而是为了防抵赖或者他们篡改数据。其核心思想是：比如A要给B发送数据，A先用摘要算法得到数据的指纹，然后用A的私钥加密指纹，加密后的指纹就是A的签名，B收到数据和A的签名后，也用同样的摘要算法计算指纹，然后用A公开的公钥解密签名，比较两个指纹，如果相同，说明数据没有被篡改，确实是A发过来的数据。假设C想改A发给B的数据来欺骗B，因为篡改数据后指纹会变，要想跟A的签名里面的指纹一致，就得改签名，但由于没有A的私钥，所以改不了，如果C用自己的私钥生成一个新的签名，B收到数据后用A的公钥根本就解不开。(来源于网络)

**数字证书格式**

数字证书格式有很多，比如.pem，.cer或者.crt等。

**PKI工作流程**

下图来源于网络，上半部分最右边就是CA机构，可以颁发证书。证书订阅人，首先去申请一个证书，为了申请这个证书，需要去登记，告诉它，我是谁，我属于哪个组织，到了登记机构，再通过CSR，发送给CA中心，CA中心通过验证通过之后 ，会颁发一对公钥和私钥，并且公钥会在CA中心存一份；证书订阅人拿到证书以后，部署在服务器；

当用户访问我们的web服务器时，它会请求我们的证书，服务器会把公钥发送给我们的客户端，客户端会去验证我们证书的合法性，客户端是如何验证证书是否有效呢？CA中心会把过期证书放在CRL服务器上面 ，这个CRL服务会把所有过期的证书形成一条链条，所以他的性能非常的差，所以又推出了OCSP程序，OCSP可以就一个证书进行查询，它是否过期，浏览器可以直接去查询OCSP响应程序，但OCSP响应程序效率还不是很高，最终往往我们会把web服务器如nginx有一个ocsp开关，当我们打开这个开关以后，会有nginx服务器主动的去ocsp服务器去查询，这样大量的客户端直接从web服务器就可以直接获取到证书是否有效。

**CFSSL介绍**

**CFSSL是什么？**

cfssl是使用go编写，由CloudFlare开源的一款PKI/TLS工具。主要程序有cfssl，是CFSSL的命令行工具，cfssljson用来从cfssl程序获取JSON输出，并将证书，密钥，CSR和bundle写入文件中。

### 安装CFSSL

## 安装 cfssl 工具集

```
sudo mkdir -p /opt/k8s/{work,bin,cert} && cd /opt/k8s/work
wget https://github.com/cloudflare/cfssl/releases/download/v1.4.1/cfssl_1.4.1_linux_amd64
mv cfssl_1.4.1_linux_amd64 /opt/k8s/bin/cfssl

wget https://github.com/cloudflare/cfssl/releases/download/v1.4.1/cfssljson_1.4.1_linux_amd64
mv cfssljson_1.4.1_linux_amd64 /opt/k8s/bin/cfssljson

wget https://github.com/cloudflare/cfssl/releases/download/v1.4.1/cfssl-certinfo_1.4.1_linux_amd64
mv cfssl-certinfo_1.4.1_linux_amd64 /opt/k8s/bin/cfssl-certinfo

chmod +x /opt/k8s/bin/*
export PATH=/opt/k8s/bin:$PATH
```

# 生成证书

### 注意 本文所有操作均在master1节点执行

## 创建配置文件

CA 配置文件用于配置根证书的使用场景 (profile) 和具体参数 (usage，过期时间、服务端认证、客户端认证、加密等)：

```
cat > ca-config.json << EOF
{
  "signing": {
    "default": {
      "expiry": "87600h"
    },
    "profiles": {
      "kubernetes": {
        "usages": ["signing", "keyencipherment", "serverauth", "clientauth"],
        "expiry": "876000h"
      }
    }
  }
}
EOF
```

- signing ：表示该证书可用于签名其它证书（生成的 ca.pem 证书中CA=TRUE ）；
- server auth ：表示 client 可以用该该证书对 server 提供的证书进行验证；
- client auth ：表示 server 可以用该该证书对 client 提供的证书进行验证；
- “expiry”: “876000h” ：证书有效期设置为 100 年；

## 创建证书签名请求文件

```
cd /opt/k8s/work
cat > ca-csr.json << EOF 
{
  "CN": "kubernetes-ca",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [{
    "C": "CN",
    "ST": "BeiJing",
    "L": "BeiJing",
    "O": "k8s",
    "OU": "opsnull"
  }],
  "ca": {
    "expiry": "876000h"
  }
}
EOF
```

- CN：Common Name ：kube-apiserver 从证书中提取该字段作为请求的用户名(User Name)，浏览器使用该字段验证网站是否合法；
- O：Organization ：kube-apiserver 从证书中提取该字段作为请求用户所属的组(Group)；
- kube-apiserver 将提取的 User、Group 作为 RBAC 授权的用户标识；

注意：

1. 不同证书 csr 文件的 CN、C、ST、L、O、OU 组合必须不同，否则可能出现PEER’S CERTIFICATE HAS AN INVALID SIGNATURE 错误；
2. 后续创建证书的 csr 文件时，CN 都不相同（C、ST、L、O、OU 相同），以达到区分的目的；

## 生成 CA 证书和私钥

```
cd /opt/k8s/work
cfssl gencert -initca ca-csr.json | cfssljson -bare ca
ls ca*
```

## 分发证书文件

```sh
cd /opt/k8s/work
source /opt/k8s/bin/environment.sh
NODE_IPS=("192.168.220.101" "192.168.220.102" "192.168.220.103" "192.168.220.104" "192.168.220.105")
for node_ip in ${NODE_IPS[@]}
do
echo ">>> ${node_ip}"
ssh root@${node_ip} "mkdir -p /etc/kubernetes/cert"
scp ca*.pem ca-config.json root@${node_ip}:/etc/kubernetes/cert
done
```

## 安装和配置 kubectl

本文档只需要部署一次，生成的 kubeconfig 文件是通用的，可以拷贝到需要执行kubectl 命令的机器的 ~/.kube/config 位置；

### 下载和分发 kubectl 二进制文件

```
#一次性下载
https://dl.k8s.io/v1.17.3/kubernetes-server-linux-amd64.tar.gz
```

### 分发到所有使用 kubectl 工具的节点：(master)

将命令分发到master节点`/opt/k8s/bin` 下,并添加PATH `export PATH=/opt/k8s/bin:$PATH`

## 创建 admin 证书和私钥

kubectl 使用 https 协议与 kube-apiserver 进行安全通信，kube-apiserver 对 kubectl 请求包含的证书进行认证和授权。
kubectl 后续用于集群管理，所以这里创建具有最高权限的 admin 证书。

### 创建证书签名请求：

```
cd /opt/k8s/cert
cat > admin-csr.json << EOF 
{
    "CN":"admin",
    "hosts":[],
    "key":{
        "algo":"rsa",
        "size":2048
    },
    "names":[
        {
            "C":"CN",
            "ST":"BeiJing",
            "L":"BeiJing",
            "O":"system:masters",
            "OU":"opsnull"
        }
    ]
}
EOF
```

- O: system:masters ：kube-apiserver 收到使用该证书的客户端请求后，为请求添加组（Group）认证标识 system:masters ；
- 预定义的 ClusterRoleBinding cluster-admin 将 Group system:masters 与Role cluster-admin 绑定，该 Role 授予操作集群所需的最高权限；
- 该证书只会被 kubectl 当做 client 证书使用，所以 hosts 字段为空；

### 生成证书和私钥：

```
cd /opt/k8s/cert
cfssl gencert -ca=/opt/k8s/cert/ca.pem \
-ca-key=/opt/k8s/cert/ca-key.pem \
-config=/opt/k8s/cert/ca-config.json \
-profile=kubernetes admin-csr.json | cfssljson -bare admin
ls admin*
```

- 忽略警告消息 [WARNING] This certificate lacks a “hosts” field. ；

## 创建 kubeconfig 文件

kubectl 使用 kubeconfig 文件访问 apiserver，该文件包含 kube-apiserver 的地址和认证信息（CA 证书和客户端证书）：

```
cd /opt/k8s/work
source /opt/k8s/bin/environment.sh
# 设置集群参数
kubectl config set-cluster kubernetes \
--certificate-authority=/opt/k8s/cert/ca.pem \
--embed-certs=true \
--server=192.168.220.101:6443 \
--kubeconfig=kubectl.kubeconfig
# 设置客户端认证参数
kubectl config set-credentials admin \
--client-certificate=/opt/k8s/certcert/admin.pem \
--client-key=/opt/k8s/cert/admin-key.pem \
--embed-certs=true \
--kubeconfig=kubectl.kubeconfig
# 设置上下文参数
kubectl config set-context kubernetes \
--cluster=kubernetes \
--user=admin \
--kubeconfig=kubectl.kubeconfig
# 设置默认上下文
kubectl config use-context kubernetes --kubeconfig=kubectl.kubeconfig
```

- –certificate-authority ：验证 kube-apiserver 证书的根证书；
- –client-certificate 、 –client-key ：刚生成的 admin 证书和私钥，与kube-apiserver https 通信时使用；
- –embed-certs=true ：将 ca.pem 和 admin.pem 证书内容嵌入到生成的kubectl.kubeconfig 文件中(否则，写入的是证书文件路径，后续拷贝 kubeconfig 到其它机器时，还需要单独拷贝证书文件，不方便。)；
- –server ：指定 kube-apiserver 的地址，这里指向第一个节点上的服务；

## 分发 kubeconfig 文件

```
mkdir -p ~/.kube
cp kubectl.kubeconfig ~/.kube/config
```

# 部署 etcd 集群

etcd 是基于 Raft 的分布式 KV 存储系统，由 CoreOS 开发，常用于服务发现、共享配置以及并发控制（如 leader 选举、分布式锁等）。

kubernetes 使用 etcd 集群持久化存储所有 API 对象、运行数据。

本文档介绍部署一个三节点高可用 etcd 集群的步骤：

- 下载和分发 etcd 二进制文件；
- 创建 etcd 集群各节点的 x509 证书，用于加密客户端(如 etcdctl) 与 etcd 集群、etcd 集群之间的通信；
- 创建 etcd 的 systemd unit 文件，配置服务参数；
- 检查集群工作状态；

etcd 集群节点名称和 IP 如下：

```
k8s-01：192.168.220.100
k8s-02：192.168.220.101
k8s-03：192.168.220.102
```

注意：

1. 如果没有特殊指明，本文档的所有操作均在k8s-01 节点上执行；
2. flanneld 与本文档安装的 etcd v3.4.x 不兼容，如果要安装 flanneld（本文档使用calio），则需要将 etcd 降级到 v3.3.x 版本；

## 下载和分发 etcd 二进制文件

### 到 etcd 的 release 页面 下载最新版本的发布包：

```shell
cd /opt/k8s/work
wget https://github.com/coreos/etcd/releases/download/v3.4.3/etcd-v3.4.3-linux-amd64.tar.gz
tar -xvf etcd-v3.4.3-linux-amd64.tar.gz
```

### 分发二进制文件到集群所有节点：

```shell
cd /opt/k8s/work
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
echo ">>> ${node_ip}"
scp etcd-v3.4.3-linux-amd64/etcd* root@${node_ip}:/opt/k8s/bin
ssh root@${node_ip} "chmod +x /opt/k8s/bin/*"
done
```

## 创建 etcd 证书和私钥

### 创建证书签名请求：

```json
cd /opt/k8s/work
cat > etcd-csr.json << EOF 
{
  "CN": "etcd",
  "hosts": [
    "127.0.0.1",
    "192.168.220.103",
    "192.168.220.104",
    "192.168.220.105"
  ],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [{
    "C": "CN",
    "ST": "BeiJing",
    "L": "BeiJing",
    "O": "k8s",
    "OU": "opsnull"
  }]
}
EOF
```

- hosts  ：指定授权使用该证书的 etcd 节点 IP 列表，需要将 etcd 集群所有节点 IP都列在其中；

### 生成证书和私钥：

```shell
cd /opt/k8s/work
cfssl gencert -ca=/opt/k8s/cert/ca.pem \
-ca-key=/opt/k8s/cert/ca-key.pem \
-config=/opt/k8s/cert/ca-config.json \
-profile=kubernetes etcd-csr.json | cfssljson -bare etcd
ls etcd*pem
```

### 分发生成的证书和私钥到各 etcd 节点：

```shell
cd /opt/k8s/work
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
echo ">>> ${node_ip}"
ssh root@${node_ip} "mkdir -p /etc/etcd/cert"
scp etcd*.pem root@${node_ip}:/etc/etcd/cert/
done
```

## 创建 etcd 的 systemd unit 模板文件

```sh
cd /opt/k8s/work
source /opt/k8s/bin/environment.sh
cat > etcd.service.template <<EOF
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target
Documentation=https://github.com/coreos
[Service]
Type=notify
WorkingDirectory=${ETCD_DATA_DIR}
ExecStart=/opt/k8s/bin/etcd \\
--data-dir=${ETCD_DATA_DIR} \\
--wal-dir=${ETCD_WAL_DIR} \\
--name=##NODE_NAME## \\
--cert-file=/etc/etcd/cert/etcd.pem \\
--key-file=/etc/etcd/cert/etcd-key.pem \\
--trusted-ca-file=/etc/kubernetes/cert/ca.pem \\
--peer-cert-file=/etc/etcd/cert/etcd.pem \\
--peer-key-file=/etc/etcd/cert/etcd-key.pem \\
--peer-trusted-ca-file=/etc/kubernetes/cert/ca.pem \\
--peer-client-cert-auth \\
--client-cert-auth \\
--listen-peer-urls=https://##NODE_IP##:2380 \\
--initial-advertise-peer-urls=https://##NODE_IP##:2380 \\
--listen-client-urls=https://##NODE_IP##:2379,http://127.0.0.1:2379 \\
--advertise-client-urls=https://##NODE_IP##:2379 \\
--initial-cluster-token=etcd-cluster-0 \\
--initial-cluster=${ETCD_NODES} \\
--initial-cluster-state=new \\
--auto-compaction-mode=periodic \\
--auto-compaction-retention=1 \\
--max-request-bytes=33554432 \\
--quota-backend-bytes=6442450944 \\
--heartbeat-interval=250 \\
--election-timeout=2000
Restart=on-failure
RestartSec=5
LimitNOFILE=65536
[Install]
WantedBy=multi-user.target
EOF
```

- WorkingDirectory  、 --data-dir  ：指定工作目录和数据目录为${ETCD_DATA_DIR}  ，需在启动服务前创建这个目录；
- --wal-dir  ：指定 wal 目录，为了提高性能，一般使用 SSD 或者和  --data-dir  不同的磁盘；
- --name  ：指定节点名称，当  --initial-cluster-state  值为  new  时， --name  的参数值必须位于  --initial-cluster  列表中；
- --cert-file  、 --key-file  ：etcd server 与 client 通信时使用的证书和私钥；
- --trusted-ca-file  ：签名 client 证书的 CA 证书，用于验证 client 证书；
- --peer-cert-file  、 --peer-key-file  ：etcd 与 peer 通信使用的证书和私钥；
- --peer-trusted-ca-file  ：签名 peer 证书的 CA 证书，用于验证 peer 证书；

### 为各节点创建和分发 etcd systemd unit 文件

替换模板文件中的变量，为各节点创建 systemd unit 文件：

```shell
cd /opt/k8s/work
source /opt/k8s/bin/environment.sh
for (( i=0; i < 3; i++ ))
do
sed -e "s/##NODE_NAME##/${ETCD_NAMES[i]}/" -e "s/##NODE_IP##/${ETCD_IPS[i]}/" etcd.service.template > etcd-${NODE_IPS[i]}.service
done
ls *.service
```

- NODE_NAMES 和 NODE_IPS 为相同长度的 bash 数组，分别为节点名称和对应的 IP；

分发生成的 systemd unit 文件：

```shell
cd /opt/k8s/work
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
echo ">>> ${node_ip}"
scp etcd-${node_ip}.service root@${node_ip}:/etc/systemd/system/etcd.service
done
```

### 启动 etcd 服务

```shell
cd /opt/k8s/work
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
echo ">>> ${node_ip}"
ssh root@${node_ip} "mkdir -p ${ETCD_DATA_DIR} ${ETCD_WAL_DIR}"
ssh root@${node_ip} "systemctl daemon-reload && systemctl enable etcd && systemctl restart etcd &"
done
```

- 必须先创建 etcd 数据目录和工作目录;
- etcd 进程首次启动时会等待其它节点的 etcd 加入集群，命令  systemctl startetcd  会卡住一段时间，为正常现象；

### 检查启动结果

```shell
cd /opt/k8s/work
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
echo ">>> ${node_ip}"
ssh root@${node_ip} "systemctl status etcd|grep Active"
done
```

确保状态为  active (running)  ，否则查看日志，确认原因：

```
journalctl -u etcd
```

### 验证服务状态

部署完 etcd 集群后，在任一 etcd 节点上执行如下命令：

```shell
cd /opt/k8s/work
source /opt/k8s/bin/environment.sh
for node_ip in ${ETCD_IPS[@]}
do
echo ">>> ${node_ip}"
/opt/k8s/bin/etcdctl \
--endpoints=https://${node_ip}:2379 \
--cacert=/etc/kubernetes/cert/ca.pem \
--cert=/etc/etcd/cert/etcd.pem \
--key=/etc/etcd/cert/etcd-key.pem endpoint health
done
```

- 3.4.3 版本的 etcd/etcdctl 默认启用了 V3 API，所以执行 etcdctl 命令时不需要再指定环境变量  ETCDCTL_API=3  ；
- 从 K8S 1.13 开始，不再支持 v2 版本的 etcd；

预期输出：

![1584426870289](k8s/k8s-%E4%BA%8C%E8%BF%9B%E5%88%B6%E5%AE%89%E8%A3%8516-6/1584426870289.png)

输出均为  healthy  时表示集群服务正常。

### 查看当前的 leader

```shell
source /opt/k8s/bin/environment.sh
/opt/k8s/bin/etcdctl \
-w table --cacert=/etc/kubernetes/cert/ca.pem \
--cert=/etc/etcd/cert/etcd.pem \
--key=/etc/etcd/cert/etcd-key.pem \
--endpoints=${ETCD_ENDPOINTS} endpoint status
```

输出：

![1584426824645](k8s/k8s-%E4%BA%8C%E8%BF%9B%E5%88%B6%E5%AE%89%E8%A3%8516-6/1584426824645.png)

- 可见，当前的 leader 为192.168.220.100。