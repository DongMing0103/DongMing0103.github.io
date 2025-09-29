---
layout:     post
title:      openvpn
subtitle:   通过docker搭建openvpn
date:       2020-12-07
author:     dm
header-img: img/post-bg-keybord.jpg
catalog: true
tags:
    - Docker



---

## 通过docker搭建openvpn

1. 拉取openvpn镜像

   `docker pull kylemanna/openvpn:latest`

   

2. 创建目录

   ``mkdir -p /data/tools/openvpn``

   

3. 生成配置文件 **（192.168.10.214这个ip是我当前服务器的公网IP）**

   ``docker run -v /data/openvpn:/etc/openvpn --rm kylemanna/openvpn:latest ovpn_genconfig -u udp://192.168.10.214``

   ````bash
   Processing PUSH Config: 'block-outside-dns'
   Processing Route Config: '192.168.254.0/24'
   Processing PUSH Config: 'dhcp-option DNS 8.8.8.8'
   Processing PUSH Config: 'dhcp-option DNS 8.8.4.4'
   Successfully generated config
   Cleaning up before Exit ...
   ````

4. 生成秘钥文件

   `docker run -v /data/openvpn:/etc/openvpn --rm -it kylemanna/openvpn:latest ovpn_initpki`

   ``` tex
   输入私钥密码（输入时是看不见的）：
   Enter PEM pass phrase:12345678
   再输入一遍
   Verifying - Enter PEM pass phrase:12345678
   输入一个CA名称（我这里直接回车）
   Common Name (eg: your user, host, or server name) [Easy-RSA CA]:
   输入刚才设置的私钥密码（输入完成后会再让输入一次）
   Enter pass phrase for /etc/openvpn/pki/private/ca.key:12345678
   ```

   

   

5. 生成客户端证书**（这里的whsir改成你想要的名字）**

   `docker run -v /data/openvpn:/etc/openvpn --rm -it kylemanna/openvpn:2.4 easyrsa build-client-full whsir nopass`

   `输入刚才设置的密码
   Enter pass phrase for /etc/openvpn/pki/private/ca.key:12345678`

   

6. 导出客户端配置

   ``` shell
   mkdir -p /data/openvpn/conf
   docker run -v /data/openvpn:/etc/openvpn --rm kylemanna/openvpn:2.4 ovpn_getclient whsir > /data/openvpn/conf/whsir.ovpn
   ```

   

7. 启动openvpn服务

   ``` dockerfile
   docker run --name openvpn -v /data/openvpn:/etc/openvpn -d -p 5001:5001/udp --cap-add=NET_ADMIN kylemanna/openvpn:latest
   ```

   

8. 保存防火墙配置

   `iptables-save > /etc/sysconfig/iptables`

   

9. 设置防火墙

   `关闭firewalld防火墙，关闭开机自启`

   ``` shell
   systemctl stop firewalld.service
   systemctl disable firewalld.service
   ```

   

   `安装iptables防火墙，设置开机自启`

   `yum -y install iptables-services net-toolssystemctl enable iptables.service`

   `编辑防火墙配置`

   `vi /etc/sysconfig/iptables`

   `在最后COMMIT前添加以下规则`

   ``` shell
   -A INPUT -p tcp -m tcp --dport 22 -j ACCEPT
   -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
   -A INPUT -p icmp -m icmp --icmp-type 8 -j ACCEPT
   -A INPUT -j REJECT --reject-with icmp-host-prohibited
   -A FORWARD -j REJECT --reject-with icmp-host-prohibited
   ```

   

10. 重启防火墙

    `systemctl restart iptables`

11. 将登录证书下载本地

    ``` shell
    yum install lrzsz -y
    sz /data/openvpn/conf/whsir.ovpn
    ```

    

12. 脚本信息

* **openvpn创建用户脚本**

``` shell
#!/bin/bash d
read -p "please your username: " NAME
docker run -v /data/openvpn:/etc/openvpn --rm -it kylemanna/openvpn:2.4 easyrsa build-client-full $NAME nopass
docker run -v /data/openvpn:/etc/openvpn --rm kylemanna/openvpn:2.4 ovpn_getclient $NAME > /data/openvpn/conf/"$NAME".ovpn
docker restart openvpn
```

* **openvpn删除用户脚本**

```shell
#!/bin/bash
read -p "Delete username: " DNAME
docker run -v /data/openvpn:/etc/openvpn --rm -it kylemanna/openvpn:2.4 easyrsa revoke $DNAME
docker run -v /data/openvpn:/etc/openvpn --rm -it kylemanna/openvpn:2.4 easyrsa gen-crl
docker run -v /data/openvpn:/etc/openvpn --rm -it kylemanna/openvpn:2.4 rm -f /etc/openvpn/pki/reqs/"$DNAME".req
docker run -v /data/openvpn:/etc/openvpn --rm -it kylemanna/openvpn:2.4 rm -f /etc/openvpn/pki/private/"$DNAME".key
docker run -v /data/openvpn:/etc/openvpn --rm -it kylemanna/openvpn:2.4 rm -f /etc/openvpn/pki/issued/"$DNAME".crt
docker restart openvpn
```

> 完整实例

``` bash
*nat
:PREROUTING ACCEPT [0:0]
:INPUT ACCEPT [0:0]
:OUTPUT ACCEPT [3:228]
:POSTROUTING ACCEPT [3:228]
:DOCKER - [0:0]
-A PREROUTING -m addrtype --dst-type LOCAL -j DOCKER
-A OUTPUT ! -d 127.0.0.0/8 -m addrtype --dst-type LOCAL -j DOCKER
-A POSTROUTING -s 172.17.0.0/16 ! -o docker0 -j MASQUERADE
-A POSTROUTING -s 172.17.0.2/32 -d 172.17.0.2/32 -p udp -m udp --dport 1194 -j MASQUERADE
-A DOCKER -i docker0 -j RETURN
-A DOCKER ! -i docker0 -p udp -m udp --dport 1194 -j DNAT --to-destination 172.17.0.2:1194
COMMIT
*filter
:INPUT ACCEPT [60:4900]
:FORWARD DROP [0:0]
:OUTPUT ACCEPT [50:4784]
:DOCKER - [0:0]
:DOCKER-ISOLATION - [0:0]
-A FORWARD -j DOCKER-ISOLATION
-A FORWARD -o docker0 -j DOCKER
-A FORWARD -o docker0 -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A FORWARD -i docker0 ! -o docker0 -j ACCEPT
-A FORWARD -i docker0 -o docker0 -j ACCEPT
-A DOCKER -d 172.17.0.2/32 ! -i docker0 -o docker0 -p udp -m udp --dport 1194 -j ACCEPT
-A DOCKER-ISOLATION -j RETURN
-A INPUT -p tcp -m tcp --dport 22 -j ACCEPT
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p icmp -m icmp --icmp-type 8 -j ACCEPT
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -j REJECT --reject-with icmp-host-prohibited
COMMIT
```

[转载](https://blog.whsir.com/post-2809.html)