---
layout: post
title: 利用Jekyll+GitHub Page搭建博客
categories:
- Tools
tags:
- 招式
---


### 0. 前言

  本文旨在记录自己搭建这个博客的过程，也希望能给有同个想法的人一点参考。需注意的是**需要的步骤**都会列出来，过程就不会写得太详细。一般会引用到官方的文档。

### 1. 这个博客是什么

  是利用[ Jekyll ](http://jekyllcn.com/)部署在[GitHub Page](https://pages.github.com/)的博客。
  
> Jekyll 是一个简单的博客形态的静态站点生产机器。它有一个模版目录，其中包含原始文本格式的文档，通过 Markdown （或者 Textile） 以及 Liquid 转化成一个完整的可发布的静态网站，你可以发布在任何你喜爱的服务器上。
 
> GitHub Pages are public webpages freely hosted and easily published through our site. You can create and publish them online using the Automatic Page Generator.

### 2. 可以怎么做

  利用 **Jekyll** + **GitHub Page**搭建博客一般有以下三种做法,难度递减.

- A. 完全自己定制博客
- B. 找一份框架，修改后使用
- C. 从GitHub上fork别人的博客代码，在其中添加自己的文章


### 3. 动手开始做

*对于我这种网站新手，用的是当然是C方法。←_←*

**注意这里默认你有：**

- Github帐号 ([if no get one](https://github.com/join))
- Ubuntu系统 (如果不是没关系linux系的都差不多)
- 系统安装了git,并且ssh什么的都弄好了 (为了能正常的push到远程仓库)
 
#### 3.1 测试一个最简单的GitHub Page

*注:这里是没用到Jekyll的*

1. 最好的资料当然是GitHub的官方文档了。 → [check here](https://pages.github.com/)
2. 按照这个一步一步下来，访问 **http://你的用户名.github.io**，如果看到 *Hello World*，那么离成功就不远了。
 

#### 3.2 找一个别人家的博客代码

  可以在[GitHub官方推荐的 Jekyll](https://github.com/jekyll/jekyll/wiki/Sites)这里找，也欢迎来fork [我的Github Page](https://github.com/runylin/runylin.github.io)。

  *fork* 完，*download* 下来，*copy* 到 *刚才hello world的本地仓库*。

  接下来再 *push* 到远程仓库过10min就能发现你的和你fork的博客一样了。

大概用到的命令:
```
	git rm index.html
	git add *
	git commit -m "use other's jekyll blog"
	git push origin master
```

#### 3.3 修改别人家的代码

1. 这里应该具体博客的具体分析，通用的就是**_config.yml**文件的*name*,*autor*等属性吧.还有一些布局文件的链接吧。
2. 还有记得删除掉 **_post/** 文件夹下别人家的博文.
3. 修改个人信息

#### 3.4 添加自己的文章

  具体看工程下的Rakefile的task怎么写的，比如这个用法如下
```
Usage: rake post title="A Title" [date="2012-02-09"]
```
  然后修改**_post/** 文件夹下新增的文件即可.

### 4. 参考链接

- A. [Jekyll中文官方文档](http://jekyllcn.com/)
- B. [GitHub Page官方文档](https://pages.github.com/)
- C. [Git简易教程](http://rogerdudler.github.io/git-guide/index.zh.html)
- D. [Git练习](https://try.github.io/levels/1/challenges/1)
- E. [MarkDown简易教程](http://runylin.github.io/2014/10/26/how-to-use-markdown.html)
- F: [GitHub Jekyll推荐模板](https://github.com/jekyll/jekyll/wiki/Sites)
- G. [知乎 Jekyll模板推荐](http://www.zhihu.com/question/20223939)
- H. [其他参考资料](http://beiyuu.com/github-pages/)
- Z. [我的博客](https://github.com/runylin/runylin.github.io)

### 5. 写在最后

*talk is cheap,see the code*

用力点 [这里](https://github.com/runylin/runylin.github.io) 开始吧。

最后的最后，搭建好博客记得要坚持写，这个才是关键哦 !


