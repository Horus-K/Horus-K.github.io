---
title: etcd-v3可视化管理
date: 2020-04-14 11:44:23
tags:
- etcd
- k8s
categories: k8s
---

![1586836528227](两种etcd-v3可视化管理/1586836528227.png)

```SH
git clone https://github.com/ringtail/lucas.git
```

<!--more-->

```yaml
cat kubernetes-deployment.yaml
---
apiVersion: v1
kind: Service
metadata:
  annotations:
    derrick.service.type: nodeport
    derrick.version: 0.0.14
  labels:
    derrick.service: lucas
  name: lucas
spec:
  ports:
  - name: "8080"
    port: 8080
    targetPort: 8080
  selector:
    derrick.service: lucas
  type: LoadBalancer
---
apiVersion: apps/v1  ### 1.16版本要改为apps/v1
kind: Deployment
metadata:
  annotations:
    derrick.version: 0.0.14
  labels:
    derrick.service: lucas
  name: lucas
spec:
  selector:
    matchLabels:
      derrick.service: lucas
  replicas: 1
  template:
    metadata:
      labels:
        derrick.service: lucas
    spec:
      nodeSelector:
        kubernetes.io/hostname: k8s-01  # 固定
      tolerations:
        - key: node-role.kubernetes.io/master
          operator: Exists
      containers:
      - image: registry.cn-hangzhou.aliyuncs.com/ringtail/lucas:0.0.2
        name: web
        ports:
        - containerPort: 8080
        env:
        - name: CA_FILE
          value: /etc/kubernetes/pki/etcd/ca.pem
        - name: CERT_FILE
          value: /etc/kubernetes/pki/etcd/etcd.pem
        - name: KEY_FILE
          value: /etc/kubernetes/pki/etcd/etcd-key.pem
        - name: ENDPOINTS
          value: "https://192.168.220.100:2379,https://192.168.220.101:2379,https://192.168.220.102:2379"
        volumeMounts:
        - mountPath: /etc/kubernetes/pki/etcd
          name: etcd-certs-0
          readOnly: true
      volumes:
      - hostPath:
          path: /etc/kubernetes/pki/etcd
          type: DirectoryOrCreate
        name: etcd-certs-0

```