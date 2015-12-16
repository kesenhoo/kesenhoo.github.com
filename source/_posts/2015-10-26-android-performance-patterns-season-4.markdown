---
layout: post
title: "Android性能优化典范 - 第4季"
date: 2015-11-03 22:19
comments: true
sidebar: false
categories: Android Android:Performance
---

![android_perf_patterns_season_4](/images/android_perf_patterns_season_4.png)

[Android性能优化典范](https://www.youtube.com/playlist?list=PLWz5rJ2EKKc9CBxr3BVjPTPoDPLdPIFCE)第4季的课程终于更新了，这次一共17个短视频课程，包括的内容大致有：

下面是对这些课程的总结摘要，认知有限，理解偏差的地方请多多交流指正！

## 1)Cachematters for networking
你知道在Android上面效率最高，速度最快的下载一段数据内容的方式是什么吗？当我们讨论到网络操作性能的时候，处理好网络数据的缓存是最基础的步骤之一。从设备的缓存中直接读取数据肯定比从网络上获取要更加的便捷高效，特别是在某些数据会需要频繁访问到的情况下，把这些数据存放到设备上以便快速进行访问就显得十分有必要了。

Android系统上关于网络请求的Http Response Cache是默认关闭的，这样会导致每次即使请求的数据内容一致也会需要重复被执行，我们可以通过下面的代码示例开启[HttpResponseCache](http://developer.android.com/reference/android/net/http/HttpResponseCache.html)。

<!-- More -->

![android_perf_4_network_cache_enable](/images/android_perf_4_network_cache_enable.png)

开启Http Response Cache之后，Http操作相关的返回数据就会缓存到文件系统上，他们可能发生的场景有普通的http请求，或者是打开某个URL去获取数据。

我们有两种方式来清除HttpResponseCache的缓存数据：第一种方式是缓存溢出的时候删除最旧最老的文件，第二种方式是通过Http返回Header中的Cache-Control字段来进行控制的。

通常来说，HttpResponseCache会缓存所有的返回信息，包括实际的数据与Header的部分，一般情况下，这个Cache会自动根据协议返回Cache-Control的内容与当前缓存的数据量来决定哪些数据应该继续保留，哪些数据应该删除。但是在一些极端的时候，例如服务器返回的数据没有设置Cache废弃的时间，或者是本地的Cache文件系统与返回的缓存数据有冲突，或者是某些特殊的网络环境导致HttpResponseCache工作异常。在这种情况下，就需要我们自己来实现Http的缓存Cache：设计DiskCacheManager与Cache的缓存策略。关于第一点，我们可以扩展Android系统提供的DiskLruCache来实现。而第二点，就相对来说复杂一些，我们可能需要把某些JSON数据设计成不能缓存的，另外一些JSON数据设计成缓存几天的，缩略图设计成缓存一两天等等。想要做到这两件事情，自己从头开始写会比较繁琐复杂，所幸的是，有不少著名的开源框架帮助我们快速的解决了那些问题。我们可以使用[Volly](https://developer.android.com/training/volley/index.html)，[okHTTP](http://square.github.io/okhttp/)，[Picasso](http://square.github.io/picasso/)来实现网络缓存。

实现好网络缓存之后，我们可以使用Android Studio里面的`Network Traffic Tools`来查看网络访问的情况，另外我们还可以使用[AT&T ARO](https://developer.att.com/application-resource-optimizer?utm_campaign=android_series_#cachematters_for_networking_101315&utm_source=anddev&utm_medium=yt-annt)工具来抓取网络数据包进行分析查看。

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
网络延迟通常来说很容易被用户察觉到，严重的网络延迟会对用户体验造成很大的影响，用户会抱怨App写的不好，而不会觉得是网络不好。

一个典型的网络操作行为，通常包含以下几个步骤：首先手机端发起网络请求，到达网络服务运营商的基站，再转移到服务提供者的服务器上，经过解码之后，接着访问本地的存储数据库，获取到数据之后，进行编码，最后按照原路进行逐层返回。如下图所示：

![android_perf_4_network_latency](/images/android_perf_4_network_latency.png)

在上面的网络请求链路当中的任何一个环节都有可能导致严重的延迟，成为性能瓶颈。但是这些环节可能出现的问题，客户端应用是无法进行调节控制的，应用能够做的就只是根据当前的网络环境选择当下最佳的策略来降低出现网络延迟的概率。主要的实施步骤有两步：第1步检测收集当前的网络环境信息，第2步根据当前收集到的信息进行网络请求的调整。

关于第1步检测当前的网络环境，我们可以使用系统提供的API来获取到相关的信息，如下图所示：

![android_perf_4_network_latency_detect](/images/android_perf_4_network_latency_detect.png)

通过上面的示例，我们可以获取到移动网络的详细子类型，例如4G(LTE),3G等等，详细分类见下图，获取到详细的移动网络类型之后，我们可以根据当前网络的速率来调整网络请求的行为：

![android_perf_4_network_latency_subtype](/images/android_perf_4_network_latency_subtype.png)

关于第2步根据收集到的信息进行策略的调整，我们可以参考这样一个策略。通常来说，我们可以把网络请求延迟划分为三档：例如网络延迟小于60ms的划分为GOOD，大于220ms的划分为BAD，介于两者之间的划分为OK（这里的60ms，220ms会需要根据不同的场景提前进行预算推测）。如果网络延迟属于GOOD的范畴，我们就可以做更多激进的预取数据的操作，如果网络延迟属于BAD的范畴，我们就应该考虑把当下的网络请求进行等待处理直到网络恢复到良好的状态。

![android_perf_4_network_latency_three_category](/images/android_perf_4_network_latency_three_category.png)

前面提到说60ms，220ms是需要提前自己预测的，可是预测的工作相当复杂。首先针对不同的机器与环境，网络延迟的三档阈值都不太一样，出现的概率也不尽相同，我们会需要针对这些不同的用户与设备选择不同的阈值进行处理：

![android_perf_4_network_latency_three_level](/images/android_perf_4_network_latency_three_level.png)

Android官方为了帮助我们设计自己的网络请求策略，为我们提供了模拟器的网络流量控制功能来对实际环境进行模拟测量，或者还可以使用AT&T提供的[AT&T Network Attenuator](http://developer.att.com/developer/legalAgreementPage.jsp?passedItemId=14500040)来帮助预估网络延迟。

## 5)Minimizing Asset Payload
对于网络传输的数据需要做压缩处理，这能够减少传输的数据量，提高网络操作的性能。不同的网络环境，下载速度以及网络延迟都存在差异：

![android_perf_4_min_asset_load](/images/android_perf_4_min_asset_load.png)

如果我们选择在网速更低的网络环境下进行数据传输，这就意味着需要执行更长的时间，而更长的网络操作行为，会导致电量消耗更加严重。另外传输的数据如果不做压缩处理，也同样会增加网络传输的时间，消耗更多的电量。不仅如此，未经过压缩的数据，也会消耗更多的流量，使得用户需要付出更多的流量费。通常来说，网络传输数据量的大小主要由两部分组成：图片与序列化的数据。

A)首先需要做的是减少图片的大小，选择合适的图片保存格式是第一步。下图展示了PNG,JPEG,WEBP三种主流格式在占用空间与图片质量之间的对比：

![android_perf_4_min_asset_png_jpeg_webp](/images/android_perf_4_min_asset_png_jpeg_webp.png)

对于JPEG与WEBP格式的图片，不同的清晰度对占用空间的大小会产生很大的影响：

![android_perf_4_min_asset_jpeg_level](/images/android_perf_4_min_asset_jpeg_level.png)

我们需要为不同的场景提供当前场景下最合适的图片，例如针对全屏显示的情况我们会需要一张清晰度比较高的图片，而如果只是显示为缩略图的形式，就只需要一个相对清晰度低很多的图片即可。服务器应该支持到为不同的使用场景分别准备多套清晰度不一样的图片，以便在对应的场景下能够获取到最适合自己的图片。

B)其次需要做的是减少序列化数据的大小。JSON与XML为了提高可读性，在文件中加入了大量的符号，空格等等字符，而这些字符对于程序来说是没有任何意义的。我们应该使用Protocal Buffers，Nano-Proto-Buffers，FlatBuffer来减少序列化的数据的大小。

Android系统为我们提供了工具来查看网络传输的数据情况，打开Android Studio的Monitor，里面有网络访问的模块。或者是打开AT&T提供的[ARO](https://developer.att.com/application-resource-optimizer)工具来查看网络请求状态。

## 6)







![android_perf_3_arraymap_two_array](/images/android_perf_3_arraymap_two_array.png)

***
首发于CSDN：[Android性能优化典范（三）](http://www.csdn.net/article/2015-08-12/2825447-android-performance-patterns-season-3)
