---
layout: post
title: Mac下安装OGRE
category: 技术
tags: OGRE
keywords: OGRE, 游戏引擎
description:
---

[OGRE](http://www.ogre3d.org/)官网提供了各种开发平台下的搭建步骤，并且很多依赖都能够下载到。但是组织不够清晰，需要花一些时间去理清思路。这花了我大概一天时间去尝试各种搭建方法。

<!-- more -->

总的来说搭建步骤很简单。
## 1. 搭建环境
Mac OS 10.10.2

CMake 3.2.1

Xcode 6.2
## 2. 搭建步骤
### 2.1 下载OGRE源代码，
https://bitbucket.org/sinbad/ogre

这个步骤需要hg支持，安装一下就可以了，当然git应该也是可以的。
下载完成后我们会看到这样一个目录：

![OGRE目录](/images/OGRE_directory.png)

### 2.2 下载OGRE的依赖库
https://bitbucket.org/cabalistic/ogredeps

依赖库放到ogre源代码的根目录下，跟OGREMain同级。

### 2.3 生成OGRE依赖库在MAC平台下的工程文件并编译
使用CMake编译，源代码的路径选择 步骤2.2中的\*\*/ogre/ogredeps 的目录；在ogredeps下面新建build目录，用于放build的结果。

如下图所示：

![编译OGRE依赖1](/images/OGRE_generate_deps.png)

编译完成之后，在build目录下，我们会看到 OGREDEPS.xcodeproj 这个文件
打开之后，会看到如下图：
![编译OGRE依赖2](/images/OGRE_compile_deps.png)


需要分 2个 步骤编译，安装依赖库，

a. 首先选择 ALL_BUILD编译一次

b. 然后选择 install编译一次


### 2.4 生成OGRE在MAC平台下的工程文件编译并编译
同样适用CMake编译，路径配置如图所示：

![编译OGRE源代码](/images/OGRE_generate_src.png)

首次点击Configure之后，会看到全部是红色选项，没有关系。
找到其中的 OGRE_DEPENDENCIES_DIR， 把它的值设置成 \*\*/ogre/ogredeps/build/ogredeps，再点击Configure, 会看到顺利完成，然后点击generate。
这样编译完成后，打开 \*\*/ogre/build目录，会看到如下结果

![ogre/build_dir](/images/OGRE_build_dir.png)

其中有一个文件是 OGRE.xcodeproj。打开之后会看到如下图：

![编译OGRE源代码2](/images/OGRE_compile_src.png)

需要分 3个 步骤编译(跟上面编译依赖类似)，安装依赖库，

a. 首先选择 ALL_BUILD编译一次

b. 然后选择 install编译一次

c. 最后编译SampleBrowser(编译列表往下拖就可以看到这个选项)

## 3.搭建完成
如果顺利编译完成的话，我们在\*\*/ogre/build/bin/debug下面就可以看到SampleBrowser的可执行文件了，打开就可以浏览了。同样你也可以编译出Release版，在Xcode的scheme中将debug改成release再编译一次即可。
如图所示：

![OGRE例子](/images/OGRE_sample.png)
