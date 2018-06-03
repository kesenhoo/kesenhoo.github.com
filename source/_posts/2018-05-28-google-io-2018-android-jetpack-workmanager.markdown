---
layout: post
title: "Android Jetpack - 使用WorkManager处理简单的后台任务"
date: 2018-05-28 23:24
comments: true
sidebar: false
categories: Android Jetpack Google
---

时间来到2018年的当下，当我们讨论后台处理任务的时候，一般可能涉及的行为类型有下面一些类型，例如：

* 发送程序运行日志
* 上传图片和视频
* 同步数据
* 处理数据

这些行为都需要在后台进行操作，在Android平台上，我们可以利用如下的这些可选方式来实现后台任务：

![google_io_2018_android_jetpack_workmanager_01](/images/google_io_2018_android_jetpack_workmanager_01.png)

那么到底我们如何做出合理的选择呢？过去的几年，Android系统随着版本的更新针对电量优化这一块做出了不同程度的限制优化，例如在Android M上的`Doze Mode`，Android N上的`Limit Implicit  Broadcast`，Android O上的`Background Service Limitations`以及最新的Android P上面的`App Standby Buckets`。为了确保后台任务对电量的消耗影响足够小，对待后台任务的处理要更加的慎重小心。

## 0）Types of Background Work
通常来说，我们可以把所有的后台任务按照任务紧迫性(是马上需要执行的任务/还是可以缓期执行的任务)和任务重要性(是确保一定要被执行的任务/还是最好能够执行的任务)进行四象限的划分。通常来说对于非确保一定要执行的任务，无论时间是否紧迫，我们都可以使用ThreadPool来完成这个任务。对于那些比较重要的又时间紧迫的任务，我们一般会使用Foreground Service来完成这个操作。比较有有意思的是最后一个象限：那些希望确保可以被执行但是又可以接受延期执行的任务。这些任务可以使用JobScheduler/JobDispatcher/AlarmManager/BroadcastReceivers来完成。WorkManager也刚好是用来解决这一类的问题的。

![google_io_2018_android_jetpack_workmanager_02](/images/google_io_2018_android_jetpack_workmanager_02.png)

## 1）WorkManager Features
下面是WorkManager的一些突出特点：

* 确保可以被执行，并且可以设置执行的限定条件(例如仅仅在有网络连接的时候才进行图片的上传)
* 同样受到系统后台任务的限制管理(如APP进入Doze Mode的时候，任务不会被执行)
* 向后兼容；无论是否集成了Google Play Service服务，都是向后兼容的
* 任务可查询；如论当下在执行什么任务，都是可以直接查询获取到任务状态信息的(例如正在运行的状态是什么，结果是成功还是失败了)
* 任务可串联；例如执行任务A之前需要任务B或者C先进行完成
* 任务伺机执行：在条件满足的时候会尽快尝试触发任务的执行，不需要等待JobScheduler的唤醒，也不会需要等待JobScheduler进行批量任务处理的才被执行

WorkManager中的核心类有：

* Worker：这个类是真正干活的，工作逻辑都在这里面
* WorkRequest：
  * OneTimeWorkRequest：只执行一次的任务请求
  * PeriodicWorkRequest：重复执行的任务请求

举个例子：图片上传的后台任务是如何执行的。下面是上传图片的Worker示例：

![google_io_2018_android_jetpack_workmanager_03](/images/google_io_2018_android_jetpack_workmanager_03.png)

![google_io_2018_android_jetpack_workmanager_04](/images/google_io_2018_android_jetpack_workmanager_04.png)

其中uploadPhoto是执行在后台线程的，返回值可以是成功或者失败，还可以是重试，这意味着告诉系统这个任务需要后面找机会重新执行。有了上面那些基础，接下去就只需要利用Worker创建对应的WorkRequest，并并添加到WorkManager的执行队列中就好了。

![google_io_2018_android_jetpack_workmanager_05](/images/google_io_2018_android_jetpack_workmanager_05.png)

正常情况下，放到任务队列中的任务会被立马执行，可是如果遇到网络连接失败的情况，这样就会执行失败。此时我们就可以通过添加限定执行条件来达到优化的目的，例如设置限定只在网络连接成功的时候才进行任务的执行。

