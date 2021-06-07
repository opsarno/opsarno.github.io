---
title: Configure SFTP Service
date: 2021-04-08 15:55:39
tags: [linux, sftp]
categories: [linux]
---
<!--more-->

# SFTP

SFTP (SSH File Transfer Protocol)是一种安全的文件传输协议。它通过SSH协议运行。它支持SSH的完整安全和身份验证功能。

SFTP已几乎取代了旧版FTP作为文件传输协议，并且正在迅速取代FTP/S。它提供了这些协议提供的所有功能，但更安全，更可靠，配置更简单。

SFTP还可以防止密码嗅探和中间人攻击。它使用加密和加密哈希函数保护数据的完整性，并对服务器和用户进行身份验证。


## Setup Chroot SFTP



### 创建用户组 & 用户

```bash
# 创建用户组
groupadd sftpusers

# 创建用户
# -g 指定账号 用户组
# -s 指定账号 login shell
# -d 指定账号 home directory， 默认 /home/username
# -M 不创建 home directory
useradd -g sftpusers -s /sbin/nologin -M s-beijing
useradd -g sftpusers -s /sbin/nologin -d /uploads -M s-shanghai

# 配置密码
passwd s-beijing
passwd s-shanghai
```



### 创建目录

```bash
# SFTP chroot directory
mkdir -p /sftp/s-beijing
mkdir -p /sftp/s-shanghai

# SFTP s-shanghai home directory
mkdir /sftp/s-shanghai/uploads
chown s-shanghai:sftpusers /sftp/s-shanghai/uploads/
```

Tips：

- SFTP 用户家目录的 / 根路径 是相对于 ChrootDirectory 声明的目录的
- 创建 SFTP 用户家目录后，默认登录会处在家目录中，否则处在根目录中
- 一定要在根目录下，创建一个 SFTP 用户拥有权限的目录，否则无权限创建文件
- 根目录一定是非 SFTP 账号权限，否则登录时会报错

```bash
# 客户端终端报错
s-shanghai@itdevops.cn's password:
packet_write_wait: Connection to 10.93.1.10 port 22: Broken pipe
Couldn't read packet: Connection reset by peer

# 服务端日志 /var/log/secure
Apr  8 15:22:07 itdevops.cn sshd[13974]: fatal: bad ownership or modes for chroot directory "/sftp/s-shanghai" [postauth]
```



查看权限情况

```bash
# ls -ld /sftp/
drwxr-xr-x 4 root root 4096 Apr  8 15:11 /sftp/

# ls -ld /sftp/s-beijing/
drwxr-xr-x 2 root root 4096 Apr  8 15:10 /sftp/s-beijing/

# ls -ld /sftp/s-shanghai/
drwxr-xr-x 3 root root 4096 Apr  8 15:11 /sftp/s-shanghai/

# ls -ld /sftp/s-shanghai/uploads/
drwxr-xr-x 2 s-shanghai sftpusers 4096 Apr  8 15:11 /sftp/s-shanghai/uploads/
```



### 配置 sshd_config

在 `sshd_config` 中注释 `sftp-server` ，并启用 `internal-sftp` 子系统。

注释内容

```bash
# override default of no subsystems
# Subsystem   sftp    /usr/libexec/openssh/sftp-server

# 添加内容 
Subsystem       sftp    internal-sftp
# 匹配组，并启用 ChrootDirestory
Match Group sftpusers
        ChrootDirectory /sftp/%u
        ForceCommand internal-sftp
```

Tips：

- ChrootDirectory /sftp/%u 中的 %u 表示登录账号的用户名 

- `sftp-server` 以及 `internal-sftp` 均具有 SFTP 功能，但 `internal-sftp` 实现了一个进程内SFTP服务，且简化了 ChrootDirectory 配置，强制不同用户使用不同位置的根目录。



### 启用SFTP & 验证

重启 sshd 以启用 SFTP 子系统。

```bash
systemctl restart sshd.service
```



测试验证 s-beijing 账号

```bash
# sftp s-beijing@itdevops.cn
s-beijing@itdevops.cn's password:
Connected to itdevops.cn.
sftp> ls
sftp> pwd
Remote working directory: /
sftp> mkdir abc
Couldn't create directory: Permission denied
sftp> bye
```

测试验证 s-shanghai 账号

```bash
# sftp s-shanghai@itdevops.cn
s-shanghai@itdevops.cn's password:
Connected to itdevops.cn.
sftp> ls
sftp> pwd
Remote working directory: /uploads
sftp> mkdir abc
sftp> ls
abc
sftp> cd /
sftp> ls
uploads
sftp> mkdir aaa
Couldn't create directory: Permission denied
sftp> bye
```



### Logging sftp

如果想要记录详细的 SFTP 日志，有以下三种方式。

Via Monitor

```bash
Subsystem   sftp    internal-sftp -l VERBOSE
```

日志结果将在 `/var/log/secure` 中

```bash
Mar  9 11:24:04 rhel7 sshd[20327]: received client version 3 [postauth]
Mar  9 11:24:04 rhel7 sshd[20327]: realpath "." [postauth]
Mar  9 11:24:05 rhel7 sshd[20327]: opendir "/" [postauth]
```



Via socket in chroot

如果需要独立的日志文件输出，还需要额外调整 `/etc/rsyslog.conf`

```bash
# /etc/ssh/sshd_config
Subsystem   sftp    internal-sftp -l VERBOSE

# /etc/rsyslog.conf
input(type="imuxsock" HostName="user" Socket="/chroots/user/dev/log" CreatePath="on")
if $fromhost == 'user' then /var/log/sftp.log
& stop

# 重启 sshd & rsyslog
```



3、Via socket in chroot, filtering using log facility

```bash
# /etc/ssh/sshd_config
Subsystem   sftp    internal-sftp -l VERBOSE -f LOCAL3

# /etc/rsyslog.conf
input(type="imuxsock" Socket="/chroots/user/dev/log" CreatePath="on")
local3.*                                            /var/log/sftp.log
```




## 参考资料

- https://man.openbsd.org/sshd_config
- https://www.ssh.com/academy/ssh/sftp
- https://access.redhat.com/articles/1374633

