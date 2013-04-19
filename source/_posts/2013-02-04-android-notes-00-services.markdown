---
layout: post
title: "Android Notes 00 - Services"
date: 2013-02-04 18:37
comments: true
sidebar: false
categories: Android Android:Note
---

## 概要
* Service是一个可以在后台长时间进行工作的一个程序组件。这个组件并没有提供UI。其他的程序组件可以Start一个Service，并且可以在用户切换到另外一个程序的时候继续工作。另外，某个组件可以Bind到一个Service上，并与他进行交互，甚至是进行IPC操作。例如，一个Service可以处理网络交互，播放音乐，执行I/O操作，或者与Content Provider进行交互，他们都是在后台的。
	* Service可以跑在后台进行工作，即使用户在切换到另外一个程序。
	* Service可以允许其他组件与它进行Bind，从而进行IPC操作。
	* Service默认是跑在host程序的main thread。

<!-- more -->

* 两种形式:
	* **Started:** 当程序的某个组件（例如一个activity）通过执行startService()来启动某个Service时，我们认为这个Service是"Started"的。一旦被started,这个Service可以在后台执行，但是却不确定它的状态，即使启动这个Service的组件已经被销毁了，Service的状态还是无法确定（有可能存在，也有可能已经消失了）。通常情况下，一个started的Service执行单一操作，并且不会给叫起它的组件返回任何结果。例如，它可能是做通过网络执行上传或者下载一个文件的动作。当操作结束后，这个service应该自动结束自己。
	* **Bound:** 当某个程序组件通过执行bindService()来bind到一个service时，我们认为这个service是"Bound"的。一个bound的service提供一个client-server的界面来允许这个组件与service进行交互。发送请求，获取结果，甚至是IPC操作。一个bound的service，只要是还有组件是bind状态的，它就会一直运作。Service允许多个组件同时bind到它。只要所有的bind对象都unbind之后，这个service就会被销毁掉。
	* 尽管这是两种不同的type的Service，但是我们还是可以同时使用它的，也就是可以允许做started的同时进行bind的操作。这取决与你的实现方式：通过startCommand()方法来start一个service,通过onBind()的回调来设置允许bind动作。
	* 无论这个service是哪种形式的，它都可以被任何组件(即使是另外一个Process)所使用，就像任何组件都可以使用activity一样。当然，你也可以把一个service声明为private的，这样可以阻止其他程序的访问。
* **注意：**Service默认是跑在host程序的main thread里面的，它既不会主动创建它自己的Thread，也不会跑在另外一个Process(除非你特别指定)。这就意味着，假如你要在service里面做一些很耗CPU的操作(例如播放音乐，网络下载等)，你应该在service里面创建另外一个thread来做那些操作，从而避免ANR。

## 基础
* 为了创建一个service，我们需要创建一个继承自Service的类。并override一些callback方法。最重要的一些方法如下：当系统资源不足时，会强制停止service，并在用户重新回到activity时，系统对其进行恢复。如果一个service是bind到一个活动的activity上，则不容易被kill掉。假设service被声明为run in the foreground，那就几乎不会被kill掉。随着service被不断执行，它会被慢慢降低优先级，当系统资源不足时，会先stop这些优先级低的service,假设这个service被kill掉，那么会在资源足够时马上恢复它。
	* onStartCommand()： 通过call startService()的方式，需要handle这个callback。service需要通过stopSelf()或者是其他组件通过call stopService（）来stop这个service。 如果你只想提供bind service的方式，那么这个function就不需要实现了。
	* onBind()： 通过call bindService() 的方式，需要handle这个callback。返回一个IBinder对象。
	* onCreate()： Service第一次被创建是才会执行。
	* onDestroy()： Service最后被销毁时才会执行。

