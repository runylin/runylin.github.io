---
layout: post
title: "Activity的启动模式"
categories:
- Android
tags:
- 安卓应用 温故知新
---

我们知道，android与用户最直接的就是界面了，而与界面跳转联系比较紧密的概念是Task（任务）和Back Stack（回退栈）。activity的启动模式会影响Task和Back Stack的状态，Intent类中定义的一些标志（以FLAG_ACTIVITY_开头）和activity的属性taskAffinity也影响Task和Back Stack的状态。也是这篇的内容，因为在这个问题上遇到过坑。

### 一、定义启动模式
按照官方文档说明如下，具体见[链接](https://developer.android.com/guide/components/tasks-and-back-stack.html?hl=zh-cn#ActivityState)：


定义启动模式
启动模式允许您定义 Activity 的新实例如何与当前任务关联。 您可以通过两种方法定义不同的启动模式：

- 1、使用清单文件：
在清单文件中声明 Activity 时，您可以指定 Activity 在启动时应该如何与任务关联。

- 2、使用 Intent 标志：
调用 startActivity() 时，可以在 Intent 中加入一个标志，用于声明新 Activity 如何（或是否）与当前任务关联。

- 因此，如果 Activity A 启动 Activity B，则 Activity B 可以在其清单文件中定义它应该如何与当前任务关联（如果可能），并且 Activity A 还可以请求 Activity B 应该如何与当前任务关联。如果这两个 Activity 均定义 Activity B 应该如何与任务关联，则 Activity A 的请求（如 Intent 中所定义）优先级要高于 Activity B 的请求（如其清单文件中所定义）。

注：某些适用于清单文件的启动模式不可用作 Intent 标志，
同样，某些可用作 Intent 标志的启动模式无法在清单文件中定义。

上面官方文档说明代码实现的Intent属性优先级要高，比如launchMode为singleTop，Intent属性为FLAG_ACTIVITY_NEW_TASK，会以后者为准。

#### 二、使用清单文件
[官方文档链接](https://developer.android.com/guide/topics/manifest/activity-element.html?hl=zh-cn)

如下表所示，android:launchMode 这些模式分为两大类，“standard”和“singleTop”Activity为一类，Activity 可多次实例化；“singleTask”和“singleInstance”为另一类，只能保留一个 Activity 实例。

用例 | 启动模式 | 多个实例？ | 注释
---|---|---|---
大多数 Activity 的正常启动 | “standard” | 是 | 默认值。系统始终会在目标任务中创建新的 Activity 实例并向其传送 Intent。【A-B栈中再启动A，会变成A-B-A；A-B-A栈中再启动A，会变成A-B-A-A】
大多数 Activity 的正常启动 | “singleTop” | 有条件 | 如果目标任务的顶部已存在一个 Activity 实例，则系统会通过调用该实例的 onNewIntent() 方法向其传送 Intent，而不是创建新的 Activity 实例。【A-B栈中再启动A，会变成A-B-A；A-B-A栈中再启动A，还是A-B-A】
专用启动（不建议用作常规用途）| “singleTask” | 否 | 系统在新任务的根位置创建 Activity 并向其传送 Intent。 不过，如果已存在一个 Activity 实例，则系统会通过调用该实例的 onNewIntent() 方法向其传送 Intent，而不是创建新的 Activity 实例。
专用启动（不建议用作常规用途）| “singleInstance” | 否 | 与“singleTask"”相同，只是系统不会将任何其他 Activity 启动到包含实例的任务中。 该 Activity 始终是其任务唯一仅有的成员。


注：除了singleTask外，其他三个比较容易理解，写了demo测试也和想象一样。singleTask实验后发现如下：

*一个Activity的Launch Mode为singleTask时，在新建这个Activity时，默认taskAffinity情况下，一般不会新建task，taskAffinity属性默认application中是相同的，那么就会新建在当前栈。如果这个Activity已经在某个task的stack中了，此时只会调用它的onNewIntent()，而不会调用onCreate()。(谷歌的官方文档上称，如果一个activity的启动模式为singleTask，那么系统总会在一个新任务的最底部（root）启动这个activity，并且被这个activity启动的其他activity会和该activity同时存在于这个新任务中。如果系统中已经存在这样的一个activity则会重用这个实例，并且调用他的onNewIntent()方法。即，这样的一个activity在系统中只会存在一个实例。这种说法是不准确的，或者翻译的隔阂，已验证不是总会在栈底启动singletask的activity，已打印验证int taskId = getTaskId())其实，把启动模式设置为singleTask，framework在启动该activity时只会把它标示为可在一个新任务中启动，至于是否在一个新任务中启动，还要受其他条件的限制。增加一个taskAffinity属性给启动的singleTask的aty，那么将在新task启动。*

*Android Framework既能在同一个任务中对Activity进行调度，也能以Task为单位进行整体调度。在启动模式为standard或singleTop时，一般是在同一个任务中对Activity进行调度，而在启动模式为singleTask或singleInstance是，一般会对Task进行整体调度。*

### 三、使用 Intent 标志
[官方文档链接](https://developer.android.com/guide/components/tasks-and-back-stack.html?hl=zh-cn)

上面四种模式与 Intent 对象中的 Activity 标志（FLAG_ACTIVITY_* 常量）协同工作。主要 Intent 标志包括：
- FLAG_ACTIVITY_SINGLE_TOP 这会产生与 "singleTop"launchMode 值相同的行为。
- FLAG_ACTIVITY_NEW_TASK 这会产生与 "singleTask"launchMode 值相同的行为
- FLAG_ACTIVITY_CLEAR_TOP：
如果正在启动的 Activity 已在当前任务中运行，则会销毁当前任务顶部的所有 Activity，并通过 onNewIntent() 将此 Intent 传递给 Activity 已恢复的实例（现在位于顶部），而不是启动该 Activity 的新实例。
产生这种行为的 launchMode 属性没有值。FLAG_ACTIVITY_CLEAR_TOP 通常与 FLAG_ACTIVITY_NEW_TASK 结合使用。
一起使用时，通过这些标志，可以找到其他任务中的现有Activity，
并将其放入可从中响应 Intent 的位置。


### 三、之前遇到过的一个问题分析
### 3.1 看这个例子：
    待补充
### 3.2 查看任务栈
dumpsys activity 的 ACTIVITY MANAGER RECENT TASKS 条目下：

```
ACTIVITY MANAGER RECENT TASKS (dumpsys activity recents)
  Recent tasks:
  * Recent #0: TaskRecord{f1ce3b3 #3808 A=com.linxtu.fortest U=0 StackId=1 sz=2}
  * Recent #1: TaskRecord{242f4cf #3649 A=com.miui.home U=0 StackId=0 sz=1}
  * Recent #2: TaskRecord{3240eeb #3658 A=com.android.systemui U=0 StackId=0 sz=1}
  * Recent #3: TaskRecord{c1fad70 #3787 A=com.miui.bugreport U=0 StackId=-1 sz=0}
  * Recent #4: TaskRecord{536423a #3762 A=com.miui.tsmclient U=0 StackId=-1 sz=0}
  * Recent #5: TaskRecord{7a82448 #3751 I=com.android.deskclock/.activity.AlarmAlertFullScreenActivity U=0 StackId=1 sz=1}
  * Recent #6: TaskRecord{55592e1 #3725 I=com.android.settings/.Settings$WifiSettingsActivity U=0 StackId=-1 sz=0}
  * Recent #7: TaskRecord{3364806 #3701 A=com.android.incallui U=0 StackId=-1 sz=0}
  * Recent #8: TaskRecord{2b2aec7 #3698 A=com.miui.packageinstaller U=0 StackId=-1 sz=0}
  * Recent #9: TaskRecord{c3f3cf4 #3684 A=com.android.quicksearchbox U=0 StackId=1 sz=1}
  * Recent #10: TaskRecord{4377c1d #3650 A=android.task.stk.StkLauncherActivity U=0 StackId=-1 sz=0}
  * Recent #11: TaskRecord{edede92 #0 I=com.android.settings/.FallbackHome U=0 StackId=-1 sz=0}

```

### 四、参考文档
http://blog.csdn.net/jijiaxin1989/article/details/42242393

