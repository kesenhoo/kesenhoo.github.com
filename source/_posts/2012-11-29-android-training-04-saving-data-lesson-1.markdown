---
layout: post
title: "Android Training 04 - 保存数据(Lesson 1 - 保存Key-Value)"
date: 2012-11-29 13:59
comments: true
sidebar: false
categories: Android Android:Training
---

**章节概要：**

虽然可以在onPause()的时候保存一些信息以免用户的使用进度被丢失，但是大多数Android app仍然是需要做保存数据的动作。大多数比较好的apps都需要保存用户的设置信息，而且有一些apps必须维护大量的文件信息与DB信息。这一章节会介绍给你在Android中一些重要的数据存储方法，例如：

* 以key-value的方式保存一些简单的数据到shared preferences文件中
* 在Android文件系统中保存任意格式的文件
* 通过SQLite来使用DB

<!-- more -->

# Lesson 1: Saving Key-Value Sets
如果你有一个相对较小的key-value集合需要保存，你应该使用[SharedPreferences](http://developer.android.com/reference/android/content/SharedPreferences.html) APIs。 SharedPreferences 对象指向了一个保存key-value pairs的文件，并且它提供了简单的方法来读写这个文件。每一个 SharedPreferences 文件都是由framework管理的并且可以是私有或者可分享的。  
这节课会演示如何使用 SharedPreferences APIs 来存储与检索简单的数据。  
**Note:** SharedPreferences APIs 仅仅提供了读写key-value对的功能，请不要与 Preference APIs相混淆。后者可以帮助你建立一个设置用户配置的页面（尽管它实际上是使用SharedPreferences 来实现保存用户配置的)。如果想了解更多关于Preference APIs的信息，请参考Settings 指南。

## Get a Handle to a SharedPreferences [获取SharedPreference的Handle]
你可以通过下面两个方法之一来创建或者访问shared preference 文件:

* **getSharedPreferences()** — 如果你需要多个通过名称参数来区分的shared preference文件, 名称可以通过第一个参数来指定。你可以在你的app里面通过任何一个Context 来执行这个方法。
* **getPreferences()** — 当你的activity仅仅需要一个shared preference文件时。因为这个方法会检索activitiy下的默认的shared preference文件，并不需要提供文件名称。

例如：下面的示例是在 Fragment 中被执行的，它会访问名为 R.string.preference_file_key 的shared preference文件，并使用private模式来打开它，这样的话，此时文件就仅仅可以被你的app访问了。
```java
Context context = getActivity();
SharedPreferences sharedPref = context.getSharedPreferences(
        getString(R.string.preference_file_key), Context.MODE_PRIVATE);
```
当命名你的shared preference文件时，你应该像 "com.example.myapp.PREFERENCE_FILE_KEY" 这样来命名。

当然，如果你的activity仅仅需要一个shared preference文件时，你可以使用[getPreferences()](http://developer.android.com/reference/android/app/Activity.html#getPreferences(int))方法：
```java
SharedPreferences sharedPref = getActivity().getPreferences(Context.MODE_PRIVATE);
```
**Caution:** 如果你创建了一个[MODE_WORLD_READABLE](http://developer.android.com/reference/android/content/Context.html#MODE_WORLD_READABLE)或者[MODE_WORLD_WRITEABLE](http://developer.android.com/reference/android/content/Context.html#MODE_WORLD_WRITEABLE) 模式的shared preference文件，那么任何其他的app只要知道文件名，则可以访问这个文件。

## Write to Shared Preferences[写Shared Preference]
为了写shared preferences文件，需要通过执行 edit() 来创建一个 SharedPreferences.Editor。

通过类似 putInt() 与 putString()方法来传递keys与values。然后执行 commit() 来提交改变. (*后来有建议除非是出于线程同步的需要，否则请使用apply()方法来替代commit()，因为后者有可能会卡到UI Thread.*)
```java
SharedPreferences sharedPref = getActivity().getPreferences(Context.MODE_PRIVATE);
SharedPreferences.Editor editor = sharedPref.edit();
editor.putInt(getString(R.string.saved_high_score), newHighScore);
editor.commit();
```

## Read from Shared Preferences[读Shared Preference]
为了从shared preference中检索读取数据，可以通过类似 getInt() 与 getString()等方法来读取。在那些方法里面传递你想要获取value对应的key，并且提供一个默认的value。如下：
```java
SharedPreferences sharedPref = getActivity().getPreferences(Context.MODE_PRIVATE);
long default = getResources().getInteger(R.string.saved_high_score_default));
long highScore = sharedPref.getInt(getString(R.string.saved_high_score), default);
```

*********************************
**学习自：[http://developer.android.com/training/basics/data-storage/shared-preferences.html](http://developer.android.com/training/basics/data-storage/shared-preferences.html)，请多指教，谢谢！**  
**转载请注明出自[http://kesenhoo.github.com](http://kesenhoo.github.com)，谢谢配合！**






