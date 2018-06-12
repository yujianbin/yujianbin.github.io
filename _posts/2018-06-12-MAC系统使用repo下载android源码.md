---
layout: post
title: "MAC系统使用repo下载Android源码"
description: "MAC, repo, Android源码"
category: Android
tags: [android]
---
{% include JB/setup %}

### 一，要安装 Repo，请执行以下操作：
###### 1,确保主目录下有一个 bin/ 目录，并且该目录包含在路径中:

```
mkdir ~/bin
PATH=~/bin:$PATH
```
###### 2, 下载 Repo 工具，并确保它可执行：

```
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
```
上面是Google的repo文件，由于在墙外，所以请使用下面清华大学提供的repo；

```
# 清华大学repo
curl https://mirrors.tuna.tsinghua.edu.cn/git/git-repo > ~/bin/repo

#修改repo权限，可以用 ‘ll ~/bin/repo’ 命令查看
chmod a+x ~/bin/repo
```

### 二，初始化repo

###### 1, 创建目录，多级目录，加 -p 参数；
```
puppy@bogon ~ : mkdir -p AOSP/android6.0
```
###### 2, 进入到目录；
```
 puppy@bogon ~ : cd AOSP/android6.0
```

###### 3, 初始化仓库：

```
#如果后面不加 -b 分支参数，默认初始化的是master分支的最新版本
repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest -b android-6.0.1_r78
```
执行完上面命令，会卡住，然后会失败。这是因为repo会默认去Google找repo的最新版本；解决方法如下：

1), 由于repo就是Python脚本，所以可以直接改代码：

```
vim ~/bin/repo
```

可以改下面的REPO_URL, 但是不建议；
```
#!/usr/bin/env python

# repo default configuration
#
import os
REPO_URL = os.environ.get('REPO_URL', None)
if not REPO_URL:
  REPO_URL = 'https://gerrit.googlesource.com/git-repo'
REPO_REV = 'stable'

# Copyright (C) 2008 Google Inc.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
```

2), 导入清华大学的变量：

在终端执行：
```
export REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo'
```
由于每次更新版本，都需要执行上面命令；为了方便，配置每次打开终端时候都执行此命令；由于我使用的item2的终端：

```
vim ~/.zshrc
```
在后面加入如下命令：

```
# 加入repo的环境变量
PATH=~/bin:$PATH

# 每次启动时执行导入变量命令
export REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo'
```

###### 4, 删掉repo目录下面的.repo，然后再执行初始化仓库的命令；

```
repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest -b android-6.0.1_r78
```
然后就会初始化成功；

###### 5，同步代码：

```
repo sync
```
