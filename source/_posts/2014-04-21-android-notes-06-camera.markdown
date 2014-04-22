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

## 3)Using Existing Camera Apps
在你的应用中快速的实现拍照与录制视频的方法是使用一个Intent来调用已经存在系统中的相机程序。通过已经存在的相机程序拍照或者录制视频，然后返回数据给请求方。这一部分会演示如何使用这种技术。

触发Camera Intent需要遵守如下几个步骤：

* **Compose a Camera Intent** - 创建一个请求拍照或者录像的Intent，使用下面的intent类型：
	* [MediaStore.ACTION_IMAGE_CAPTURE](http://developer.android.com/reference/android/provider/MediaStore.html#ACTION_IMAGE_CAPTURE) - 请求拍照的Intent。
	* [MediaStore.ACTION_VIDEO_CAPTURE](http://developer.android.com/reference/android/provider/MediaStore.html#ACTION_VIDEO_CAPTURE) - 请求录像的Intent。
	
* **Start the Camera Intent** - 使用[startActivityForResult()](http://developer.android.com/reference/android/app/Activity.html#startActivityForResult(android.content.Intent, int))方法来执行这个Intent。在启动这个Intent之后，相机程序会被唤起并提供拍照或者录像的功能。

* **Receive the Intent Result** - 在你的程序里面实现[onActivityResult()](http://developer.android.com/reference/android/app/Activity.html#onActivityResult(int, int, android.content.Intent))的方法用来接收相机程序返回的数据。当用户结束拍照或者录像之后，系统会调用到这个方法。

### 3.1)Image capture intent
使用Camera Intent是一种使用最少的代码为你的程序开启拍照功能的一种简便的方法。一个拍照程序可以包含下面的附加信息：

**[MediaStore.EXTRA_OUTPUT](http://developer.android.com/reference/android/provider/MediaStore.html#EXTRA_OUTPUT)** - 这定义了一个Uri对象来指定存放图片的路径与文件名。这个设置信息是可选的，但是强烈建议添加。如果你不指定这个值，相机程序会使用默认的文件名保存图片到默认的位置，这个值可以从Intent.getData()的字段中获取到。

下面的示例代码演示了如何构建一个拍照Intent并执行它。`getOutputMediaFileUri()`方法可以从**Saving Media Files**的段落中涉及到。

```java
private static final int CAPTURE_IMAGE_ACTIVITY_REQUEST_CODE = 100;
private Uri fileUri;

@Override
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.main);

    // create Intent to take a picture and return control to the calling application
    Intent intent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);

    fileUri = getOutputMediaFileUri(MEDIA_TYPE_IMAGE); // create a file to save the image
    intent.putExtra(MediaStore.EXTRA_OUTPUT, fileUri); // set the image file name

    // start the image capture Intent
    startActivityForResult(intent, CAPTURE_IMAGE_ACTIVITY_REQUEST_CODE);
}
```

当startActivityForResult()方法被执行，用户会看到一个相机拍照的界面。用户执行了拍照(或者取消操作)，用户界面会回退到你的程序，你必须在onActivityResult()方法里面接收返回的数据。关于如何接受完整的intent，可以参考下面的**Receiving camera intent result**段落。

### 3.2)Video capture intent
视频录制的原理和拍照一致。一个视频录制的Intent可以包含如下的参数信息：

* [MediaStore.EXTRA_OUTPUT](http://developer.android.com/reference/android/provider/MediaStore.html#EXTRA_OUTPUT) - 和拍照类似，这里指定保存视频的位置。同样这个字段是可选的，但是也被强烈建议进行填写。如果没有传递这个参数，相机程序会使用默认的文件名保存文件到默认的存储位置。你可以通过在返回的Intent.getData()字段中获取到这个值。
* [MediaStore.EXTRA_VIDEO_QUALITY](http://developer.android.com/reference/android/provider/MediaStore.html#EXTRA_VIDEO_QUALITY) - 这里的值可以为0或者1，分别表示低质量与高质量。
* [MediaStore.EXTRA_DURATION_LIMIT](http://developer.android.com/reference/android/provider/MediaStore.html#EXTRA_DURATION_LIMIT) - 设置这个值用来限制视频的长度，用毫秒计算。
* [MediaStore.EXTRA_SIZE_LIMIT](http://developer.android.com/reference/android/provider/MediaStore.html#EXTRA_SIZE_LIMIT) - 设置这个值用来限制文件的大小，用btye做单位。

下面演示了如何构建一个Video Intent并执行：
```java
private static final int CAPTURE_VIDEO_ACTIVITY_REQUEST_CODE = 200;
private Uri fileUri;

@Override
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.main);

    //create new Intent
    Intent intent = new Intent(MediaStore.ACTION_VIDEO_CAPTURE);

    fileUri = getOutputMediaFileUri(MEDIA_TYPE_VIDEO);  // create a file to save the video
    intent.putExtra(MediaStore.EXTRA_OUTPUT, fileUri);  // set the image file name

    intent.putExtra(MediaStore.EXTRA_VIDEO_QUALITY, 1); // set the video image quality to high

    // start the Video Capture Intent
    startActivityForResult(intent, CAPTURE_VIDEO_ACTIVITY_REQUEST_CODE);
}
```
和拍照类似，也需要在activity的onActivityResult里面去接收数据并做处理。

### 3.3)Receiving camera intent result
一旦你构建并执行了一个拍照或者录像的Intent，你的程序必须确保能够正确接收返回的数据。为了正确的接收到Intent，你必须重写onActivityResult()的方法，下面会演示如何获取到上面示例代码返回的数据。

```java
private static final int CAPTURE_IMAGE_ACTIVITY_REQUEST_CODE = 100;
private static final int CAPTURE_VIDEO_ACTIVITY_REQUEST_CODE = 200;

@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    if (requestCode == CAPTURE_IMAGE_ACTIVITY_REQUEST_CODE) {
        if (resultCode == RESULT_OK) {
            // Image captured and saved to fileUri specified in the Intent
            Toast.makeText(this, "Image saved to:\n" +
                     data.getData(), Toast.LENGTH_LONG).show();
        } else if (resultCode == RESULT_CANCELED) {
            // User cancelled the image capture
        } else {
            // Image capture failed, advise user
        }
    }

    if (requestCode == CAPTURE_VIDEO_ACTIVITY_REQUEST_CODE) {
        if (resultCode == RESULT_OK) {
            // Video captured and saved to fileUri specified in the Intent
            Toast.makeText(this, "Video saved to:\n" +
                     data.getData(), Toast.LENGTH_LONG).show();
        } else if (resultCode == RESULT_CANCELED) {
            // User cancelled the video capture
        } else {
            // Video capture failed, advise user
        }
    }
}
```

一旦你的activity成功接收了数据，那么你的程序就可以在指定的位置获取到图片或者视频了。

## 4)Building a Camera App
一些开发者也许需要开发一个定制的相机应用，用来提供特殊的功能与体验。创建一个定制的相机界面比起使用Intent需要更多的代码，但是它能够提供一种更加优秀的用户体验。

通常来说创建一个定制化的相机界面有如下几个步骤：

* **Detect and Access Camera** - 检查相机是否存在并可访问。
* **Create a Preview Class** - 创建一个继承自SurfaceView的preview类，并implement SurfaceHolder的接口的interface。这个类用来预览相机的动态图片。
* **Build a Preview Layout** - 一旦你拥有了preview class。创建一个Layout用来承载preview并提供交互控制界面。
* **Setup Listeners for Capture** - 为控制界面建立监听器，用来启动拍照或者录像。
* **Capture and Save Files** - 建立拍照录像的代码并进行保存。
* **Release the Camera** - 使用完相机之后，你的程序必须正确的释放它，以便其他程序使用。

相机硬件是一个共享资源，它必须被小心谨慎的管理使用。因此你的程序不应该和其他可能使用相机硬件的程序有冲突。下面的段落会介绍如何检测相机硬件，如何请求获取权限，如何拍照录像以及如何在使用完毕时释放相机。

**注意:** 当你的程序执行完任务之后，需要记得通过执行Camera.release()来释放相机对象。如果你的相机没有合理的释放相机，后续包括你自己的应用在内的所有的相机应用，都将无法正常打开相机并且可能导致程序崩溃。

### 4.1)Detecting camera hardware
如果你的程序没有在manifest中声明需要使用相机，你应该在运行时去检查相机是否可用。为了执行这个检查，需要使用到[PackageManager.hasSystemFeature()](http://developer.android.com/reference/android/content/pm/PackageManager.html#hasSystemFeature(java.lang.String)) 方法，如下所示：

```java
/** Check if this device has a camera */
private boolean checkCameraHardware(Context context) {
    if (context.getPackageManager().hasSystemFeature(PackageManager.FEATURE_CAMERA)){
        // this device has a camera
        return true;
    } else {
        // no camera on this device
        return false;
    }
}
```

Android设备可以拥有多个摄像头，例如前置与后置摄像头。从Android 2.3 (API Level 9)开始，可以通过[Camera.getNumberOfCameras()](http://developer.android.com/reference/android/hardware/Camera.html#getNumberOfCameras())方法获取到摄像头的个数。

### 4.2)Accessing cameras
如果你已经判断到程序运行的设备上有摄像头，你需要获取到摄像头的话，需要通过一个相机实例来进行访问。

为了访问到主摄像头，如下所示，使用[Camera.open()](http://developer.android.com/reference/android/hardware/Camera.html#open())方法。
```java
/** A safe way to get an instance of the Camera object. */
public static Camera getCameraInstance(){
    Camera c = null;
    try {
        c = Camera.open(); // attempt to get a Camera instance
    }
    catch (Exception e){
        // Camera is not available (in use or does not exist)
    }
    return c; // returns null if camera is unavailable
}
```

**注意:** 当使用Camera.open方法时总是需要做检查exceptions的动作。如果没有检查exception，有可能会因为相机正在使用或者相机不存在而使得程序崩溃。

在Android 2.3 (API Level 9)开始, 你可以使用通过[Camera.open(int)](http://developer.android.com/reference/android/hardware/Camera.html#open(int))方法来访问特定的摄像头。上面演示的代码会优先获取主摄像头。

### 4.3)Checking camera features
一旦你获取到相机，你可以使用[Camera.getParameters()](http://developer.android.com/reference/android/hardware/Camera.html#getParameters())方法来获取到更多的相机信息。也可以通过获取到的相机参数对象得到相机能够支持的功能。从android 2.3开始，使用[Camera.getCameraInfo()](http://developer.android.com/reference/android/hardware/Camera.html#getCameraInfo(int, android.hardware.Camera.CameraInfo))可以获取到相机是前置还是后置摄像头以及拍摄出来的图片角度。

### 4.4)Creating a preview class
为了给用户提供有效的拍照与录像体验，用户需要能够看到摄像头捕获的数据。相机预览是使用SurfaceView，它能够显示来自摄像头的数据，因此用户可以分割捕获图片或者视频。

The following example code demonstrates how to create a basic camera preview class that can be included in a View layout. This class implements SurfaceHolder.Callback in order to capture the callback events for creating and destroying the view, which are needed for assigning the camera preview input.


## 5)代码举例说明
![dispatchtouchevent_demo.jpg](/images/articles/dispatchtouchevent_demo.jpg "Demo的layout层级")

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

