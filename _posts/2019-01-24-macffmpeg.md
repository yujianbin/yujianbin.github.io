---
layout: post
title: "二，MAC安装FFMPEG"
description: "ffmpeg, homebrew"
category: FFMPEG
tags: [FFMPEG]
---
{% include JB/setup %}

### 一，mac下使用Homebrew安装FFmpeg     

```
brew install ffmpeg

```

### 二，mac 下编译安装ffmpeg     

##### 1，安装[yasm](http://yasm.tortall.net/Download.html) (这个是最新版本)      
（OSX 下载Source.tar.gz ，需要用下面的命令去解压，然后在安装；）

---  
Yasm是一个完全重写的NASM汇编。目前，它支持x86和AMD64指令集，接受NASM和气体汇编语法，产出二进制， ELF32 ， ELF64 ， COFF ， Mach - O的（ 32和64 ） ， RDOFF2 ，的Win32和Win64对象的格式，并生成STABS 调试信息的来源，DWARF 2 ，CodeView 8格式。

---

终端命令如下：
下载(这个是1.2.0的版本)：

```
curl http://www.tortall.net/projects/yasm/releases/yasm-1.2.0.tar.gz >yasm.tar.gz
```

解压

```
tar xzvf yasm.tar.gz
```

进入目录

```
cd yasm-1.2.0
```
执行配置

```
./configure
```

编译

```
make
```

安装

```
sudo make install
```

##### 2，编译源码安装FFmpeg
1,github下载ffmpeg源代码   
[ffmepg源码下载](https://github.com/FFmpeg/FFmpeg)

2，进入目录，执行下面指令，进行编译和安装
配置

```
./configure
```
编译

```
make
```

安装

```
sudo make install
```
手动编译，安装的ffmpeg，有问题：
卸载： 进入到目录：

```
sudo make uninstall
```

