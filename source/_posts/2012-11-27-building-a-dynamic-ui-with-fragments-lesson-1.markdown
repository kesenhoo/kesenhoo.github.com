---
layout: post
title: "Android Training 03 - 使用Fragments建立动态的UI[Lesson 1 - 使用Support Library]"
date: 2012-11-27 15:08
comments: true
sidebar: false
categories: Android Android:Training
---

# Using the Support Library

* Android Support Library 提供了一个包含了API库的JAR 文件，它可以允许你在你的app在更老的Android平台上使用一些比较新的API。例如，它提供了一些fragment的API，这样你可以在1.6或者更高的平台上使用fragment。
* 这节课会演示如何使用fragment来建立一个动态的app UI。

### 使用Support Library来建立你的Project
* 使用SDK Manager来下载Android Support Library。
* 在Android项目的根目录下创建 libs的目录。

<!-- more -->

* 把刚才下载的jar文件复制到libs目录下。下载好的jar文件在 <sdk>/extras/android/support/v4/android-support-v4.jar.
* 更新你的manifest文件，设置最低API level 为4，并且设置目标API为最新的Level，如下
```xml
 <uses-sdk android:minSdkVersion="4" android:targetSdkVersion="15" />
```
### 导入Support Library APIs
* Support Library 包含了一些最新的系统上的API，这样你就可以在那些没有最新API的平台上开发特定的功能。你可以在android.support.v4.*.下寻找所有关于这个support library的文档解释。

* 为了确保在旧的平台上可以使用新平台上的API，请在代码里面做导入的动作，像下面一样：
```java
import android.support.v4.app.Fragment;
import android.support.v4.app.FragmentManager;
...
```
* 当创建一个actvity的时候，如果这个activity包含了fragment，那么这个activity应该继承自[FragmentActivity](http://developer.android.com/reference/android/support/v4/app/FragmentActivity.html)而不是通常的 Activity。你在下一节课会看到fragment与activity的sample code。


*********************************
**学习自：[http://developer.android.com/training/basics/fragments/support-lib.html](http://developer.android.com/training/basics/fragments/support-lib.html)，请多指教，谢谢！**  
**转载请注明出自[http://kesenhoo.github.com](http://kesenhoo.github.com)，谢谢配合！**






