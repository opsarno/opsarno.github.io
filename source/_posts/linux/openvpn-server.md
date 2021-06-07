---
title: linux openvpn server
date: 2015-07-02 12:00:00
tags: openvpn
categories: linux
---

# 实验环境
本文用 Vmware Workstation 模拟出来的一个VPN环境，其中主机参数如下：
```bash
# 任意的WEB服务
LAN WEB SERVER      10.1.1.11

# Centos 6.X（Openvpn 2.3.X）  
VPN  SERVER         10.1.1.10（LAN）   10.1.2.10（WAN）

# Windows 7 （Openvpn Client）
WAN Client PC       10.1.2.11
```

<!--more-->

目的

客户端通过VPN连接到 VPN Server，然后使之能够访问到内部的WEB 服务，实际生产环境中应用大同小异。

# 具体配置

LAN WEB Server 只需搭建一个简单的 web 服务即可，能够有一个简单的WEB页面用来测试。

WAN Client 则只要安装一个Openvpn Client就可以了；

下面重点说一下VPN SERVER中 openvpn的实现过程与配置

## 安装

① 安装epel源和openvpn

```
https://dl.fedoraproject.org/pub/epel/epel-release-latest-6.noarch.rpm

rpm -Uvh epel-release-latest-6.noarch.rpm

yum install openvpn easy-rsa -y
```

② 使用easy-rsa生成密钥和证书
```bash
mkdir /etc/openvpn/easy-rsa
cp -rf /usr/share/easy-rsa/2.0/* /etc/openvpn/easy-rsa/
cd /etc/openvpn/easy-rsa/

vim vars  #根据自己情况修改文本中下面的这些内容
export KEY_COUNTRY="CN"    #国家
export KEY_PROVINCE="BJ"   #省份
export KEY_CITY="BeiJing"  #城市
export KEY_ORG="personal"  #组织
export KEY_EMAIL="admin@fandenggui.com" #邮箱
export KEY_OU="myserver"  #单位

source ./vars #加载vars参数到当前环境中
./clean-all   #第一次使用时需要执行，会在当前目录中建立keys目录 
./build-ca    #建立ca证书
./build-key-server server  #建立服务器证书
./build-key client  #建立客户端证书
./build-dh  #为OpenVPN服务端生成 Diffie Hellman 参数
```

③ 配置服务器配置文件
```bash
cp /usr/share/doc/openvpn-2.3.7/sample/sample-config-files/server.conf /etc/openvpn/
cd /etc/openvpn
mkdir log  #在当前目录下创建log目录，用于存放openvpn中的log文件

vim server.conf  #egrep -v "#|;|^$" server.conf过滤结果如下
port 1194
proto tcp
dev tun
ca easy-rsa/keys/ca.crt
cert easy-rsa/keys/server.crt
dh easy-rsa/keys/dh2048.pem
server 10.8.0.0 255.255.255.0
ifconfig-pool-persist ipp.txt
push "route 10.1.1.0 255.255.255.0"
push "dhcp-option DNS 208.67.220.220"
keepalive 10 120
comp-lzo
user nobody
group nobody
persist-key
persist-tun
status log/openvpn-status.log
log         log/openvpn.log
log-append  log/openvpn.log
verb 3   
```

④ 配置 iptables
```bash
iptables -F
iptables -X
iptables -A INPUT -p tcp --dport 22 -j ACCEPT #开启ssh端口
iptables -P INPUT DROP 
iptables -P FORWARD DROP 
iptables -P OUTPUT ACCEPT 
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT #保持已经建立的连接
iptables -A INPUT -p tcp --dport 1194 -j ACCEPT  #允许openvpn的端口连接 
iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE #将所有10.8.0.0网段的包转发到eth0口
iptables -A FORWARD -i tun+ -j ACCEPT #添加FORWARD白名单
iptables -A INPUT -s 10.8.0.0/24 -j ACCEPT #允许虚拟网段的所有连接
```

⑤ 配置系统内核参数，使其支持转发
```bash
# a. 临时生效
echo "1" > /proc/sys/net/ipv4/ip_forward  

# b. 配置文件，长久生效
vim /etc/sysctl.conf
net.ipv4.ip_forward = 1  #更改这个条目，保存退出；
sysctl -p  #使之生效
```

⑥ 客户端连接

WIN7下，安装Openvpn客户端后，从服务器上面下载 `ca.crt、client.crt、client.key、client.ovpn` 文件到 Openvpn Client 安装路径下的 config 目录下。(`默认是c:\Program Files\OpenVPN\config`)

> 其中ovpn模板配置文件可参见：cp /usr/share/doc/openvpn-2.3.7/sample/sample-config-files/client.conf /etc/openvpn/easy-rsa/keys/client.ovpn
