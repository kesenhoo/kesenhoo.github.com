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

下面的示例代码演示了如何创建一个基础的相机预览类，该类可以included到另外一个layout中。为了捕获拍照事件的回调，需要implement[SurfaceHolder.Callback](http://developer.android.com/reference/android/view/SurfaceHolder.Callback.html) ，之后可以在这些回调里面进行创建与销毁View的操作。

```java
/** A basic Camera preview class */
public class CameraPreview extends SurfaceView implements SurfaceHolder.Callback {
    private SurfaceHolder mHolder;
    private Camera mCamera;

    public CameraPreview(Context context, Camera camera) {
        super(context);
        mCamera = camera;

        // Install a SurfaceHolder.Callback so we get notified when the
        // underlying surface is created and destroyed.
        mHolder = getHolder();
        mHolder.addCallback(this);
        // deprecated setting, but required on Android versions prior to 3.0
        mHolder.setType(SurfaceHolder.SURFACE_TYPE_PUSH_BUFFERS);
    }

    public void surfaceCreated(SurfaceHolder holder) {
        // The Surface has been created, now tell the camera where to draw the preview.
        try {
            mCamera.setPreviewDisplay(holder);
            mCamera.startPreview();
        } catch (IOException e) {
            Log.d(TAG, "Error setting camera preview: " + e.getMessage());
        }
    }

    public void surfaceDestroyed(SurfaceHolder holder) {
        // empty. Take care of releasing the Camera preview in your activity.
    }

    public void surfaceChanged(SurfaceHolder holder, int format, int w, int h) {
        // If your preview can change or rotate, take care of those events here.
        // Make sure to stop the preview before resizing or reformatting it.

        if (mHolder.getSurface() == null){
          // preview surface does not exist
          return;
        }

        // stop preview before making changes
        try {
            mCamera.stopPreview();
        } catch (Exception e){
          // ignore: tried to stop a non-existent preview
        }

        // set preview size and make any resize, rotate or
        // reformatting changes here

        // start preview with new settings
        try {
            mCamera.setPreviewDisplay(mHolder);
            mCamera.startPreview();

        } catch (Exception e){
            Log.d(TAG, "Error starting camera preview: " + e.getMessage());
        }
    }
}
```

如果你想为你的相机预览界面设置特定的预览大小，可以在`surfaceChanged()`的回调里面进行操作(注意上面演示代码的注释)。设置预览大小时，你**必须**使用从[getSupportedPreviewSizes()](http://developer.android.com/reference/android/hardware/Camera.Parameters.html#getSupportedPreviewSizes())方法获取到的预览值，不能在[setPreviewSize()](http://developer.android.com/reference/android/hardware/Camera.Parameters.html#setPreviewSize(int, int))方法里设置随意的预览值。

### 4.5)Placing preview in a layout
在上一段落演示的Camera Preview Class,必须放置在一个activity的layout中。这一段落会演示为了预览如何创建一个基础的layout与activity。

下面的代码提供了一个能够显示相机预览界面的基础layout。在这段代码中，FrameLayout是相机预览类的container。使用framelayout可以在相机预览界面上叠加额外的图片信息或者是操作控制组件。

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="horizontal"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    >
  <FrameLayout
    android:id="@+id/camera_preview"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:layout_weight="1"
    />

  <Button
    android:id="@+id/button_capture"
    android:text="Capture"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_gravity="center"
    />
</LinearLayout>
```

在大多数设备上，相机预览的角度默认是横屏的。演示的layout指定了horizontal，并且下面的代码使得activity固定成横屏的模式。

```xml
<activity android:name=".CameraActivity"
          android:label="@string/app_name"

          android:screenOrientation="landscape">
          <!-- configure this activity to use landscape orientation -->

          <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```

**Note:** 相机预览界面不一定是要横屏的。从android 2.2开始，你可以使用[setDisplayOrientation()](http://developer.android.com/reference/android/hardware/Camera.html#setDisplayOrientation(int))方法来设置预览图片的角度。为了在用户旋转手机时改变相机预览的角度，在`surfaceChanged()`方法里面，首先要使用[Camera.stopPreview()](http://developer.android.com/reference/android/hardware/Camera.html#stopPreview())停止预览，然后再使用[Camera.startPreview()](http://developer.android.com/reference/android/hardware/Camera.html#startPreview())方法来重新开启相机预览。

为了在activity中添加相机界面，你的Camera Activity必须确保在activity pause或者是destory的时候释放相机资源。下面的代码演示了如何添加camera preview class到camera activity中。

```java
public class CameraActivity extends Activity {

    private Camera mCamera;
    private CameraPreview mPreview;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);

        // Create an instance of Camera
        mCamera = getCameraInstance();

        // Create our Preview view and set it as the content of our activity.
        mPreview = new CameraPreview(this, mCamera);
        FrameLayout preview = (FrameLayout) findViewById(R.id.camera_preview);
        preview.addView(mPreview);
    }
}
```

**Note:** 上面演示的`getCameraInstance()`方法出现在4.2)Accessing camera段落中。

### 4.6)Capturing pictures
一旦你建立了preview class并且创建好了显示的layout。那么就可以开始做拍照的动作了。

为了获取到一张图片，需要使用[Camera.takePicture()](http://developer.android.com/reference/android/hardware/Camera.html#takePicture(android.hardware.Camera.ShutterCallback, android.hardware.Camera.PictureCallback, android.hardware.Camera.PictureCallback))方法。为了获取到JPEG格式的图片数据，你必须implement一个[Camera.PictureCallback](http://developer.android.com/reference/android/hardware/Camera.PictureCallback.html)接口来接收图片数据并把它写到文件中。

```java
private PictureCallback mPicture = new PictureCallback() {

    @Override
    public void onPictureTaken(byte[] data, Camera camera) {

        File pictureFile = getOutputMediaFile(MEDIA_TYPE_IMAGE);
        if (pictureFile == null){
            Log.d(TAG, "Error creating media file, check storage permissions: " +
                e.getMessage());
            return;
        }

        try {
            FileOutputStream fos = new FileOutputStream(pictureFile);
            fos.write(data);
            fos.close();
        } catch (FileNotFoundException e) {
            Log.d(TAG, "File not found: " + e.getMessage());
        } catch (IOException e) {
            Log.d(TAG, "Error accessing file: " + e.getMessage());
        }
    }
};
```

触发拍照的动作，需要使用下面演示到的方法。

```java
// Add a listener to the Capture button
Button captureButton = (Button) findViewById(id.button_capture);
captureButton.setOnClickListener(
    new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            // get an image from the camera
            mCamera.takePicture(null, null, mPicture);
        }
    }
);
```

### 4.7)Capturing videos
录制视频的内容，暂时跳过，下次再看。

### 4.8)Releasing the camera
相机是被共享的一种资源。获取到一个相机实例之后，程序才可以使用相机，但是，在使用完相机的时候，程序必须小心谨慎的释放它。在程序进入到pause状态时，立即释放相机资源。如果你的程序没有合理的释放相机资源，包括自己程序本身在内，后续所有的相机请求都将失败，甚至可能会导致程序崩溃。下面的代码演示了如何释放相机资源。

```java
public class CameraActivity extends Activity {
    private Camera mCamera;
    private SurfaceView mPreview;
    private MediaRecorder mMediaRecorder;

