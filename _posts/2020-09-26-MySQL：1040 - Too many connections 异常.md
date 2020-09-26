layout:     post
title:      MySQL问题
subtitle:   1040 - Too many connections
date:       2020-09-26
author:     dm
header-img: img/post-bg-kuaidi.jpg
catalog: true
tags:

    - MySQL



---

## 异常报错 1040 - Too many connections

```MySQL
## 重启MySQL 

## 查看最大连接数
show variables like '%max_connections%'; 

## 查看连接进程
show processlist;

## 设置连接数
set GLOBAL max_connections=1000;
```

