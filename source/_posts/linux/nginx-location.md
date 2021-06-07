---
title: nginx location 详解
date: 2021-02-19 12:00:00
tags: [nginx]
categories: [linux, nginx]
---

location 指令用途

根据请求 URI 设置配置，进而对请求做不同的处理和响应。

<!--more-->

# 语法规则
```bash
# 关键字    修饰符  匹配的前缀字符既URI { 要执行的操作 }
location [ = | ~ | ~* | ^~ ] uri { ... }
location @name { ... }
```

语法规则很简单，关键字location后跟可选的修饰符，后面是要匹配的前缀字符既URI，花括号中是要执行的操作。

location 一般定义在 server 区块中，也可以嵌套定义在 location 区块中，但命名 location 不可嵌套。

## 修饰符说明
```bash
空  前缀字符匹配 记录匹配的最长路径，继续后续规则
=   精确字符匹配 匹配后终止搜索
^~  非正则字符匹配 匹配后终止搜索

~   正则匹配 区分大小写 匹配后终止搜索
~*  正则匹配 不区分大小写 匹配后终止搜索

@   命名 location，不用于常规请求处理，而是用于请求重定向。它们不能嵌套，也不能包含嵌套位置。
```

由修饰符的含义可以得出，location 有三种表现形式
- 字符匹配
- 正则匹配
- 命名 location


# 匹配过程

首先会有一些内部的前置工作，例如对请求中 `%xx` 编码的文本进行解码，解析uri中的`.`，`..`相对路径的引用，对两个或多个相邻的斜杠 `/` 将其压缩为一个斜杠后，再开始对一个规范化的URI执行匹配。


匹配过程：  
1. 匹配所有非正则表达式规则（空、=、^~），找到请求uri的最长匹配规则，并暂存；
2. 如果匹配到 = 或 ^~ 的规则，停止后续查找，使用匹配的规则；
3. 如果未匹配 = 或 ^~ 的规则，会按照顺序查找正则匹配规则，匹配后停止后续查找，使用匹配的规则；
4. 如果没有匹配的正则规则，则使用暂存的最长匹配规则；


启示：
- 精确匹配可以提高查找速度，例如经常请求 / 的话，可以使用`location = /`来定义。
- 正则匹配的定义顺序很重要，因为匹配后，后续的查找就终止了。


优先级总结：

`=` > `^~` > `~` > `~*` > `最长前缀字符匹配` > `/`


```bash
精确字符匹配 > 非正则字符匹配 > 区分大小写正则匹配 > 不区分大小写正则匹配 > 最长前缀字符匹配 > 默认前缀字符匹配
```


## 示例
```bash
# 等号 精确字符匹配
location = / {
    [ configuration A ]
}

# 无修饰符 前缀字符匹配，此规则匹配其它未命中的规则。
location / {
    [ configuration B ]
}

# 无修饰符 前缀字符匹配
location /documents/ {
    [ configuration C ]
}

# ^~ 非正则字符匹配
location ^~ /images/ {
    [ configuration D ]
}

# ~* 不区分大小写正则匹配
location ~* \.(gif|jpg|jpeg)$ {
    [ configuration E ]
}
```

- “/” 请求 精确匹配 A，不再往下查找。
- “/index.html” 请求 前缀字符匹配最长路径 配置B，后面无匹配的正则，使用配置B。
- “/documents/document.html” 请求 前缀字符匹配最长路径 配置C，后面无匹配的正则，使用配置C。
- “/images/1.gif” 请求 找到非正则字符匹配 配置D，不再往下查找，使用配置D。
- “/documents/1.jpg” 首先 前缀字符匹配到最长路径 配置C，后面匹配到正则匹配 配置 E，故而最终使用配置E。



# location @name 用法

@用来定义一个命名location。主要用于内部重定向，不能用来处理正常的请求。其用法如下：

```bash
location / {
    try_files $uri $uri/ @custom
}
location @custom {
    # ...do something
}
```
上例中，当尝试访问url找不到对应的文件就重定向到我们自定义的命名location（此处为custom）。



# uri 末尾带不带 / 的说明

uri 末尾带斜杠`/user/`表示目录，不带斜杠`/user`表示文件。

默认情况下，访问 `/user/`时，服务器会自动去该目录下查找加载默认文件。  
而访问 `/user` 时，服务器会先尝试加载该文件，如果文件不存在，则重定向至`/user/`去该目录下找默认文件。


## 特殊情况
如果 location 匹配定义的前缀字符是以 `/` 结尾，并且请求由 `proxy_pass, fastcgi_pass, uwsgi_pass, scgi_pass, memcached_pass, or grpc_pass` 进行特殊处理。这种情况，不管 /user 文件是否存在，请求将会触发301永久重定向到被请求的uri，并附加斜杠。

如果不希望这样，则需要明确定义不带 / 结尾的location配置：
```
location /user/ {
    proxy_pass http://user.example.com;
}

location = /user {
    proxy_pass http://login.example.com;
}
```



官方文档：http://nginx.org/en/docs/http/ngx_http_core_module.html#location
