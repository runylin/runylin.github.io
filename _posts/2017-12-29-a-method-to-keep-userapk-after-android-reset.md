---
layout: post
title: "一种预装Android用户APK且复位后还存在的方法"
categories:
- Android
tags:
- 安卓系统
---

#### 前言
现在购买安卓手机的时候，会发现出厂的时候大部分是会预装一些第三方APK，比如MIUI出厂就带了一些可以卸载的APK（虽然有些只是个图标，点击后才去下载，这不在本篇讨论范围内，虽然这更合理）。

对于厂商来说一方面方便用户使用，一方面可以在运营上获取利益。但是对于用户来说，一般会有卸载的要求。一般来说可以卸载的APP是作为用户APK安装的，那么复位完这些APP也会随着清除data分区而不见了。

如果不考虑这些APK得从网络上去下载的情况下，又要求**在recovery复位清data分区后得保留**且**用户可以卸载**，一是考虑修改PMS，一是考虑在复位/升级后的第一次开机通过代码去system下的某个路径安装到data分区。下面就介绍下后者的一种做法。


#### 一种实现方法
大概思路是在编译Android系统的时候把apk预装到system/media下，然后在init.rc判断persist.sys.apk_boot_install属性去执行installapk.sh脚本，安装成功后再把这属性设为0

##### Step1 :预装apk到system/media的Android.mk文件
```
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_32_BIT_ONLY :=true
LOCAL_MODULE := lxt
LOCAL_SRC_FILES := $(LOCAL_MODULE).apk
LOCAL_MODULE_CLASS := APPS
LOCAL_MODULE_SUFFIX := $(COMMON_ANDROID_PACKAGE_SUFFIX)
LOCAL_CERTIFICATE := platform
LOCAL_MULTILIB := 32
LOCAL_MODULE_PATH := $(TARGET_OUT)/media
include $(BUILD_PREBUILT)
```

##### Step2 :添加脚本文件installapk.sh
```
if [ $(getprop persist.sys.apk_boot_install) -ne 1 ] ; then
    exit
fi

function installApk(){
	for filename in $1
	do
		if [ -f $filename ]; then
			echo $filename  > /dev/console
			tmpresult=`pm install -r $filename`
			if [ "$tmpresult"x != "Success"x ];then
				result="Failed"
			fi
		fi
	done
}

installApk /system/media/*.apk
installApk /system/media/*/*.apk

if [ "$result"x == "Success"x ]; then
    setprop persist.sys.preinstall.apk 0
    echo "install finish" > /dev/console
fi
```

##### Step3 :在相关mk文件添加上面脚本到系统中
```
PRODUCT_COPY_FILES += \
    packages/lxt/preinstall/bin/installapk.sh:/system/bin/installapk.sh
```

##### Step4 :添加init.rc开机后执行的脚本
```
service bootinstallapk /system/bin/installapk.sh
    class install_apk
    user root
    group root
    disabled
    oneshot

on property:persist.sys.apk_boot_install=1
    start bootinstallapk
```

##### Step5 :在相关mk文件添加系统属性
```
PRODUCT_PROPERTY_OVERRIDES += persist.sys.apk_boot_install=1
```

##### Step6 :如果开了selinux需要根据具体情况加上相关te文件
具体可以参考上篇文章



