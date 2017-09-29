---
layout: post
title: "Android4.4添加新语言和字体"
categories:
- Android
tags:
- 安卓系统
---

### 一、修改编译配置文件
**目的**：让PRODUCT_LOCALES := 后面有我们需要添加的语言。

一般原生安卓代码是修改这两个文件 Android/build/target/product/languages_full.mk| languages_small.mk.

### 二、添加系统资源文件
上面添加好了但是还发现Resources.getSystem().getAssets().getLocales()没有我们添加的语言，这时候需要检查一下在Android/frameworks/base/core/res/res/目录添加values-xxx相应的资源

### 三、命名规则
#### 2.1 资源文件
其中资源文件[命名规则](https://developer.android.com/guide/topics/resources/providing-resources.html?hl=zh-cn#AlternativeResources)
官方说明如下：
![](http://7xt9nx.com2.z0.glb.clouddn.com/android_res_local.png)

 [ISO 639-1](http://www.loc.gov/standards/iso639-2/php/code_list.php) + r +  [ISO 3166-1-alpha-2](https://zh.wikipedia.org/wiki/ISO_3166-1)

#### 2.2 PRODUCT_LOCALES
PRODUCT_LOCALES 的命名规则去掉r，也就是 [ISO 639-1](http://www.loc.gov/standards/iso639-2/php/code_list.php) + [ISO 3166-1-alpha-2](https://zh.wikipedia.org/wiki/ISO_3166-1)

后面一个的链接失效了，可以用维基上的[这个](https://zh.wikipedia.org/wiki/ISO_3166-1)。比如加印地语，就是hi-IN

### 四、添加字库
```
Step1:  
Copy custom font .ttf into frameworks/base/data/fonts

Step2:  
Modify framworks/base/data/fonts/Android.mk ，Add your custom font into list of ‘font_src_files’  

Step3:  
Modify frameworks/base/data/fonts/fonts.mk ，Add your custom font into list of PRODUCT_PACKAGES  
```

完成上面三步之后在文件系统下的/system/fonts/就有添加的字库了，也可以通过mm命令然后在out/下检查。

### 五、字库资源载入
Android4.0+ 和之前版本方式不一样

```
Modify frameworks/base/data/fonts/fallback_fonts.xml，Add your custom font like below :

    <family>
        <fileset>
            <file>Himalaya.ttf</file>
        </fileset>
    </family>

```

### 六、可能遇到的问题

#### 1、在setting里面没有语言选项。

* languages_full.mk没有添加到语言
* framework res和settings res没有需添加语言的资源
* 语言的名称不是规定的。如缅甸语的名称是my_MM，可以在[维基](https://zh.wikipedia.org/wiki/ISO_639-1)查找。
* ICU文件没有相应语言


#### 2、在settings的语言栏有选项，但是空的。

* 检查安卓文件系统下system/etc/fallback_fonts.xml下面是否有添加新的字库
* 检查安卓文件系统下system/fonts/是否有新的字库文件


### 七、参考文章：

* [Android系统应用开发（四）系统语言以及添加字体库](http://blog.csdn.net/sky_pjf/article/details/52515526)
* [android添加新语言之缅甸语](http://blog.csdn.net/yicao821/article/details/17559851)
* [Unicode字符检查](http://unicode-table.com/cn/blocks/tibetan/)
