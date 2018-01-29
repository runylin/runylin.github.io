---
layout: post
title: "Android中的Selinux机制"
categories:
- Selinux
tags:
- 安卓系统
---

### 一、Selinux是什么?
SELinux 全称 Security Enhanced Linux (安全强化 Linux)，是MAC (Mandatory Access Control，强制访问控制系统)的一个实现。
https://source.android.com/security/selinux/

Android的安全模型是基于一部分应用程序沙箱(sandbox)的概念, 每个应用程序都运行在自己的沙箱之中。在Android 4.3之前的版本，系统在应用程序安装时为每一个应用程序创建一个独立的uid，基于uid来控制访问进程来访问资源，这种安全模型是基于Linux传统 的安全模型DAC(Discretionary Access Control，翻译为自主访问控制)来实现的。从Android 4.3开始，安全增强型Linux (SElinux)用于进一步定义应用程序沙箱的界限。作为Android安全模型的一部分，Android使用SELinux的强制访问控制(MAC) 来管理所有的进程，即使是进程具有root(超级用户权限)的能力，SELinux通过创建自动话的安全策略(sepolicy)来限制特权进程来增强 Android的安全性。从Android 4.4开始Android打开了SELinux的Enforcing模式，使其工作在默认的AOSP代码库定义的安全策略(sepolicy)下。在 Enforcing模式下，违反SELinux安全策略的的行为都会被阻止，所有不合法的访问都会记录在dmesg和logcat中。因此，我们通过查看dmesg或者logcat, 可以收集有关违背SELinux策略的错误信息，来完善我们自己的软件和SELinux策略。

### 二、安全机制
#### 2.1 DAC和MAC的区别：
DAC核心思想：进程理论上所拥有的权限与执行它的用户的权限相同。比如，以root用户启动Browser，那么Browser就有root用户的权限，在Linux系统上能干任何事情。
MAC核心思想：即任何进程想在SELinux系统中干任何事情，都必须先在安全策略配置文件中赋予权限。凡是没有出现在安全策略配置文件中的权限，进程就没有该权限。
#### 2.2 DAC和MAC的联系：
Linux系统先做DAC检查，如果没有通过DAC权限检查，则操作直接失败。通过DAC检查之后，再做MAC权限检查。

### 三、安全策略
#### 3.1 Selinux的三种状态
- permissive：即为有效，但是即使你违反了策略的话它让你继续操作，但是把你的违反的内容记录下来。在我们开发策略的时候非常的有用。相当于Debug模式。
- Enforcing：违反了策略，你就无法继续操作下去。
- Disabled：关掉不使用

#### 3.2 安全上下文
SEAndroid是一种基于安全策略的MAC 安全机制。这种安全策略又是建立在对象的安全上下文的基础上的。这里所说的对象分为两种类型，一种称 主体（Subject），一种称为客体（Object）。主体通常就是指进程，而客体就是指进程所要访问的资源，例如文件、系统属性等。安全上下文实际上就是一个附加在对象上的标签（label）。这个标签实际上就是一个字符串，它由四部分内容组成，分别是SELinux用户、SELinux 角色、类型、安全级别，每一个部分都通过一个冒号来分隔，格式为“user:role:type:rank”。

其中查看一个进程的LABLE可以通过ps -Z (pid)查看第一个字段，比如init进程
```
Hi3751V553:/ # ps -Z 1
LABEL                          USER      PID   PPID  VSIZE  RSS   WCHAN            PC  NAME
u:r:init:s0                    root      1     0     7360   1516  SyS_epoll_ 0008dafc S /init
```


