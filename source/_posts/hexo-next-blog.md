---
title: Hexo Next 博客
date: 2021-02-19 11:52:35
tags: 
  - hexo
  - blog
  - next
categories:
  - others
---

# 准备工作

- Node
- Git
- GitHub

安装 Hexo
```bash
npm install -g hexo-cli
```


初始化基础环境
```bash
hexo init <folder>
cd <folder>
npm install
```

`_config.yml` 文件中配置博客的基本信息


## 创建分类文件夹
```
hexo new page categories
```

编辑 `source/categories/index.md` 文件，添加 `type: "categories"`
```
---
title: categories
date: 2019-07-15 23:30:33
type: "categories"
---
```

## 创建标签文件夹
```
hexo new page tags
```

编辑 `source/tags/index.md` 文件，添加 `type: "tags"`
```
---
title: tags
date: 2019-07-15 23:30:33
type: "tags"
---
```
