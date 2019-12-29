---
title: 为git操作设置别名
date: 2019-12-28 22:37:19
categories: 
    - git
    - alias
tags:
---

## 1. git config --global alias.别名 原名
当然也可针对某一仓库设置：  
去掉--global即可

一些建议：
``` git 
$ git config --global alias.st status
$ git config --global alias.co checkout
$ git config --global alias.ci commit
$ git config --global alias.br branch
$ git config --global alias.unstage 'reset HEAD'
$ git config --global alias.last 'log -1'
$ git config --global alias.lg "log --graph --abbrev-commit --decorate --date=relative --format=format:'%C(bold blue)%h%C(reset) - %C(bold green)(%ar)%C(reset) %C(white)%s%C(reset) %C(dim white)- %an%C(reset)%C(bold yellow)%d%C(reset)' --all"
```

## 2. 配置文件
每个仓库的Git配置文件都放在.git/config文件中  
别名就在[alias]后面，要删除别名，直接把对应的行删掉即可。

{% asset_image 1537259071_18043.png %}

而当前用户的Git配置文件放在用户主目录下的一个隐藏文件.gitconfig中：  
配置别名也可以直接修改这个文件，如果改错了，可以删掉文件重新通过命令配置。

{% asset_image 1537259094_30013.png %}