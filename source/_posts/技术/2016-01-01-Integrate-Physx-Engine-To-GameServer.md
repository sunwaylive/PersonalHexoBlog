---
layout: post
title: 为服务器集成NVIDIA Physx物理引擎
category: 技术
tags: Unity,Physx
keywords: Physx, 物理引擎
description:
---

&#160; &#160; &#160; &#160;[PhysX](https://developer.nvidia.com/physx-sdk)最初是AGEIA公司开发的物理运算引擎，跟Havok，Bullet这两款旗鼓相当，后来2008年AGEIA被NVIDIA收购，NVIDIA将PhysX保密，作为NVIDIA显卡的专利，这逐渐导致了其没落，在封闭了相当长一段时间后，NVIDIA最终将其开源，目前源代码已经托管在了Github上。

PhysX支持平台和引擎如下：
> 硬件平台：Win, OSX, Linux, XBOX, PlayStation, Andriod, IOS

> 游戏引擎：Unreal 3, Unreal 4, Unity

本文的**最终目的**：
>当美术在Unity中制作好游戏场景后(为gameobject拖好collider)，通过我们写的工具导出这份场景的collider配置，在服务器端能够生成一份一模一样的物理世界，从而由权威服务器去计算物理，诸如子弹有没有击中玩家等等。

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<!-- 336*280 -->
<ins class="adsbygoogle"
     style="display:inline-block;width:336px;height:280px"
     data-ad-client="ca-pub-1116434893916666"
     data-ad-slot="5557981430"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

## 1. 安装PhysX
### 1.1 获取GitHub上PhysX源代码的权限

&#160; &#160; &#160; &#160;PhysX源码并不对所有人开放，需要向Repo的holder申请，操作步骤参考如下：[详细步骤](https://developer.nvidia.com/physx-source-github)

