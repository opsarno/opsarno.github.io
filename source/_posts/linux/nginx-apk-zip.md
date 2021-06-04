---
title: NGINX 修改 Response Headers，解决 apk 文件下载变 zip 后缀问题
date: 2021-06-03 11:35:53
tags: nginx
categories: [linux, nginx]
---
# 问题
部分用户手机下载 apk 文件时，后缀会变为 zip 文件。

同样适用于修改 Response Headers 的场景。


# 分析

请求链路：用户手机 --> CDN 域名 --> 源站域名 ---> Ceph 存储

通过调试请求，发现请求下载文件时的 Response Headers 中 `Content-Type` 是 `zip` 类型。
```
content-type: application/zip
```

<!--more-->

故而以为是 Mine Types 问题，于是修改 **源站域名 nginx 的 mine.types 文件** 添加以下内容，但并未解决。
```
application/vnd.android.package-archive          apk;
```

接着，尝试通过声明请求 apk 结尾文件时，add_header 添加文件类型。
```bash
    location ~* \.(apk)$ {
        add_header Content-Type application/vnd.android.package-archive;
        proxy_pass  http://xxx;
        ...
    }
```
但请求结果就会加载 2个 Content-Type，也是有问题的。

也就是说，用户在上传 apk 文件到存储时，存储端存在 Content-Type 文件类型，但并未声明正确的值，由于开发人员反馈无法修改存储文件的 Metadata 数据，故而我们还是在NGINX层处理先解决问题。



# 解决方法
1. 移除后端返回的 Content-Type
2. 添加指定类型的 Content-Type

```
    location ~* \.(apk)$ {
        proxy_hide_header Content-Type;
        add_header Content-Type application/vnd.android.package-archive;
        proxy_pass  http://xxxx;
        ....
    }
```

至此，问题得到解决。


# 参考资料
- [proxy_hide_header](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_hide_header)
- [add_header](http://nginx.org/en/docs/http/ngx_http_headers_module.html)
