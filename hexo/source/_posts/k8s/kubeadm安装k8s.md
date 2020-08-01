---
title: kubeadm安装k8s
date: 2020-06-17 20:31:55
tags: k8s
categories: k8s
---

# 安装前提

```
设置主机名
$ hostnamectl set-hostname k8s-master
$ hostnamectl set-hostname k8s-node1
 
关闭防火墙：
$ systemctl stop firewalld
$ systemctl disable firewalld
 
关闭selinux：
$ sed -i 's/enforcing/disabled/' /etc/selinux/config
$ setenforce 0
 
关闭swap：
$ swapoff -a  $ 临时
$ vim /etc/fstab  $ 永久
 
添加主机名与IP对应关系（记得设置主机名）：
cat <<EOF > /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.31.62 k8s-master
192.168.31.63 k8s-node1
EOF
 
将桥接的IPv4流量传递到iptables的链：
$ cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
 
$ sysctl --system

安装Docker
$ wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo
$ yum -y install docker-ce-18.06.1.ce-3.el7
$ systemctl enable docker && systemctl start docker
$ docker --version
Docker version 18.06.1-ce, build e68fc7a

docker 加速镜像配置
$ mkdir /etc/docker/
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=cgroupfs"],
  "registry-mirrors": ["http://f1361db2.m.daocloud.io"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m", "max-file":"3"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
EOF
 
# Restart docker.
systemctl daemon-reload
systemctl restart docker
 
##max-file=3，意味着一个容器有三个日志，分别是id+.json、id+1.json、id+2.json
 
尝试过修改kubelet的cgroup dirver(文件位置：/lib/systemd/system/kubelet.service.d/10-kubeadm.conf)，但是每次启动kubeadm时会被覆盖掉，导致kubelet启动失败，只能修改docker的cgroup dirver设置；
文档中docker驱动是systemd
 
参考：https://kubernetes.io/docs/setup/production-environment/container-runtimes/
```

<!--more-->

# 添加源

```sh
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```



# 修改docker cgroup

docker的Cgroup Driver和kubelet的Cgroup Driver不一致导致的，此处选择修改docker的和kubelet一致

```sh
docker info | grep Cgroup
Cgroup Driver: cgroupfs
编辑文件/usr/lib/systemd/system/docker.service

ExecStart=/usr/bin/dockerd --exec-opt native.cgroupdriver=systemd
systemctl daemon-reload
systemctl restart docker
 docker info | grep Cgroup
Cgroup Driver: systemd
```

```sh
yum install kubelet-1.16.9 kubeadm-1.16.9 kubectl-1.16.9
```

```sh
cat kubeadm-config 
apiVersion: kubeadm.k8s.io/v1beta1
kind: ClusterConfiguration
kubernetesVersion: v1.16.9
imageRepository: registry.aliyuncs.com/google_containers
networking:
  dnsDomain: cluster.local
  podSubnet: 10.128.0.0/23
  serviceSubnet: 10.192.0.0/22
```

```
kubeadm init --config=/root/kubeadm-config --node-name=192.168.220.100
```

# 安装Pod网络插件（CNI）

```sh
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

或者

```
kubectl apply -f https://docs.projectcalico.org/v3.8/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml
```

# 检查证书到期时间

```
kubeadm alpha certs check-expiration
```

# 续订全部证书

```
kubeadm alpha certs renew all
```

# kubectl自动补全

```
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc
#或者
echo "source <(kubectl completion bash)" >> /etc/profile
```

# 添加污点

```
kubectl taint nodes node1 key=value:NoSchedule
kubectl taint node 192.168.88.9 node-role.kubernetes.io=master:NoSchedule
```

# 删除污点

```
kubectl taint nodes kube11 key:NoSchedule-
```