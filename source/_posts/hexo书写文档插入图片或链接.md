---
title: hexo书写文档插入图片或链接
date: 2019-12-29 21:18:03
categories:
    - hexo
    - 插件标签
tags:
---

## 开启文章资源文件夹
对于那些想要更有规律地提供图片和其他资源以及想要将他们的资源分布在各个文章上的人来说，Hexo也提供了更组织化的方式来管理资源。这个稍微有些复杂但是管理资源非常方便的功能可以通过将 `config.yml` 文件中的 `post_asset_folder` 选项设为 `true` 来打开。

{% codeblock _config.yml lang:yml %}
post_asset_folder: true
{% endcodeblock %}

## 插入资源

当资源文件管理功能打开后，`Hexo`将会在你每一次通过 `hexo new [layout] <title>` 命令创建新文章时自动创建一个文件夹。这个资源文件夹将会有与这个文章文件一样的名字。将所有与你的文章有关的资源放在这个关联文件夹中之后，你可以通过相对路径来引用它们，这样你就得到了一个更简单而且方便得多的工作流。其中`[..]`内为可选项。

- 插入路径 `{% raw %} {% asset_path 同名文件夹下的文件路径 %} {% endraw %}`
- 插入图片 `{% raw %} {% asset_img 同名文件夹下的文件路径 [title] %} {% endraw %}`
- 插入链接 `{% raw %} {% asset_link 同名文件夹下的文件路径 [title] %} {% endraw %}`

比如说：当你打开文章资源文件夹功能后，你把一个 example.jpg 图片放在了你的资源文件夹中。然后插入

`{% raw %}{% asset_img example.jpg This is an example image %}{% endraw %}`

通过这种方式，图片将会同时出现在文章和主页以及归档页中。
