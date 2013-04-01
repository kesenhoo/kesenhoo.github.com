---
layout: post
title: "Android Notes - Process and Thread"
date: 2013-03-31 16:27
comments: true
categories: Android
---

# 进程与线程
当程序的第一个组件开始启动时，Android系统会为这个程序启动一个新的Linux进程。默认的，程序中的后续其他组件都是运行在这个进程的线程中(这个线程被成为"主"线程:main thread)。如果程序的组件在启动时发现已经存在这个程序的进程了(因为其他组件正在运行)， 那么这个组件将启动在该进程中，并使用同一线程。然而，你可以安排程序中的不同组件运行在另外一个进程中，而且你可以为任何进程创建其它的线程。

## Process：进程
默认的，同一程序的所有组件都是运行在一个Proces里面的，并且大多数程序都不应该去改变这一规则。然而，如果你需要控制某一确定的组件的Proces，你可以在manifest文件中做特殊设置。*Music播放器的Playback Service就可以这样做*

<!-- more -->

manifest中的activity，service，receiver与provider的标签都可以支持**android:process**的属性，它可以为这个组件的运行指定一个特定的进程。这样你可以为某些组件设置运行的进程而其他组件共享一个进程。你还可以通过设置进程属性使得不同程序的运行在同一个进程，共享同一个Linux ID，并且签有同样的签名。

application标签也可以支持设置**android:process**属性，这样会给所有的组件设置一个默认的进程值。

Android会在系统内存紧张时决定关闭某些进程。那么程序中的运行的组件会因此被摧毁掉。当他们需要再次运行时会重新启动一个进程。

当决定杀掉哪一个进程时，Android系统会自动衡量进程的重要性。例如，一个在屏幕上不再可见的进程相对于那些有组件正在被显示的进程更容易被杀掉是显得合理的。那么衡量的权重后面会讲到。

## Process lifecycle：进程生命周期
Android系统会尝试尽可能的维持程序的存在。但是当需要为新的或者更重要的进程开辟内存空间的时候，最终某些程序是会要被拿掉。为了决定存活当中的程序哪些该拿掉，哪些该留下，系统会根据每一个进程的组件与组件运行状态来生成一个"importance hierarchy"（权重层级）。那些权重低的进程将依次被移除，直到系统恢复了足够的资源。

在权重层级中，一个有5个层次。下面列出了不同类型进程的权重：

1. **Foreground process**：  
   用户目前正在使用的进程。要成为此类型的进程需要满足下面的任意一点：  
   * 该进程拥有一个用户正在交互的页面。（onResume方法正在执行）
   * 该进程拥有一个Service，该Service绑定到正在与用户交互的Activity中。
   * 该进程拥有一个in the foreground的Service,通过执行startForeground().
   * 该进程拥有一个Service，正在执行Service的某些callbacks方法(onCreate()， onStart()，或者onDestroy()).
   * 该进程拥有一个BroadcastReceiver，并且在执行它的onReceive()方法。
2. **Visible process**
   一个没有任何foreground组件，但是仍然能够影响屏幕呈现内容的进程。需要满足下面条件之一：
   * 该进程没有任何foreground的组件，但是仍然对用户可见。例如onPause()被调用的情况，started dialog。
   * 该进程拥有一个一个Service，并且该Service绑定到某个Visible的activity上。
3. **Service process**  
   一个进程拥有正在运行的Service，该Service是通过startService()的方式被启动的，并且不会进入到前面的两种高权重的层级。尽管Service进程没有与用户看到的部分有直接关系，但是他们通常是在做用户关心在意的工作（例如后台播放音乐，后台下载网络数据），因此系统会保持他们能够运行，除非现有的内存已经不够维持前面2个权重层级的进程使用。  
4. **Background process**  
   进程拥有一个activity，并且这个activity不被用户所见（例如activity的onStop方法被执行）。这些进程对用户体验没有直接的影响，系统可以杀掉这些进程为前面三个层级的进程空出内存。通常来说，系统中存在许多后台进程正在运行，因此他们被保存为一个LRU(least recently used)列表，用来确保最近被使用过的activity会被最后杀死。如果一个activity正确的实现了它的生命周期函数，可以保存它的当前状态。那么杀掉该进程并不会对用户体验有明显的影响。因为当用户重新回到这个Activity时，activity可以所有可见时的状态。  
5. **Empty process**  
   该进程没有拥有任何激活状态的程序组件。保持该进程存在的唯一理由是为了缓存。使用缓存可以提升程序的下次启动时间。系统通常会权衡所有的资源来决定杀掉哪些缓存程序。

**取最高优先级**  
如果一个程序中有好几种优先级的组件，Android系统会把其中最高级别的当作整个程序的权重。例如，如果一个进程拥有一个service与一个visible activity，这个进程会被当作是一个visible进程而不是service进程。

**提升优先级**  
另外，一个进程的排名会因为其依赖的组件的权重提升而提升。例如，进程A本来是权重为3的，但是它的某个组件与另外一个权重为1的进程B进行绑定后，进程A的权重也会被提升为1。

因为一个执行service的进程的排名比一个后台activity的进程排名要高，所以，如果一个activity启动时要执行一段长时间的操作，应该选择使用Service而不是创建一个worker thread。例如，一个activity做上传图片的操作，应该选择启动一个Service做上传的动作。使用service能确保这个操作会至少有"service process"的优先级。

