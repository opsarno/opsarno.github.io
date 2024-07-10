---
title: Terminal proxy config
date: 2021-04-08 15:49:52
tags: [linux, proxy]
categories: [Linux]
---

# Linux Terminal Proxy 代理配置

如果需要在 Terminal 中使用代理，则需配置一些代理变量来实现。此方式适用于所有 Linux 发行版、MacOS 等类Unix系统。

<!--more-->

## 通用配置

如果代理启用了身份认证，需要提供账号密码

- 在当前 BASH 终端中输入可临时生效
- 将内容写入 `~/.bashrc` 中，对当前用户生效
- 将内容写入 `/etc/profile` 中，对所有用户生效
- 在脚本中写入，仅对脚本中的请求生效


```bash
export username='myuser'
export password='mypass123com'
export proxy="http://${username}:${password}@myproxy.itdevops.cn:3128"

# 无账号密码
# export proxy="http://myproxy.itdevops.cn:3128"

export http_proxy=${proxy}
export https_proxy=${proxy}
export ftp_proxy=${proxy}
export all_proxy=${proxy}

# 验证代理是否生效
curl -I https://www.google.com/
```



## 特别说明

如果密码中有特殊字符，一般会报以下错误

```bash
curl: (5) Could not resolve proxy: com@myproxy.itdevops.cn; Unknown error
```



此时需要将特殊字符用URL编码后输入，以  `%+Hex` 形式写入。

```bash
# ~ : 0x7E
# @ : 0x40
# % : 0x25

# 如果密码是  P@ssw%rd~ ，那么代理时密码需要如下设定
export password='P%40ssw%25rd%7E'
```



更多ASCII码可参见：[ASCII Encoding Reference](https://www.w3schools.com/tags/ref_urlencode.ASP)



如果代理启用了认证，未输入账号密码，则会出现类似以下的状态码或提示

```bash
HTTP/1.1 407 Proxy Authentication Required
```

