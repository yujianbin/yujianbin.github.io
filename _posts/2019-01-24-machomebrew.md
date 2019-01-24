---
layout: post
title: "一，MAC安装Homebrew"
description: "ffmpeg, homebrew"
category: FFMPEG
tags: [FFMPEG]
---
{% include JB/setup %}

####1, 安装homebrew
建议大家去Homebrew官网去安装，网上博客给出的连接，不安全，也不稳定，可能会导致Homebrew装偏（我就遇到了这个问题）

[Homebrew 官网](https://brew.sh/index_zh-cn.html)

检查ruby是否安装：    

```
ruby --version
ruby 2.0.0p648 (2015-12-16 revision 53162) [universal.x86_64-darwin15]
```

执行安装homebrew指令：

```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

当brew安装成功后，就可以随意安装自己想要的软件了，例如wget，命令如下：

```
sudo brew install wget  
```

卸载的话，命令如下：

```
sudo brew uninstall wget
```

查看安装软件的话，命令如下：

```
sudo brew search /apache*/
```
注意/apache*/是使用的正则表达式，用/分割。

####2, 卸载homebrew

```
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/uninstall)"
```

问题:
安装完homebrew之后，用homebrew安装工具包的时候，提示没有权限，如下：

```
error: could not lock config file .git/config: Permission denied
Error: Failure while executing: git config --local --replace-all homebrew.private false
```

解决办法:
执行下面两条指令：

```
//确保目录归属管理组
sudo chgrp -R admin /usr/local
//确保管理组可读
sudo chmod -R g+w /usr/local
```
然后就可以正常安装程序包了；




