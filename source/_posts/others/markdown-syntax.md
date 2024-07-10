---
title: Markdown 语法指南
date: 2024-07-10 16:13:27
tags: [Markdown]
categories: [Others, Markdown]
---

# 概述 (Overview)

Markdown 是一种轻量级标记语言，可用于向纯文本文档添加格式元素。
Markdown 语言在 2004 由 [约翰·格鲁伯（John Gruber）](https://daringfireball.net/projects/markdown/)创建，现在是世界上最流行的标记语言之一。

<!--more-->

# 标题 (Heading)

在文本前面加 `#` 号，即可创建 1-6 级标题。标题级别与 `#` 号个数有关。
```txt
# 一级标题

内容......

## 二级标题

### 三级标题

......
```

> Tips: 建议 # 号后面加空格，再写标题文本，以及标题与内容之间也应加个空格。

# 段落 (Paragraphs)

段落之间使用空行分隔
```txt
不要在段落前面加 tab 或者 空格。

像这样保持左对齐。
```

# 换行 (Line Breaks)
在行的末尾加上两个及以上的空格 (这称为 trailing whitespace，行末空格)，或者用 `<br>` HTML 标记实现。
```txt
第一行，后面有两个空格。  
然后是换行内容。

第一行，后面有 HTML 标记.<br>
然后是换行内容。
```

# 强调 (Emphasis)

您可以通过将文本设为粗体或斜体来增加强调。

## 粗体 (Bold)

若要加粗文本，在单词或短语前后加两个 `*` 号或 `_` 下划线。单词或词语的中间则只能用 `*` 号。

**这是粗体**

```txt
我喜欢用 **加粗文本**

我喜欢用 __加粗文本__

词语中间**粗体**只能用星号
```

## 斜体 (Italic)

若要斜体文本，在单词和短语前后加一个 `*` 号或 `_` 下划线。单词或词语的中间则只能用 `*` 号。 

*这是斜体*

```txt
请看，*这是斜体的内容*。

请看，_这是斜体的内容_。

词语中间的*斜体*内容。
```

## 加粗加斜

与上面的粗体、斜体类似，这里加三个 `*` 号或 `_` 下划线。 

***这是加粗加斜***

```txt
心累了，***就这样吧***。
```

# 块引用 (Blockquotes)

创建块引用，在段落前面添加 `>` ，块引用中可嵌套其它元素。

> 这是块引用

```txt
> 这是块引用

> 我是第一行块引用
> 我是第二行块引用

> #### 块引用中的标题
> - 列表
> **加粗的文本内容**
>> 嵌套块引用
```

# 列表 (Lists)

## 有序列表

要创建有序列表，请添加数字后跟句点的行项目。数字不必按数字顺序排列，但列表应该以数字1开头。

1. 列表项一
2. 列表项二
    1. 缩进项一
    2. 缩进项二
3. 列表项...

```txt
1. 列表项一
2. 列表项二
    1. 缩进项一
    2. 缩进项二
3. 列表项...
```

## 无需列表

要创建无序列表，请在行项目前添加 `短划线（-）、星号（*）或加号（+）`。缩进一个或多个项目以创建嵌套列表。

- 第一项
- 第二项
    - 缩进项
    - 缩进项
- 第三项...

```txt
- 第一项
- 第二项
    - 缩进项
    - 缩进项
- 第三项...
```

## 列表中添加其它元素

要在列表中添加另一个元素，同时保持列表的连续性，请将该元素缩进四个空格或一个制表符，如以下示例所示。

- 这是第一行项目
- 这是第二行项目
    > 这是块引用内容
- 这是其它项目

```txt
- 这是第一行项目
- 这是第二行项目
    > 这是块引用内容
- 这是其它项目
```

# 代码 (Code)

要将单词或短语表示为代码，请将其括在反引号（`）中。

Linux 中编辑文档用 `vim` 命令。

```txt
Linux 中编辑文档用 `vim` 命令。
```

## 跳脱反引号

使用两个反引号括起来的文本中的一个或多个反引号将被转义。

``使用 `code` 在 Markdown 文件中。``

```txt
``使用 `code` 在 Markdown 文件中。``
```

## 代码块

基础语法中，要创建代码块，请将代码块的每一行缩进至少四个空格或一个制表符。

    <html>
      <head>
      </head>
    </html>


要创建不带缩进的代码块，使用扩展语法中的 `fenced code blocks` 使用三个反引号来实现。有些编辑器支持语法高亮。

```json
{
  "firstName": "John",
  "lastName": "Smith",
  "age": 25
}
```

~~~
```json
{
  "firstName": "John",
  "lastName": "Smith",
  "age": 25
}
```
~~~

# 分割线 (Horizontal Rules)

要创建分割线，请在一行中单独使用三个或三个以上星号（***）、短划线（---）或下划线（___）。

---

# 链接 (Links)

要创建链接，请将链接文本括在中括号中，然后立即在其后面加上圆括号中写 URL。

例如：这是 [Github](https://github.com/) 的访问地址。

`这是 [Github](https://github.com/) 的访问地址`.

## 添加链接标题

可选的为链接添加标题。当用户将鼠标悬停在链接上时，这将显示为提示。要添加标题，请在URL后用引号将其括起来。

例如：这是 [Github Pages](https://pages.github.com/ "Github Pages") 的访问地址。

`这是 [Github Pages](https://pages.github.com/ "Github Pages") 的访问地址`.

## URLs 和 Email

要将URL或电子邮件地址快速转换为链接，请将其括在尖括号中。

<https://opsarno.github.io>
<opsarno@qq.com>

```txt
<https://opsarno.github.io>
<opsarno@qq.com>
```

## 格式化链接

要强调链接，请在括号和圆括号前后添加星号。要将链接表示为代码，请在括号中添加反勾号。

*[Github](https://github.com/)* 真好用
**[Github](https://github.com/)** 确实好用。
查看章节 [`链接`](#链接-links)

```txt
*[Github](https://github.com/)* 真好用
**[Github](https://github.com/)** 确实好用。
查看章节 [`链接`](#链接-links)
```

## 参考类型链接

了解即可。

例如我们引用 [维基百科 Markdown][2] 的参考链接


[1]: <https://en.wikipedia.org/wiki/Markdown> "维基百科 Markdown"
[2]: <https://zh.wikipedia.org/wiki/Markdown> "维基百科 Markdown"


```txt
例如我们引用 [维基百科 Markdown][2] 的参考链接


[1]: <https://en.wikipedia.org/wiki/Markdown> "维基百科 Markdown"
[2]: <https://zh.wikipedia.org/wiki/Markdown> "维基百科 Markdown"
```

# 图片 (Images)

添加图片，感叹号（！），后跟中括号中的alt文本，并在括号中添加图像资源的路径或URL。您可以选择在路径或URL后添加带引号的标题。

```txt
![alt图片文本](url "可选的title文本")
```

## 图片链接

添加图片链接，就是结合链接语法结合起来。

```txt
[![alt图片文本](url "可选的title文本")](URL)
```

# 转义字符 (Escaping Characters)

对于Markdown文档中格式化文本的字符，要显示其字符本身，可在前面添加反斜杠转义。

\* 转义星号，避免成为无序列表

```txt
\* 转义星号，避免成为无序列表
```

# HTML

大部分 Markdown 应用允许在 Markdown 格式文本中使用 HTML 标签。这能够定制化更多的元素属性。

例如：<a href="https://github.com/" target="_blank">Github</a> 链接在新标签打开

```html
<a href="https://github.com/" target="_blank">Github</a>
```


# 参考资料

- [Markdown Guide - Basic Syntax](https://www.markdownguide.org/basic-syntax/)
- [Markdown Guide - Extended Syntax](https://www.markdownguide.org/extended-syntax/)
