---
layout: post
title: "Android Notes(06) - Camera"
date: 2014-04-21 21:24
comments: true
sidebar: false
categories: Android Android:Notes
---

Android framework为各种不同的Camera与Camera的特色功能提供了支持，使得可以在应用中进行拍照与录像。这篇文章会讨论一种简便，快速的拍照录像方式，为了给用户创建定制的相机体验，文章也会概述相机的高级功能。

## 0)开始之前
在应用中开启Android设备的相机功能之前，应该考量如下几个问题：

* **必须的相机硬件** - 当然不能把一个包含相机功能的应用安装到一个连相机硬件都没有的设备上。因此，应该在mainfest文件中声明需要使用到相机。
* **快速获取图片还是定制相机** - 应用将如何使用相机？是想做一个快速的抓拍还是录制一小段视频剪辑？还是说想提供一种新的相机使用方式？如果是快速的获取一张抓拍图片或者是一小段视频剪辑，建议查看下面的**3)使用已经安装的相机应用。**如果是为了开发一个定制相机功能的应用，查看下面的**4)创建一个相机应用**。
* **存储位置** - 生成的图片与视频是只对自己的应用可见还是其它类似Gallery的应用也可以访问？即使自己的应用被卸载后也不能被其他应用访问吗？建议查看**5)保存媒体文件**

## 1)简要概述
Android framework通过提供Camera API来支持拍照与录制视频的功能。下面是相关的类：

