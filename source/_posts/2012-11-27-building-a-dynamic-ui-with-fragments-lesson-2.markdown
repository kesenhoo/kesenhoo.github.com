---
layout: post
title: "Android Training 03 - 使用Fragments建立动态的UI[Lesson 2 - 新建一个Fragment]"
date: 2012-11-27 16:48
comments: true
sidebar: false
categories: Android
---

* 你可以把fragment当作activity的一部分，它有自己的lifecycle,它会接受自己的输入事件。你可以在activity运行的时候增加或者拿掉fragment。（类似子activity，你可以在不同的activity中重用fragment）。这节课演示如何使用support library来创建一个继承自 Fragment 的类。  
* Note: 如果你的app决定跑在Level 11或者更高的系统上，你不需要添加这个support library,你可以使用那个版本的系统自带的API来实现。请注意这节课是专注于使用support library里面的API，这里面的API名称与那些本身就包含这些功能的平台上的名称有可能不一样。

## Create a Fragment Class[创建一个fragment类]
* 为了新建一个fragment,需要继承 Fragment 类，然后重写关键的lifecycle中的callback方法，类似Activity的写法。  
* 创建fragment，其中一个不同的地方是你必须使用 onCreateView() 的回调来定义Layout。实际上，这是使得fragment开始运行的唯一的callback。如下:
```java
import android.os.Bundle;
import android.support.v4.app.Fragment;
import android.view.LayoutInflater;
import android.view.ViewGroup;

public class ArticleFragment extends Fragment {
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, 
        Bundle savedInstanceState) {
        // Inflate the layout for this fragment
        return inflater.inflate(R.layout.article_view, container, false);
    }
}
```
* 就想一个activity一样，一个fragment应该实现一些其他的callback，这样你可以操控当它添加到activity，从activity移除，或者activity在不同生命周期进行切换时fragment该做的动作。l例如，当activity的 onPause() 方法被呼叫时,所有在这个activity中的fragment也都会呼叫到 onPause().方法。  
* 更多关于fragment的lifecycle与callback的方法，请参考[Fragments](http://developer.android.com/guide/components/fragments.html)开发者指南。

## Add a Fragment to an Activity using XML[使用XML添加fragment到activity]
* 因为fragment可以重用，是模块化的组件，每一个fragment的实例都与父类FragmentActivity有依赖，你可以通过定义每一个fragment的XML来实现这样的联系。  
  **Note:** FragmentActivity 是一个在Support Library中的特殊的activity，使用它来向Level 11以下的系统进行兼容。如果你是在Level 11以上开发，则可以使用通常的 Activity.

* 这里有一个layout的示例文件，当设备屏幕被认为是large级别时，它会添加了两个fragment到一个activity中。  
res/layout-large/news_articles.xml:
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="horizontal"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent">

    <fragment android:name="com.example.android.fragments.HeadlinesFragment"
              android:id="@+id/headlines_fragment"
              android:layout_weight="1"
              android:layout_width="0dp"
              android:layout_height="match_parent" />

    <fragment android:name="com.example.android.fragments.ArticleFragment"
              android:id="@+id/article_fragment"
              android:layout_weight="2"
              android:layout_width="0dp"
              android:layout_height="match_parent" />

</LinearLayout>
```
* 下面则演示了一个activity如何运用这个layout:
```java
import android.os.Bundle;
import android.support.v4.app.FragmentActivity;

public class MainActivity extends FragmentActivity {
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.news_articles);
    }
}
```

* Note: 当你通过定义一个layout XML的方式来添加fragment到activity时，你不可以在运行时移除这个fragment.如果你计划在与用户进行交互时对fragment进行载入与移除，你必须在activity第一次启动时进行添加fragment，下一节课会讲到。

*********************************
**学习自：[http://developer.android.com/training/basics/fragments/creating.html](http://developer.android.com/training/basics/fragments/creating.html)，请多指教，谢谢！**  
**转载请注明出自[http://kesenhoo.github.com](http://kesenhoo.github.com)，谢谢配合！**






