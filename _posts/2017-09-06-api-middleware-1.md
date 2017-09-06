---
layout: post
title: "TV中间件实现杂谈（一）"
categories:
- Middleware
tags:
- 中间件
---

> 之前参考公司的中间件写了一个demo，其中有些知识点是第一次接触到，有些有新的认识。打算写个系列记录一下

### 零、前言
可能很多人对中间件的理解不一样，那就先介绍下本文的中间件是指啥。现在我们上层APK（比如开机向导和设置菜单等）需要使用TV相关的接口（比如搜台，切通道等），这些接口一般由电视芯片厂商提供，现在智能电视芯片主要有hisilicon，Mstar，MTK，Amlogic等。每个厂商都会有模块来实现和TV相关的功能，Mstar的叫supernova，hisilicon的叫hippo和hidolphin。这些模块都会有JAVA的接口。

使用这些的芯片的厂商会因为规模有不同的软件架构去对接，这也是一年多和同事原厂交流总结出来的。当然可能其他公司存在更好的解决方案。有时候也很好奇手机厂商比如国际米是怎么维护它们那多款手机型号的。回归正题：
- 有些就直接在各个上层apk直接引入上面所说的jar包，对接不同原厂的芯片时需要直接修改上层代码，维护性差更新芯片时导入慢。
- 有些就自己定义一套TV的api，然后分层，上层调用这套api，具体实现单独一个apk去对接上述jar包。当芯片更新时，上层apk不用修改，只需要重新对接原厂的jar包即可。
- 有些就不对接JAVA的接口，这套自己写，对接的是驱动（比如tunner，pannel等）的接口。因为驱动接口基本不会有变动，当这套稳定了，其实对接其他芯片是很快的。

我们采用的是第二种，对接的是原厂提供给我们的JAVA层的接口，那么使用这个接口我们就需要导入相应的jar包。里面主要包含以下内容：

![](http://7xt9nx.com2.z0.glb.clouddn.com/hisi-api.png)

### 一、系统相应模块生成jar包
#### 把java代码打包为jar
```
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
# Build all java files in the java subdirectory
LOCAL_SRC_FILES := $(call all-subdir-java-files)
# Any libraries that this library depends on
LOCAL_JAVA_LIBRARIES := android.test.runner
# The name of the jar file to create
LOCAL_MODULE := HitvShare
# Build a static jar file.
include $(BUILD_STATIC_JAVA_LIBRARY)
```

#### 另外如果已经有jar包直接预装到系统的Android.mk文件
```
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_PREBUILT_STATIC_JAVA_LIBRARIES := libdemoapi:demoapi.jar

include $(BUILD_MULTI_PREBUILT)
include $(CLEAR_VARS)
LOCAL_MODULE := cn.linxtu.demo.api
LOCAL_MODULE_TAGS := optional
LOCAL_STATIC_JAVA_LIBRARIES := libdemoapi
include $(BUILD_JAVA_LIBRARY)
```

**notices**
- 如果是在编译整个android的源码，这个LOCAL_MODULE需要添加到了PRODUCT_PACKAGES和PRODUCT_BOOT_JARS 变量里
- 如果需要系统为应用自动加载动态库，需要在/system/etc/permissions中添加<library>条目。
```
1、add build/target/product/linxtu_base.mk in build/target/product/core_minimal.mk\
+PRODUCT_PACKAGES += \
+    HitvShare 
+
+PRODUCT_BOOT_JARS += \
+    HitvShare \
+
+PRODUCT_COPY_FILES += \
+    device/common/preinstall/permission/linxtu-demo-api.xml:system/etc/permissions/linxtu-demo-api.xml
+
2、add device/common/preinstall/permission/cvte-tv-api.xml
--- /dev/null
+++ b/device/common/preinstall/permission/cvte-tv-api.xml
@@ -0,0 +1,5 @@
+<?xml version="1.0" encoding="utf-8"?>
+<permissions>
+    <library name="cn.linxtu.demo.api" file="/system/framework/cn.linxtu.demo.api.jar"/>
+</permissions>
```

这样编译完在out/target/product/XXX/system/framework下就会生成HitvShare.jar，这个是生成机器的字节码，运行时用;
在out/target/common/obj/JAVA_LIBRARIES/HitvShare_intermediates会有一个classes.jar，这个里面有class文件，可以拷贝出来，在apk中依赖用。

### 二、APK依赖这个jar包
apk分源码编译和Gradle或者其他IDE编译。其实源码编译只需在Android.mk中LOCAL_JAVA_LIBRARIES指定就可以了。所以这里说的是Gradle编译。

Gradle编译依赖jar有两种方式：compile和provided。后者仅仅在编译时使用，但最终不会被编译到apk或aar里。

因为我们上面已经在安卓整包编译时在system/framework已经生成库文件，而且有时候也有可能改变到逻辑，所以这里用provided编译，只要接口没变。我们的apk所依赖的jar也不用更新。

### 三、其他
这篇也主要大概介绍了怎么去生成和使用jar包，其中也看了一些java加载的相关知识，不甚理解，等整理。下面一篇不出意外会整理aidl相关，回调，动态代理，注解应用，，，

Night