* [**Camera**](http://developer.android.com/reference/android/hardware/Camera.html)  
该类是控制相机硬件的基础的API。它可以用来拍照或者录制视频。
* [**SurfaceView**](http://developer.android.com/reference/android/view/SurfaceView.html)  
该类是用来呈现一个动态的相机预览界面。
* [**MediaRecorder**](http://developer.android.com/reference/android/media/MediaRecorder.html)  
该类用来使用相机录制视频。
* [**Intent**](http://developer.android.com/reference/android/content/Intent.html)  
使用[MediaStore.ACTION_IMAGE_CAPTURE](http://developer.android.com/reference/android/provider/MediaStore.html#ACTION_IMAGE_CAPTURE) 或者 [MediaStore.ACTION_VIDEO_CAPTURE](http://developer.android.com/reference/android/provider/MediaStore.html#ACTION_VIDEO_CAPTURE)作为Intent的action可以用来拍照与录制视频。

<!-- More -->

## 2)Manifest声明
在使用Camera API开发应用之前，应该确保应用的mainfest中有做恰当的声明，表明需要使用相机或者是相机的相关功能。

* **Camera Permission** - 为了使用相机硬件，你的应用必须请求使用Camera的权限。  
```xml
<uses-permission android:name="android.permission.CAMERA" />
```  
**Note:**如果你是通过Intent来使用Camera，你的应用程序则不需要请求这个权限。

* **Camera Features** - 你的应用还必须声明使用相机功能，例如：  
```xml
<uses-feature android:name="android.hardware.camera" />
```  
关于相机功能列表，请参考[功能引用](http://developer.android.com/guide/topics/manifest/uses-feature-element.html#hw-features)。增加相机功能到你的mainfest文件，这样Google Play可以阻止那些没有相机硬件或者没有相机特定功能的设备安装你的应用。关于Google Play如何做过滤的信息，请参考[Google Play and Feature-Based Filtering](http://developer.android.com/guide/topics/manifest/uses-feature-element.html#market-feature-filtering)。  
你还可以为每个相机特性设置`android:required`的属性，表示这个功能是否为必须的。

* **Storage Permission** - 如果你的应用需要保存图片或者视频到设备的外置存储空间(SD card)上，你也需要在manifest中指定存取权限。  
```xml
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
```

* **Audio Recording Permission** - 为了录制音频或者视频，你的程序必须请求audio capture permission。  
```xml
<uses-permission android:name="android.permission.RECORD_AUDIO" />
```
* **Location Permission** - 如果你的应用需要为图片添加位置信息，你还需要请求location permission:  
```xml
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
```  
关于获取用户位置信息的更多细节，请参考[Location Strategies](http://developer.android.com/guide/topics/location/strategies.html).

## 3)Using Existing Camera Apps使用现有的相机应用
在你的应用中快速A quick way to enable taking pictures or videos in your application without a lot of extra code is to use an Intent to invoke an existing Android camera application. A camera intent makes a request to capture a picture or video clip through an existing camera app and then returns control back to your application. This section shows you how to capture an image or video using this technique.

## 3)dispatchTouchEvent()流程图
![dispatchtouchevent_process.jpg](/images/articles/dispatchtouchevent_process.jpg "演示ViewGroup中的dispatchTouchEvent的流程")

## 4)代码举例说明
![dispatchtouchevent_demo.jpg](/images/articles/dispatchtouchevent_demo.jpg "Demo的layout层级")

[Demo Source Code](https://github.com/kesenhoo/TouchEventDemo.git)下面是截取的片段
```java
@Override
public boolean dispatchTouchEvent(MotionEvent ev) {
    switch (ev.getAction()) {
        case MotionEvent.ACTION_DOWN:
            Log.d(TAG, "[dispatchTouchEvent] -> ACTION_DOWN");
            break;
            //Log.i(TAG, "[dispatchTouchEvent] -> ACTION_DOWN, return true");
            //return true;
        case MotionEvent.ACTION_MOVE:
            Log.d(TAG, "[dispatchTouchEvent] -> ACTION_MOVE");
            break;
            //Log.i(TAG, "[dispatchTouchEvent] -> ACTION_MOVE, return true");
            //return true;
        case MotionEvent.ACTION_UP:
            Log.d(TAG, "[dispatchTouchEvent] -> ACTION_UP");
            break;
        case MotionEvent.ACTION_CANCEL:
            Log.d(TAG, "[dispatchTouchEvent] -> ACTION_CANCEL");
            break;
        default:
            break;
    }
    boolean superReturn = super.dispatchTouchEvent(ev);
    Log.i(TAG, "[dispatchTouchEvent] return super. = " + superReturn);
    return superReturn;
}
```
下面演示的每一种情况，操作均为点击中间的Button，然后松开。请仔细看下面的案例，里面均有对应的解释。

### Case 0:没有任何的分发丢弃，也没有任何的拦截
![Touch_Case_0_All_Normal.png](/images/articles/Touch_Case_0_All_Normal.png "CASE 0")

### Case 1:Activity层的dispatch函数对ACTION_DOWN进行return true.
![Touch_Case_1_Activity_Dispatch_Down_Return_True.png](/images/articles/Touch_Case_1_Activity_Dispatch_Down_Return_True.png "CASE 1")

### Case 2:ParentLayout层的dispatch函数对ACTION_DOWN进行return true.
![Touch_Case_2_Parent_Dispatch_Down_Return_True.png](/images/articles/Touch_Case_2_Parent_Dispatch_Down_Return_True.png "CASE 2")

因为ChildLayout层的dispatch函数对ACITON_DOWN进行return true和在activity层,ParentLayout层是类似的逻辑，因为都没有找到Target组件，又没有拦截的因素影响，所以后续的MOVE与UP都只传递到DOWN被return true的那一层截至，然后都回传，也都没有被消费掉。(注意发生在ChildLayout层的return true与ParentLayout层的差异在于：回传时，只有return层与activity层才可以接收到onTouchEvent()的回调，但是默认都无法消费)。

### Case 3:Activity层的dispatch函数对ACTION_MOVE进行return true.
![Touch_Case_2_Activity_Dispatch_Move_Return_True.png](/images/articles/Touch_Case_3_Activity_Dispatch_Move_Return_True.png "CASE 3")

因为ParentLayout层的dispatch函数对ACITON_MOVE进行return true和在activity层是类似的道理，不做新的分析

### Case 4:ParentLayout层的intercept函数对ACTION_DOWN进行return true.
![Touch_Case_4_Parent_Intercept_Down_Return_True.png](/images/articles/Touch_Case_4_Parent_Intercept_Down_Return_True.png "CASE 4")

### Case 5:ChildLayout层的intercept函数对ACTION_DOWN进行return true.
![Touch_Case_5_Child_Intercept_Down_Return_True.png](/images/articles/Touch_Case_5_Child_Intercept_Down_Return_True.png "CASE 5")

### Case 6:ChildLayout层的intercept函数对ACTION_MOVE进行return true.
![Touch_Case_6_Child_Intercept_Move_Return_True.png](/images/articles/Touch_Case_6_Child_Intercept_Move_Return_True.png "CASE 6")

## 5)写在最后
- 对于dispatch分发某个事件的情况：
	- 如果是ACTION_DOWN被return true,那么在哪一层return的，后续的MOVE与UP都只传递到该层，然后回传(**Case 1,2**)(注意在回传的过程中只有在return层与activity层才会触发onTouchEvent，中间若是有其他层，均会被跳过。这一规律暂时没有找到比较有力的解释，需要查看更多的源码。)
	- 如果非ACTION_DOWN被return true,意味着DOWN事件正常被下发并找到Target组件，那么后续只有被return的事件会无法正常下发，并只传递到return层，没有return的事件还能够正常下方到Target组件并被Target消费。(**Case 3**)
- 对于intercept拦截某个事件的情况：
	- 如果ACTION_DOWN被拦截，无论拦截发生在哪一层，都会导致Target组件都无法找到，那么后续的MOVE与UP事件都只在Activity层处理，不会下发(**Case 4,5**)。
	- 如果ACTION_DOWN没被拦截，此时可以找到Target组件，DOWN事件是正常被消费。后续的MOVE如果被拦截，会对子组件触发CANCEL的事件，并且UP事件只能传递到拦截MOVE的那一层，无法消费并返回(**Case 6**)。Ps:因为Case 6演示的是在ChildLayout层对MOVE进行拦截，所以看到的效果是Button直接收到了CANCEL,实际上如果是ParentLayout对MOVE进行拦截，那么CANCEL事件需要经过ChildLayout(如果有需要的话，可以在这里继续拦截CANCEL)，最终CANCEL事件都是由Button进行消费。

**经过上面的描述，对Android的Touch事件传递机制应该有更深入的了解，理解错误或者有偏差的地方，欢迎提出一起讨论，谢谢！**