* 用户(user)和角色(role)：在SEAndroid中，只定义了一个SELinux用户u，也只定义了一个SELinux角色r。*备注：经常见到object_r，表示文件是被进程访问，它没法扮演角色，所以在SELinux中，只能被进程访问的资源都用object_r来表示它的role。*
* 安全级别：s0 - mls_systemhigh
* 类型(type)：在安全上下文中，只有类型（Type）才是最重要的，SELinux用户、SELinux角色和安全级别都几乎可以忽略不计的。正因为如此，SEAndroid安全机制又称为是基于TE（Type Enforcement）策略的安全机制。通常将用来标注**文件**的安全上下文中的类型称为**file_type**，而用来标注**进程**的安全上下文的类型称为**domain**，并且每一个用来描述文件安全上下文的类型都将file_type设置为其属性，每一个用来进程安全上下文的类型都将domain设置为其属性。以checkdata.te举个栗子：
```
type checkdata, domain; #设置checkdata属于domain域，是一个进程的type
type checkdata_exec, exec_type, file_type; #设置checkdata_exec属于可执行文件的类型.
typeattribute checkdata mlstrustedsubject; #设置shelld是一个可信任的主题
```

### 四、TE文件介绍
#### 4.1 最基本的语法
```
/*
rule_name：规则名，主要有allow,dontaudit,neverallow和auditallow
source_type：源类型，对应一个很重要的概念--------域（domain）
target_type：目标的类型，SELinux一个重要的判断对象
class：类别，主要有File,Dir,Socket,SEAndroid还有Binder等，在这些基础上又细分出设备字符类型（chr_file），链接文件（lnk_file）等
perm_set：动作集
*/
rule_name source_type target_type:class perm_set
```
依次对上面的关键字进行介绍：
#### rule_name:
- allow：允许某个进程执行某个动作
- auditallow：audit含义就是记录某项操作。默认SELinux只记录那些权限检查失败的操作。 auditallow则使得权限检查成功的操作也被记录。注意，allowaudit只是允许记录，它和赋予权限没关系。赋予权限必须且只能使用allow语句。
- dontaudit：对那些权限检查失败的操作不做记录。
- neverallow：没有被allow到的动作默认就不允许执行的。neverallow只是显式地写出某个动作不被允许，如果添加了该动作的allow，则会编译错误

#### source_type：
指定一个“域”（domain），一般用于描述进程，该域内的的进程，受该条TE语句的限制。用type关键字，把一个自定义的域与原有的域相关联
常见的样例有：type mtkeepinit domain
mtkeepinit就是这个source_type，也就是需要指定的进程名，上面意思是赋予mtkeepinit domain属性，mtkeepinit与属于domain这个集合里面。

#### target_type：
指定进程需要操作的客体（文件，文件夹等）类型，同时是用type与一些已有的类型，属性相关联
比如：type mtkeepinit_exec, exec_type, file_type;
说明target mtkeepinit_exec的属性为exec_type和file_type.
在源码中system/sepolicy/attributes可以找到一些常见属性的详细注释

#### class：
客体的具体类别。用class来定义一个客体类别，具体定义方式在system/sepolicy/security_classes里面有定义.

#### perm_set：
就是进程对某个文件或者文件类型的具体的操作。系统的定义在system/sepolicy/access_vectors.

#### 4.2 添加新服务TE文件过程介绍
##### 添加te文件
```
type checkdata, domain;
#type checkdata_exec, exec_type, file_type;
# permissive checkdata;
init_daemon_domain(checkdata);

```

##### 如果加了exec_type属性 需要修改file_contexts
```
/system/bin/checkdata.sh       u:object_r:checkdata_exec:s0
```

##### 如果你定义服务在init.rc中 需要使用对应的seclabel
```
service mtkeepinit /system/bin/checkdata
    class core
    user root
    group root
    #disabled
    oneshot
    seclabel u:r:checkdata:s0

```

##### 使用audit2allow工具添加具体权限
需要在linux系统的电脑安装这个功能。再把板卡/平台如下的打印拷贝到1.txt文件，然后audit2allow -i 1.txt。
```
type=1400 audit(1514765145.022:126): avc: denied { getattr } for pid=1630 comm="sh" path="/system/bin/checkdata.sh" dev="mmcblk0p18" ino=286 scontext=u:r:shell:s0 tcontext=u:object_r:checkdata_exec:s0 tclass=file permissive=0

```
即可得到下面基本语法的语句，再拷贝到对应的te文件即可。
```
#============= shell ==============
allow shell checkdata_exec:file getattr;
```

### 参考链接：
- http://www.jianshu.com/p/a3572eee341c
- C厂KB文档
