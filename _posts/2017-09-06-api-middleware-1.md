---
layout: post
title: "Android 生成和使用jar包"
categories:
- Middleware
tags:
- 中间件
---

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
- 如果是在编译整个android的源码，这个LOCAL_MODULE需要添加到PRODUCT_PACKAGES和PRODUCT_BOOT_JARS 变量里
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

