---
layout: post
title: "Android性能优化典范 - 第4季"
date: 2015-11-03 22:19
comments: true
sidebar: false
categories: Android Android:Performance
---

![android_perf_patterns_season_4](/images/android_perf_patterns_season_4.png)

[Android性能优化典范](https://www.youtube.com/playlist?list=PLWz5rJ2EKKc9CBxr3BVjPTPoDPLdPIFCE)第4季的课程终于更新了，这次一共17个短视频课程，包括的内容大致有：更高效的ArrayMap容器，使用Android系统提供的特殊容器来避免自动装箱，避免使用枚举类型，注意onLowMemory与onTrimMemory的回调，避免内存泄漏，高效的位置更新操作，重复layout操作的性能影响，以及使用Batching，Prefetching优化网络请求，压缩传输数据等等使用技巧。下面是对这些课程的总结摘要，认知有限，理解偏差的地方请多多交流指正！

## 1)Cachematters for networking
你知道在Android上面效率最高，速度最快的下载一段数据内容的方式是什么吗？当我们讨论到网络操作性能的时候，处理好网络数据的缓存是最基础的步骤之一。从设备的缓存中直接读取数据肯定比从网络上获取要更加的便捷高效，特别是在某些数据会需要频繁访问到的情况下，把这些数据存放到设备上以便快速进行访问就显得十分有必要了。

Android系统上的Http Response Cache是默认关闭的，这样每次即使一样的数据请求也需要重复被执行，我们可以通过下面的代码示例开启[HttpResponseCache](http://developer.android.com/reference/android/net/http/HttpResponseCache.html)。

![android_perf_4_network_cache_enable](/images/android_perf_4_network_cache_enable.png)

开启Cache之后，Http操作相关的返回数据就会缓存到文件系统上，他们可能发生的场景有普通的http请求，或者是打开某个URL去获取数据。HttpResponseCache的数据清除通过两种方式：第一种方式是缓存溢出的时候删除最旧最老的文件，第二种方式是通过Http返回Header中的Cache-Control字段来进行控制的。

通常来说，HttpResponseCache会缓存所有的返回信息，包括实际的数据与Header的部分，一般情况下，这个Cache会自动根据协议返回Cache-Control的内容与当前缓存的数据量来决定哪些数据应该继续保留，哪些数据应该删除。但是在一些极端的时候，例如服务器返回的数据没有设置Cache废弃的时间，或者是本地的Cache文件系统与返回的缓存数据有冲突，或者是某些特殊的网络环境导致HttpResponseCache工作异常。在这种情况下，就需要我们自己来实现Http的缓存Cache：设计DiskCacheManager与Cache的缓存策略。关于第一点，我们可以扩展Android系统提供的DiskLruCache来实现。而第二点，就相对来说复杂一些，我们可能需要把某些JSON数据设计成不能缓存的，另外一些JSON数据设计成缓存几天的，缩略图设计成缓存一两天等等。想要做到这两件事情，自己从头开始写会比较繁琐复杂，所幸的是，有不少著名的开源框架帮助我们快速的解决了那些问题。我们可以使用[Volly](https://developer.android.com/training/volley/index.html)，[okHTTP](http://square.github.io/okhttp/)，[Picasso](http://square.github.io/picasso/)来实现网络缓存。

实现好网络缓存之后，我们可以使用Android Studio里面的Network Traffic Tools来查看网络访问的情况，另外我们还可以使用[AT&T ARO](https://developer.att.com/application-resource-optimizer?utm_campaign=android_series_#cachematters_for_networking_101315&utm_source=anddev&utm_medium=yt-annt)工具来抓取网络数据包进行分析查看。

## 2)Optimizing Network Request Frequencies



<!-- More -->

![android_perf_3_arraymap_two_array](/images/android_perf_3_arraymap_two_array.png)

***
首发于CSDN：[Android性能优化典范（三）](http://www.csdn.net/article/2015-08-12/2825447-android-performance-patterns-season-3)
