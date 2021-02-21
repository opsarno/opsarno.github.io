---
title: Hexo Next 博客
date: 2020-02-19 11:52:35
tags: [hexo, next]
categories: [others]

---

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

`_config.next.yml` 文件中配置博客主题相关信息

<!--more-->
## 创建分类页
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

## 创建标签页
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

# 参考资料
- https://hexo.io/zh-cn/docs/
- https://github.com/theme-next/hexo-theme-next
