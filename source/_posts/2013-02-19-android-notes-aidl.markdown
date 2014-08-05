---
layout: post
title: "Android Notes(02) - AIDL"
date: 2013-02-19 21:18
comments: true
sidebar: false
categories: Android Android:Notes
---

* 在Android process之间不能用通常的方式去访问彼此的内存数据。 他们把需要传递的数据解析成基础对象，使得系统能够识别并处理这些对象。因为这个处理过程很难写，所以Android使用AIDL来解决这个问题。
* 在定义AIDL接口之前，请意识到call这些接口是direct function call。 请不要认为这些call 接口的行为是发生在另外一个线程里面的。具体的不同因这个调用是发生在local process还是remote process而异。
	* 发生在local process里面的调用会跑在这个local process的thread里面。如果这是你的主UI线程，那么AIDL接口的调用也会发生在这个UI thread里面。如果这是发生在另外一个thread，那么调用会发生在service里面。因此，如果仅仅是发生在local process的调用，则你可以完全控制这些调用，当然这样的话，也就不需要用AIDL了。因为你完全可以使用Bound Service的第一种方式去实现。
	* 发生在remote process里面的调用是会跑在你自己的process所维护的一个thread pool里面。那么你需要注意可能会在同一时刻接受到多个请求。所以AIDL的操作需要做到thread-safe。(*每次请求，都会交给Service，在线程池里面启动一个thread去执行那些请求，所以那些方法需要是线程安全的*)
	* oneway关键字改变了remote call的行为。当使用这个关键字时，remote call不会被阻塞住，它仅仅是发送交互数据后再立即返回。IBinder thread pool 之后会把它当作一个通常的remote call呼叫。

<!-- more -->

## 定义一个AIDL接口
* 需要在src的目录下使用Java的语法去定义一个.aidl的文件，Android SDK tools会基于这个文件在gen目录下生成一个IBinder的接口（与.aidl同名，后缀名为java的文件）。Service需要实现这个接口，然后Client程序才可以bind到service并执行接口的方法，从而实现IPC。
* 具体实现一个AIDL接口，需要下面几个步骤：(请注意，aidl的文件需要向后兼容[能增加，不能减少与修改之前的接口]，因为这个文件需要提供给客户端程序)
	* 创建一个.aidl文件
	* 实现接口
	* 暴露接口给Client

### 1)Create the .aidl file
```java
// IRemoteService.aidl
package com.example.android;

// Declare any non-default types here with import statements

/** Example service interface */
interface IRemoteService{
    /** Request the process ID of this service, to do evil things with it. */
    int getPid();

    /** Demonstrates some basic types that you can use as parameters
     * and return values in AIDL.
     */
    void basicTypes(int anInt,long aLong,boolean aBoolean,float aFloat,
            double aDouble,String aString);
}
```

### 2)Implement the interface
```java
private final IRemoteService.Stub mBinder = new IRemoteService.Stub(){
    public int getPid(){
        returnProcess.myPid();
    }
    public void basicTypes(int anInt,long aLong,boolean aBoolean,
        float aFloat,double aDouble,String aString){
        // Does nothing
    }
};
```

### 3)Expose the interface to clients
* Service端：
```java
public class RemoteService extends Service{    @Override
    public void onCreate(){
        super.onCreate();
    }

    @Override
    public IBinder onBind(Intent intent){
        // Return the interface
        return mBinder;
    }

    private finalI RemoteService.Stub mBinder = newIRemoteService.Stub(){
        public int getPid(){
            returnProcess.myPid();
        }
        public void basicTypes(int anInt,long aLong,boolean aBoolean,
            float aFloat,double aDouble,String aString){
            // Does nothing
        }
    };
}
```
* Client端
```java
IRemoteService mIRemoteService;
private ServiceConnection mConnection = new ServiceConnection(){
    // Called when the connection with the service is established
    public void onServiceConnected(ComponentName className,IBinder service){
        // Following the example above for an AIDL interface,
        // this gets an instance of the IRemoteInterface, which we can use to call on the service
        mIRemoteService = IRemoteService.Stub.asInterface(service);
    }

    // Called when the connection with the service disconnects unexpectedly
    public void onServiceDisconnected(ComponentName className){
        Log.e(TAG,"Service has unexpectedly disconnected");
        mIRemoteService =null;
    }
};
```

