---
title: jenkins镜像初始化
date: 2020-07-21 19:24:11
tags:
categories: jenkins
---

### 镜像初始化

```sh
cat Dockerfile 

FROM jenkins/jenkins:2.235.2-lts-centos7

USER root

RUN sed -i '/jenkins/d' /etc/passwd 
    && sed -i 's#/root#/var/jenkins_home#' /etc/passwd
    && cp -a /root/.bash_profile /var/jenkins_home/
    && cp -a /root/.bashrc /var/jenkins_home/      
    && cp -a /root/.cshrc /var/jenkins_home/
```

### 修改插件源

由于默认插件源在国外服务器，大多数网络无法顺利下载，需修改国内插件源地址：

```sh
cd jenkins_home/updates
sed -i 's/http:\/\/updates.jenkins-ci.org\/download/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g' default.json && \
sed -i 's/http:\/\/www.google.com/https:\/\/www.baidu.com/g' default.json
```

### yaml

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins-v1
  namespace: devops
spec:
  selector:
    matchLabels:
      app: jenkins-v1
  template:
    metadata:
      labels:
        app: jenkins-v1
    spec:
      terminationGracePeriodSeconds: 10
      serviceAccountName: jenkins
      containers:
        - name: jenkins
          image: registry.cn-hangzhou.aliyuncs.com/xqbl-share/jenkins_root:2.235.2-lts-centos7
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 8080
              name: web
              protocol: TCP
            - containerPort: 50000
              name: agent
              protocol: TCP
          volumeMounts:
            - mountPath: /var/jenkins_home
              name: jenkinshome
            - mountPath: /var/run/docker.sock
              name: dockersock
            - mountPath: /usr/bin/docker
              name: dockercmd
            - name: kubectl


          resources:
            {}
          env:
            - name: JAVA_OPTS
              value: -Xmx3048m -XshowSettings:vm -Dhudson.slaves.NodeProvisioner.initialDelay=0 -Dhudson.slaves.NodeProvisioner.MARGIN=50 -Dhudson.slaves.NodeProvisioner.MARGIN0=0.85 -Duser.timezone=Asia/Shanghai -Dfile.encoding=utf-8
      volumes:
        - name: jenkinshome
          persistentVolumeClaim:
            claimName: jenkinsdatafile
        - name: dockersock
          hostPath:
            path: /var/run/docker.sock
        - name: dockercmd
          hostPath:
            path: /usr/bin/docker
        - name: kubectl
          hostPath:
            path: /usr/bin/kubectl
```

