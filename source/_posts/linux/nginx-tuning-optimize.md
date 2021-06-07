---
title: NGINX WEBSERVER 调优
date: 2021-06-04 11:35:53
tags: nginx
categories: [linux, nginx]
---

# Linux 系统调优

## Backlog 队列
- [net.core.somaxconn](https://www.kernel.org/doc/Documentation/networking/ip-sysctl.txt)
    ```bash
    somaxconn - INTEGER
        Limit of socket listen() backlog, known in userspace as SOMAXCONN.
        Defaults to 4096. (Was 128 before linux-5.4)
        See also tcp_max_syn_backlog for additional tuning for TCP sockets.
    ```
    `net.core.somaxconn` 参数是 socket listen() 的 backlog 限制。**用于控制全连接队列长度**。默认值是4096（内核5.4版本以前是128）。如果 `socket server` 处理请求较慢，以至于监听队列填满后，新来的请求会被拒绝。

<!--more-->


- [net.core.netdev_max_backlog](https://www.kernel.org/doc/Documentation/sysctl/net.txt)
    ```bash
    netdev_max_backlog
        Maximum number of packets, queued on the INPUT side, 
        when the interface receives packets faster than kernel can process them.
    ```
    `net.core.netdev_max_backlog` 参数是当网络接口接收的数据包速度比内核处理速度快时，用于控制网络设备的最大队列数。增大此值可以提高高带宽机器的性能。

扩展：  
- [net.ipv4.tcp_max_syn_backlog](https://www.kernel.org/doc/Documentation/networking/ip-sysctl.txt)
    ```bash
    tcp_max_syn_backlog - INTEGER
        Maximal number of remembered connection requests (SYN_RECV),
        which have not received an acknowledgment from connecting client.
        This is a per-listener limit.
        The minimal value is 128 for low memory machines, and it will
        increase in proportion to the memory of machine.
        If server suffers from overload, try increasing this number.
        Remember to also check /proc/sys/net/core/somaxconn
        A SYN_RECV request socket consumes about 304 bytes of memory.
    ```
    `net.ipv4.tcp_max_syn_backlog` 参数是记录连接请求（SYN_RECV）的最大数值。**用于控制半连接队列长度**。因其会随着机器的内存成比例增加，通常不需要手动修改它。


## 文件描述符
系统级别文件句柄数
```bash
# sys.fs.file-max – The system‑wide limit for file descriptors
```

用户级别文件句柄数
```bash
# nofile – The user file descriptor limit, set in the /etc/security/limits.conf file
```


## 端口范围
当 NGINX 用作 reverse proxy 时，与每个 `upstream server` 都需要使用一个临时端口建立连接。
- [net.ipv4.ip_local_port_range](https://www.kernel.org/doc/Documentation/networking/ip-sysctl.txt) 
    ```bash
    ip_local_port_range - 2 INTEGERS
        Defines the local port range that is used by TCP and UDP to
        choose the local port. The first number is the first, the
        second the last local port number.
        If possible, it is better these numbers have different parity
        (one even and one odd value).
        Must be greater than or equal to ip_unprivileged_port_start.
        The default values are 32768 and 60999 respectively.
    ```
    `net.ipv4.ip_local_port_range` 定义TCP/UDP连接时的本地端口范围。  
    默认值是 32768 60999。最小值不得低于 `ip_unprivileged_port_start`（默认1024）。


扩展：  
[ip_unprivileged_port_start](https://www.kernel.org/doc/Documentation/networking/ip-sysctl.txt)
```bash
ip_unprivileged_port_start - INTEGER
	This is a per-namespace sysctl.  It defines the first
	unprivileged port in the network namespace.  Privileged ports
	require root or CAP_NET_BIND_SERVICE in order to bind to them.
	To disable all privileged ports, set this to 0.  They must not
	overlap with the ip_local_port_range.

	Default: 1024
```


## Linux 系统调优总结
编辑文件 `/etc/sysctl.conf` 添加以下内容
```bash
# Increase number of incoming connections. Default: 4096
net.core.somaxconn = 32768

# Increase number of incoming connections backlog
net.core.netdev_max_backlog = 16384

# Increase size of file handles and inode cache
fs.file-max = 209708

# Allowed local port range
net.ipv4.ip_local_port_range = 1024 65535
```

编辑 `/etc/security/limits.conf`
```bash
# End of file
*	soft	nproc	65535
*	hard	nproc	65535
*	soft	nofile	65535
*	hard	nofile	65535
```
- nproc - maximum number of processes
- nofile - maximum number of open file descriptors

# NGINX 服务调优

## Worker
[worker_processes](https://nginx.org/en/docs/ngx_core_module.html#worker_processes) 定义工作进程数量。auto 将自动检测并设为与内核数一致。
```bash
worker_processes auto;
```

[worker_rlimit_nofile](https://nginx.org/en/docs/ngx_core_module.html#worker_rlimit_nofile) 更改工作进程打开文件的最大数量的限制(RLIMIT NOFILE)。可在不重启主进程的情况下增加限制。
```bash
worker_rlimit_nofile 65535;
```

[worker_connections](https://nginx.org/en/docs/ngx_core_module.html#worker_connections) 设置工作进程可以打开的最大同时连接数。包含客户端连接、与代理服务器的连接。同样连接数不能超过工作进程打开文件的最大数量的限制(worker_rlimit_nofile)。
```
worker_connections 65535;
```

## backlog
NGINX [listen backlog](https://nginx.org/en/docs/http/ngx_http_core_module.html#listen)，限制了等待连接队列的最大长度。在高并发场景，可以适当的增大它。
```bash
# backlog=number
# sets the backlog parameter in the listen() call that limits the maximum length for the queue of pending connections. By default, backlog is set to -1 on FreeBSD, DragonFly BSD, and macOS, and to 511 on other platforms.

server {
    listen 80 backlog=1024;

}
```

## Keepalived
`Keepalive Connections` 可以减少打开和关闭连接所需的CPU和网络开销。
- `keepalive_requests` 客户端可以通过单个连接的最大请求数。在达到最大请求数后，连接关闭。  
为了释放每个连接分配的内存，需要定期关闭连接。因此，使用过高的最大请求数可能会导致过多的内存使用，不建议这样做。  
更高的值对于使用 `load-generation` 工具测试尤为有用。  
    ```bash
    Syntax:	    keepalive_requests number;
    Default:	keepalive_requests 1000;
    Context:	http, server, location
    ```

- `keepalive_timeout` 设置闲置连接的超时时间。
    ```bash
    Syntax:	    keepalive_timeout timeout [header_timeout];
    Default:	keepalive_timeout 75s;
    Context:	http, server, location
    ```

- `keepalive` 设置后启用每个工作进程的缓存中保留到 `upstream server` 的最大空闲 keepalive 连接数。
    ```
    Syntax:	    keepalive connections;
    Default:	—
    Context:	upstream
    ```
    要使上游服务器保持连接，配置中还需包含以下指令
    ```bash
    proxy_http_version 1.1;
    proxy_set_header Connection "";
    ```

## Nginx 配置示例

主配置文件 `nginx.conf`
```bash
# Generated by opsarno

user                 nginx;
pid                  /var/run/nginx.pid;
worker_processes     auto;
worker_rlimit_nofile 65535;

# Load modules
include              /etc/nginx/modules-enabled/*.conf;

events {
    multi_accept       on;
    worker_connections 65535;
}

http {
    charset                utf-8;
    sendfile               on;
    tcp_nopush             on;
    tcp_nodelay            on;
    server_tokens          off;
    log_not_found          off;
    types_hash_max_size    2048;
    types_hash_bucket_size 64;
    client_max_body_size   16M;

    # MIME
    include                mime.types;
    default_type           application/octet-stream;

    # Limits
    # limit_req_log_level    warn;
    # limit_req_zone         $binary_remote_addr zone=login:10m rate=10r/m;

    # SSL
    ssl_session_timeout    1d;
    ssl_session_cache      shared:SSL:10m;
    ssl_session_tickets    off;

    # Diffie-Hellman parameter for DHE ciphersuites
    ssl_dhparam            /etc/nginx/dhparam.pem;

    # Mozilla Intermediate configuration
    ssl_protocols          TLSv1.2 TLSv1.3;
    ssl_ciphers            ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;

    # OCSP Stapling
    ssl_stapling           on;
    ssl_stapling_verify    on;
    resolver               1.1.1.1 1.0.0.1 8.8.8.8 8.8.4.4 208.67.222.222 208.67.220.220 valid=60s;
    resolver_timeout       2s;

    # Custom log format: main
    log_format main     '[$time_local] $remote_addr - $remote_user '
                        '$scheme $http_host "$request" $body_bytes_sent $request_time $status '
                        '"$http_referer" "$http_user_agent" "$http_x_forwarded_for" '
                        '$upstream_addr $upstream_response_time $upstream_status';

    # Logging
    access_log             /var/log/nginx/access.log main;
    error_log              /var/log/nginx/error.log warn;

    # Load configs
    # include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```


# 参考资料
- [Optimizing Performance for Serving Content](https://docs.nginx.com/nginx/admin-guide/web-server/serving-static-content/#optimize)
- [Tuning NGINX for Performance](https://www.nginx.com/blog/tuning-nginx/)
- [kernel networking ip-sysctl.txt](https://www.kernel.org/doc/Documentation/networking/ip-sysctl.txt)
- [kernel sysctl](https://www.kernel.org/doc/Documentation/sysctl/)