## Thread
当一个程序首次启动，系统会为这个程序创建一个**"main thread"**。这个线程非常重要，因为它将肩负起UI的控制调度，还包含绘制图像的事件。同时，它还是与UI相关的组件（来自android.widget与android.view下的组件）进行交互的中介。因此，有些时候main thread 也被成为**"UI thread"**.

系统不会为每一个组件的实例创建单独的线程。所有运行在同一个进程中的组件都会在UI Thread中被实例化。系统调用组件与他们自身的回调函数都是运行在UI Thread的。

例如，当用户点击屏幕上的一个button，程序的UI thread会把这个事件分发至button这个组件上，然后button会执行它的presss state并post an invalidate请求到事件队列中。UI thread然后从事件队列中取出消息并通知组件进行重绘。

当你的app执行一个比较重的工作时，单线程模式有可能会卡到UI。特别是，在UI线程里面做网络请求操作或者是db查询会严重卡到整个UI。当UI thread被阻塞时，没有事件能够继续被分发，包括绘制事件。那么在用户看来，这样的程序是糟糕的。更糟糕的是，如果UI线程被阻塞超过5秒，程序会就出现ANR的错误提示。那么用户可能会决定推出程序，并对该程序进行卸载。

另外，Andoid的UI组件不是thread-safe的。因此，你不应该在另外一个线程去操控UI组件。有两个原则需要遵守：  
* 不要阻塞UI线程。  
* 不要在UI线程之外访问UI组件。

### Worker threads
为了实现执行耗时的操作，你应该确保那些动作执行在另外一个线程("background" or "worker" threads)。

例如，下面的代码演示了点击事件后开启另外一个线程来下载并显示图片的操作：  

    public void onClick(View v) {
	    new Thread(new Runnable() {
	        public void run() {
	            Bitmap b = loadImageFromNetwork("http://example.com/image.png");
	            mImageView.setImageBitmap(b);
	        }
	    }).start();
    }

上面的例子看起来没有问题，实际上违法了第二条规则：不要在UI线程之外访问UI组件。
Android提供了下面三个方法来解决这个问题：

* Activity.runOnUiThread(Runnable)
* View.post(Runnable)
* View.postDelayed(Runnable, long)

例如下面就是使用View.post的方式实现的代码示例

    public void onClick(View v) {
	    new Thread(new Runnable() {
	        public void run() {
	            final Bitmap bitmap = loadImageFromNetwork("http://example.com/image.png");
	            mImageView.post(new Runnable() {
	                public void run() {
	                    mImageView.setImageBitmap(bitmap);
	                }
	            });
	        }
	    }).start();
	}


上面的代码虽然实现了功能，可是当系统变复杂时，会显得不好处理。也许我们可以考虑使用Handler，但是更好的方案也许是使用AsyncTask。

### Using AsyncTask
关于什么是AsyncTask与如何使用[AsyncTask](http://developer.android.com/reference/android/os/AsyncTask.html)，不再赘述。
下面是使用AsyncTask来实现上面的例子：

    public void onClick(View v) {
    	new DownloadImageTask().execute("http://example.com/image.png");
	}

    private class DownloadImageTask extends AsyncTask<String, Void, Bitmap> {
	    /** The system calls this to perform work in a worker thread and
	      * delivers it the parameters given to AsyncTask.execute() */
	    protected Bitmap doInBackground(String... urls) {
	        return loadImageFromNetwork(urls[0]);
    	}
    
	    /** The system calls this to perform work in the UI thread and delivers
	      * the result from doInBackground() */
	    protected void onPostExecute(Bitmap result) {
	        mImageView.setImageBitmap(result);
	    }
    }


### Thread-safe methods
在某些情况下，你实现的一些方法有可能会被不止一个线程中执行到，因此这些方法必须是线程安全的。

**在bound service的情况下，回调函数通常都需要是线程安全的。如果IBinder的Client与Server是在同一进程的话，那么被Client调用的方法是执行在Client的线程当中的。然而如果Client是在另外一个进程的话，被调用的方法则是执行在来自系统为Server端维护的一个线程池当中的某个线程中（非UI Thread）。例如，既然Service的onBind()的方法可以被service进程的UI线程所调用执行，那么onBind所返回的对象（Client端）所实现的方法则可以被线程池中的线程所调用执行。因为一个service可以拥有多个client，那么在同一时刻可以有不止一个线程可以占用同一个IBinder的回调函数。所以IBinder的方法必须是线程安全的。**

同样的，一个content provider可以接受来自另外一个进程的数据请求。尽管ContentResolver与ContentProvider类隐藏了实现细节，但是ContentProvider所提供的query()，insert()，delete()，update()与getType()都是在content provider进程的线程池中被调用执行的，而不是进程的主线程中。因为那些方法可能同时被多个线程所调用，所以他们都应该是线程安全的。

## Interprocess Communication
Android提供了为远程过程调用（RPC）提供了一种进程间通信（IPC）的机制。调用发生在activity或者其他组件中，执行却在另外一个进程，最后再把结果返回给调用者。这需要把调用的数据解析成操作系统能够识别的格式，解码，传递，再编码返回。Android提供了IPC交互的实现细节，因此我们只需要专注于定义与实现RPC接口。

为了执行IPC，你的程序必须通过bindService()方法绑定到service上，更多细节，请查看[Services](http://developer.android.com/guide/components/services.html)开发指南。

*****************
** 文章学习自http://developer.android.com/guide/components/processes-and-threads.html**  
** 转载请注明出自[四方城](http:://kesenhoo.github.com)，谢谢**

