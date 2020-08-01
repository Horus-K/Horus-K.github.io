---
title: linux配置代理
date: 2020-07-24 15:43:13
tags:
- 代理
categories: Linux
---

**Linux系统下终端proxy代理配置**

工作中有时会遇到需要翻墙下载软件的问题，这个时候就用到了正向代理。

变量的设置方法
1、在 /etc/profile文件
2、在 ~/.bashrc
3、在 /etc/profile.d/文件夹下新建一个文件xxx.sh

- 写入如下配置：

```sh
export proxy="http://10.20.56.32:8000"
export http_proxy=$proxy
export https_proxy=$proxy
export ftp_proxy=$proxy
export no_proxy="localhost, 127.0.0.1, ::1"
```

- 取消设置的方法

```sh
unset http_proxy
unset https_proxy
unset ftp_proxy
unset no_proxy
```

