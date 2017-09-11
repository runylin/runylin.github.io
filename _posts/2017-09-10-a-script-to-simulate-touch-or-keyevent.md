---
layout: post
title: "一个通过模拟点击或者按键事件来复现概率性问题的脚本"
categories:
- Tools
tags:
- 调试
---

测试同学报了一个打开模拟器，然后点击左上角的X，概率性是回到Setting而不是Laucher的问题。概率性大概是1/30.

改了相关问题，复现很麻烦。所以写个脚本跑几百次，没出现就pass了。

先打开开发者选项，打开显示触摸操作，然后通过鼠标点击顶部有dx，dy。这个就是对用的横纵坐标。然后通过串口/adb，输入input tap 360（X坐标） 532（Y坐标），就可以模拟点击事件了。加上sleep，按照流程一步步往下，等到需要验证的问题，根据需要截图或者抓设备的某些信息保持在/data/文件夹即可。

设置页面和脚本文件如下：

![](http://7xt9nx.com2.z0.glb.clouddn.com/touch-position.png)


```
#!/system/bin/sh

count=1

while [ 1 ]
do

echo "=========================COUNT: " $count "============================"

# click browser
input tap 360 532
sleep 5

# click x
input tap 223 38
sleep 5

# screencap
filename="`date +%y%m%d_%H%M%S`.png" 
/system/bin/screencap -p /data/3X05X0-916/${filename} 
sync

sleep 1
let count=count+1
done
```