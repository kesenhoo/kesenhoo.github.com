---
layout: post
title: "Android Training Performance - 提升布局文件的性能(Lesson 3 - 使用viewStub按需载入视图)"
date: 2012-03-21 18:47
comments: true
sidebar: false
categories: Android Android:Training Android:Training:Performance
---

## Loading Views On Demand
某些时候，我们需要一些很复杂的视图却仅仅很少用到。如果我们在它仅仅需要的时候再载入，这样可以减少内存的使用并且给用户带来流畅的体验。

## 1)Define a ViewStub
[ViewStub](http://developer.android.com/reference/android/view/ViewStub.html)是一个轻量级的view，没有占有空间，没有花费draw的资源，也没有参与在任何一个layout的计算与绘制里面。

创建它仅需要很少的系统资源，而且存留在View的层级也是个比较不花费资源的动作。

<!-- More -->

每一个ViewStub简单的包含一个`android:layout`的属性来指定待创建的布局文件。

下面是一个包含Progress bar的ViewStub例子，这对于overlay来说是透明的，progress bar仅仅会在需要导入的时候才会可见。
```xml
<ViewStub  
    android:id="@+id/stub_import"  
    android:inflatedId="@+id/panel_import"  
    android:layout="@layout/progress_overlay"  
    android:layout_width="fill_parent"  
    android:layout_height="wrap_content"  
    android:layout_gravity="bottom" />  
```

## 2)Load the ViewStub Layout
当你想要载入在ViewStub中定义的布局的时候，可以calling setVisibility(View.VISIBLE) 或者是调用 inflate().
```java
 ((ViewStub) findViewById(R.id.stub_import)).setVisibility(View.VISIBLE);

// or
View importPanel = ((ViewStub) findViewById(R.id.stub_import)).inflate();
```
一旦被设置可见或者被创建，这个ViewStub组件则从View层级中消失，它被创建出来的布局所替代，而且这个布局的ID就是ViewStub里面用android:inflatedId属性所定义的。（用来定义这个ViewStub的ID的属性andoid:id直到被可见才是有效的）。

**Note:**ViewStub的一个缺陷是目前并不支持创建包含有<merge>标签的布局文件。更多ViewStub的信息请看：http://developer.android.com/resources/articles/layout-tricks-stubs.html

***
**文章学习自[http://developer.android.com/training/improving-layouts/loading-ondemand.html
](http://developer.android.com/training/improving-layouts/loading-ondemand.html)**
**转载请注明出自[http://kesenhoo.github.com](http:://kesenhoo.github.com)，谢谢**

