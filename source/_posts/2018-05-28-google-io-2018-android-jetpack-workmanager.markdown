---
layout: post
title: "Android Jetpack - 使用WorkManager处理简单的后台任务"
date: 2018-05-28 23:24
comments: true
sidebar: false
categories: Android Jetpack Google
---

时间切换到2018年的当下，当我们讨论后台处理任务的时候，一般可能涉及的类型有下面几种，例如：
* 发送程序运行日志
* 上传图片和视频
* 同步数据
* 处理数据
这些行为都需要在后台进行操作，在Android平台上，我们可以利用如下的这些可选方式来实现后台任务：

![google_io_2018_android_jetpack_workmanager_01](/images/google_io_2018_android_jetpack_workmanager_01.png)

那么到底我们如何做出合理的选择呢？过去的几年，Android系统随着版本的更新针对电量优化这一块做出了不同程度的限制优化，例如在Android M上的`Doze Mode`，Android N上的`Limit Implicit  Broadcast`，Android O上的`Background Service Limitations`以及最新的Android P上面的`App Standby Buckets`。为了确保后台任务对电量的消耗影响足够小，开发者需要很谨慎的处理后台任务。

## Types of Background Work
通常来说，我们可以把所有的后台任务按照任务紧迫性(是马上需要执行的任务/还是可以缓期执行的任务)和任务重要性(是确保一定要被执行的任务/还是最好能够执行的任务)进行四象限的划分。通常来说对于非确保一定要执行的任务，无论时间是否紧迫，我们都可以使用ThreadPool来完成这个任务。对于那些比较重要的又时间紧迫的任务，我们一般会使用Foreground Service来完成这个操作。比较有有意思的是最后一个象限：那些希望确保可以被执行但是又可以接受延期执行的任务。这些任务可以使用JobScheduler/JobDispatcher/AlarmManager/BroadcastReceivers来完成。WorkManager也刚好是用来解决这一类的问题的。

![google_io_2018_android_jetpack_workmanager_02](/images/google_io_2018_android_jetpack_workmanager_02.png)

## WorkManager Features
下面是WorkManager的一些突出特点：
* 确保可以被执行，并且可以设置执行的限定条件(例如仅仅在有网络连接的时候才进行图片的上传)
* 同样受到系统后台任务的限制管理(如APP进入Doze Mode的时候，任务不会被执行)
* 向后兼容；无论是否集成了Google Play Service服务，都是向后兼容的
* 任务可查询；如论当下在执行什么任务，都是可以直接查询获取到任务状态信息的(是成功还是失败了)
* 任务可串联；例如执行任务A之前需要任务B或者C先进行完成
* 
