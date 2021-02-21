---
title: npm Error EACCES permission denied
date: 2021-02-21 17:25:01
tags: [nodejs, npm, troubleshooting]
categories: [dev, npm]
---
# 问题

基础环境
```
系统：Windows WSL
工具：NVM (Node Version Manager)
版本：node-v12.20.1、npm-6.14.10
```


全局安装工具包时抛错 `npm ERR! Error: EACCES: permission denied` 

<!--more-->

详情如下：
```bash
opsarno@DESKTOP-T3JD9UU:~$ npm install nrm -g
npm WARN deprecated request@2.88.2: request has been deprecated, see https://github.com/request/request/issues/3142
npm WARN deprecated coffee-script@1.7.1: CoffeeScript on NPM has moved to "coffeescript" (no hyphen)
npm WARN deprecated mkdirp@0.3.5: Legacy versions of mkdirp are no longer supported. Please update to mkdirp 1.x. (Note that the API surface has changed to use Promises in 1.x.)
npm WARN deprecated har-validator@5.1.5: this library is no longer supported
npm ERR! code EACCES
npm ERR! syscall rename
npm ERR! path /home/opsarno/.nvm/versions/node/v12.20.1/lib/node_modules/.staging/npm-753362b2/node_modules/string-width
npm ERR! dest /home/opsarno/.nvm/versions/node/v12.20.1/lib/node_modules/.staging/string-width-8aef3ce2
npm ERR! errno -13
npm ERR! Error: EACCES: permission denied, rename '/home/opsarno/.nvm/versions/node/v12.20.1/lib/node_modules/.staging/npm-753362b2/node_modules/string-width' -> '/home/opsarno/.nvm/versions/node/v12.20.1/lib/node_modules/.staging/string-width-8aef3ce2'
npm ERR!  [OperationalError: EACCES: permission denied, rename '/home/opsarno/.nvm/versions/node/v12.20.1/lib/node_modules/.staging/npm-753362b2/node_modules/string-width' -> '/home/opsarno/.nvm/versions/node/v12.20.1/lib/node_modules/.staging/string-width-8aef3ce2'] {
npm ERR!   cause: [Error: EACCES: permission denied, rename '/home/opsarno/.nvm/versions/node/v12.20.1/lib/node_modules/.staging/npm-753362b2/node_modules/string-width' -> '/home/opsarno/.nvm/versions/node/v12.20.1/lib/node_modules/.staging/string-width-8aef3ce2'] {
npm ERR!     errno: -13,
npm ERR!     code: 'EACCES',
npm ERR!     syscall: 'rename',
npm ERR!     path: '/home/opsarno/.nvm/versions/node/v12.20.1/lib/node_modules/.staging/npm-753362b2/node_modules/string-width',
npm ERR!     dest: '/home/opsarno/.nvm/versions/node/v12.20.1/lib/node_modules/.staging/string-width-8aef3ce2'
npm ERR!   },
npm ERR!   errno: -13,
npm ERR!   code: 'EACCES',
npm ERR!   syscall: 'rename',
npm ERR!   path: '/home/opsarno/.nvm/versions/node/v12.20.1/lib/node_modules/.staging/npm-753362b2/node_modules/string-width',
npm ERR!   dest: '/home/opsarno/.nvm/versions/node/v12.20.1/lib/node_modules/.staging/string-width-8aef3ce2',
npm ERR!   parent: 'nrm'
npm ERR! }
npm ERR!
npm ERR! The operation was rejected by your operating system.
npm ERR! It is likely you do not have the permissions to access this file as the current user
npm ERR!
npm ERR! If you believe this might be a permissions issue, please double-check the
npm ERR! permissions of the file and its containing directories, or try running
npm ERR! the command again as root/Administrator.

npm ERR! A complete log of this run can be found in:
npm ERR!     /home/opsarno/.npm/_logs/2021-02-19T13_28_26_969Z-debug.log
```

# 分析
默认 NVM 安装在用户家目录下，WSL环境中相当于安装在Windows C盘，此时如果有一些高危操作的话（如删除、重命名），可能会触发系统的保护性机制，导致权限不足。

也有资料怀疑是  [Symlink Problems](https://github.com/MicrosoftDocs/WSL/issues/26) 问题。

解决方法，将 NVM （如未使用NVM，则是 node），安装到非系统盘符，默认WSL将所有Windows盘符挂载到 `/mnt/` 目录下。

```bash
# 我这里在 d 盘下，新建了一个目录，用于存放 .nvm 工具
mkdir /mnt/d/data/
mv ~/.nvm /mnt/d/data/.nvm

# 更新 ~/.bashrc 中的 nvm 变量配置
export NVM_DIR="/mnt/d/data/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm
```

重新打开 terminal，再次执行安装，成功！


# 参考资料
- https://im.shellj.com/2020/05/wsl-npm-install-permission-denied-error.html
- https://github.com/MicrosoftDocs/WSL/issues/26