#### 在manifest文件中声明service
```xml
<manifest ... >
  ...
  <application ... >
      <service android:name=".ExampleService" />
      ...
  </application>
</manifest>
``` 
## 创建一个started的Service
* Service在 onStartCommand() 方法里面接收Intent。我们有两个类可以继承用来实现一个service
	* [Service](http://developer.android.com/reference/android/app/Service.html)：跑在main thread
	* [IntentService](http://developer.android.com/reference/android/app/IntentService.html)： 会在Service里面new 一个worker thread。我们需要做的只是实现onHandleIntent()，在这里接受每次启动service的参数。**注意**:这适合没有多线程并发的情况使用。每次start都是独立的操作。

#### 继承自IntentService (适合于不会有同时发出请求的情况)
* Creates a default worker thread that executes all intents delivered to onStartCommand() separate from your application's main thread.
* Creates a work queue that passes one intent at a time to your onHandleIntent() implementation, so you never have to worry about multi-threading.
* Stops the service after all start requests have been handled, so you never have to call stopSelf().
* Provides default implementation of onBind() that returns null.
* Provides a default implementation of onStartCommand() that sends the intent to the work queue and then to your onHandleIntent() implementation.
* 要做的只是提供constructor并实现 onHandleIntent().
```java
public class HelloIntentService extends IntentService {

  /** 
   * A constructor is required, and must call the super IntentService(String)
   * constructor with a name for the worker thread.
   */
  public HelloIntentService() {
      super("HelloIntentService");
  }

  /**
   * The IntentService calls this method from the default worker thread with
   * the intent that started the service. When this method returns, IntentService
   * stops the service, as appropriate.
   */
  @Override
  protected void onHandleIntent(Intent intent) {
      // Normally we would do some work here, like download a file.
      // For our sample, we just sleep for 5 seconds.
      long endTime = System.currentTimeMillis() + 5*1000;
      while (System.currentTimeMillis() < endTime) {
          synchronized (this) {
              try {
                  wait(endTime - System.currentTimeMillis());
              } catch (Exception e) {
              }
          }
      }
  }
}
```
#### 继承自Service（适合于可能会有同时发出请求的情况）
* 使用一个work queue（handler）来处理同时发出的请求，一次执行一个job。当然，也可以在接受到消息的时候，立即启动一个thread来做这个job，那么就会出现并发的情况。
```java
public class HelloService extends Service {
  private Looper mServiceLooper;
  private ServiceHandler mServiceHandler;

  // Handler that receives messages from the thread
  private final class ServiceHandler extends Handler {
      public ServiceHandler(Looper looper) {
          super(looper);
      }
      @Override
      public void handleMessage(Message msg) {
          // Normally we would do some work here, like download a file.
          // For our sample, we just sleep for 5 seconds.
          long endTime = System.currentTimeMillis() + 5*1000;
          while (System.currentTimeMillis() < endTime) {
              synchronized (this) {
                  try {
                      wait(endTime - System.currentTimeMillis());
                  } catch (Exception e) {
                  }
              }
          }
          // Stop the service using the startId, so that we don't stop
          // the service in the middle of handling another job
          stopSelf(msg.arg1);
      }
  }

  @Override
  public void onCreate() {
    // Start up the thread running the service.  Note that we create a
    // separate thread because the service normally runs in the process's
    // main thread, which we don't want to block.  We also make it
    // background priority so CPU-intensive work will not disrupt our UI.
    HandlerThread thread = new HandlerThread("ServiceStartArguments",
            Process.THREAD_PRIORITY_BACKGROUND);
    thread.start();
    
    // Get the HandlerThread's Looper and use it for our Handler 
    mServiceLooper = thread.getLooper();
    mServiceHandler = new ServiceHandler(mServiceLooper);
  }

  @Override
  public int onStartCommand(Intent intent, int flags, int startId) {
      Toast.makeText(this, "service starting", Toast.LENGTH_SHORT).show();

      // For each start request, send a message to start a job and deliver the
      // start ID so we know which request we're stopping when we finish the job
      Message msg = mServiceHandler.obtainMessage();
      msg.arg1 = startId;
      mServiceHandler.sendMessage(msg);
      
      // If we get killed, after returning from here, restart
      return START_STICKY;
  }

  @Override
  public IBinder onBind(Intent intent) {
      // We don't provide binding, so return null
      return null;
  }
  
  @Override
  public void onDestroy() {
    Toast.makeText(this, "service done", Toast.LENGTH_SHORT).show(); 
  }
}
```
* 关于onStartCommand()的返回值：在系统因资源不足时杀死这个service,之后以何种方式restart与这里的返回值有关
	* START_NOT_STICKY： If the system kills the service after onStartCommand() returns, do not recreate the service, unless there are pending intents to deliver.
	* START_STICKY： recreate the service and call onStartCommand(), but do not redeliver the last intent.除非系统有pending的intent，否则会丢给onstartCommand()一个null的intent。这适合于media player一类的service。（This is suitable for media players (or similar services) that are not executing commands, but running indefinitely and waiting for a job.）
	* START_REDELIVER_INTENT： recreate the service and call onStartCommand() with the last intent that was delivered to the service. This is suitable for services that are actively performing a job that should be immediately resumed, such as downloading a file. 适合于下载等需要立即恢复的工作。

#### 启动Service
```java
Intent intent = newIntent(this,HelloService.class);
startService(intent);
```
* 若这个service没有启动过，这先执行onCreate()，然后才是onStartCommand()。 否则，若这个service已经启动了，则会直接执行onStartComand()。
* 默认的startService方式是无法返回数据给叫起它的对象的，若是需要返回数据，可以使用pendingIntent的getBoardcast方式发送给service，然后service再使用这个broadcast进行return value。

#### 停止Service
* 一个started类型的service必须自己来manage own lifecycle。一旦执行下面的方法，系统会立即停止这个service.
	* 自己执行stopSelf()
	* 其他组件执行stopService()
* 当多个并发时，需要通过执行stopSelf(int)来避免其他正在执行的动作也被一起杀掉，这个int id是onStartCommand()里面的参数。
* 请注意，在任务执行完毕时，请及时关闭service，以避免电量的浪费。

## 创建一个Bound的Service
* 通过bindService（）的方式创建的service。
* 通常来说，当你需要这个Service提供一些与其他程序组件进行IPC交互的功能时，会使用这种方式。
* 这种方式启动的service，并不需要自己去停止service。当没有组件再bind上的时候，系统会kill掉它。
* 创建bound方式的service的第一件事情是，定义客户端与Service交互的接口。这些接口通过onBind()里面的IBinder对象来进行操作。
* 可以有多个客户端同时bind到Service,当客户端做完事情之后，会执行unbindService()来进行解绑。

## 发送通知给用户
* Service可以通过[Toast Notifications](http://developer.android.com/guide/topics/ui/notifiers/toasts.html) or [Status Bar Notifications](http://developer.android.com/guide/topics/ui/notifiers/notifications.html)来通知用户。
* 最好的方式是通过Status Bar Notification来通知用户，例如下载完成等操作，用户可以拉开notification，再进行其他操作。

## 使得Service跑在foregound
* A foreground service: 一个即使在系统资源低的情况下，也不会杀掉的service。可以认为这个Service是与用户正在交互的。它会在status bar上显示一个ongoing的notification，除非这个service的工作已经被停止或者是不再是foreground的了。例如，播放音乐是一个foreground的service，需要在status bar上一直显示正在播放的歌曲。用户可以点击notification再launch起music或者做切歌等动作。
* startForeground()：为了使得service跑在foreground,需要执行这个方法。
```java
Notification notification =new Notification(R.drawable.icon, getText(R.string.ticker_text),
        System.currentTimeMillis());
Intent notificationIntent =new Intent(this,ExampleActivity.class);
PendingIntent pendingIntent =PendingIntent.getActivity(this,0, notificationIntent,0);
notification.setLatestEventInfo(this, getText(R.string.notification_title),
        getText(R.string.notification_message), pendingIntent);
startForeground(ONGOING_NOTIFICATION, notification);
```
* stopForeground(boolean)： 使用这个方法把service从foreground移除（变成background）

## 管理Service的Lifecycle
* A started service 与 A bound service：
* 上面两种方式并不总是独立存在的。你也可以在一个Start的service做bind的动作。例如，播放音乐的Service可以由start的方式来启动，之后还可以做bind的操作，这个时候，执行stopService并不会使得Service立即停止，除非所有bind的对象都解绑，Service才会停止。
* **如果一个service既可以started与bound。当一个service是started的，系统并不会在所有的Client都unbind之后去杀死这个service，我们需要显示的去停止这个service，通过stopSelf或者stopService的方式。**

#### 实现生命周期的一些callback方法
```java
public class ExampleService extends Service {
    int mStartMode;       // indicates how to behave if the service is killed
    IBinder mBinder;      // interface for clients that bind
    boolean mAllowRebind; // indicates whether onRebind should be used

    @Override
    public void onCreate() {
        // The service is being created
    }
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        // The service is starting, due to a call to startService()
        return mStartMode;
    }
    @Override
    public IBinder onBind(Intent intent) {
        // A client is binding to the service with bindService()
        return mBinder;
    }
    @Override
    public boolean onUnbind(Intent intent) {
        // All clients have unbound with unbindService()
        return mAllowRebind;
    }
    @Override
    public void onRebind(Intent intent) {
        // A client is binding to the service with bindService(),
        // after onUnbind() has already been called
    }
    @Override
    public void onDestroy() {
        // The service is no longer used and is being destroyed
    }
}
```
#### 两种service的lifecycle图示：

![service_lifecycle.png](/images/articles/service_lifecycle.png)

onCreate与onDestory负责创建与释放一些资源。例如开的线程等。


*****************
**文章学习自[http://developer.android.com/guide/components/services.html](http://developer.android.com/guide/components/services.html)**  
**转载请注明出自[http://kesenhoo.github.com](http:://kesenhoo.github.com)，谢谢**

