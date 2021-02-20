---
title: Yum Repository Mirrors
date: 2021-02-20 11:20:57
tags: [centos, yum, repository]
categories: [linux]
---

# 说明
通常情况下，企业内部会维护一套 `Yum Repository Mirrors`，以便于快速的安装更新一些软件包。即使是在阿里云VPC环境下，也会有一些三方或自定义的软件包需要内部镜像源来管理。

针对这些需求，目前有三种镜像源的管理方式
- rsync
- Nexus
- createrepo

<!--more-->

# rsync
针对系统发行版的官方源，以及常用的核心仓库进行全量的同步。
- centos
- epel
- remi

优点是内部访问时稳定、速度快，缺点是会占用较大的数据空间。

示例脚本：
```bash
#!/bin/bash
mkdir -p /opt/data/repo

# 过滤一些不必要的文件
rsync -avz --delete-after --no-iconv \
    --exclude=atomic/ \
    --exclude=cr/ \
    --exclude=dotnet/ \
    --exclude=fasttrack/ \
    --exclude=paas/ \
    --exclude=rt/ \
    --exclude=isos/ \
    --exclude=debug/ \
    rsync://mirrors.tuna.tsinghua.edu.cn/centos/ \
    /opt/data/repo/centos/
```



# Nexus
[Sonatype Nexus Repository Manager](https://www.sonatype.com/nexus/repository-oss)

对于一些低频使用、非核心的三方包，我们通过 Nexus 代理官方仓库的方式，缓存安装的 RPM 包。

- AdoptOpenJDK
- elasticstack
- nvidia-cuda
- .....

优点是仅缓存所需的依赖包，无需占用太多存储资源；缺点是对于文件较大的RPM包，在首次安装时会有失败的情况。

## 扩展说明
`Sonatype Nexus Repository Manager` 功能非常强大，能够存储主流的文件仓库格式，如 apt, docker, go, npm, pypi yum... 等。

更多细节，参见官方文档：[Repository Manager Documentation](https://help.sonatype.com/docs)


# createrepo
主要用于 rpmbuild 自封装的 RPM 包，或者固定版本的三方RPM包，存放到指定目录下，生成 repodata 进行管理使用。

```bash
createrepo --update /opt/data/repo/self-repo
```
