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

<!-- More -->

![android_perf_4_network_cache_enable](/images/android_perf_4_network_cache_enable.png)

开启Cache之后，Http操作相关的返回数据就会缓存到文件系统上，他们可能发生的场景有普通的http请求，或者是打开某个URL去获取数据。HttpResponseCache的数据清除通过两种方式：第一种方式是缓存溢出的时候删除最旧最老的文件，第二种方式是通过Http返回Header中的Cache-Control字段来进行控制的。

通常来说，HttpResponseCache会缓存所有的返回信息，包括实际的数据与Header的部分，一般情况下，这个Cache会自动根据协议返回Cache-Control的内容与当前缓存的数据量来决定哪些数据应该继续保留，哪些数据应该删除。但是在一些极端的时候，例如服务器返回的数据没有设置Cache废弃的时间，或者是本地的Cache文件系统与返回的缓存数据有冲突，或者是某些特殊的网络环境导致HttpResponseCache工作异常。在这种情况下，就需要我们自己来实现Http的缓存Cache：设计DiskCacheManager与Cache的缓存策略。关于第一点，我们可以扩展Android系统提供的DiskLruCache来实现。而第二点，就相对来说复杂一些，我们可能需要把某些JSON数据设计成不能缓存的，另外一些JSON数据设计成缓存几天的，缩略图设计成缓存一两天等等。想要做到这两件事情，自己从头开始写会比较繁琐复杂，所幸的是，有不少著名的开源框架帮助我们快速的解决了那些问题。我们可以使用[Volly](https://developer.android.com/training/volley/index.html)，[okHTTP](http://square.github.io/okhttp/)，[Picasso](http://square.github.io/picasso/)来实现网络缓存。

实现好网络缓存之后，我们可以使用Android Studio里面的Network Traffic Tools来查看网络访问的情况，另外我们还可以使用[AT&T ARO](https://developer.att.com/application-resource-optimizer?utm_campaign=android_series_#cachematters_for_networking_101315&utm_source=anddev&utm_medium=yt-annt)工具来抓取网络数据包进行分析查看。

## 2)Optimizing Network Request Frequencies
保持应用呈现的数据最新是一个很重要的基础功能，例如最新的新闻，天气，信息流等等。但是，过于频繁的同步最新数据会对性能产生很大的负面影响，在进行网络请求操作的时候一定要避免多度同步。

有时候退到后台的应用为了能够在切换回前台的时候呈现最新的数据，会偷偷在后台不停的做同步的操作。这种行为会带来很严重的问题，首先因为网络请求的行为异常的耗电，其次不停的进行网络同步会耗费很多带宽流量。

为了能够尽量的减少不必要的同步操作，我们需要遵守下面的一些规则：

* 首先我们要对网络行为进行分类，区分需要立即更新数据的行为和其他可以进行延迟的更新行为，在不同的场景下进行差异化处理。
* 其次要避免客户端对服务器的轮询操作，这样会浪费很多的电量与带宽流量。解决这个问题，我们可以使用Google Cloud Message来对更新的数据进行推送。
* 最后在某些必须做同步的场景下，需要避免使用固定相同的时间进行更新，应该在返回数据无更新的时候，使用双倍的时间进行下一次同步。更进一步，我们还可以通过判断当前设备的状态来决定同步的频率，例如判断设备处于休眠，运动等不同的状态设计不同的同步频率。

![android_perf_4_network_frequencies_backoff](/images/android_perf_4_network_frequencies_backoff.png)

另外，我们还可以通过判断设备是否连接上WiFi，是否正在充电来决定更新的频率。为了能够方便的实现这个功能，Android为我们提供了[GCMNetworkManager](https://developers.google.com/android/reference/com/google/android/gms/gcm/GcmNetworkManager)来判断设备当下的状态，从而设计更加高效的网络同步操作，如下图所示：

![android_perf_4_network_frequencies_gcm](/images/android_perf_4_network_frequencies_gcm.png)

## 3)Effective Prefetching
关于提升网络操作的性能，除了避免频繁的网络同步操作之外，还可以使用捆绑批量访问的方式来减少访问的频率，这说的就是Prefetching。

举个例子，在某个场景下，一开始发出了网络请求需要得到一张图片，隔了10s之后，发出第二次请求想要拿到另外一张图片，再隔了6s发出第三张图片的网络请求。这会导致设备的无线蜂窝一直处于高消耗的状态。Prefetching就是预先判定那些可能马上就会使用到的网络资源，捆绑一起集中进行网络请求。这样能够极大的减少电量的消耗，提升设备的续航时间。

![android_perf_4_prefetching_bundle](/images/android_perf_4_prefetching_bundle.png)

使用Prefetching的难点在于如何平衡事先获取的数据量到底是多少，如果预取的数据量偏少，那么就起不到什么效果，但是如果预期过多，又会导致访问的时间过长。

![android_perf_4_prefetching_tricky](/images/android_perf_4_prefetching_tricky.png)

那么问题来了，到底预取多少才比较合适呢？一个比较普适的规则是，在3G网络下可以预取1-5Mb的数据量，或者是按照提前预期后续1-2分钟的数据作为衡量标准。在实际的操作当中，我们还需要考虑当前的网络速度来决定预取的数据量，例如在同样的时间下，4G网络可以获取到12张图片的数据，而2G网络则只能拿到3张图片的数据。所以，我们还需要把当前的网络环境情况添加到设计预取数据量的策略当中去。判断当前设备的状态与网络情况，可以使用前面提到过的[GCMNetworkManager](https://developers.google.com/android/reference/com/google/android/gms/gcm/GcmNetworkManager)。

## 4)Adapting to Latency




![android_perf_3_arraymap_two_array](/images/android_perf_3_arraymap_two_array.png)

***
首发于CSDN：[Android性能优化典范（三）](http://www.csdn.net/article/2015-08-12/2825447-android-performance-patterns-season-3)
