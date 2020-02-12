---
title: ss+混淆加密自动代理全攻略
date: 2020-01-23T09:49:57.793Z
coverImage: 
categories: 
    - ss
tags: 
    - ss
    - shadowsocks
    - 代理
---
<!-- toc -->

该教程中使用到的命令基于debian系linux操作系统。教程中涉及到ss客户端的安装、混淆插件`simple-obfs`的安装、加密方法`xchacha20-ietf-poly1305`的安装、自动代理和开机自启等相关内容。

<!-- more -->


## 安装混淆插件(可选)

混淆插件可以帮助混淆访问的目的ip, 避免某ip高频率访问而引起相关部门注意导致封锁服务器。此处推荐使用的是[`simple-obfs`](https://github.com/shadowsocks/simple-obfs)。

``` shell
sudo apt-get install --no-install-recommends build-essential autoconf libtool libssl-dev libpcre3-dev libev-dev asciidoc xmlto automake
git clone https://github.com/shadowsocks/simple-obfs.git
cd simple-obfs
git submodule update --init --recursive
./autogen.sh
./configure && make
sudo make install
```

## 安装附加加密方法(可选)

有时候ss自带的加密方法可能没有我们需要的加密方法，例如`xchacha20-ietf-poly1305`。这个时候可以安装[`libsodium`](https://github.com/jedisct1/libsodium/releases)以支持一些新的加密方法。

``` shell
下载最新版：libsodium-x.x.xx.tar.gz
解压并进入文件夹
./autogen.sh
./configure && make -j 
sudo make install
sudo ldconfig
```

## 安装ss本体

不多说，一条命令搞定。

``` shell
pip install https://github.com/shadowsocks/shadowsocks/archive/master.zip -U
```

## 运行

编写配置文件

``` json
{
    "server": "xx.xx.xx.xx",                // 服务器ip
    "server_port": xxx,                     // 服务器port
    "password": "xxxx",                     // 服务器密码
    "method": "xchacha20-ietf-poly1305",    // 加密方法
    "plugin": "obfs-local-x64",             // 混淆插件
    "plugin_opts": "obfs=http;obfs-host=www.bing.com",  // 混淆设置
    "remarks": "",
    "timeout": 500
}
```

运行： `sslocal -c 配置文件路径`

## 全局代理(不推荐)

配置系统全局代理：

deepin设置->网络->系统代理->手动代理

socks5: 127.0.0.1 1080

## 浏览器自动代理(推荐)

关闭系统代理选用chrome安装`proxy omega`自动代理。

autoproxy网址：https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt

## 开启开机自启

打开终端执行 `which python`,记下输出的内容 `/xxx/bin/python` 中的`xxx`,后面需要用到 

在系统home目录下打开 `.profile`文件，在结尾加上

```shell
# shadowsocks
nohup /xxx/bin/sslocal -c 配置文件路径 > shadowsocks.log 2>&1 &
```

重启即可实现开机自启，在home目录下的`shadowsocks.log`文件，可以查看程序运行的日志。
