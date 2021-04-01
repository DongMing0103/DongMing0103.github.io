---
layout:     post
title:      Docker搭建Nexus Maven私服
subtitle:   日志清理
date:       2021-04-01
author:     dm
header-img: img/post-bg-mma-2.jpg
catalog: true
tags:
    - Docker
	- Nexus
	- maven







---

## 搭建nexus仓库服务器

### 要求

- 使用docker安装
- 支持docker、maven
- 数据持久化

### 安装计划

- 数据持久化：将宿主机目录`/data/tools/nexus/nexus-data`挂在到容器的`/nexus-data`目录
- `nexus`管理网页访问端口：`8081`
- `nexus`镜像上传下载端口：`8082`

### 下载nexus3镜像

- 搜索nexus镜像 `docker search nexus`

  ![docker 搜索 nexus 镜像](https://raw.githubusercontent.com/DongMing0103/MarkdownCloudImage/master/data/docker%E6%90%9C%E7%B4%A2nexus.jpg)

  

- 下载nexus3镜像`sonatype/nexus3`,该镜像[使用教程](https://hub.docker.com/r/sonatype/nexus3/)

  ![docker下载nexus](https://raw.githubusercontent.com/DongMing0103/MarkdownCloudImage/master/data/docker%E4%B8%8B%E8%BD%BDnexus.jpg)
  
  ```bash
  root@localhost:/home$ docker volume create --name nexus-data
  ```

### 运行nexus

- 首次启动，注意`data/tools/nexus/nexus-data`目录权限需要修改为`200:200`，对应容器内部的`nexus`用户，不然会因权限不足导致启动失败；`8081`为网页端口，`8082`为镜像上传下载端口

  ```
  root@localhost:/home/docker/nexus$ sudo chown -R 200:200 /data/tools/docker/nexus/nexus-data/		
    
  root@localhost:/home/docker/nexus$ docker run -d -p 8081:8081 -p 8082:8082 --name nexus -v /home/docker/nexus/nexus-data:/nexus-data sonatype/nexus3
    526c03402bf6bb3c86105453be55203365b4491c3448c7183227968f12a03e14
    
  ```
  
- 修改默认端口

  - 修改容器内 `/nexus-data/etc/nexus.properties` 文件， `application-port=[your port]`，这里因为和其它应用冲突，所以修改为8081。

- 测试工作是否正常

  - 命令行检测，正常运行时会输出`pong`

    ![](https://raw.githubusercontent.com/DongMing0103/MarkdownCloudImage/master/data/%E6%9F%A5%E7%9C%8Bnexus%E6%98%AF%E5%90%A6%E5%90%AF%E5%8A%A8%E6%88%90%E5%8A%9F.jpg)

    

  - 浏览器打开[http://192.168.xx.xx:8081](http://192.168.xx.xx:8081/)，可正常显示网页

- 非首次启动，执行`docer start nexus`







---

### 添加docker hosted仓库

- 访问[http://192.168.1.56:8092](http://192.168.1.56:8092/)，使用默认账号/密码:`admin/admin123`登录
- 创建`docker(hosted)`仓库，名称`docker-hosted`，注意配置http端口，这里配置为`8093`，与启动参数一致

### 修改docker启动参数

- 由于我们添加的本地仓库只配置了http服务，而docker默认使用https，因此在docker启动文件`/etc/docker/daemon.json`中添加私有仓库地址：`"insecure-registries":["http://192.168.1.56:8093"]`，`daemon.json`文件内容变为：

  ```visual basic
    {
      "registry-mirrors": ["https://7uko0u1b.mirror.aliyuncs.com"],
      "insecure-registries":["http://192.168.1.56:8093"]
    }
  ```

- 重启docker服务：`service docker restart`

### 向本地仓库push镜像

- 为待上传的镜像打标签，格式为`docker tag <imageId or imageName> <nexus-hostname>:<repository-port>/<image>:<tag>`

  ```bash
    root@localhost:/home/docker/nexus$ docker images
    REPOSITORY            TAG                 IMAGE ID            CREATED             SIZE
    sonatype/nexus3       latest              9ded7bd31da5        37 hours ago        480MB
    jenkins/jenkins       lts                 b36e8b881678        37 hours ago        810MB
    httpd                 latest              b669148bb5a5        6 days ago          177MB
    stilliard/pure-ftpd   latest              193339b4053f        2 weeks ago         439MB
    ubuntu                16.04               ccc7a11d65b1        4 weeks ago         120MB
    hello-world           latest              1815c82652c0        3 months ago        1.84kB
    root@localhost:/home/docker/nexus$ docker tag hello-world 192.168.1.56:8093/hello-world:1.0
    root@localhost:/home/docker/nexus$ docker images
    REPOSITORY                      TAG                 IMAGE ID            CREATED             SIZE
    sonatype/nexus3                 latest              9ded7bd31da5        37 hours ago        480MB
    jenkins/jenkins                 lts                 b36e8b881678        37 hours ago        810MB
    httpd                           latest              b669148bb5a5        6 days ago          177MB
    stilliard/pure-ftpd             latest              193339b4053f        2 weeks ago         439MB
    ubuntu                          16.04               ccc7a11d65b1        4 weeks ago         120MB
    192.168.1.56:8082/hello-world   1.0                 1815c82652c0        3 months ago        1.84kB
    hello-world                     latest              1815c82652c0        3 months ago        1.84kB
  ```

- 登录仓库，默认账号/密码为：`admin/admin123`，不登录就push镜像会提示：`no basic auth credentials`

  ```bash
    root@localhost:/home/docker/nexus$ docker login 192.168.1.56:8093
    Username (admin): admin
    Password: 
    Login Succeeded
  ```

- push镜像

  ```bash
    root@localhost:/home/docker/nexus$ docker push 192.168.1.56:8082/hello-world:1.0
    The push refers to a repository [192.168.1.56:8093/hello-world]
    45761469c965: Layer already exists 
    1.0: digest: sha256:9fa82f24cbb11b6b80d5c88e0e10c3306707d97ff862a3018f22f9b49cef303a size: 524
  ```

- 从[http://192.168.1.56:8092](http://192.168.1.56:8092/)上可以看到docker-hosted仓库中已经有了刚刚上传的镜像

### 从本地仓库搜索镜像

```bash
	root@localhost:/home/docker/nexus$ docker search 192.168.1.56:8093/hello
	NAME                                DESCRIPTION   STARS     OFFICIAL   AUTOMATED
	192.168.1.56:8093/hello-world:1.0  
```

### 从本地仓库pull镜像

```bash
		root@localhost:/home/docker/nexus$ docker images
		REPOSITORY            TAG                 IMAGE ID            CREATED             SIZE
		sonatype/nexus3       latest              9ded7bd31da5        37 hours ago        480MB
		jenkins/jenkins       lts                 b36e8b881678        37 hours ago        810MB
		httpd                 latest              b669148bb5a5        6 days ago          177MB
		stilliard/pure-ftpd   latest              193339b4053f        2 weeks ago         439MB
		ubuntu                16.04               ccc7a11d65b1        4 weeks ago         120MB
		hello-world           latest              1815c82652c0        3 months ago        1.84kB
		root@localhost:/home/docker/nexus$ docker pull 192.168.1.56:8093/hello-world:1.0
		1.0: Pulling from hello-world
		Digest: sha256:9fa82f24cbb11b6b80d5c88e0e10c3306707d97ff862a3018f22f9b49cef303a
		Status: Downloaded newer image for 192.168.1.56:8093/hello-world:1.0
		root@localhost:/home/docker/nexus$ docker images
		REPOSITORY                      TAG                 IMAGE ID            CREATED             SIZE
		sonatype/nexus3                 latest              9ded7bd31da5        37 hours ago        480MB
		jenkins/jenkins                 lts                 b36e8b881678        37 hours ago        810MB
		httpd                           latest              b669148bb5a5        6 days ago          177MB
		stilliard/pure-ftpd             latest              193339b4053f        2 weeks ago         439MB
		ubuntu                          16.04               ccc7a11d65b1        4 weeks ago         120MB
		192.168.1.56:8082/hello-world   1.0                 1815c82652c0        3 months ago        1.84kB
		hello-world                     latest              1815c82652c0        3 months ago        1.84kB
```