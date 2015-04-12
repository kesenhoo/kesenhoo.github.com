---
layout: post
title: "Android性能优化之运算篇"
date: 2015-04-12 13:50
comments: true
sidebar: false
categories: Android Android:Deeper
---

![](/images/android_performance_course_udacity.png)
Google近期在Udacity上发布了[Android性能优化的在线课程](https://www.udacity.com/course/ud825)，分别从渲染，运算与内存，电量几个方面介绍了如何去优化性能，这些课程是Google之前在Youtube上发布的[Android性能优化典范](http://hukai.me/android-performance-patterns/)专题课程的细化与补充。

下面是运算篇章的学习笔记，部分内容与前面的性能优化典范有重合，欢迎大家一起学习交流！

### 1)Intro to Compute and Memory Problems
Android中的Java代码会需要经过编译优化再执行的过程。代码的不同写法会影响到Java编译器的优化效率。例如for循环的不同写法就会对编译器优化这段代码产生不同的效率，当程序中包含大量这种可优化的代码的时候，运算性能就会出现问题。想要知道如何优化代码的运算性能就需要知道代码在硬件层的执行差异。

### 2)Slow Function Performance
如果你写了一段代码，它的执行效率比想象中的要差很多。我们需要知道有哪些因素有可能影响到这段代码的执行效率。例如：比较两个float数值大小的执行时间是int数值的4倍左右。这是因为CPU的运算架构导致的，如下图所示：

![](/images/android_perf_compute_float_int.png)

虽然现代的CPU架构得到了很大的提升，也许并不存在上面所示的那么大的差异，但是这个例子说明了代码写法上的差异会对运算性能产生很大的影响。

<!-- More -->

通常来说有两类运行效率差的情况：第1种是相对执行时间长的方法，我们可以很轻松的找到这些方法并做一定的优化。第2种是执行时间短，但是执行频次很高的方法，因为执行次数多，累积效应下就会对性能产生很大的影响。

修复这些细节效率问题，需要使用Android SDK提供的工具，进行仔细的测量，然后再进行微调修复。

### 3)Traceview Walkthrough
通过Android Studio打开里面的Android Device Monitor，切换到DDMS窗口，点击左边栏上面想要跟踪的进程，再点击上面的Start Method Tracing的按钮，如下图所示：

![](/images/android_perf_compute_traceview.png)

启动跟踪之后，再操控app，做一些你想要跟踪的事件，例如滑动listview，点击某些视图进入另外一个页面等等。操作完之后，回到Android Device Monitor，再次点击Method Tracing的按钮停止跟踪。此时工具会为刚才的操作生成TraceView的详细视图。

![](/images/android_perf_compute_traceview_2.png)

关于TraceView中详细数据如何查看，这里不展开了，有很多文章介绍过。

### 4)Batching and Caching
为了提升运算性能，这里介绍2个非常重要的技术，Batching与Caching。

**Batching**是在真正执行运算操作之前对数据进行批量预处理，例如你需要有这样一个方法，它的作用是查找某个值是否存在与于一堆数据中。假设一个前提，我们会先对数据做排序，然后使用二分查找法来判断值是否存在。我们先看第一种情况，下图中存在着多次重复的排序操作。

![](/images/android_perf_compute_batching_1.png)

在上面的那种写法下，如果数据的量级并不大的话，应该还可以接受，可是如果数据集非常大，就会有严重的效率问题。那么我们看下改进的写法，把排序的操作打包绑定只执行一次：

![](/images/android_perf_compute_batching_2.png)

上面就是Batching的一种示例：把重复的操作拎出来，打包只执行一次。

**Caching**的理念很容易理解，在很多方面都有体现，下面举一个for循环的例子：

![](/images/android_perf_compute_caching.png)

上面这2种基础技巧非常实用，积极恰当的使用能够显著提升运算性能。

### 5)Blocking the UI Thread
提升代码的运算效率是改善性能的一方面，让代码执行在哪个线程也同样很重要。我们都知道Android的Main Thread也是UI Thread，它需要承担用户的触摸事件的反馈，界面视图的渲染等操作。这就意味着，我们不能在Main Thread里面做任何非轻量级的操作，类似I/O操作会花费大量时间，这很有可能会导致界面渲染发生丢帧的现象，甚至有可能导致ANR。防止这些问题的解决办法就是把那些可能有性能问题的代码移到非UI线程进行操作。

### 6)Container Performance
另外一个我们需要注意的运算性能问题是基础算法的合理选择，例如冒泡排序与快速排序的性能差异：

![](/images/android_perf_compute_container.png)

避免我们重复造轮子，Java提供了很多现成的容器，例如Vector，ArrayList，LinkedList，HashMap等等，在Android里面还有新增加的SparseArray等，我们需要了解这些基础容器的性能差异以及适用场景。这样才能够选择合适的容器，达到最佳的性能。

![](/images/android_perf_compute_container_2.png)

**Notes:**关于更多代码优化的小技巧，请点击[这里](http://hukai.me/android-training-performance-tips/)