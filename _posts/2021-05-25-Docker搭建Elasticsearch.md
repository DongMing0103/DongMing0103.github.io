---
layout:     post
title:      Docker搭建Elasticsearch
subtitle:   
date:       2021-05-25
author:     dm
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - Docker
    - Elasticsearch






---

# 安装流程

## Docker拉取镜像

``` shell
docker search elasticsearch

docker pull elasticsearch:7.7.0
```

## 运行镜像

``` shell
docker run --name elasticsearch -p 9200:9200 -p 9300:9300 -d -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms512m -Xmx512m" -v /data/test/config/elasticsearch.yml:/config/elasticsearch.yml -v /data/test/data:/data -v /data/test/plugins:/plugins 7ec4f35ab452
```

``` tex
--name：表示镜像启动后的容器名称  

-d: 后台运行容器，并返回容器ID；

-e: 指定容器内的环境变量

-p: 指定端口映射，格式为：主机(宿主)端口:容器端口

-v：标记 在容器中设置了一个挂载点/data（就是容器中的一个目录），并将主机上的/home/xqh/myimage 目录中的内容关联到/data下
```

## 浏览器访问IP:9200

`出现以下界面说安装成功`

![浏览器启动成功页面](https://raw.githubusercontents.com/DongMing0103/MarkdownCloudImage/master/data/Elasticsearch%E5%90%AF%E5%8A%A8%E6%88%90%E5%8A%9F.jpg)

## 安装Elasticsearch-head

``` shell
#拉取镜像
docker pull mobz/elasticsearch-head:5

#创建容器
docker create --name elasticsearch-head -p 9100:9100 mobz/elasticsearch-head:5

#启动容器
docker start elasticsearch-head
```

## 修改docker中elasticsearch的elasticsearch.yml文件

``` shell
docker exec -it elasticsearch /bin/bash （进不去使用容器id进入）

vim config/elasticsearch.yml
```

``` shell
# 最下方添加
http.cors.enabled: true 
http.cors.allow-origin: "*"
```

**vim修改文件报错 bash: vi: command not found**

``` shell
apt-get update
apt-get install vim

# 下载vim
yum -y install vim*
```

**退出重启服务**

``` shell
exit
docker restart 容器id
```

## 浏览器访问IP:9100

`出现以下界面说明安装成功`

![Elasticsearch-head安装](https://raw.githubusercontents.com/DongMing0103/MarkdownCloudImage/master/data/Elasticsearch-head%E5%AE%89%E8%A3%85.jpg)

## 修改Elasticsearch-head配置，解决406错误码

``` shell
#复制vendor.js到外部
docker cp 7ec4f35ab452:/usr/src/app/_site/vendor.js /data/test/plugins/head/_site/

#修改vendor.js
vim vendor.js
```

``` tex
修改点
1. 6886行 contentType:"application/x-www-form-urlencoded" 
改为 
					contentType:"application/json;charst=UTF-8"
					
2. 7574行 var inspectData = s.contentType === "application/x-www-form-urlencoded" && 
改为
					var inspectData = s.contentType === "application/json;charset=UTF-8" && 
```



修改完成后复制回容器

``` shell
docker cp /data/test/plugins/head/_site/vendor.js 729683e0e0f5:/usr/src/app/_site
```

重启Elasticsearch-head

## 安装IK分词器

