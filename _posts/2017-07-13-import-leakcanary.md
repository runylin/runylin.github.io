---
layout: post
title: Androi应用源码编译时导入leakcanary
categories:
- Android
tags:
- 安卓应用 招式
---

### 零、前言
[**LeakCanary**](https://github.com/square/leakcanary) -- A memory leak detection library for Android and Java.

在AS用gradle编译的时候，导入只需要在build.gradle添加
```
 dependencies {
   debugCompile 'com.squareup.leakcanary:leakcanary-android:1.5.1'
   releaseCompile 'com.squareup.leakcanary:leakcanary-android-no-op:1.5.1'
   testCompile 'com.squareup.leakcanary:leakcanary-android-no-op:1.5.1'
 }
```
当源码编译的时候需要手动下载包，并引用。如果是Gradle编译的话可以直接看三怎么使用。

### 一、Android.mk文件导入
```
diff --git a/packages/apps/Browser/Android.mk b/packages/apps/Browser/Android.mk
index 70c1b7a..88574c5 100755
--- a/packages/apps/Browser/Android.mk
+++ b/packages/apps/Browser/Android.mk
@@ -9,6 +9,12 @@ LOCAL_STATIC_JAVA_LIBRARIES := \
         android-support-v13 \
         android-support-v4 \
 
+LOCAL_STATIC_JAVA_LIBRARIES +=  haha leakcanary-watcher-jar leakcanary-analyzer-jar
+LOCAL_STATIC_JAVA_AAR_LIBRARIES := leakcanary-android-aar
+LOCAL_AAPT_FLAGS := \
+        --auto-add-overlay \
+        --extra-packages com.squareup.leakcanary
+
 LOCAL_SRC_FILES := \
         $(call all-java-files-under, src) \
         src/com/android/browser/EventLogTags.logtags
@@ -24,5 +30,13 @@ LOCAL_REQUIRED_MODULES := SoundRecorder
 
 include $(BUILD_PACKAGE)
 
+include $(CLEAR_VARS)
+LOCAL_PREBUILT_STATIC_JAVA_LIBRARIES := \
+        leakcanary-watcher-jar:libs/leakcanary-watcher-1.5.1.jar \
+        haha:libs/haha-2.0.3.jar \
+        leakcanary-analyzer-jar:libs/leakcanary-analyzer-1.5.1.jar \
+        leakcanary-android-aar:libs/leakcanary-android-1.5.1.aar
+include $(BUILD_MULTI_PREBUILT)
+
 # additionally, build tests in sub-folders in a separate .apk
 include $(call all-makefiles-under,$(LOCAL_PATH))
```
jar,aar包从as下完后拷贝出来放到libs目录。

### 二、遇到的问题
1.android:minSdkVersion没定义
* 报错信息：
```
make: Entering directory `/disk2/linrunyu/code/hisi51_tmp/Android'
# Make sure the extracted classes.jar has a new timestamp.
Merge android manifest files: out/target/common/obj/APPS/Browser_intermediates/AndroidManifest.xml <-- packages/apps/Browser/AndroidManifest.xml   out/target/common/obj/JAVA_LIBRARIES/leakcanary-android-aar_intermediates/aar/AndroidManifest.xml
Error: [AndroidManifest.xml:1, AndroidManifest.xml:3] Main manifest has <uses-sdk android:minSdkVersion='1'> but library uses minSdkVersion='14'
Note: main manifest lacks a <uses-sdk android:minSdkVersion> declaration, which defaults to value 1.
Warning: [AndroidManifest.xml:1, AndroidManifest.xml:3] Main manifest has <uses-sdk android:targetSdkVersion='1'> but library uses targetSdkVersion='14'
Note: main manifest lacks a <uses-sdk android:targetSdkVersion> declaration, which defaults to value minSdkVersion or 1.
make: *** [out/target/common/obj/APPS/Browser_intermediates/AndroidManifest.xml] Error 1
make: *** Deleting file `out/target/common/obj/APPS/Browser_intermediates/AndroidManifest.xml'
make: Leaving directory `/disk2/linrunyu/code/hisi51_tmp/Android'

```
* 解决办法
```
linrunyu@RD32(trunk5_1):~/code/hisi51_tmp/Android/packages/apps/Browser$git diff AndroidManifest.xml
diff --git a/packages/apps/Browser/AndroidManifest.xml b/packages/apps/Browser/AndroidManifest.xml
index 4340300..1ce2b98 100755
--- a/packages/apps/Browser/AndroidManifest.xml
+++ b/packages/apps/Browser/AndroidManifest.xml
@@ -20,6 +20,7 @@
 
     <original-package android:name="com.android.browser" />
 
+    <uses-sdk android:minSdkVersion="14" />
     <permission android:name="com.android.browser.permission.PRELOAD"
         android:label="@string/permission_preload_label"
         android:protectionLevel="signatureOrSystem" />
```
2.导入leakcanary-android-1.5.1.aar的AndroidManifest.xml有问题
* 报错信息
```
Warning: AndroidManifest.xml already defines minSdkVersion (in http://schemas.android.com/apk/res/android); using existing value in manifest.
out/target/common/obj/APPS/Browser_intermediates/AndroidManifest.xml:219: Tag <activity> attribute taskAffinity has invalid character '$'.
out/target/common/obj/APPS/Browser_intermediates/AndroidManifest.xml:226: Tag <activity> attribute taskAffinity has invalid character '$'.
make: *** [out/target/common/obj/APPS/Browser_intermediates/src/R.stamp] Error 1
make: Leaving directory `/disk2/linrunyu/code/hisi51_tmp/Android'
```
* [分析](http://www.cnblogs.com/shortboy/p/5241643.html)
```
因为是从AS,gradle下下来的leakcanary-android-1.5.1.aar。google发现${applicationId}写法是读build.gradle的applicationId替换进去。而Android.mk并不支持这种写法。
```
* 解决办法
```
${applicationId}直接替换成你的apk名字
leakcanary-android-1.5.1.aar重名名leakcanary-android-1.5.1.zip
改完再重命名回来
```
3.混淆
* 报错信息
```
Reading library jar [/disk2/linrunyu/code/hisi51_tmp/Android/out/target/common/obj/JAVA_LIBRARIES/framework_intermediates/classes.jar]
Warning: com.squareup.haha.guava.collect.AbstractMapBasedMultimap: can't find referenced class com.squareup.haha.guava.collect.AbstractMapBasedMultimap$com.squareup.haha.guava.collect.AbstractMapBasedMultimap$WrappedCollection
Warning: com.squareup.haha.guava.collect.AbstractMapBasedMultimap$1: can't find referenced class com.squareup.haha.guava.collect.AbstractMapBasedMultimap$com.squareup.haha.guava.collect.AbstractMapBasedMultimap$Itr
Warning: com.squareup.haha.guava.collect.AbstractMapBasedMultimap$2: can't find referenced class com.squareup.haha.guava.collect.AbstractMapBasedMultimap$com.squareup.haha.guava.collect.AbstractMapBasedMultimap$Itr
Warning: com.squareup.haha.guava.collect.AbstractMapBasedMultimap$RandomAccessWrappedList: can't find referenced class com.squareup.haha.guava.collect.AbstractMapBasedMultimap$com.squareup.haha.guava.collect.AbstractMapBasedMultimap$WrappedCollection
Warning: com.squareup.haha.guava.collect.AbstractMapBasedMultimap$RandomAccessWrappedList: can't find referenced class com.squareup.haha.guava.collect.AbstractMapBasedMultimap$com.squareup.haha.guava.collect.AbstractMapBasedMultimap$WrappedList
Warning: com.squareup.haha.guava.collect.AbstractMapBasedMultimap$SortedAsMap: can't find referenced class com.squareup.haha.guava.collect.AbstractMapBasedMultimap$com.squareup.haha.guava.collect.AbstractMapBasedMultimap$AsMap
Warning: com.squareup.haha.guava.collect.AbstractMapBasedMultimap$SortedKeySet: can't find referenced class com.squareup.haha.guava.collect.AbstractMapBasedMultimap$com.squareup.haha.guava.collect.AbstractMapBasedMultimap$KeySet
Warning: com.squareup.haha.guava.collect.AbstractMapBasedMultimap$WrappedCollection: can't find referenced class com.squareup.haha.guava.collect.AbstractMapBasedMultimap$com.squareup.haha.guava.collect.AbstractMapBasedMultimap$WrappedCollection
Warning: com.squareup.haha.guava.collect.AbstractMapBasedMultimap$WrappedCollection: can't find referenced class com.squareup.haha.guava.collect.AbstractMapBasedMultimap$com.squareup.haha.guava.collect.AbstractMapBasedMultimap$WrappedCollection
Warning: com.squareup.haha.guava.collect.AbstractMapBasedMultimap$WrappedCollection: can't find referenced class com.squareup.haha.guava.collect.AbstractMapBasedMultimap$com.squareup.haha.guava.collect.AbstractMapBasedMultimap$WrappedCollection
```
* 解决办法：
```
linrunyu@RD32(trunk5_1):~/code/hisi51_tmp/Android/packages/apps/Browser$git diff proguard.flags
diff --git a/packages/apps/Browser/proguard.flags b/packages/apps/Browser/proguard.flags
index 888c238..70d04e0 100755
--- a/packages/apps/Browser/proguard.flags
+++ b/packages/apps/Browser/proguard.flags
@@ -1,2 +1,5 @@
 # Most of the classes in this package are fragments only referenced from XML
 -keep class com.android.browser.preferences.*
+# com.squareup
+-dontwarn com.squareup.**
+-keep class com.squareup.** {*;}
```
### 三、使用
修改继承Application的类以及需要检测内存泄露的Activity的onDestroy函数。
```
//https://github.com/square/leakcanary
//http://www.jianshu.com/p/a8900eb3de12
linrunyu@RD32(trunk5_1):~/code/hisi51_tmp/Android/packages/apps/Browser$git diff src/com/android/browser/Browser.java src/com/android/browser/BrowserActivity.java
diff --git a/packages/apps/Browser/src/com/android/browser/Browser.java b/packages/apps/Browser/src/com/android/browser/Browser.java
index add8bdd..69a9980 100755
--- a/packages/apps/Browser/src/com/android/browser/Browser.java
+++ b/packages/apps/Browser/src/com/android/browser/Browser.java
@@ -19,21 +19,30 @@ package com.android.browser;
 import android.app.Application;
 import android.util.Log;
 import android.webkit.CookieSyncManager;
+import com.squareup.leakcanary.LeakCanary;
+import com.squareup.leakcanary.RefWatcher;
 
 public class Browser extends Application { 
 
     private final static String LOGTAG = "browser";
-    
     // Set to true to enable verbose logging.
     final static boolean LOGV_ENABLED = false;
 
     // Set to true to enable extra debug logging.
     final static boolean LOGD_ENABLED = true;
 
+    private static Browser mContext;
+    private RefWatcher refWatcher;
+    public static RefWatcher getRefWatcher(){
+        Browser application = (Browser) mContext.getApplicationContext();
+        return application.refWatcher;
+    }
+
     @Override
     public void onCreate() {
         super.onCreate();
-
+        mContext = this;
+        refWatcher = LeakCanary.install(this);
         if (LOGV_ENABLED)
             Log.v(LOGTAG, "Browser.onCreate: this=" + this);
 
diff --git a/packages/apps/Browser/src/com/android/browser/BrowserActivity.java b/packages/apps/Browser/src/com/android/browser/BrowserActivity.java
index 612370d..abd108a 100755
--- a/packages/apps/Browser/src/com/android/browser/BrowserActivity.java
+++ b/packages/apps/Browser/src/com/android/browser/BrowserActivity.java
@@ -41,6 +41,9 @@ import android.content.DialogInterface.OnClickListener;
 import com.android.browser.stub.NullController;
 import com.google.common.annotations.VisibleForTesting;
 
+import com.squareup.leakcanary.LeakCanary;
+import com.squareup.leakcanary.RefWatcher;
+
 import android.app.ActivityManager;
 import android.os.Debug;
 import android.os.Process;
@@ -208,6 +211,8 @@ public static final int THRESHOLD = 100000; //browser pss threshold 100M
         mController.onDestroy();
         handler.removeCallbacks(pssCheckRunnable);
         mController = NullController.INSTANCE;
+        RefWatcher refWatcher = Browser.getRefWatcher();
+        refWatcher.watch(this);
     }
 
     @Override
linrunyu@RD32(trunk5_1):~/code/hisi51_tmp/Android/packages/apps/Browser$
```
### 四、查看报错问题
当退出这个activity的时候，如果出现一个黄色的小图标，说明有内存泄露。并会在桌面生成一个Leaks的入口。点击可以查看信息。
![](http://7xt9nx.com2.z0.glb.clouddn.com/Browser-oom.png)
刚好和[这个例子](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0511/2861.html)一样，Activity销毁了还被其他类持有。
