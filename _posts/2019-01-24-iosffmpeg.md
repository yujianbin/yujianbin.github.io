---
layout: post
title: "四，iOS配置FFmpeg"
description: "ffmpeg"
category: FFMPEG
tags: [FFMPEG]
---
{% include JB/setup %}

### iOS配置FFmpeg

#####1，安装[yasm](http://yasm.tortall.net/Download.html)   
(OS X 下载 Source .tar.gz 即可 )

下载完成后，打开终端，cd 进目录   
分别执行下面命令：   

```
$ ./configure
$ make
$ make install
```

安装完成后验证，显示如下表明安装成功：

```
$ yasm --version

yasm 1.3.0
Compiled on May 12 2016.
Copyright (c) 2001-2014 Peter Johnson and other Yasm developers.
Run yasm --license for licensing overview and summary.
```

#####2，下载[gas-preprocessor.pl](https://github.com/yjbGitHub/gas-preprocessor)   
复制 gas-preprocessor.pl 到 /usr/local/bin 下 (网上很多教程都让放在/usr/bin 下，但是现在OS X的 usr/bin 目录不可写，所以放在 /usr/local/bin )    

修改文件权限 

```
$ chmod 777 /usr/local/bin/gas-preprocessor.pl
```

#####3，下载[脚本文件](https://github.com/kewlbear/FFmpeg-iOS-build-script)   
这个脚本可以一次编译，就生成适合各个框架的静态库。

解压后 cd 进目录 ，执行

```
$ ./build-ffmpeg.sh
```

脚本则会自动从github中把ffmpeg源码下到本地并开始编译。 编译结束后，在当前目录，会生成FFmpeg-IOS目录，这个目录中就有打包好的静态库和头文件，.a 静态库，一共有7个。 ffmpeg-3.0是源码，如图：


然后在终端输入如下命令，查看某一个静态库所支持的架构：

```
$ lipo -info libavcodec.a 
Architectures in the fat file: libavcodec.a are: armv7 i386 x86_64 arm64
```

#####4，把FFmpeg-iOS导入工程    
1>,将FFmpeg-IOS目录拖入工程；

2>,然后在 TARGETS - BuildSettings中找到Search Paths ，设置 Header Search Pahts 和 Library Search Paths 如下。不然会报 include“libavformat/avformat.h” file not found错误。

---
Hearder Search Paths

```
$(inherited)

"${PODS_ROOT}/Headers/Public"

"${PODS_ROOT}/Headers/Public/NSLogger"

$(PROJECT_DIR)/MyKxMovie/FFmpeg-iOS/include
```
---

Library Search Paths

```
$(inherited)
$(PROJECT_DIR)/MyKxMovie/FFmpeg-iOS/lib
```

#####5,在工程中加入其它库文件

libz.tbd 、ibbz2.tbd 、 libiconv.tbd,在 3.0以后需要添加另外2个框架 VideoToolbox.framework 和 AudioToolbox.framework.
