---
layout:     post
title:      Docker搭建RabbitMQ
subtitle:   
date:       2021-05-24
author:     dm
header-img: img/post-bg-mma-4.jpg
catalog: true
tags:
    - Docker
    - RabbitMQ






---

***这里注意获取镜像的时候要获取`management`版本的，不要获取`last`版本的，`management`版本的才带有管理界面。***

# 安装流程
## 获查询镜像

```shell
docker search rabbitmq:management
```

## 获取镜像

```
docker pull rabbitmq:management
```

## 运行镜像
```shell
docker run -d --name rabbitmq -p 5672:5672 -p 15672:15672 -v /data/rabbitmq:/var/lib/rabbitmq -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin 32497bcd639d
```
### 说明

```tex
-d 后台运行容器；
–name 指定容器名；
-p 指定服务运行的端口（5672：应用访问端口；15672：控制台Web端口号）；
-v 映射目录或文件；
-e 指定环境变量；（RABBITMQ_DEFAULT_VHOST：默认虚拟机名；RABBITMQ_DEFAULT_USER：默认的用户名；RABBITMQ_DEFAULT_PASS：默认用户名的密码）
```


## Docker目录挂载

```tex
sudo docker run -itd --name=(镜像运行起来的容器名称) -p (服务器对外的端口):(docker容器开放的端口) -v 服务器的文件路径:对应docker的文件路径 镜像id command
```

```tex
docker run -d --name rabbitmq  -v /data/rabbitmq:/var/lib/rabbitmq

通过这种方式，我们可以明确一点，即-v参数中，冒号":"前面的目录是宿主机目录，后面的目录是容器内目录。
```

## 访问管理界面
![访问管理界面](https://raw.githubusercontents.com/DongMing0103/MarkdownCloudImage/master/data/rabbitmq.jpg)

# 问题
## docker chown: changing ownership of '/var/lib/XXX': Permission denied


```tex
在docker run中加入 --privileged=true  给容器加上特定权限
```
