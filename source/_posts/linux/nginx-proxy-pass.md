---
title: nginx proxy_pass 详解
date: 2021-02-19 13:00:00
tags: [nginx]
categories: [linux, nginx]
---
# 指令语法
```bash
proxy_pass URL;
```
URL 由三部分组成：
- 协议 可以指定 http 或 https
- 地址 可以指定 域名 或 ip 以及 可选端口
- 可选的URI

示例
```bash
proxy_pass http://www.abc.com;
proxy_pass http://127.0.0.1:8000/uri/;

proxy_pass http://unix:/tmp/backend.socket:/uri/;
```

此外，可将地址指定为一个主机组 [server group](http://nginx.org/en/docs/http/ngx_http_upstream_module.html)，使用 upstream 来定义主机组。

当使用 rewrite 在代理位置更改URI时（地址重写），则重写后的URI会传递给后端服务器。

<!--more-->


# 示例解析
假设访问域名为：www.abc123.com

我们围绕`proxy_pass`指令后面，地址带不带有 `uri`，以及是否以斜杠 `/` 结尾进行分析，看看后端服务接收到的请求是什么样的。

## 示例1
无 uri， 非斜杠结尾
```bash
location /test1/ {
    proxy_pass http://192.168.10.110;
}

# 请求结果
# 请求：http://www.abc123.com/test1/1.html
# 后端：http://192.168.10.110/test1/1.html
```


## 示例2
无 uri，斜杠结尾
```bash
location /test2/ {
    proxy_pass http://192.168.10.110/;
}

# 请求结果
# 请求：http://www.abc123.com/test2/2.html
# 后端：http://192.168.10.110/2.html
```

## 示例3
有 uri，斜杠结尾
```bash
location /test3/ {
    proxy_pass http://192.168.10.110/abc/;
}

# 请求结果
# 请求：http://www.abc123.com/test3/3.html
# 后端：http://192.168.10.110/abc/3.html
```

## 示例4
有 uri，非斜杠结尾
```bash
location /test4/ {
    proxy_pass http://192.168.10.110/abc;
}

# 请求结果
# 请求：http://www.abc123.com/test4/4.html
# 后端：http://192.168.10.110/abc4.html
```