## 通过IPC传递对象
* 某些时候传递的对象不是Android默认支持的那些，我们需要自己使得这个对象Parcelable化，实现步骤如下：
	* Make your class implement the Parcelable interface.
	* Implement writeToParcel, which takes the current state of the object and writes it to a Parcel.
	* Add a static field called CREATOR to your class which is an object implementing the Parcelable.Creator interface.
	* Finally, create an .aidl file that declares your parcelable class (as shown for the Rect.aidl file, below).If you are using a custom build process, do not add the .aidl file to your build. Similar to a header file in the C language, this .aidl file isn't compiled.

* 首先需要在aidl文件中声明这个对象类型
```java
package android.graphics;

// Declare Rect so AIDL can find it and knows that it implements
// the parcelable protocol.
parcelable Rect;
import android.os.Parcel;
import android.os.Parcelable;

public final class Rect implements Parcelable{
    public int left;
    public int top;
    public int right;
    public int bottom;

    public static final Parcelable.Creator<Rect> CREATOR = new
Parcelable.Creator<Rect>(){
        public Rect createFromParcel(Parcelin){
            return new Rect(in);
        }

        public Rect[] newArray(int size){
            return new Rect[size];
        }
    };

    public Rect(){
    }

    private Rect(Parcelin){
        readFromParcel(in);
    }

    public void writeToParcel(Parcelout){
        out.writeInt(left);
        out.writeInt(top);
        out.writeInt(right);
        out.writeInt(bottom);
    }

    public void readFromParcel(Parcelin){
        left = in.readInt();
        top = in.readInt();
        right = in.readInt();
        bottom = in.readInt();
    }
}
```

