---
layout: post
title: 系统签名的APK在AS的调试
categories:
- Android
tags:
- 安卓应用 
- 招式
---

### 手动签名
之前调试需要系统签名的apk都是先生成apk，再通过signapk.jar去生成。
```
java -jar signapk.jar platform.x509.pem platform.pk8 old.apk new.apk
```
这样不能使用AS的调试功能，有时候效率较低。下面就是一种AS导入系统签名的方法。


### 制作签名文件 *.jks
利用keytool-importkeypair

下载地址 https://github.com/getfatday/keytool-importkeypair

使用，最终生成demo.jks
```
# 转换系统签名命令
./keytool-importkeypair -k demo.jks -p 123456 -pk8 platform.pk8 -cert platform.x509.pem -alias demo

# demo.jks : 签名文件
# 123456 : 签名文件密码
# platform.pk8、platform.x509.pem : 系统签名文件 //需要从平台源码中拷贝过来
# demo : 签名文件别名
```

### AS配置
在Build -> Generate Signed APK... -> Choose exitsting... 

导入上面生成的demo.jks文件，填入刚才的签名文件密码等信息

### builde.gradle配置
```
signingConfigs {
    release {
        storeFile file("../signature/demo.jks")
        storePassword '123456'
        keyAlias 'demo'
        keyPassword '123456'
    }

    debug {
        storeFile file("../signature/demo.jks")
        storePassword '123456'
        keyAlias 'demo'
        keyPassword '123456'
    }
}

buildTypes {
    release {
        minifyEnabled false
        proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.txt'
        signingConfig signingConfigs.release
    }
    debug {
        minifyEnabled false
        proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.txt'
        signingConfig signingConfigs.debug
    }
}
```

### 可以使用了

