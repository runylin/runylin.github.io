---
layout: post
title: "Android开发环境AS"
categories:
- Tools 
tags:
- 招式
---
![](http://upload-images.jianshu.io/upload_images/1236985-d168dc3aea0be8fc.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 一、墙内安装SDK
给这个网站 [Androiddevtools](http://www.androiddevtools.cn/index.html) 强烈点个赞 ;

#### 方法1: 下载解压完放到*__your_sdk/platforms__* 文件夹即可 ;
#### 方法2: 镜像, [具体方法](http://android-mirror.bugly.qq.com:8080/include/usage.html) ;

即使按照上面这样设置了，在导入新的工程文件可能会出现一直卡在Building XXX Gradle project info的情况。这是有两种解决方法：

#### 方法1: 网上下载包（比如gradle-2.4-all.zip）放到

```
C:\Users\user\.gradle\wrapper\dists\gradle-2.4-all\6r4uqcc6ovnq6ac6s0txzcpc0a 
```

#### 方法2: 电脑已翻墙的话，在项目中的gradle.propertities文件中加入下面两行:
```
    systemProp.https.proxyHost=127.0.0.1
    systemProp.https.proxyPort=8123 
```


### 二、解决工具栏字符显示不全
![](http://upload-images.jianshu.io/upload_images/1236985-b2ea3ad4629aecfa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
上图方法可以解决，但最终原因发现是装了MacType字体渲染导致。

### 三、代码框配色
1. 第三方主题下载 [链接](http://www.ideacolorthemes.org/home/) ; 
2. 选择菜单栏*__"File-Import Settings"__* 导入下载完解压出来的jar包，重启AS;
3. 再在*__"File--settings--Editor--Colors&Fonts"__*就可以看到导入的新主题了;

我用的是这个 [SublimeMonokai](https://github.com/y3sh/Intellij-Colors-Sublime-Monokai) ，文字配置用的 Courier New 16 1.0。

### 四、快捷键
用一款IDE顺不顺手首先就是熟不熟悉他的快捷键了，熟悉后明显能提高生产力。
#### 查看代码时，和工程结构或者跳转相关

| 快捷键                        | 说明         |
| :--------                        | :--------      | 
| CTRL+SHIFT+N        | 快速查找文件 | 
| CTRL + N                   |  快速查找类   | 
| CTRL + H                   |  浏览当前类的继承关系 | 
|CTRL + F12	             |  列出当前类的成员|
|Alt+向上箭头Alt+向下箭头                      | 移动到上一个/下一个方法|
|CTRL + SHIFT + F3 或 SHIFT + F3	|在当前文件中高亮显示所选文本在这些高亮文本间移动|
|Alt+左箭头 / Alt+右箭头 |在打开的多个文件中，移动到左/右一个文件|
|CTRL + B/点击	       |跳到定义的位置|
|ALT + F7	               |在工程中找出引用了所选内容的所有位置|
|Ctrl + Alt + B 	               |导航到一个抽象方法的实现代码|

#### 并照之前习惯加了下面的快捷键 :

| 快捷键                        | 说明           |
| :--------                        | :--------        | 
|Alt+逗号 / Alt+句号	     |移动到上一个/下一个光标位置|

#### 编写代码时，和代码提示/补全相关

| 快捷键                        | 说明           |
| :--------                        | :--------        | 
|CTRL + Q	             |  快速查看文档(简单版本)  |
|SHIFT + F1	             |  用浏览器打开 SDK 文档  |
|TAB	                     |  选择列出的高亮显示的补全代码  |
|CTRL + 空格	             |  补全代码   |
|CTRL + SHIFT + 空格 |  智能分析表达式，列出可能要写的方法名、变量名  |
|ALT + INSERT	     |  快速生成 getter、setter 代码  |
|CTRL + O	             |  覆写类成员  |
|CTRL + I	                     |  实现接口方法  |
|SHIFT + F6	             |  快速重命名  |
|CTRL + ALT + T	     |  生成捕获异常的代码  |
|CTRL (+ SHIFT) + /     |  添加注释  |
|Alt + J	                     | 多行操作  |

### 五、遇到的问题备注
#### 修改包名
在Android Studio默认情况下， 修改包名只能修改最后一级。要修改可以在Project工具栏勾掉Compact Empty Middle。然后Refactor->Rename 即可。

### others
[android官网教程](https://developer.android.com/training/index.html)
<br>

