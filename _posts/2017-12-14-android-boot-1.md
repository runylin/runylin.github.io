---
layout: post
title: "Android启动相关杂谈（一）概念部分"
categories:
- Android
tags:
- 安卓系统
---

### 零、启动模式
从这个图的一个流程（右下load boot.img这路）谈起
![](http://7xt9nx.com2.z0.glb.clouddn.com/recovery.png)

### 一、几个概念
相信接触过TV Android电视的同学，对下面这几个概念应该都有印象吧。但是刚开始接触可能有点混淆，下面按自己的理解记录下。
- pm
- bootloader
- fastboot
- uboot
- boot.img

#### 1.1 pm(powermanager?)
在TV或者机顶盒产品中，一般为了待机时的低功耗会多一颗C51单片机，当接收到IR/蓝牙/wifi开机信息时，再去拉主芯片引脚启动主芯片，从而进入主芯片的bootloader的程序。在这颗C51芯片的代码我们一般称为pm代码，至于为啥大家都这么叫，可能是因为它最主要的功能就是电源管理吧。

#### 1.2 bootloader/uboot
从字面上来说，bootloader为引导程序的含义。uboot是bootloader的一种。带linux系统的都有这段代码。

BootLoader可以分为两个阶段。第一个阶段使用汇编来实现，它完成一些依赖于CPU体系结构的初始化，并调用第二阶段的代码；第二阶段则通常使用C语言来实现，这样可以实现更复杂的功能，而且代码会有更好的可读性和移植性。BootLoader既然要做硬件初始化之类的，必然和硬件相关，所以它的代码并非通用的，不同的硬件需要不同的BootLoader代码，各大厂商可能都有自己的，并且加入开机画面之类的。最常听说的是uboot和hboot，后者是htc的bootloader。

```
两个阶段如下：
一、第一阶段功能
（1）硬件设备初始化；
（2）为加载bootloader的第二个阶段代码准备RAM空间。
（3）复制bootloader的第二个阶段代码到RAM中；
（4）设置好栈。
（5）跳转到第二阶段代码的C入口点。

二、第二个阶段的功能
（1）初始化本阶段要使用到的硬件设备。
（2）检测系统内存映射。
（3）将内核映像和根文件系统从Flash上读到RAM空间中。
（4）为内核设置启动参数。
（5）调用内核。
```

海思芯片的bootloader代码放在Android/device/hisilicon/bigfish/sdk/source/boot/fastboot下。
```
./bigfish/sdk/source/boot/Makefile:53:OBJ_TARGET := $(OBJ_DIR)/fastboot-burn.bin
./bigfish/build/emmc.mk:89:	cp -avf $(ANDROID_BUILD_TOP)/$(EMMC_HIBOOT_OBJ)/fastboot-burn.bin $(ANDROID_BUILD_TOP)/$(EMMC_PRODUCT_OUT)/fastboot-burn-emmc.bin
```
可以看到这段代码最终生成的是fastboot-burn-emmc.bin文件

具体uboot代码解析，之前在学校有帮师兄修修补补过一本书，叫《ARM Cortex-A8实战演练》。在P175页开始有介绍uboot。**公众号回复uboot，可以获取这本书pdf版下载链接**

#### 1.3 fastboot
fastBoot是bootloader提供的一个功能。可以通过数据线与电脑连接，然后在电脑上执行一些命令，如刷系统镜像到手机上。fastboot可以理解为实现了一个简单的通信协议，接收命令并更新镜像文件，其他什么的干不了。

#### 1.4 boot.img
以Nexus6p为例，刷机时除了boot.img还有bootloader-angler-angler-02.45.img。所以可以理解boot.img其实没有包含bootloader的代码的。

*备注：海思代码生成的kernel.img其实就是boot.img*

android 的boot.img 包括 boot header，kernel，ramdisk。


### 二、谈谈boot.img
下节谈
在我们认知里面，启动是


### 参考链接：
- http://blog.jobbole.com/67931/
- http://www.cnblogs.com/shangdawei/p/4356679.html
- http://blog.csdn.net/zhenwenxian/article/details/6095487
