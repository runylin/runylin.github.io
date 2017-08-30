---
layout: post
title: Android源码的编译与烧写
categories:
- Android
tags:
- 安卓系统 招式
---

### 一、介绍
AOSP全称是Android Open Source Project，是Google的一个开源项目。其中代码包含了大部分Android源码，通过编译这些代码，可以直接运行在Nexus和Pixel系列手机上以及PC的模拟器上。

因为手上有个Nexus6P，所以这里就直接运行在真机上了。目前对代码本身也不是很了解，这篇文章只是介绍搭载环境，做好实践的平台。源码分析推荐老罗的书和博客。 

下图是编译完手机显示的信息，可以看到编译用户名和时间。

![](http://7xt9nx.com2.z0.glb.clouddn.com/aosp_angler_build_number.png)

### 二、下载源码
![](http://7xt9nx.com2.z0.glb.clouddn.com/android_build_web_page.png)
[Android官网](https://source.android.com/source/downloading.html)已经有介绍，
但是因为google在大天朝被墙了，但是这种非黄赌毒不反社会的还是有代理可以下载的。比如在其他博客看到有清华大学的源，使用介绍在[这里](https://mirrors.tuna.tsinghua.edu.cn/help/AOSP/).

提取下个人需要用的步骤如下：

- 1.Installing Repo
```
$ mkdir ~/bin
$ PATH=~/bin:$PATH
$ curl https://mirrors.tuna.tsinghua.edu.cn/git/git-repo > ~/bin/repo
$ chmod a+x ~/bin/repo
//谷歌的地址　https://storage.googleapis.com/git-repo-downloads/repo
```

- 2.Initializing a Repo client
```
$ mkdir WORKING_DIRECTORY
$ cd WORKING_DIRECTORY
$ git config --global user.name "Your Name"
$ git config --global user.email "you@example.com"
$ repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest
//谷歌的地址 https://android.googlesource.com/platform/manifest
```
最后一个也可以选择加（-b 分支名）下某个特定的 Android 版本，因为手上有个nexus6p，目前最新的可以在[这里](https://source.android.com/source/build-numbers.html#source-code-tags-and-builds)查看支持的情况。
所以我这里就是
```
repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest -b android-7.1.1_r24
```

- 3.Downloading the Android Source Tree
```
$ repo sync
```
**注意：**　*清华大学的源的使用说明限制最多４线程*

- 4.DOEN
```
$ cd WORKING_DIRECTORY
$ ls 就能看到代码了
$ 然后 repo start android-7.1.1_r24 --all 
```

### 三、 编译环境
[Android官网](https://source.android.com/source/initializing.html)已经有介绍。本来这个应该在下载代码前介绍的，但是有代码才能实践编译嘛,也可以一边编译一边解报错的问题。所以放这里了。

* 电脑硬件要求
```
1.A 64-bit environment is required for Gingerbread (2.3.x) and newer versions, including the master branch. You can compile older versions on 32-bit systems.
2.At least 100GB of free disk space for a checkout, 150GB for a single build, and 200GB or more for multiple builds. If you employ ccache, you will need even more space.
3.If you are running Linux in a virtual machine, you need at least 16GB of RAM/swap.
```

* 操作系统
```
1.GNU/Linux
Android 6.0 (Marshmallow) - AOSP master: Ubuntu 14.04 (Trusty)
Android 2.3.x (Gingerbread) - Android 5.x (Lollipop): Ubuntu 12.04 (Precise)
Android 1.5 (Cupcake) - Android 2.2.x (Froyo): Ubuntu 10.04 (Lucid)
2.Mac OS (Intel/x86)
Android 6.0 (Marshmallow) - AOSP master: Mac OS v10.10 (Yosemite) or later with Xcode 4.5.2 and Command Line Tools
Android 5.x (Lollipop): Mac OS v10.8 (Mountain Lion) with Xcode 4.5.2 and Command Line Tools
Android 4.1.x-4.3.x (Jelly Bean) - Android 4.4.x (KitKat): Mac OS v10.6 (Snow Leopard) or Mac OS X v10.7 (Lion) and Xcode 4.2 (Apple's Developer Tools)
Android 1.5 (Cupcake) - Android 4.0.x (Ice Cream Sandwich): Mac OS v10.5 (Leopard) or Mac OS X v10.6 (Snow Leopard) and the Mac OS X v10.5 SDK
```

* JDK
```
1.The master branch of Android in AOSP: Ubuntu - OpenJDK 8, Mac OS - jdk 8u45 or newer
2.Android 5.x (Lollipop) - Android 6.0 (Marshmallow): Ubuntu - OpenJDK 7, Mac OS - jdk-7u71-macosx-x64.dmg
3.Android 2.3.x (Gingerbread) - Android 4.4.x (KitKat): Ubuntu - Java JDK 6, Mac OS - Java JDK 6
4.Android 1.5 (Cupcake) - Android 2.2.x (Froyo): Ubuntu - Java JDK 5
```

* Key packages
```
1.Python 2.6 -- 2.7 from python.org
2.GNU Make 3.81 -- 3.82 from gnu.org; Android 3.2.x (Honeycomb) and earlier will need to revert from make 3.82 to avoid build errors
3.Git 1.7 or newer from git-scm.com
```

因为这里不同人会有不同电脑不同系统，有条件的详细看[这里](https://source.android.com/source/requirements.html#jdk)。这里说明下我的电脑(Thinkpad W540)和系统（Ubuntu 15.04）,代码是android-7.1.1_r16分支。主要安装下JDK8就可以了。命令如下:
```
$ sudo apt-add-repository ppa:openjdk-r/ppa 
$ sudo apt-get update 
$ sudo apt-get install openjdk-8-jdk
```

如果有安装多个java版本的，可以通过下面查看和修改要使用的java版本 

```
$ java -version
$ sudo update-alternatives –config java
$ sudo update-alternatives –config javac
$ whereis java
$ sudo vi ~/.bashrc
//change JAVA_HOME
```
设置环境变量
```
#set JAVA_HOME and PATH in ~/.bashrc
#export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-amd64
export JRE_HOME=$JAVA_HOME/jre                                                                                                                                                                 
export CLASSPATH=.:$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH   
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH
```

### 四、开始编译
最好的资料还是[Android官网](https://source.android.com/source/downloading.html)。

- 1.Clean up
```
$ make clobber
```
- 2.Set up environment
```
$ source build/envsetup.sh
```
- 3.Choose a target
```
lunch aosp_angler-userdebug
```
选择一个target，因为有nexus6p真机，所以选的是17.aosp-angler-userdebug
，如果没有可以选择aosp-arm-eng.然后用模拟器。**注意：userdebug和eng才有root权限。**

- 4.Build the code
```
$ make -j8
```

遇到的问题:
```
[ 45% 16125/35623] Building with Jack: out/target/common/obj/JAVA_LIBRARIES/framework_intermediates/with-local/classes.dex
FAILED: /bin/bash out/target/common/obj/JAVA_LIBRARIES/framework_intermediates/with-local/classes.dex.rsp
Out of memory error (version 1.2-rc4 'Carnac' (298900 f95d7bdecfceb327f9d201a1348397ed8a843843 by android-jack-team@google.com)).
GC overhead limit exceeded.
Try increasing heap size with java option '-Xmx<size>'.
Warning: This may have produced partial or corrupted output.
[ 45% 16125/35623] Building with Jack: out/target/common/obj/JAVA_LIBRARIES/android-support-compat-honeycomb-mr1_intermediates/classes.jack
ninja: build stopped: subcommand failed.
build/core/ninja.mk:148: recipe for target 'ninja_wrapper' failed
make: *** [ninja_wrapper] Error 1
```

[解决办法](http://stackoverflow.com/questions/35579646/android-source-code-compile-error-try-increasing-heap-size-with-java-option):

```
export JACK_SERVER_VM_ARGUMENTS="-Dfile.encoding=UTF-8 -XX:+TieredCompilation -Xmx4g"
./prebuilts/sdk/tools/jack-admin kill-server
./prebuilts/sdk/tools/jack-admin start-server
```

### 五、烧写到手机上（nexus6p）
- 1.其实根据[这里](https://source.android.com/source/building.html#choose-a-target)的最开始和[这里](https://source.android.com/source/requirements.html#binaries)的最后,编译出来的并不是完整的可以用的包。其中有些二进制文件还需要从谷歌官网下载。

```
Device binaries
Download previews, factory images, drivers, over-the-air (OTA) updates, and other blobs below. See Obtaining proprietary binaries for additional details.
Preview binaries (blobs) - for AOSP master branch development
Factory images - for the supported devices running tagged AOSP release branches
Binary hardware support files - for devices running tagged AOSP release branches
OTA images - for manually updating Nexus devices over the air
```

其中[Binary hardware support files](https://developers.google.com/android/drivers)就是Vendor image。版本不对会在开机的时候弹出一个警告。这里建议在烧写自己编译的文件前提下，先刷一个相同版本的原生底包进去。地址在[这里](https://developers.google.com/android/images).

- 2.回归正传，烧写编译好的软件其实只需要几个命令。

```
$ adb reboot bootloader　#进入fastboot
$ cd /media/runylin/HDD/Andoid/AOSP/out/target/product/angler #进入类似的编译完的目录
$ fastboot flashall -w　＃等于刷进boot.img cache.img recovery.img system.img userdata.img vendor.img
```


- 3.其实这样就和我们下一个image-angler-mmb29p.zip包，然后用./flash-all.sh烧写“ROM”差不多的。
./flash-all.sh文件内容：

```
fastboot flash bootloader bootloader-angler-angler-02.45.img
fastboot reboot-bootloader
sleep 5
fastboot flash radio radio-angler-angler-02.50.img
fastboot reboot-bootloader
sleep 5
fastboot -w update image-angler-mmb29p.zip
```
image-angler-mmb29p.zip解压完文件：
```
android-info.txt boot.img cache.img recovery.img system.img userdata.img vendor.img
```

### 五、一些调试准备工作
因为userdebug版本是带root权限的，adb shell 进去再敲su发现就在root用户了，可以用whoami可以验证。

遇到问题：敲入mount -o remount,rw /system发现以下提示：
```
mount: '/dev/block/dm-0'->'/system': Device or resource busy
```

解决办法：

```
方法一： 
1.adb disable-verity and than adb reboot
方法二：
2.修改device\huawei\angler\fstab.aosp_angler(fstab.angler)文件中的
/dev/block/platform/soc.0/f9824900.sdhci/by-name/system /system ext4 ro,barrier=1,inode_readahead_blks=8 wait,verify=/dev/block/platform/soc.0/f9824900.sdhci/by-name/metadata
删除 verify 标识；
```

方法一可能遇到的问题是adb版本太低。(最新的adb 工具包才支持adb disable-verity命令，如果是Linux开发环境，则可使用工程编译结果目录out/host/linux-x86/bin下的adb执行文件，windows下1.0.32版本可以）; 

方法二需要重新编译烧写软件。

到这里就可以随便玩了。


### 参考文章
- http://www.jianshu.com/p/94187d33f730
- http://blog.csdn.net/wh_19910525/article/details/50263851
