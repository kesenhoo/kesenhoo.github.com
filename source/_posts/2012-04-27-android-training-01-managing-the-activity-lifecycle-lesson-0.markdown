---
layout: post
title: "Android Training 01 - 详解Activity声明周期[Lesson 0 - 章节概要]"
date: 2012-04-27 18:34
comments: true
sidebar: false
categories: Android
---

## Managing the Activity Lifecycle
**管理Activity生命周期**

* 当用户进入，退出，回到你的App，在程序中的[Activity](http://developer.android.com/reference/android/app/Activity.html) 实例都经历了生命周期中的不同状态。例如，当你的activity第一次启动的时候，它来到系统的前台，开始接受用户的焦点。在此期间，Android系统调用了一系列的生命周期中的方法。如果用户执行了启动另一个activity或者切换到另一个app的操作, 系统又会调用一些生命周期中的方法。
* 在生命周期的回调方法里面，你可以声明当用户离开或者重新进入这个Activity所需要执行的操作。例如, 如果你建立了一个streaming video player, 在用户切换到另外一个app的时候，你应该暂停video 并终止网络连接。当用户返回时，你可以重新建立网络连接并允许用户从同样的位置恢复播放。
* 这一章会介绍一些[Activity](http://developer.android.com/reference/android/app/Activity.html)中重要的生命周期回调方法，如何使用那些方法使得程序符合用户的期望且在activity不需要的时候不会导致系统资源的浪费。
* 完整的Demo示例：[ActivityLifecycle.zip](http://developer.android.com/shareables/training/ActivityLifecycle.zip)

<!-- more -->

### Lessons 
这一章我们将学习下面4个内容:
#### [Starting an Activity](http://developer.android.com/training/basics/activity-lifecycle/starting.html)
[Android Training - 01:详解Activity生命周期[Lesson 1 - 启动与销毁Activity]](http://blog.csdn.net/kesenhoo/article/details/7519270)

#### [Pausing and Resuming an Activity](http://developer.android.com/training/basics/activity-lifecycle/pausing.html)
[Android Training - 01:详解Activity生命周期[Lesson 2 - 暂停与恢复Activity]](http://blog.csdn.net/kesenhoo/article/details/7519985)]

#### [Stopping and Restarting an Activity](http://developer.android.com/training/basics/activity-lifecycle/stopping.html)
[Android Training - 01:详解Activity生命周期[Lesson 3 - 停止与重启Activity]](http://blog.csdn.net/kesenhoo/article/details/7520679)

#### [Recreating an Activity](http://developer.android.com/training/basics/activity-lifecycle/recreating.html)
[Android Training - 01:详解Activity生命周期[Lesson 4 - 重新创建Activity]](http://blog.csdn.net/kesenhoo/article/details/7524011)

**最后闲扯：**好吧，又是Activity的生命周期这个基本概念，感觉我是在做重新发明轮子的事情，因为网上关于这个概念的文章太多了，既然这次官方的基础训练课程又有涉及，就当是重温下好了，看看有没有新的观点与体会。

学习自：[http://developer.android.com/training/basics/activity-lifecycle/index.html](http://developer.android.com/training/basics/activity-lifecycle/index.html)，请多指教，谢谢！

**转载请注明出自:[四方城](http://kesenhoo.github.com)，谢谢配合！**
