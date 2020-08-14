---
layout:     post
title:      springboot数据源配置
subtitle:   单数据源、多数据源区别
date:       2020-03-19
author:     dm
header-img: img/post-bg-coffee.jpg
catalog: true
tags:
    - springboot

---

###  1.单数据源配置

> * 配置信息datasource:url:XXX 

```java
server:
  port: 8808
  servlet:
    context-path: /
spring:
  application:
    name: xcbdepot
  datasource:
    url: jdbc:mysql://xx.xxx.xxx:3306/xxx?autoReconnect=true&useSSL=false
    username: root
    password: xxxxxx
    driver-class-name: com.mysql.jdbc.Driver
```

### 2.多数据源配置

> * 配置信息datasource: master: jdbc-url: XXXX

```java
server:
  port: 8808
  servlet:
    context-path: /
spring:
  application:
    name: xxxx
  datasource:
    #主库
    master:
      jdbc-url: jdbc:mysql://xx.xxx.xxx:3306/xxx?autoReconnect=true&useSSL=false
      username: root
      password: xxxxxx
      driver-class-name: com.mysql.jdbc.Driver
    #从库
    slave:
      jdbc-url: jdbc:mysql://xx.xxx.xxx:3306/xxx?characterEncoding=GBK&zeroDateTimeBehavior=convertToNull
      username: root
      password: xxxxxxx
      connection-timeout: 600000
      driver-class-name: com.mysql.jdbc.Driver
    #equipment
    equipment:
      jdbc-url: jdbc:mysql://xx.xxx.xxx:3306/xxx?autoReconnect=true&useSSL=false
      username: root
      password: xxxxxxx
      driver-class-name: com.mysql.jdbc.Driver
```

