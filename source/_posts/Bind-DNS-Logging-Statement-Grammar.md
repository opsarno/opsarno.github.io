---
title: Bind DNS Logging Statement Grammar
date: 2017-02-20 14:26:19
tags: [bind, dns]
categories: [linux, bind]
---

# 语法
```bash
logging {
    # 定义 channel
    [ channel channel_name {
        ( file path_name
          [ versions ( number | unlimited ) ]
          [ size size_spec ]
        | syslog syslog_facility
        | stderr
        | null );

        [ severity (critical | error | warning | notice | info | debug [ level ] | dynamic ); ]

        [ print-category yes or no; ]
        [ print-severity yes or no; ]
        [ print-time yes or no; ]
    }; ]

    # 定义 category
    [ category category_name {
        channel_name ; [ channel_name ; ... ]
    }; ]
    ...
};
```

<!--more-->


# 定义和使用
如果没有明确定义logging语句，默认配置为：
```bash
logging {
    category default { default_syslog; default_debug; };
    category unmatched { null; };
};
```

channel 语句
- 所有log可以输出到一个或多个channel.
- 每个channel 定义都必须包含一个目标语句，该语句指出为该channel选择的消息是转到file、syslog、stderr，还是被丢弃。
- 它还可以选择性地限制channel将接受的消息严重性级别（默认为info），以及是否包括命名生成的时间戳，类别名称和/或严重性级别（默认值不包括任何 ）。

Example usage of the size and versions options:
```bash
channel an_example_channel {
    file "example.log" versions 3 size 20m;
    print-time yes;
    print-category yes;
}; 
```

有四个预定义的默认日志记录，如下所示。
```
channel default_syslog {
    // send to syslog’s daemon facility
    syslog daemon; 
    // only send priority info and hig
    severity info;
};
channel default_debug {
    // write to named.run in the worki
    // Note: stderr is used instead of
    // the server is started with the
    file "named.run";
    // log at the server’s current deb
    severity dynamic;
};
channel default_stderr {
    // writes to stderr
    stderr;
    // only send priority info and hig
    severity info;
};
channel null {
    // toss anything sent to this chann
    null;
};
```
channel default_debug 有特殊属性，它只在服务器的调试级别非零时才产生输出。 它通常写入服务器工作目录中名为named.run的文件

以下是其包含的日志信息类型的可用类别：
```
client    处理客户端请求
cname     域名服务器日志,跳过由于它们是一个CNAME而不是A/AAAA的记录
config    配置文件解析和处理。
database  与名称服务器内部使用的用于存储区域和缓存数据的数据库相关的消息。
default   默认类别定义了没有定义特定配置的那些类别的日志记录选项。
delegation-only  记录由于仅限授权区域或授权（仅在转发，提示或存根区域声明中）强制执行NXDOMAIN的查询。
dispatch  将传入的数据包发送到要处理的服务器模块。
dnssec    DNSSEC和TSIG协议处理。
edns-disabled    记录由于超时而被强制使用纯DNS的查询。
general   很多事情仍然没有分为类别,最终他们都在这里。
lame-servers     这些是远程服务器配置错误，BIND9在解析过程中尝试查询这些服务器时发生。
network   网络操作信息
notify    NOTIFY协议
queries   查询信息
query-errors     查询解析的一些失败信息
rate-limit 在此类别的信息严重性中记录了响应流的速率限制的开始，周期和最终通知。
resolver   DNS解析，例如由缓存名称服务器代表客户端执行的递归查找
rpz        有关响应策略区域文件，重写响应以及最高调试级别错误的信息，只需重写尝试。
security   批准和拒绝请求。
spill      通过删除或响应SERVFAIL来记录已被终止的查询，这是由于超出了限制配额限制的结果。
unmatched  named无法确定的消息类或没有匹配的view，一行摘要也记录到客户端类别。此类别最好发送到文件或stderr，默认情况下发送到空信道。
update     动态更新
update-security  批准和拒绝更新请求。
xfer-in    服务器接收的zone传输信息
xfer-out   服务器发送的zone传输信息
```



使用的配置示例：
```bash
# LOGGING
# severity (critical | error | warning | notice | info | debug [ level ] | dynamic ); 
logging {
    channel default_log {
        file "/var/log/named.log" versions 10 size 200m;
        severity dynamic;
        print-category yes;
        print-severity yes;
        print-time yes;
    };
    channel query_log {
        file "/var/log/query.log" versions 10 size 200m;
        severity dynamic;
        print-category yes;
        print-severity yes;
        print-time yes;
    };
    channel resolver_log {
        file "/var/log/resolver.log" versions 10 size 200m;
        severity dynamic;
        print-category yes;
        print-severity yes;
        print-time yes;
    };

    category default {default_log;};
    category queries {query_log;};
    category query-errors {query_log;};
    category resolver {resolver_log;};
    category lame-servers {null;};
    category edns-disabled {null;};
};
```