![google_io_2018_android_jetpack_workmanager_06](/images/google_io_2018_android_jetpack_workmanager_06.png)

## 2）Observing Work
有了上面的任务触发逻辑之后，那么如何做任务的监听呢？例如正在处理过程中显示一个进度圈，处理成功的时候消失进度等等。我们可以使用如下演示的范例来监听任务的执行状态。

![google_io_2018_android_jetpack_workmanager_07](/images/google_io_2018_android_jetpack_workmanager_07.png)

`LiveData`是Google开发的一个感知生命周期的架构组件。使用这个组件来hook监听request任务的`WorkStatus`。在WorkStatus里面有任务的id和status，其中status有6种状态，分别是`ENQUEUED`,`RUNNING`,`SUCCEEDED`,`FAILED`,`BLOCKED`,`CANCELLED`。

## 3）Chaining Work
通常来说，上传任务真正被执行之前，我们会对数据做一次压缩，因为每一个任务都需要在后台进行，并且需要保证执行顺序。我们可以使用下面的示例方式，先进行压缩，成功之后，再进行上传。

![google_io_2018_android_jetpack_workmanager_08](/images/google_io_2018_android_jetpack_workmanager_08.png)

只所以可以类似上面那样写，是因为每一步任务返回的都是`WorkContinuation`，使用它可以对不同的任务进行串联。

![google_io_2018_android_jetpack_workmanager_09](/images/google_io_2018_android_jetpack_workmanager_09.png)

如果想要多项任务并发执行，可以同时建立多个WorkRequest，一起交给WorkManager进行执行(根据CPU核心数和架构的不同，并发数量有所差异)。

![google_io_2018_android_jetpack_workmanager_10](/images/google_io_2018_android_jetpack_workmanager_10.png)

我们再把任务链设置的更加复杂一点，例如图片要先分别经过不同的滤镜处理，之后再进行压缩，最后才可以上传，那么使用WorkManager该如何实行呢？

![google_io_2018_android_jetpack_workmanager_11](/images/google_io_2018_android_jetpack_workmanager_11.png)

## 4）Inputs and Outputs
任务之间如何进行数据的传递呢？在介绍这个之前，我们需要了解下什么叫做MapReduce。例如，我们想要从三本书里面找出使用最多的词语，先把所有词语都进行计算一遍，然后对词语的使用次数进行排序，最后才可以找出使用最多的词语，我们把这个行为叫做MapReduce。

使用WorkManager的输入和输出数据具备如下的特点：

* 简单的KEY-VALUE
  * KEY都是String类型的
  * VALUE可以是基础数据类型和String
* 数据本身已经做了序列化处理
* 限定10KB大小以内

我们使用如下的方式进行输入的数据传递，构造一个map类型的Data，通过WorkManager的`setInputData()`给Worker进行传输数据。

![google_io_2018_android_jetpack_workmanager_12](/images/google_io_2018_android_jetpack_workmanager_12.png)

接下去Worker可以通过`getInputData()`来获取到输入的数据。

![google_io_2018_android_jetpack_workmanager_13](/images/google_io_2018_android_jetpack_workmanager_13.png)

一般来说，我们会需要把处理的结果进行返回，那么使用`setOutputData()`来完成这个操作就可以了

![google_io_2018_android_jetpack_workmanager_14](/images/google_io_2018_android_jetpack_workmanager_14.png)

有意思的事情是，在任务链中，输出的数据一般就是下一个任务的输入。那么当某个环节的一个任务是由多个任务的输出构成的时候，改如何处理呢？

![google_io_2018_android_jetpack_workmanager_15](/images/google_io_2018_android_jetpack_workmanager_15.png)

为了解决这个问题，我们需要了解`InputMergers`，顾名思义，它是用来合并多个输入数据变成一个的。一般来说有两种合并实现的方式(也可以自己自定义)

* `OverwritingInputMerger`(系统默认)：按照输入数据的先后顺序，相同KEY会被覆盖，不同的KEY内容会被保留