    ...

    @Override
    protected void onPause() {
        super.onPause();
        releaseMediaRecorder();       // if you are using MediaRecorder, release it first
        releaseCamera();              // release the camera immediately on pause event
    }

    private void releaseMediaRecorder(){
        if (mMediaRecorder != null) {
            mMediaRecorder.reset();   // clear recorder configuration
            mMediaRecorder.release(); // release the recorder object
            mMediaRecorder = null;
            mCamera.lock();           // lock camera for later use
        }
    }

    private void releaseCamera(){
        if (mCamera != null){
            mCamera.release();        // release the camera for other applications
            mCamera = null;
        }
    }
}
```

## 5)Saving Media Files
用户创建的图片或者视频均需要保存到设备的external storage目录下(SD Card)。可以有多种可能的位置用来保存文件，但是作为一个开发人员，只有下面两种标准的路径进行保存。

* [Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_PICTURES)](http://developer.android.com/reference/android/os/Environment.html#getExternalStoragePublicDirectory(java.lang.String)) - 这个方法会返回用来保存图片与视频所推荐使用的标准共享目录。这个目录是共享开放的，所以其他程序可以轻易的发现，读取并删除这个目录下的文件。如果你的程序被用户卸载，在这个目录下的文件不会被移除。为了避免干扰到用户已经存在的图片与视频目录，你应该为你的程序创建一个子目录。如下面的代码所示。这个方法从Android 2.2 (API Level 8)开始就可以使用。
* [Context.getExternalFilesDir(Environment.DIRECTORY_PICTURES)](http://developer.android.com/reference/android/content/Context.html#getExternalFilesDir(java.lang.String)) - 这个方法会返回一个和你的程序相关联的，用来保存图片与视频的，标准目录。如果你的程序被卸载，在这个目录下的文件也会被一起移除。这个目录并不能阻止其他程序的读写。

```java
public static final int MEDIA_TYPE_IMAGE = 1;
public static final int MEDIA_TYPE_VIDEO = 2;

