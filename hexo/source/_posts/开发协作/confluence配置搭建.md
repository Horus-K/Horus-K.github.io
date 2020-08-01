---
title: confluence配置搭建
date: 2020-04-10 09:30:57
tags:
categories: confluence
---

# 环境准备：

需要安装jdk、需要mysql5.7数据库，创建数据库confluence并创建用户和密码

<!--more-->

JAVA:

![1586482387063](confluence6-3-1配置搭建/1586482387063.png)

数据库：

confluence

Confluence!@#

```
CREATE DATABASE `confluence` DEFAULT CHARACTER SET utf8 COLLATE utf8_bin;
```

或者直接使用docker镜像

```
docker run -d \
--name confluence \
-p 8090:8090 \
-v /Data/atlassian:/opt/atlassian \
-v /etc/localtime:/etc/localtime:ro \
-v /Data/atlassian_data:/var/atlassian/application-data/confluence \
docker.io/atlassian/confluence-server:6.13.11
```



# 下载启动包以及其他文件

链接：https://pan.baidu.com/s/1-CBUNkbWxMUnCR9254Xs8Q
提取码：lsaj

```sh
chmod +x atlassian-confluence-6.3.1-x64.bin
mkdir /Data/Confluence -p
mkdir /Data/Confluence_app-data -p
./atlassian-confluence-6.3.1-x64.bin
```

![1586483006124](confluence6-3-1配置搭建/1586483006124.png)

![1586483044927](confluence6-3-1配置搭建/1586483044927.png)

```sh
http://192.168.220.100:8090/setup/setupstart.action
```

选择中文语言后，点击下一步

![1586483134908](confluence6-3-1配置搭建/1586483134908.png)

![1586483165108](confluence6-3-1配置搭建/1586483165108.png)

![1586483292410](confluence6-3-1配置搭建/1586483292410.png)

 # 这两个可以不选，看个人需求，下一步，然后复制id，然后stop掉confluence进行破解：

![1586483351399](confluence6-3-1配置搭建/1586483351399.png)

```sh
/etc/init.d/confluence stop
```

用下载的文件替换atlassian-extras-decoder-v2-3.2.jar文件：

![1586483670110](confluence6-3-1配置搭建/1586483670110.png)

用下载的文件替换confluence自带的atlassian-universal-plugin-manager-plugin-2.22.1.jar：

![1586483751140](confluence6-3-1配置搭建/1586483751140.png)

顺便将mysql的驱动也安装到指定目录：

/Data/Confluence/confluence/WEB-INF/lib/

然后启动confluence：

![1586483938353](confluence6-3-1配置搭建/1586483938353.png)

然后继续访问url：点击获得试用权限

![1586483963684](confluence6-3-1配置搭建/1586483963684.png)

然后登陆账号，无账号可注册

![1586484108472](confluence6-3-1配置搭建/1586484108472.png)

点击创建证书后，等待弹出下面的窗口，然后点击yes

![1586484136556](confluence6-3-1配置搭建/1586484136556.png)

![1586484161380](confluence6-3-1配置搭建/1586484161380.png)

随后会跳转到安装界面，然后点击下一步：

![1586484195261](confluence6-3-1配置搭建/1586484195261.png)

![1586484267492](confluence6-3-1配置搭建/1586484267492.png)

然后输入数据库信息，并下一步(有点慢)

![1586484451533](confluence6-3-1配置搭建/1586484451533.png)

然后选择空白站点

![1586484660336](confluence6-3-1配置搭建/1586484660336.png)

![1586486573492](confluence6-3-1配置搭建/1586486573492.png)

![1586488500002](confluence6-3-1配置搭建/1586488500002.png)

