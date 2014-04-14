---
layout: post
title: "Android Training(02) - 提升布局文件的性能(Lesson 2 - 使用include标签重用Layout)"
date: 2012-03-21 18:06
comments: true
sidebar: false
categories: Android Android:Training
---

## Re-using Layouts with <include/>
尽管Android提供了很多种小的组件可以重用，我们还需要自定义一些稍微复杂一点的小组件进行重用。我们可以使用`<include/>` 与 `<merge/>` 标签来对当前的layout嵌入一些其他的layout.

在创建一个稍微复杂一点的layout时，重用layout是个很给力的方法。比如我们需要一个YES/NO的控制栏，包含文字提示的Progress bar。像这种的布局会在很多地方需要重用到.

<!-- More -->

## 1)Create a Re-usable Layout
如果你已经知道哪些组件是会重用的，我们可以创建一个XML并且定义这个layout。例如：下面定义了一个需要在每个Activity都需要显示的titlebar.xml
```xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"  
    android:layout_width=”match_parent”  
    android:layout_height="wrap_content"  
    android:background="@color/titlebar_bg">  
  
    <ImageView android:layout_width="wrap_content"  
               android:layout_height="wrap_content"   
               android:src="@drawable/gafricalogo" />  
</FrameLayout>
```  

## 2)Use the <include> Tag
下面示例了一个包含了titlebar控件的布局：
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"  
    android:orientation="vertical"   
    android:layout_width=”match_parent”  
    android:layout_height=”match_parent”  
    android:background="@color/app_bg"  
    android:gravity="center_horizontal">  
  
    <include layout="@layout/titlebar"/>  
  
    <TextView android:layout_width=”match_parent”  
              android:layout_height="wrap_content"  
              android:text="@string/hello"  
              android:padding="10dp" />  
  
    ...  
  
</LinearLayout>
```  
我们可以重写任何include里面的属性，例如：
```xml
<include android:id=”@+id/news_title”  
         android:layout_width=”match_parent”  
         android:layout_height=”match_parent”  
         layout=”@layout/title”/>  
```

## 3)Use the <merge> Tag
某些时候，自定义可重用的布局包含了过多的层级标签，比如我们需要在LinearLayout里面嵌入一个重用的组件，而恰恰这个自定义的可重用的组件根节点也是LinearLayout，这样就多了一层没有用的嵌套，无疑这样只会拖慢程序速度。而这个时候如果我们使用merge根标签就可以避免那样的问题。例如：
```xml
<merge xmlns:android="http://schemas.android.com/apk/res/android">  
  
    <Button  
        android:layout_width="fill_parent"   
        android:layout_height="wrap_content"  
        android:text="@string/add"/>  
  
    <Button  
        android:layout_width="fill_parent"   
        android:layout_height="wrap_content"  
        android:text="@string/delete"/>  
  
</merge>  
```
这样的话，使用`<include>`包含上面的布局的时候，系统会自动忽略merge层级，而把两个button直接放置与include平级。

***
**文章学习自[http://developer.android.com/training/improving-layouts/reusing-layouts.html
](http://developer.android.com/training/improving-layouts/reusing-layouts.html)**
**转载请注明出自[http://kesenhoo.github.com](http:://kesenhoo.github.com)，谢谢**

