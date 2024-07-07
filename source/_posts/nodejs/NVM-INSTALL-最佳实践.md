---
title: NVM INSTALL 最佳实践
date: 2024-07-07 22:01:59
tags:
categories:
---

# 说明
如果网络条件好，可以直接使用官方仓库安装，本文主要记录使用国内仓库加速安装方法。

- Github 官方仓库 - https://github.com/nvm-sh/nvm
- Gitee 码云镜像仓库 - https://gitee.com/repository-mirror/nvm

> Gitee 码云镜像仓库是个人同步的代码，可能不是最新的，如有需要可联系作者同步更新一下。

<!--more-->

# NVM 安装
这里以 v0.39.7 版本为例

1、下载脚本
```bash
wget https://gitee.com/repository-mirror/nvm/raw/v0.39.7/install.sh
```

2、修改脚本
```bash
# 找到以下 URL 内容，并镜像修改。主要目的是改为镜像仓库。

  if [ "_$NVM_METHOD" = "_script-nvm-exec" ]; then
    # NVM_SOURCE_URL="https://raw.githubusercontent.com/${NVM_GITHUB_REPO}/${NVM_VERSION}/nvm-exec"
    NVM_SOURCE_URL="https://gitee.com/repository-mirror/nvm/raw/${NVM_VERSION}/nvm-exec"
  elif [ "_$NVM_METHOD" = "_script-nvm-bash-completion" ]; then
    # NVM_SOURCE_URL="https://raw.githubusercontent.com/${NVM_GITHUB_REPO}/${NVM_VERSION}/bash_completion"
    NVM_SOURCE_URL="https://gitee.com/repository-mirror/nvm/raw/${NVM_VERSION}/bash_completion"
  elif [ -z "$NVM_SOURCE_URL" ]; then
    if [ "_$NVM_METHOD" = "_script" ]; then
      # NVM_SOURCE_URL="https://raw.githubusercontent.com/${NVM_GITHUB_REPO}/${NVM_VERSION}/nvm.sh"
      NVM_SOURCE_URL="https://gitee.com/repository-mirror/nvm/raw/${NVM_VERSION}/nvm.sh"
    elif [ "_$NVM_METHOD" = "_git" ] || [ -z "$NVM_METHOD" ]; then
      # NVM_SOURCE_URL="https://github.com/${NVM_GITHUB_REPO}.git"
      NVM_SOURCE_URL="https://gitee.com/repository-mirror/nvm.git"
    else
      nvm_echo >&2 "Unexpected value \"$NVM_METHOD\" for \$NVM_METHOD"
      return 1
    fi
  fi
```

3、执行安装
```bash
bash install.sh
```

会自动在当前 shell 环境配置文件中  `.bashrc` or `.zshrc` 添加以下配置
```bash
# NVM
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
```

> 说明：如果当前用户的 SHELL 环境是 BASH，则文件是 .bashrc；如果是 ZSH，则文件是 .zshrc

# node binaries 镜像配置
NVM 管理工具安装完成后，我们还可以配置 node 二进制文件包的镜像，用于安装加速。

编辑 `.bashrc` or `.zshrc` 文件，追加写入
```bash
export NVM_NODEJS_ORG_MIRROR=https://npmmirror.com/mirrors/node
```


# 使用示例
```bash
# 查看可安装的 LTS 版本
nvm ls-remote --lts

# 安装指定版本
nvm install v18.20.3 

# 验证安装
node -v
npm -v

# 安装 nrm 实用工具 - 用于便捷切换 registry 
npm install -g nrm --registry=https://registry.npmmirror.com
nrm ls
nrm use taobao
```
