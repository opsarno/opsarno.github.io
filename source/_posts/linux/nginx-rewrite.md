---
title: nginx rewrite 详解
date: 2021-02-19 14:00:00
tags: nginx
categories: [linux, nginx]
---
<!--more-->

# 概述
NGINX 重写模块 `ngx_http_rewrite_module`  用于使用 PCRE正则表达式 更改请求URI，返回重定向，并有条件地选择配置。

主要的指令 `break, if, return, rewrite, set`

指令执行顺序：
1. 按顺序执行 server 区块中的 rewrite 模块指令
2. 如果发生 rewrite 地址重写，执行 location 匹配
3. 按顺序执行 匹配 location 中的 rewrite指令

如果URI发生重写，就会重新循环执行1-3，直到找到真实存在的文件。

如果循环超过10次，则返回 [500 Internal Server Error](http://nginx.org/en/docs/http/ngx_http_core_module.html#internal) 错误。


# 指令语法

## break 指令
```bash
Syntax:	    break;
Default:    —
Context:	server, location, if
```

停止执行当前设置的 rewrite 模块指令集。

如果指令在 location 中定义，那么请求的进一步处理将在 location 继续运行。

示例
```bash
if ($slow) {
    limit_rate 10k;
    break;
}
```

## if 指令
```bash
Syntax:	    if (condition) { ... }
Default:	—
Context:	server, location
```
评估指定的条件(condition)，如果为真(true)，大括号内指定的模块指令将被执行。

条件(condition)可以是下列任何一种
- 一个变量名称；如果变量的值是 `空字符串` 或 `0`，结果则为false。
- 使用 `=` 和 `!=` 操作符比较变量和字符串。
- 使用 `〜`（区分大小写）和 `〜*`（不区分大小写）运算符将变量与正则表达式进行匹配。正则表达式可以包含用于稍后在 `$1..$9` 中重用的捕获。也可以使用负运算符 `!~` 和 `!〜*` 。如果正则表达式包含大括号 `{` 或 分号 `;` 字符，整个表达式应该用单引号或双引号括起来。
- 使用 `-f` 和 `!-f` 操作符检查文件是否存在。
- 使用 `-d` 和 `!-d` 操作符检查目录是否存在。
- 使用 `-e` 和 `!-e` 操作符检查文件、目录或符号链接是否存在。
- 使用 `-x` 和 `！-x` 运算符检查可执行文件。


示例
```bash
if ($http_user_agent ~ MSIE) {
    rewrite ^(.*)$ /msie/$1 break;
}

if ($http_cookie ~* "id=([^;]+)(?:;|$)") {
    set $id $1;
}

if ($request_method = POST) {
    return 405;
}

if ($slow) {
    limit_rate 10k;
}

if ($invalid_referer) {
    return 403;
}
```

## return 指令
```bash
Syntax:	    return code [text];
            return code URL;
            return URL;
Default:	—
Context:	server, location, if
```
停止处理并将指定的代码返回给客户机。

非标准代码444关闭连接时不发送响应头。

响应正文text 和 重定向URL 可以包含变量。



## rewrite 指令
```bash
Syntax:	    rewrite regex replacement [flag];
Default:	—
Context:	server, location, if
```

如果指定的正则表达式与请求URI匹配，则URI将按照替换字符串中的指定进行更改。

rewrite 指令按照它们在配置文件中出现的顺序依次执行。

可以使用标记符 flag 终止指令的进一步处理。 

如果 `replacement` 字符串以 `http：//, https：//, 或 $scheme ` 开头，则处理将停止并将重定向返回给客户端。


### 标记符
- last  
    停止处理当前的 ngx_http_rewrite_module 模块指令集，开始搜索重写uri后的 location 匹配。浏览器地址不变。

- break  
    和 break指令一样，停止处理 ngx_http_rewrite_module 模块指令集。

- redirect  
    302 临时重定向。客户端或搜索引擎不记录新地址。

- permanent  
    301 永久重定向。客户端或搜索引擎会记录新地址。

> Tips：  
last与break的相同点在于，立即停止执行所有当前上下文的rewrite模块指令；  
不同点在于last参数接着用新的URI马上搜寻新的location，而break不会搜寻新的location，直接用这个新的URI来处理请求，这样能避免重复rewite。因此，通常在server上下文中使用last，而在location上下文中使用break。


示例
```bash
server {
    ...
    rewrite ^(/download/.*)/media/(.*)\..*$ $1/mp3/$2.mp3 last;
    rewrite ^(/download/.*)/audio/(.*)\..*$ $1/mp3/$2.ra  last;
    return  403;
    ...
}
```
如果这些指令放在 `location /download/` 中，最后一个标志应该替换为 break，否则nginx将进行10个循环并返回500错误。
```bash
location /download/ {
    rewrite ^(/download/.*)/media/(.*)\..*$ $1/mp3/$2.mp3 break;
    rewrite ^(/download/.*)/audio/(.*)\..*$ $1/mp3/$2.ra  break;
    return  403;
}
```


如果替换字符串包括新的请求参数，则先前的请求参数将附加在它们之后。
如果不希望这样，请在替换字符串的末尾添加问号，避免附加它们，例如：
```bash
rewrite ^/users/(.*)$  /show?user=$1? last;
```

如果正则表达式包含 `}` 或 `;` 字符，整个表达式应该用单引号或双引号括起来。


# 301/302/307/308 状态码扩展

## 301 Moved Permanently  
被请求的资源已永久移动到新位置，并且将来任何对此资源的引用都应该使用本响应返回的若干个 URI 之一。

如果可能，拥有链接编辑功能的客户端应当自动把请求的地址修改为从服务器反馈回来的地址。除非额外指定，否则这个响应也是可缓存的。

HTTP 301 永久重定向 说明请求的资源已经被移动到了由 Location 头部指定的url上，是固定的不会再改变。搜索引擎会根据该响应修正。

尽管标准要求浏览器在收到该响应并进行重定向时不应该修改http method和body，但是有一些浏览器可能会有问题。**所以最好是在应对GET 或 HEAD 方法时使用301，其他情况使用308 来替代301。**


## 302 Found  
请求的资源现在临时从不同的 URI 响应请求。由于这样的重定向是临时的，客户端应当继续向原有地址发送以后的请求。只有在Cache-Control或Expires中进行了指定的情况下，这个响应才是可缓存的。

HTTP 302 Found 重定向状态码表明请求的资源被暂时的移动到了由Location 头部指定的 URL 上。浏览器会重定向到这个URL， 但是搜索引擎不会对该资源的链接进行更新 (In SEO-speak, it is said that the link-juice is not sent to the new URL)。


## 307 Temporary Redirect  
请求的资源现在临时从不同的URI 响应请求。由于这样的重定向是临时的，客户端应当继续向原有地址发送以后的请求。只有在Cache-Control或Expires中进行了指定的情况下，这个响应才是可缓存的。

状态码 307 与 302 之间的唯一区别在于，当发送重定向请求的时候，307 状态码可以确保请求方法和消息主体不会发生变化。如果使用 302 响应状态码，一些旧客户端会错误地将请求方法转换为 GET：也就是说，在 Web 中，如果使用了 GET 以外的请求方法，且返回了 302 状态码，则重定向后的请求方法是不可预测的；但如果使用 307 状态码，之后的请求方法就是可预测的。对于 GET 请求来说，两种情况没有区别。

## 308 Permanent Redirect  
这意味着资源现在永久位于由 Location: HTTP Response 标头指定的另一个 URI。 这与 `301 Moved Permanently` HTTP 响应代码具有相同的语义，但用户代理不能更改所使用的 HTTP 方法：如果在第一个请求中使用 POST，则必须在第二个请求中使用 POST。

在重定向过程中，请求方法和消息主体不会发生改变，然而在返回 301 状态码的情况下，请求方法有时候会被客户端错误地修改为 GET 方法。



# 参考资料
[官方文档 - ngx_http_rewrite_module](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html)

[MDN Web Docs - HTTP 响应代码](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status)
