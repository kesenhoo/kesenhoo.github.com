---
layout: post
title: "Android Training[Performance] - 管理应用的内存"
date: 2013-10-19 15:18
comments: true
sidebar: false
categories: Android Android:Training Android:Training:Performance
---

Random Access Memory(RAM)在任何软件开发环境中都是一个很宝贵的资源。这一点在物理内存通常很有限的移动操作系统上，显得尤为突出。尽管Android的Dalvik虚拟机扮演了常规的垃圾回收的角色，但这并不意味着你可以忽视app的内存分配与释放的时机与地点。

为了GC能够从你的app中及时回收内存，你需要避免Memory Leaks(这通常由引用的不能释放而导致)并且在适当的时机(下面会讲到的lifecycle callbacks)来释放引用。对于大多数apps来说，Dalvik的GC会自动把离开活动线程的对象进行回收。

这篇文章会解释Android如何管理app的进程与内存分配，并且你可以在开发Android应用的时候主动的减少内存的使用。关于Java的资源管理机制，请参加其它书籍或者线上材料。如果你正在寻找如何分析你的内存使用情况的文章，请参考这里[Investigating Your RAM Usage](http://developer.android.com/tools/debugging/debugging-memory.html)。

<!-- More -->

## 第1部分:Android是如何管理内存的 ##
Android并没有提供内存的交换区(Swap space)，但是它有使用[paging](http://en.wikipedia.org/wiki/Paging)与[memory-mapping(mmapping)](http://en.wikipedia.org/wiki/Memory-mapped_files)的机制来管理内存。这意味着任何你修改的内存(无论是通过分配新的对象还是访问到mmaped pages的内容)都会贮存在RAM中，而且不能被paged out。因此唯一完整释放内存的方法是释放那些你可能hold住的对象的引用，这样使得它能够被GC回收。只有一种例外是：如果系统想要在其他地方进行reuse。

### 1)共享内存 ###
Android通过下面几个方式在不同的Process中来共享RAM:

* 每一个app的process都是从同一个被叫做Zygote的进程中fork出来的。Zygote进程在系统启动并且载入通用的framework的代码与资源之后开始启动。为了启动一个新的程序进程，系统会fork Zygote进程生成一个新的process，然后在新的process中加载并运行app的代码。这使得大多数的RAM pages被用来分配给framework的代码与资源，并在应用的所有进程中进行共享。
* 大多数static的数据被mmapped到一个进程中。这不仅仅使得同样的数据能够在进程间进行共享，而且使得它能够在需要的时候被paged out。例如下面几种static的数据: 
	* Dalvik code (by placing it in a pre-linked .odex file for direct mmapping
	* App resources (by designing the resource table to be a structure that can be mmapped and by aligning the zip entries of the APK)
	* Traditional project elements like native code in .so files.
* 在许多地方，Android通过显式的分配共享内存区域(例如ashmem或者gralloc)来实现一些动态RAM区域的能够在不同进程间的共享。例如，window surfaces在app与screen compositor之间使用共享的内存，cursor buffers在content provider与client之间使用共享的内存。

关于如何查看app所使用的共享内存，请查看[Investigating Your RAM Usage](http://developer.android.com/tools/debugging/debugging-memory.html)

### 2)分配与回收内存 ###
这里有下面几点关于Android如何分配与回收内存的事实：

* 每一个进程的Dalvik heap都有一个限制的虚拟内存范围。这就是逻辑上讲的heap size，它可以随着需要进行增长，但是会有一个系统为它所定义的上限。
* 逻辑上讲的heap size和实际物理上使用的内存数量是不等的，Android会计算一个叫做Proportional Set Size(PSS)的值，它记录了那些和其他进程进行共享的内存大小。（假设共享内存大小是10M，一共有20个Process在共享使用，根据权重，可能认为其中有0.3M才能真正算是你的进程所使用的）
* Dalvik heap与逻辑上的heap size不吻合，这意味着Android并不会去做heap中的碎片整理用来关闭空闲区域。Android仅仅会在heap的尾端出现不使用的空间时才会做收缩逻辑heap size大小的动作。但是这并不是意味着被heap所使用的物理内存大小不能被收缩。在垃圾回收之后，Dalvik会遍历heap并找出不使用的pages，然后使用madvise把那些pages返回给kernal。因此，成对的allocations与deallocations大块的数据可以使得物理内存能够被正常的回收。然而，回收碎片化的内存则会使得效率低下很多，因为那些碎片化的分配页面也许会被其他地方所共享到。

### 3)限制应用的内存 ###
为了维持多任务的功能环境，Android为每一个app都设置了一个硬性的heap size限制。准确的heap size限制随着不同设备的不同RAM大小而各有差异。如果你的app已经到了heap的限制大小并且再尝试分配内存的话，会引起OutOfMemoryError的错误。

在一些情况下，你也许想要查询当前设备的heap size限制大小是多少，然后决定cache的大小。可以通过getMemoryClass()来查询。这个方法会返回一个整数，表明你的app heap size限制是多少megabates。

### 4)切换应用 ###
当用户在不同应用之间进行切换的时候，不是使用交换空间的办法。Android会把那些不包含foreground组件的进程放到LRU cache中。例如，当用户刚开始启动了一个应用，这个时候为它创建了一个进程，但是当用户离开这个应用，这个进程并没有离开。系统会把这个进程放到cache中，如果用户后来回到这个应用，这个进程能够被resued，从而实现app的快速切换。

如果你的应用有一个被缓存的进程，它被保留在内存中，并且当前不再需要它了，这会对系统的整个性能有影响。因此当系统开始进入低内存状态时，它会由系统根据LRU的规则与其他因素选择杀掉某些进程，为了保持你的进程能够尽可能长久的被cached，请参考下面的章节学习何时释放你的引用。

更对关于不在foreground的进程是Android是如何决定kill掉哪一类进程的问题，请参考[Processes and Threads](http://developer.android.com/guide/components/processes-and-threads.html).

## 第2部分:你的应用该如何管理内存 ##
待续……


***
**文章学习自[http://developer.android.com/training/articles/memory.html](http://developer.android.com/training/articles/memory.html)**
**转载请注明出自[http://kesenhoo.github.com](http:://kesenhoo.github.com)，谢谢**

