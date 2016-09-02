---
layout: post
title: "Android性能优化典范 - 第6季"
date: 2016-08-31 23:12
comments: true
sidebar: false
categories: Android Android:Performance
---

![android_perf_patterns_season_5](/images/android_perf_patterns_season_5.png)

> 这是[Android性能优化典范](https://www.youtube.com/watch?v=Vw1G1s73DsY&index=74&list=PLWz5rJ2EKKc9CBxr3BVjPTPoDPLdPIFCE)第6季的课程学习笔记，最近个人事情比较多，从被@起，这篇学习笔记就一直被惦记着，现在学习记录分享一下，请多多包涵担待指正！这次才一共6个段落，涉及的内容主要有：程序启动时间相关的三个方面：activity的创建过程，臃肿的application启动对象，主题启动显屏。另外还介绍了减少安装包大小的checklist以及如何使用VectorDrawable来减少安装包的大小。

## 1）App Launch time 101
提高程序的启动速度意义重大，很显然，启动时间越短，用户才越有耐心等待打开这个APP进行使用，反之启动时间越长，用户则越有可能来不及等到APP打开就已经切换到其他APP了。程序启动过程中的那些复杂错误的操作很可能导致严重的性能问题。Android系统会根据用户的操作行为调整程序的显示策略，用来提高程序的显示性能。例如，一旦用户点击桌面图标，Android系统会立即显示一个启动窗口，这个窗口会一直保持显示直到画面中的元素成功加载并绘制完第一帧。这种行为常见于程序的冷启动，或者程序的热启动场景（程序从后台被唤起或者从其他APP界面切换回来）。那么关键的问题是，用户很可能会因为从启动窗口到显示画面的过程耗时过长而感到厌烦，从而导致用户没有来得及等程序启动完毕就切换到其他APP了。更严重的是，如果启动时间过长，可能导致程序出现ANR。我们应该避免出现这两种糟糕的情况。

从技术角度来说，当用户点击桌面图标开始，系统会立即为这个APP创建独立的专属进程，然后显示启动窗口，直到APP在自己的进程里面完成了程序的创建以及主线程完成了Activity的初始化显示操作，再然后系统进程就会把启动窗口替换成APP的显示窗口。

![android_perf_6_launch_time_start_process](/images/android_perf_6_launch_time_start_process.png)

上述流程里面的绝大多数步骤都是由系统控制的，一般来说不会出现什么问题，可是对于启动速度，我们能够控制并且需要特别关注的地方主要有三处：

* 1）Activity的onCreate流程，特别是UI的布局与渲染操作，如果布局过于复杂很可能导致严重的启动性能问题。
* 2）Application的onCreate流程，对于大型的APP来说，通常会在这里做大量的通用组件的初始化操作。
* 3）目前有部分APP会提供自定义的启动窗口，这里可以做成品牌宣传界面或者是给用户提供一种程序已经启动的视觉效果。

在正式着手解决问题之前，我们需要掌握一套正确测量评估启动性能的方法。所幸的是，Android系统有提供一些工具来帮助我们定位问题。

* 1）首先是**display time**：从Android KitKat版本开始，Logcat中会输出从程序启动到某个Activity显示到画面上所花费的时间。这个方法比较适合测量程序的启动时间。

![android_perf_6_launch_time_display_time](/images/android_perf_6_launch_time_display_time.png)

* 2）其次是**reportFullyDrawn**方法：我们通常来说会使用异步懒加载的方式来提升程序画面的显示速度，这通常会导致的一个问题是，程序画面已经显示，可是内容却还在加载中。为了衡量这些异步加载资源所耗费的时间，我们可以在异步加载完毕之后调用`activity.reportFullyDrawn()`方法来告诉系统此时的状态，以便获取整个加载的耗时。

![android_perf_6_launch_time_report_fully_drawn](/images/android_perf_6_launch_time_report_fully_drawn.png)

* 3）然后是**Method Tracing**：前面两个方法提供了启动耗时的总时间，可是却无法提供具体的耗时细节。为了获取具体的耗时分布情况，我们可以使用Method Tracing工具来进行详细的测量。

![android_perf_6_launch_time_method_tracing](/images/android_perf_6_launch_time_method_tracing.png)

* 4）最后是**Systrace**：我们可以在onCreate方法里面添加trace方法来声明需要跟踪的起止位置，系统会帮忙统计中间经历过的函数调用耗时，并输出报表。

![android_perf_6_launch_time_systrace](/images/android_perf_6_launch_time_systrace.png)

## 2）





![android_perf_5_threading_main_thread](/images/android_perf_5_threading_main_thread.png)

<!-- More -->

***
首发于CSDN：[Android性能优化典范（五）](http://ms.csdn.net/geek/71031)