/** Create a file Uri for saving an image or video */
private static Uri getOutputMediaFileUri(int type){
      return Uri.fromFile(getOutputMediaFile(type));
}

/** Create a File for saving an image or video */
private static File getOutputMediaFile(int type){
    // To be safe, you should check that the SDCard is mounted
    // using Environment.getExternalStorageState() before doing this.

    File mediaStorageDir = new File(Environment.getExternalStoragePublicDirectory(
              Environment.DIRECTORY_PICTURES), "MyCameraApp");
    // This location works best if you want the created images to be shared
    // between applications and persist after your app has been uninstalled.

    // Create the storage directory if it does not exist
    if (! mediaStorageDir.exists()){
        if (! mediaStorageDir.mkdirs()){
            Log.d("MyCameraApp", "failed to create directory");
            return null;
        }
    }

    // Create a media file name
    String timeStamp = new SimpleDateFormat("yyyyMMdd_HHmmss").format(new Date());
    File mediaFile;
    if (type == MEDIA_TYPE_IMAGE){
        mediaFile = new File(mediaStorageDir.getPath() + File.separator +
        "IMG_"+ timeStamp + ".jpg");
    } else if(type == MEDIA_TYPE_VIDEO) {
        mediaFile = new File(mediaStorageDir.getPath() + File.separator +
        "VID_"+ timeStamp + ".mp4");
    } else {
        return null;
    }
    return mediaFile;
}
```

## 6)Camera Features
Android提供了控制相机特性的方法，例如图片格式化，闪光灯模式，设置聚焦等等。这一段落列出了通常的相机功能并简短的介绍如何使用这些功能。大多数相机特性可以通过**Camera.Parameters**对象来获取并进行相关的设置。然而，有几个重要的功能不仅仅是通过**Camera.Parameters**能够实现的。请看下面的内容介绍：

* Metering and focus areas：测光并进行聚焦
* Face detection：人脸检测
* Time lapse video：延时视频

关于上面3个常用的功能会在下面进行更加详细的介绍，除此之外的其他相机功能，请参考下面这张表：

![camera_features_table.png](/images/articles/camera_features_table.png "Camera Common Features")

**Note:** 因为软硬件的差异性，那些功能并不一定都是支持的。对于检查功能是否可用，请参考下面的Checking feature availability.

### 6.1)Checking feature availability
相机的有些功能在所有手机上并不一定是都支持的。在开发相机应用时就需要提前考虑应该适配到哪个Level。然后开发的时候需要动态的去根据功能是否支持来做不同的处理。

你可以通过获取到相机参数的对象来做检测。下面的例子演示了如何检查autofocus功能是否可用：
```java
// get Camera parameters
Camera.Parameters params = mCamera.getParameters();

