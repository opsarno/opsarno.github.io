---
title: Github Pages + Hexo Next 博客
date: 2020-02-21 12:00:00
tags: [hexo, next]
categories: [others]

---
# GitHub Pages 博客
主要组件
- 博客框架 [Hexo - 快速、简洁且高效的博客框架](https://hexo.io/zh-cn/)
- 博客主题 [hexo-theme-next](https://github.com/theme-next/hexo-theme-next)
- 博客托管 [Github Pages](https://pages.github.com/)
- 自动部署 [Travis CI](https://travis-ci.com/)

Hexo 其实是一种静态站点生成器，也可以用 [Hugo](https://gohugo.io/)、[JekyII](https://jekyllrb.com/)、[MKkDocs](https://www.mkdocs.org/) 等其它静态站点生成器替代。更多生成器参见：[Site Generators](https://jamstack.org/generators/)。

博客示例：
- [GitHub Pages 域名 - opsarno.github.io](opsarno.github.io)
- [GitHub Pages 自定义域名 - www.itdevops.cn](www.itdevops.cn) 
<!--more-->

# 快速安装
主要步骤
1. 开发者电脑安装 Hexo 基础环境，编写博客内容
2. 定制化 Hexo 主题，使博客内容展示更加友好
3. 配置 GitHub + Travis CI，实现自动部署

[Hexo 的官方文档](https://hexo.io/zh-cn/docs/) 其实非常详细了，本文会有一些额外的内容补充，例如 GitHub Pages 自定义域名，自动部署配置需要额外增加一步等，仅供大家参考。


安装 Hexo 相当简单，只需要先安装下列应用程序即可：
- Node.js (建议用 [nvs](https://github.com/jasongin/nvs/) 或 [nvm](https://github.com/nvm-sh/nvm) 方式安装)
- Git

## 安装 Hexo
```bash
npm install -g hexo-cli
```

## 初始化基础环境
```bash
hexo init opsarno.github.io
cd opsarno.github.io
npm install
```

`_config.yml` 文件中配置博客的基本信息


## 安装主题
```bash
git clone https://github.com/theme-next/hexo-theme-next themes/next

cp themes/next/_config.yml _config.next.yml
```
独立的主题配置文件  `_config.[theme].yml`  应放置于站点根目录下，支持 `yml` 或 `json` 格式。该特性自 Hexo 5.0.0 起提供。


配置站点文件 `_config.yml` 中的 `theme` 以供 Hexo 寻找 `_config.[theme].yml` 文件。
```bash
# _config.yml
# ...
## Themes: https://hexo.io/themes/
theme: next
theme_config: _config.next.yml
```

配置主题文件 `_config.next.yml`，自定义定义菜单。
```yml
# _config.next.yml
# ...
menu:
  home: / || fa fa-home
  categories: /categories/ || fa fa-th
  tags: /tags/ || fa fa-tags
  archives: /archives/ || fa fa-archive
  友链: /links/ || fa fa-link
  about: /about/ || fa fa-user
```

## 基础操作

### 新建一篇文章
```bash
hexo new "Hello World"
```
上述命令，会创建一个 `source/_posts/Hello-World.md` 的文件。

另外，可以通过 `--path` 选项新建子目录，以便于在 `source/_posts/` 下创建更好的目录结构管理文章源文件。

示例，`source/_posts/linux/nginx-install.md`
```bash
hexo new --path linux/nginx-install "nginx install"
```

### 创建分类页
```bash
hexo new page categories
```

分类页，是一种特殊页面，编辑 `source/categories/index.md` 文件，添加  `type: "categories"` 即可，无需其它内容。
```yml
---
title: categories
date: 2020-02-19 11:55:35
type: "categories"
---
```

### 创建标签页
```
hexo new page tags
```

标签页，也是一种特殊页面，编辑 `source/tags/index.md` 文件，添加 `type: "tags"` 即可，无需其它内容。
```yml
---
title: tags
date: 2020-02-19 11:56:35
type: "tags"
---
```

### 友链 & 关于
```bash
hexo new page links
hexo new page about
```

编辑 `source/about/index.md`，`source/links/index.md` 文件，与文章类似，填写需要展示的内容。


### 启动服务器
```bash
hexo server

# server 可简写为 s
hexo s

# INFO  Validating config
# INFO  Start processing
# INFO  Hexo is running at http://localhost:4000 . Press Ctrl+C to stop.
```
可以方便的本机访问，查看博客内容及样式。


## 自动部署
官方文档：[将 Hexo 部署到 GitHub Pages](https://hexo.io/zh-cn/docs/github-pages)

这里只补充一点，就是当指定了自定义域名后，项目中会多一个`CNAME`文件，文件内容就是自定义的域名。

此时，`.travis.yml` 文件需要增加一个新增动作，同步CNAME文件。
```yml
sudo: false
language: node_js
node_js:
  - 12 # use nodejs v10 LTS
cache: npm
branches:
  only:
    - main # build main branch only
script:
  - hexo generate # generate static files
  - cp CNAME public/ # 新增动作，同步CNAME文件。
deploy:
  provider: pages
  skip-cleanup: true
  github-token: $GH_TOKEN
  keep-history: true
  on:
    branch: main
  local-dir: public
```

## 注意
- 主分支 `main/master`，管理的是实际站点项目的源文件
- 自动部署会自动新增`gh-pages`分支，用来托管实际的站点文件
- GitHub项目 settings 中的 GitHub Pages项，Branchs 需改为 `gh-pages` 分支。
- 自定义域名，需要将其主机记录 CNAME 指向 username.github.io ，username 为你自己的实际名称。

# 参考资料
更多细节，请参见官方文档。
- https://hexo.io/zh-cn/docs/
- https://github.com/theme-next/hexo-theme-next