总的来说就是4步：

 - 注册为nvidia的[developer programs](http://developer.nvidia.com/registered-developer-programs)
 - [申请权限](http://developer.nvidia.com/content/apply-access-nvidia-physx-source-code)，在最下面填上自己的GitHub Username就可以了。
 - 1个小时后查看github绑定的邮箱，就可以看到通过申请的回复邮件了
 - 访问Github上[PhysX的Repo](https://github.com/NVIDIAGameWorks/PhysX-3.3)， 就可以fork,clone下来了

### 1.2. PhysX安装

 - 源码clone下来之后， 按照根目录下的README.md很容的找到 /PhysX-3.3/PhysXSDK/Source/compile/目录下所需要的平台，编译出对应平台下的静态和动态链接库。
 - 集成/PhysX-3.3/Include 和 /PhysX-3.3/Lib 和 /PhysX-3.3/Bin到自己的程序使用。

**备注:** 

在根目录下，我们会看到另外一个APEXSDK， 这个是一套读取设计师通过[PhysX Lab](https://developer.nvidia.com/gameworksdownload#?dn=physx-lab-1-3-0)等工具导出的设计的资源的API(资源有自己定义的格式如 “.apb”).

例如：场景中物体的摧毁效果，布料，粒子等效果，比方说其中有一个模块： Destruction Module， 有兴趣的可以看一下[这套视频](https://developer.nvidia.com/apex-destruction-physxlab-tutorials)，半个小时左右，就了解了，这个不是这篇文章的重点。
 
## 2. Unity Editor中导出物理场景的配置

&#160; &#160; &#160; &#160;Unity中基本的Collider分为：Box Collider, Sphere Collider, Capsule Collider,以及针对精度要求较高的Mesh Collider，还有其它的一些特定应用场景下会用到的诸如Wheel Collider，Terrain Collider等等。

&#160; &#160; &#160; &#160;我们今天主要目的搭建好整个流程，只用基本类型的Collider作为例子。

&#160; &#160; &#160; &#160;我们前后台都统一使用[Google Proto buffers](https://developers.google.com/protocol-buffers/)定义场景导出的格式，以二进制格式序列化和反序列化场景。

### 2.1 定义Scene.proto
&#160; &#160; &#160; &#160;整个proto代码贴在了下方，我们简单说明一下，很容易就明白了。

每一个战斗场景导出一个文件，格式如下：

```c#

message U3DPhysxScene
{
    optional int32 id                         = 1;
    optional string scene_name                = 2;
    repeated U3DPhysxBox box_collider         = 3;
    repeated U3DPhysxSphere sphere_collider   = 4;
    repeated U3DPhysxCapsule capsule_collider = 5;
    repeated U3DPhysxMesh mesh_collider       = 6;
}
```


一个场景由若干个U3DPhysxBox，U3DPhysxSphere，U3DPhysxCapsule，U3DPhysxMesh组成.

每个collider的定义很容易理解， 其中Capsule Collider在PhysX的定义如下：

![PhysX Capsule Collider定义](/images/PhysX_Capsule_Collider.png)

```c#
package killer.proto;

enum ColliderType
{
	BOX     = 1;
	SPHERE  = 2;
	CAPSULE = 3;
	MESH    = 4;
}

message Position
{
	optional double x = 1 [default = 0];
	optional double y = 2 [default = 0];
	optional double z = 3 [default = 0];
}

message U3DPhysxScene
{
	optional int32 id                         = 1;
	optional string scene_name                = 2;
	repeated U3DPhysxBox box_collider         = 3;
	repeated U3DPhysxSphere sphere_collider   = 4;
	repeated U3DPhysxCapsule capsule_collider = 5;
	repeated U3DPhysxMesh mesh_collider       = 6;
}

message U3DPhysxSphere
{
	optional int32 id          = 1;
	optional ColliderType type = 2;
	optional Position pos      = 3;
	optional double radius     = 4;
}

message U3DPhysxBox
{
	optional int32 id          = 1;
	optional ColliderType type = 2;
	optional Position pos      = 3;
	optional double x_extents  = 4;
	optional double y_extents  = 5;
	optional double z_extents  = 6;
}

message U3DPhysxCapsule
{
	optional int32 id          = 1;
	optional ColliderType type = 2;
	optional Position pos      = 3;
	optional double raduis     = 4;
	optional double height     = 5;
}

message U3DPhysxMesh
{
	optional int32 id           = 1;
	optional ColliderType type  = 2;
	optional int32 vertex_count = 3;
	repeated Position vertices  = 4;
}
```

### 2.2 修改Scene.proto

- 进入 Editor/BuildProtocol/Protocal， 修改Scene.proto, 然后运行 Editor/BuildProtocol/build.bat，这样protobuf会在 Editor/ProtoGeneratedScrips 目录下生成新的Scene.cs。
- 然后刷新Unity，修改 Editor/SceneExtractor.cs 脚本使用新的Scene.proto的字段。

### 2.3 代码下载并运行
整个Unity Editor的代码(c#) 和 依赖的第三方库(proto-net)，都已经打包好了.

下载下来直接放到Unity的Assets目录下，刷新unity.

在菜单栏中： Tools->SceneExtractor->ExtractColliders即可导出当前打开的场景为一份物理配置给后台使用。

代码托管在了Github上: [https://github.com/sunwaylive/UsePhysXWithUnity/tree/master/1.ExportScenesInUnity](https://github.com/sunwaylive/UsePhysXWithUnity/tree/master/1.ExportScenesInUnity)


# 3. 服务器端导入场景的配置文件，生成物理场景
因为后台在跑的服务器没有显卡，所以我们服务器端的工作，分为如下两步

## 3.1 在windows平台上，反序列化出场景，用OpenGL渲染测试正确性

### 1. 这部分工程代码也封装好了，可以下载下来直接跑
代码托管在了Github上: [https://github.com/sunwaylive/UsePhysXWithUnity/tree/master/2.RunPhysicsOnWindows](https://github.com/sunwaylive/UsePhysXWithUnity/tree/master/2.RunPhysicsOnWindows)

平台：windows7 x64 + Visual Studio 2013 + OpenGL

语言： C++

具体每个文件夹的含义以及如何导proto，请看工程根目录下的README.md文件。

在跑工程的时候，有可能会遇到一些问题，比方说：OpenGL没有安装等，请移步 /external 目录,里面有两张png是解决没有安装OpenGL问题的。

- 如果没有安装OpenGL，请将 /external/GL 目录拷贝到 \*\*/Program Files(x86)/Microsoft Visual Studio 12.0/VC/include/GL。对于Visual Studio的其它版本，放到对应的目录下去就好了。
- 同样如果在链接阶段报lib文件找不到的话，将external/\*.lib 这3个lib放到 \*\*/Program Files(x86)/Microsoft Visual Studio 12.0/VC/lib/amd64 这个目录下去。

### 2. 代码结构说明
* PhysicsSceneManager是**核心**的类，管理了初始化PhysX，导入场景等核心功能
* PhysicsSceneRender和ServerCooperationRender负责渲染
* PhysicsSceneCamera是控制相机镜头的类
* Main.cpp是工程入口，通过RENDERFORTEST宏去控制是否开启OpenGL渲染

我们打开PhysicsSceneManager.cpp, 看到最重要的两个函数： InitiPhysics()和InitiSceneFromFile().

需要注意的是m\_foundation， m\_physics是在程序一启动的时候就必须初始化的。

其它的代码配合PhysX的Documentation看，很容易就看明白了。


## 3.2 关闭OpenGL渲染，将Simulation部分的代码放到Linux上去
&#160; &#160; &#160; &#160;代码托管在了Github上: [https://github.com/sunwaylive/UsePhysXWithUnity/tree/master/3.RunPhysicsOnLinux](https://github.com/sunwaylive/UsePhysXWithUnity/tree/master/3.RunPhysicsOnLinux)

&#160; &#160; &#160; &#160;最后关闭 **RENDERFORTEST** 宏， 并删除PhysicsSceneRender.h, PhysicsSceneRender.cpp和SeverCooperationRender.cpp，并将对代码进行简单的封装，就可以放到服务器上运行了。

这一部分的代码部分可以在这里下载，但是linux下的physx环境配置需要自行安装，安装过程也很简单， 不能直接make。

目录下有一个可执行的二进制文件 PhysXTestBinary，是我们编译出来测试的。



# 4.运行测试
1. Unity制作测试场景
![Unity PhysX Test Scene](/images/Unity_PhysX_test_scene.png)

2. 在Windows下服务器端渲染的场景
![Server Collider Render Scene](/images/Server_Collider_Render_Scene.png)

3. 在Linux服务器上，利用PhysX的Raycast的API， 可以正确检测其中一对点可以到达， 另外一对点由于中间Box collider的阻挡无法达到。

至此，本文完， 欢迎指正交流。