List<String> focusModes = params.getSupportedFocusModes();
if (focusModes.contains(Camera.Parameters.FOCUS_MODE_AUTO)) {
  // Autofocus mode is supported
}
```

对于大多数的相机特性，都可以使用类型上面的代码来处理。Camera.Parameters对象提供了一系列的类似`getSupported...()`, `is...Supported()` 与 `getMax...()` 方法来判断某个功能是否可用的。

如果你的程序确定需要相机的某个特性，你可以在mainfest文件中就进声明。例如你声明了flash与auto-focus的功能，那么Google Play会阻止那些不支持这些功能的设备安装这个应用。关于相机功能的声明列表，请参考[Features Reference.](http://developer.android.com/guide/topics/manifest/uses-feature-element.html#hw-features)

### 6.2)Using camera features
前面已经提到过，通过Camera.Parameters对象来操控相机。如下所示：
```java
// get Camera parameters
Camera.Parameters params = mCamera.getParameters();
// set the focus mode
params.setFocusMode(Camera.Parameters.FOCUS_MODE_AUTO);
// set Camera parameters
mCamera.setParameters(params);
```
上面这种方法对大多数相机功能都是可用的，在你获取到相机的实例之后，大多数参数都是在任意时间均可以修改的。参数修改的效果在相机预览的界面是可以立即看到效果的。在软件层面，实际上可能是需要花几帧的时间来产生效果的，因为需要发送指令给相机硬件产生效果。

**Important:** 部分功能是不能修改的。特别是，修改相机预览的角度与大小，是需要先停止预览的。从Android Android 4.0 (API Level 14)开始，预览角度可以不用重启预览就可以进行修改。

下面快速的介绍前面提到的3个功能：

#### 6.2.1）Metering and focus areas
在某些拍照场景下，自动聚焦与测光并不能拍出我们想要的效果。从Android 4.0 (API Level 14)开始，支持指定特定的区域用来focus与metering。下面的代码演示了如何指定特定的区域用来focus与metering。

```java
// Create an instance of Camera
mCamera = getCameraInstance();

// set Camera parameters
Camera.Parameters params = mCamera.getParameters();

if (params.getMaxNumMeteringAreas() > 0){ // check that metering areas are supported
    List<Camera.Area> meteringAreas = new ArrayList<Camera.Area>();

    Rect areaRect1 = new Rect(-100, -100, 100, 100);    // specify an area in center of image
    meteringAreas.add(new Camera.Area(areaRect1, 600)); // set weight to 60%
    Rect areaRect2 = new Rect(800, -1000, 1000, -800);  // specify an area in upper right of image
    meteringAreas.add(new Camera.Area(areaRect2, 400)); // set weight to 40%
    params.setMeteringAreas(meteringAreas);
}

mCamera.setParameters(params);
```
[Camera.Area](http://developer.android.com/reference/android/hardware/Camera.Area.html)对象包含了2个参数: 一个Rect对象用来指定区域，另外一个是权重用来告诉相机这部分区域的重要性。

Rect的值是基于2000*2000的坐标体系。(-1000, -1000)表示的左上角，(1000, 1000)表示的是右下角。如下图所示：

![camera-area-coordinates.png](/images/articles/camera-area-coordinates.png "Figure 1. The red lines illustrate the coordinate system for specifying a Camera.Area within a camera preview. The blue box shows the location and shape of an camera area with the Rect values 333,333,667,667.")

这个坐标系统是于相机的可见预览区域相吻合的，不会因为预览效果的放大缩小二改变。同样的，通过Camera.setDisplayOrientation()旋转预览并不会导致坐标系统的变化。

#### 6.2.2)Face detection
对于包含人像的图片，头像是最重要的区域。在拍照的时候，这个区域应该用来作为focus与while balance操作的重要参数。从Android 4.0 (API Level 14)开始，系统能够提供API来动态检测人脸并且可以使用人脸识别技术来计算拍照图片的设置参数。

**Note:** 当人脸检测功能开启的时候，`setWhiteBalance(String)`, `setFocusAreas(List)` 与` setMeteringAreas(List)`并没有效果。

使用人脸检测功能需要下面的几个步骤：

* 检测在设备上是否有人脸检测的功能。
* 创建一个人脸检测的监听器。
* 把这个人脸检测器添加到Camera对象中。
* 在预览开启之后启动人脸检测(每次预览重启也是)。


通过[getMaxNumDetectedFaces()](http://developer.android.com/reference/android/hardware/Camera.Parameters.html#getMaxNumDetectedFaces())方法来获取设备是否支持检测人脸。如下面的`startFaceDetection()`方法中演示的：


为了把人脸检测到得结果反馈到界面上，还需要实现检测的监听器，如下所示：
```java
class MyFaceDetectionListener implements Camera.FaceDetectionListener {