![google_io_2018_android_jetpack_workmanager_16](/images/google_io_2018_android_jetpack_workmanager_16.png)

* `ArrayCreatingInputMerger`：相同KEY的VALUE值进行合并，需要确保VALUE是相同数据类型的，否者会出现异常

![google_io_2018_android_jetpack_workmanager_17](/images/google_io_2018_android_jetpack_workmanager_17.png)

## 5）Cancelling Work
想要取消一个任务，只需要调用`cancelWorkById()`就好了，但是需要注意的是，这个方法只是**尽力而为**，因为相关想要取消的任务有可能已经在运行，也有可能已经执行结束了。

![google_io_2018_android_jetpack_workmanager_18](/images/google_io_2018_android_jetpack_workmanager_18.png)

## 6）Tags
前面我们有提到过好几次任务id，这个id是系统自动生成的，类似UUID这样的数值。我们无法通过这个id来判断这是一个什么样的任务，tags就是为了解决这个任务可读性的问题的。我们可以给任务打上一个或者多个tag来标记这是一个什么样的任务，然后可以通过这个tag来查询，取消任务等等。

![google_io_2018_android_jetpack_workmanager_19](/images/google_io_2018_android_jetpack_workmanager_19.png)

![google_io_2018_android_jetpack_workmanager_20](/images/google_io_2018_android_jetpack_workmanager_20.png)

使用Tag可以给我们提供很大的帮助，我们可以根据不同的模块和依赖给任务设置不同的tag，也可以根据任务的类型进行设置tag，这样就可以方便的进行批量任务操作了。

## 7）Unique Work
为了解决多个任务的同步问题，引入了Unique Work的机制。它有三种类型，分别为

* `KEEP`：新启动的Unique任务，如果之前已经存在，就继续保留旧的任务，如果不存在，则触发这次新的任务

![google_io_2018_android_jetpack_workmanager_21](/images/google_io_2018_android_jetpack_workmanager_21.png)

* `REPLACE`：取消或者删除之前的所有此类Unique任务，使用这次的任务作为最新任务，重复调用多次的时候，会以最后一次为准

![google_io_2018_android_jetpack_workmanager_22](/images/google_io_2018_android_jetpack_workmanager_22.png)

* `APPEND`：按照添加顺序，逐个执行任务

![google_io_2018_android_jetpack_workmanager_23](/images/google_io_2018_android_jetpack_workmanager_23.png)

## 8）Periodic Work
重复任务和我们之前认知的其他重复任务一样，具备一些如下的特点：

* 最短间隔时间15分钟(和JobScheduler一样)
* 同样受系统doze mode和其他的后台任务限制
* 不可以有任务链
* 不可以有触发延迟

## 9）Under The Hood
当系统接受到一个Work任务的时候，会先记录到自己的任务数据库中，接下去系统是如何判断执行的呢？如果任务符合当下执行的条件，那么会由Executor(可自定义，系统默认有实现)立即执行；如果我们的进程已经被杀死，那么任务什么时候可以被执行呢？如果设备运行在>=API 23，会交给JobScheduler触发IPC请求，唤醒我们的进程进行任务的执行；如果设备运行在< API 23的情况下，系统会判断设备是否有Firebase JobDispatcher，如果有会交给它进行处理；如果那些Google Play Service服务都没有，系统会使用AlarmManager和BroadcastReceivers的方式在合适的时候唤醒应用进行处理。

![google_io_2018_android_jetpack_workmanager_24](/images/google_io_2018_android_jetpack_workmanager_24.png)

## 10）Best Practice
* 何时使用WorkManager呢？下面给一些最佳实践的例子：
  * OK：上传图片和视频
  * OK：解析数据并存储到数据库中
  * NO：从调色板中获取颜色并设置到图片上
  * NO：解析数据并呈现到视图上进行显示
  * NO：处理交易的请求
* 不要使用WorkManager来存储数据，记得只有10kb的限制
* 记得给不同的任务设置各自的执行限定条件，避免无谓的资源浪费

![google_io_2018_android_jetpack_workmanager_25](/images/google_io_2018_android_jetpack_workmanager_25.png)
