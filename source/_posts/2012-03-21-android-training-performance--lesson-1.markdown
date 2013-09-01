---
layout: post
title: "Android Training Performance - 提升布局文件的性能(Lesson 4 - 使用ViewHolder来提升ListView的性能)"
date: 2012-03-21 21:20
comments: true
sidebar: false
categories: Android Android:Training Android:Training:Performance
---

## Making ListView Scrolling Smooth
使得滚动ListView平滑的关键在与保持APP的UI thread与复杂的操作隔离。确保另起一个Thread来处理Disk IO，network access或者SQL access.
为了测试AP的状态，可以enable [StrictMode](http://developer.android.com/reference/android/os/StrictMode.html).(Android ICS 4.0上已经默认开启了StrickMode)

<!-- More -->

## 1)Use a Background Thread
使用后台线程，这样可以使得UI线程可以专注于描绘UI。大多数时候，AsycnTask实现了一种简单把需要做的事情与main thread隔离的方法。下面是一个例子：
```java
// Using an AsyncTask to load the slow images in a background thread  
new AsyncTask<ViewHolder, Void, Bitmap>() {  
    private ViewHolder v;  
  
    @Override  
    protected Bitmap doInBackground(ViewHolder... params) {  
        v = params[0];  
        return mFakeImageLoader.getImage();  
    }  
  
    @Override  
    protected void onPostExecute(Bitmap result) {  
        super.onPostExecute(result);  
        if (v.position == position) {  
            // If this item hasn't been recycled already, hide the  
            // progress and set and show the image  
            v.progress.setVisibility(View.GONE);  
            v.icon.setVisibility(View.VISIBLE);  
            v.icon.setImageBitmap(result);  
        }  
    }  
}.execute(holder); 
``` 
从Android 3.0开始，对于AsyncTask有个额外的特色：在多核处理器的情况下，我们可以使用[executeOnExecutor()](http://developer.android.com/reference/android/os/AsyncTask.html#executeOnExecutor(java.util.concurrent.Executor, Params...)) 来替代execute()，这样系统会根据当前设备的内核数量同时进行多个任务。

## 2)Hold View Objects in a View Holder
你的程序在滚动ListView的时候也许会重复频繁的call findViewById()，这样会降低性能。尽管Adapter会因为View的循环机制返回一个创建好的View。你仍然需要查找到这些组件并更新它，避免这样的重复，我们可以使用ViewHolder的设计模式。

A ViewHolder对象存放每一个View组件于Layout的tag属性中，因此我们可以立即访问tag中的组件从而避免重复call findViewById()。下面是定义了一个ViewHolder的例子：
```java
static class ViewHolder {  
  TextView text;  
  TextView timestamp;  
  ImageView icon;  
  ProgressBar progress;  
  int position;  
}
```  
这样之后，我们可以填充这个ViewHolder，并且保存到tag field.
```java
ViewHolder holder = new ViewHolder();  
holder.icon = (ImageView) convertView.findViewById(R.id.listitem_image);  
holder.text = (TextView) convertView.findViewById(R.id.listitem_text);  
holder.timestamp = (TextView) convertView.findViewById(R.id.listitem_timestamp);  
holder.progress = (ProgressBar) convertView.findViewById(R.id.progress_spinner);  
convertView.setTag(holder);  
```
那么我们就可以直接访问里面的数据了，省去了重复查询，提升了性能。

***
**文章学习自[http://developer.android.com/training/improving-layouts/smooth-scrolling.html
](http://developer.android.com/training/improving-layouts/smooth-scrolling.html)**
**转载请注明出自[http://kesenhoo.github.com](http:://kesenhoo.github.com)，谢谢**

