---
title: hexo书写文档插入代码块和引用
date: 2019-12-31 22:50:55
categories:
    - hexo
    - 插件标签
tags:
---

使用hexo编写博客时，使用Markdown语法来插入代码、引用、图片时有时候并不起作用，推荐使用hexo标签插件来实现相同功能。

## 插入代码块

{% codeblock %}
{% raw %}{% codeblock [title] [lang:language] [option: value] %}
代码内容
{% endcodeblock %}
{% endraw %}{% endcodeblock %}

`[..]`内为可选项。其中 `[option: value]`有以下选项:

| Extra Options | Description                                                                                                                                                  | Default |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------- |
| `line_number` | Show line number                                                                                                                                             | `true`  |
| `highlight`   | Enable code highlighting                                                                                                                                     | `true`  |
| `first_line`  | Specify the first line number                                                                                                                                | `1`     |
| `mark`        | Line highlight specific line(s), each value separated by a comma. Specify number range using a dashExample: `mark:1,4-7,10` will mark line 1, 4 to 7 and 10. |         |
| `wrap`        | Wrap the code block in [`<table>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/table)                                                          | `true`  |


## 插入引用

{% codeblock %}
{% raw %}{% blockquote [author[, source]] %}
引用内容
{% endblockquote %}
{% endraw %}{% endcodeblock %}

`[..]`内为可选项。

## 插入图片和链接

要想能够正确使用，需要将 `config.yml` 文件中的 `post_asset_folder` 选项设为 `true` 来打开。

{% codeblock _config.yml lang:yml %}
post_asset_folder: true
{% endcodeblock %}

- 插入路径 `{% raw %} {% asset_path 同名文件夹下的文件路径 %} {% endraw %}`
- 插入图片 `{% raw %} {% asset_img 同名文件夹下的文件路径 [title] %} {% endraw %}`
- 插入链接 `{% raw %} {% asset_link 同名文件夹下的文件路径 [title] %} {% endraw %}`

## 插入url链接

{% codeblock  %}
{% raw %}{% link title url %}{% endraw %}
{% endcodeblock %}

## 其他

另外还有一起其他不常用插件标签 {% link 传送门 https://hexo.io/zh-cn/docs/tag-plugins %}