## 调用IPC的方法
* Include the .aidl file in the project src/ directory.
* Declare an instance of the IBinder interface (generated based on the AIDL).
* Implement ServiceConnection.
* Call Context.bindService(), passing in your ServiceConnection implementation.
* In your implementation of onServiceConnected(), you will receive an IBinder instance (called service). Call YourInterfaceName.Stub.asInterface((IBinder)service) to cast the returned parameter toYourInterface type.
* Call the methods that you defined on your interface. You should always trap DeadObjectException exceptions, which are thrown when the connection has broken; this will be the only exception thrown by remote methods.
* To disconnect, call Context.unbindService() with the instance of your interface.
```java
public static class Binding extends Activity {
    /** The primary interface we will be calling on the service. */
    IRemoteService mService = null;
    /** Another interface we use on the service. */
    ISecondary mSecondaryService = null;

    Button mKillButton;
    TextView mCallbackText;

    private boolean mIsBound;

    /**
     * Standard initialization of this activity.  Set up the UI, then wait
     * for the user to poke it before doing anything.
     */
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        setContentView(R.layout.remote_service_binding);

        // Watch for button clicks.
        Button button = (Button)findViewById(R.id.bind);
        button.setOnClickListener(mBindListener);
        button = (Button)findViewById(R.id.unbind);
        button.setOnClickListener(mUnbindListener);
        mKillButton = (Button)findViewById(R.id.kill);
        mKillButton.setOnClickListener(mKillListener);
        mKillButton.setEnabled(false);

        mCallbackText = (TextView)findViewById(R.id.callback);
        mCallbackText.setText("Not attached.");
    }

    /**
     * Class for interacting with the main interface of the service.
     */
    private ServiceConnection mConnection = new ServiceConnection() {
        public void onServiceConnected(ComponentName className,
                IBinder service) {
            // This is called when the connection with the service has been
            // established, giving us the service object we can use to
            // interact with the service.  We are communicating with our
            // service through an IDL interface, so get a client-side
            // representation of that from the raw service object.
            mService = IRemoteService.Stub.asInterface(service);
            mKillButton.setEnabled(true);
            mCallbackText.setText("Attached.");

            // We want to monitor the service for as long as we are
            // connected to it.
            try {
                mService.registerCallback(mCallback);
            } catch (RemoteException e) {
                // In this case the service has crashed before we could even
                // do anything with it; we can count on soon being
                // disconnected (and then reconnected if it can be restarted)
                // so there is no need to do anything here.
            }

            // As part of the sample, tell the user what happened.
            Toast.makeText(Binding.this, R.string.remote_service_connected,
                    Toast.LENGTH_SHORT).show();
        }

        public void onServiceDisconnected(ComponentName className) {
            // This is called when the connection with the service has been
            // unexpectedly disconnected -- that is, its process crashed.
            mService = null;
            mKillButton.setEnabled(false);
            mCallbackText.setText("Disconnected.");

            // As part of the sample, tell the user what happened.
            Toast.makeText(Binding.this, R.string.remote_service_disconnected,
                    Toast.LENGTH_SHORT).show();
        }
    };

    /**
     * Class for interacting with the secondary interface of the service.
     */
    private ServiceConnection mSecondaryConnection = new ServiceConnection() {
        public void onServiceConnected(ComponentName className,
                IBinder service) {
            // Connecting to a secondary interface is the same as any
            // other interface.
            mSecondaryService = ISecondary.Stub.asInterface(service);
            mKillButton.setEnabled(true);
        }

        public void onServiceDisconnected(ComponentName className) {
            mSecondaryService = null;
            mKillButton.setEnabled(false);
        }
    };

    private OnClickListener mBindListener = new OnClickListener() {
        public void onClick(View v) {
            // Establish a couple connections with the service, binding
            // by interface names.  This allows other applications to be
            // installed that replace the remote service by implementing
            // the same interface.
            bindService(new Intent(IRemoteService.class.getName()),
                    mConnection, Context.BIND_AUTO_CREATE);
            bindService(new Intent(ISecondary.class.getName()),
                    mSecondaryConnection, Context.BIND_AUTO_CREATE);
            mIsBound = true;
            mCallbackText.setText("Binding.");
        }
    };

    private OnClickListener mUnbindListener = new OnClickListener() {
        public void onClick(View v) {
            if (mIsBound) {
                // If we have received the service, and hence registered with
                // it, then now is the time to unregister.
                if (mService != null) {
                    try {
                        mService.unregisterCallback(mCallback);
                    } catch (RemoteException e) {
                        // There is nothing special we need to do if the service
                        // has crashed.
                    }
                }

                // Detach our existing connection.
                unbindService(mConnection);
                unbindService(mSecondaryConnection);
                mKillButton.setEnabled(false);
                mIsBound = false;
                mCallbackText.setText("Unbinding.");
            }
        }
    };

    private OnClickListener mKillListener = new OnClickListener() {
        public void onClick(View v) {
            // To kill the process hosting our service, we need to know its
            // PID.  Conveniently our service has a call that will return
            // to us that information.
            if (mSecondaryService != null) {
                try {
                    int pid = mSecondaryService.getPid();
                    // Note that, though this API allows us to request to
                    // kill any process based on its PID, the kernel will
                    // still impose standard restrictions on which PIDs you
                    // are actually able to kill.  Typically this means only
                    // the process running your application and any additional
                    // processes created by that app as shown here; packages
                    // sharing a common UID will also be able to kill each
                    // other's processes.
                    Process.killProcess(pid);
                    mCallbackText.setText("Killed service process.");
                } catch (RemoteException ex) {
                    // Recover gracefully from the process hosting the
                    // server dying.
                    // Just for purposes of the sample, put up a notification.
                    Toast.makeText(Binding.this,
                            R.string.remote_call_failed,
                            Toast.LENGTH_SHORT).show();
                }
            }
        }
    };

    // ----------------------------------------------------------------------
    // Code showing how to deal with callbacks.
    // ----------------------------------------------------------------------

    /**
     * This implementation is used to receive callbacks from the remote
     * service.
     */
    private IRemoteServiceCallback mCallback = new IRemoteServiceCallback.Stub() {
        /**
         * This is called by the remote service regularly to tell us about
         * new values.  Note that IPC calls are dispatched through a thread
         * pool running in each process, so the code executing here will
         * NOT be running in our main thread like most other things -- so,
         * to update the UI, we need to use a Handler to hop over there.
         */
        public void valueChanged(int value) {
            mHandler.sendMessage(mHandler.obtainMessage(BUMP_MSG, value, 0));
        }
    };

    private static final int BUMP_MSG = 1;

    private Handler mHandler = new Handler() {
        @Override public void handleMessage(Message msg) {
            switch (msg.what) {
                case BUMP_MSG:
                    mCallbackText.setText("Received from service: " + msg.arg1);
                    break;
                default:
                    super.handleMessage(msg);
            }
        }

    };
}
```

*****************
**文章学习自[http://developer.android.com/guide/components/aidl.html](http://developer.android.com/guide/components/aidl.html)**  
**转载请注明出自[http://kesenhoo.github.com](http:://kesenhoo.github.com)，谢谢**