    @Override
    public void onFaceDetection(Face[] faces, Camera camera) {
        if (faces.length > 0){
            Log.d("FaceDetection", "face detected: "+ faces.length +
                    " Face 1 Location X: " + faces[0].rect.centerX() +
                    "Y: " + faces[0].rect.centerY() );
        }
    }
}
```

创建完这个类之后，需要把这个Listener加到Camera的对象中，如下所示：
```java
mCamera.setFaceDetectionListener(new MyFaceDetectionListener());
```

你得程序必须在每次启动或者重启相机预览的时候开始人脸检测。如下所示：
```java
public void startFaceDetection(){
    // Try starting Face Detection
    Camera.Parameters params = mCamera.getParameters();

    // start face detection only *after* preview has started
    if (params.getMaxNumDetectedFaces() > 0){
        // camera supports face detection, so can start it:
        mCamera.startFaceDetection();
    }
}
```

如果使用前面4.4)Creating a preview class段落中提到的预览类，你需要在`surfaceCreated()` 与 `surfaceChanged()`方法中执行`startFaceDetection()`方法。如下所示：
```java
public void surfaceCreated(SurfaceHolder holder) {
    try {
        mCamera.setPreviewDisplay(holder);
        mCamera.startPreview();

        startFaceDetection(); // start face detection feature

    } catch (IOException e) {
        Log.d(TAG, "Error setting camera preview: " + e.getMessage());
    }
}

public void surfaceChanged(SurfaceHolder holder, int format, int w, int h) {

    if (mHolder.getSurface() == null){
        // preview surface does not exist
        Log.d(TAG, "mHolder.getSurface() == null");
        return;
    }

    try {
        mCamera.stopPreview();

    } catch (Exception e){
        // ignore: tried to stop a non-existent preview
        Log.d(TAG, "Error stopping camera preview: " + e.getMessage());
    }

    try {
        mCamera.setPreviewDisplay(mHolder);
        mCamera.startPreview();

        startFaceDetection(); // re-start face detection feature

    } catch (Exception e){
        // ignore: tried to stop a non-existent preview
        Log.d(TAG, "Error starting camera preview: " + e.getMessage());
    }
}
```
**Note:** 要在startPreview()方法之后才能开启人脸检测。请不要尝试在onCreate()方法里面启动人脸检测。

#### 6.2.3)Time lapse video
Time lapse video使得用户可以通过组合一段时间的图片生成视频片段。这个功能利用了[MediaRecorder](http://developer.android.com/reference/android/media/MediaRecorder.html)每隔一段时间来记录一张图片。

为了实现这个功能，你需要像配置一个普通的recorder对象一样。如下面的代码所示：
```java
// Step 3: Set a CamcorderProfile (requires API Level 8 or higher)
mMediaRecorder.setProfile(CamcorderProfile.get(CamcorderProfile.QUALITY_TIME_LAPSE_HIGH));
...
// Step 5.5: Set the video capture rate to a low number
mMediaRecorder.setCaptureRate(0.1); // capture a frame every 10 seconds
```
上面的设置只是对MediaRecorder进行参数设置的一小部分。一旦配置结束，就可以开启视频录制。关于更多的信息，请参考前面的4.7)Capture Video(还没有开始学习)

====

学习自<http://developer.android.com/guide/topics/media/camera.html>







