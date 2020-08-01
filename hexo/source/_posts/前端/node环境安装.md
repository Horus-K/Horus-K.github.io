---
title: node环境安装
date: 2020-05-14 13:53:40
tags:
categories:
---

### 源码安装

```sh
cd
wget https://npm.taobao.org/mirrors/node/v14.2.0/node-v14.2.0-linux-x64.tar.xz
tar xf node-v14.2.0-linux-x64.tar.xz
mkdir /Data/apps/src -pv
mv node-v14.2.0-linux-x64 /Data/apps/src/
cd /Data/apps
chown -R `whoami`:`whoami` node./*
ln -s src/node-v14.2.0-linux-x64 node
echo 'PATH=$PATH:/Data/apps/node/bin' >> /etc/profile
source /etc/profile
```

