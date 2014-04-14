---
layout: post
title: "Android Training(01) - 适配不同的屏幕(Lesson 3 - 实现可适配的UI流程)"
date: 2012-03-21 14:38
comments: true
sidebar: false
categories: Android Android:Training
---
# Implementing Adaptative UI Flows
根据显示不同的layout，我们需要设计不同的UI flow。比如如果你的AP是dual-pane的模式，点击左边list的item的时候，会在右边直接显示对应的内容，如果是single-pane的模式，那么需要跳转到另外一个Activity来显示对于的内容

**注：**个人认为目前很多AP都会针对比较大的屏幕设计一个对于的版本，比如QQ Pad版，QQ HD版，QQ Pad Mini版，这些信息可以看出来大多数情况，还是不太会采取同一份代码适应所有屏幕的方案。
这一课主要就是讲如何在运行的时候判断当前的布局，从而让AP选择不同行为。

<!-- more -->

## Determine the Current Layout[判断当前的布局]
显然，为了现实不同UI flow的设计，我们首先需要知道当前使用的是哪个布局，two-pane or single-pane，因为前面讲到系统会自动根据当前屏幕来选择显示对应的布局文件

* 方法一：我们可以查询对应的View是否存在并且可见来判断目前的布局是哪个
```java
public class NewsReaderActivity extends FragmentActivity {  
    boolean mIsDualPane;  
  
    @Override  
    public void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.main_layout);  
  
        View articleView = findViewById(R.id.article);  
        mIsDualPane = articleView != null &&   
                        articleView.getVisibility() == View.VISIBLE;  
    }  
}  
```
* 方法二：我们可以判断某个组件是否存在来执行需要的操作（比如two-pane下没有categoryButton(因为two-pane下被actionBar替代)，而single-pane下则有）
```java
Button catButton = (Button) findViewById(R.id.categorybutton);  
OnClickListener listener = /* create your listener here */;  
if (catButton != null) {  
    catButton.setOnClickListener(listener);  
}  
```
## React According to Current Layout [根据当前布局有不同的反应]
某些动作会根据当前的布局而有不同的反映。例如如果你的AP是dual-pane的模式，点击左边list的item的时候，会在右边直接显示对应的内容，如果是single-pane的模式，那么需要跳转到另外一个Activity来显示对于的内容
```java
@Override  
public void onHeadlineSelected(int index) {  
    mArtIndex = index;  
    if (mIsDualPane) {  
        /* display article on the right pane */  
        mArticleFragment.displayArticle(mCurrentCat.getArticle(index));  
    } else {  
        /* start a separate activity */  
        Intent intent = new Intent(this, ArticleActivity.class);  
        intent.putExtra("catIndex", mCatIndex);  
        intent.putExtra("artIndex", index);  
        startActivity(intent);  
    }  
}  
```
## Reuse Fragments in Other Activities [在其他Activities里重用Fragments]
某些时候，我们可以重用一些组件，比如in the News Reader sample, the news article text is presented in the right side pane on large screens, but is a separate activity on smaller screens.  
在这种情况下，我们可以重用同一个Fragment在若干个Activities里面而避免duplication.例如，ArticleFragment 被使用在dual-pane的布局里面
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"  
    android:layout_width="fill_parent"  
    android:layout_height="fill_parent"  
    android:orientation="horizontal">  
    <fragment android:id="@+id/headlines"  
              android:layout_height="fill_parent"  
              android:name="com.example.android.newsreader.HeadlinesFragment"  
              android:layout_width="400dp"  
              android:layout_marginRight="10dp"/>  
    <fragment android:id="@+id/article"  
              android:layout_height="fill_parent"  
              android:name="com.example.android.newsreader.ArticleFragment"  
              android:layout_width="fill_parent" />  
</LinearLayout>  
```
并且被重用在small screens里面
```java
ArticleFragment frag = new ArticleFragment();  
getSupportFragmentManager().beginTransaction().add(android.R.id.content, frag).commit();  
```
显然，这样做的效果可以和声明一个布局文件效果一致，在这样的情况下，布局文件其实是另外一个Activity的组件，我们可以直接重复利用  
当设计fragments的时候我们需要记住的是：不要为特定的activity创建一个强耦合的fragment，我们可以使用定义一个接口来和host activity进行交互  
For example, the News Reader app's HeadlinesFragment does precisely that:
```java
public class HeadlinesFragment extends ListFragment {  
    ...  
    OnHeadlineSelectedListener mHeadlineSelectedListener = null;  
  
    /* Must be implemented by host activity */  
    public interface OnHeadlineSelectedListener {  
        public void onHeadlineSelected(int index);  
    }  
    ...  
  
    public void setOnHeadlineSelectedListener(OnHeadlineSelectedListener listener) {  
        mHeadlineSelectedListener = listener;  
    }  
}  
```
这样，当用户点击头条项的时候，这个fragment会通知host activity的listener进行操作，在listener的onHeadlineSelected方法里面会进行当前布局的判断，从而选择合适的UI（是显示在右边还是另起一个activity）
```java
public class HeadlinesFragment extends ListFragment {  
    ...  
    @Override  
    public void onItemClick(AdapterView<?> parent,   
                            View view, int position, long id) {  
        if (null != mHeadlineSelectedListener) {  
            mHeadlineSelectedListener.onHeadlineSelected(position);  
        }  
    }  
    ...  
}  
```

## Handle Screen Configuration Changes [Handle屏幕配置的改变]
如果我们在使用分离的activity来实现不同的模块，那么为了保持UI的连续性，我们需要记住目前的配置信息。  
比如，在跑3.0或者更高版本系统的7“的平板上，News Reader会在竖屏的时候使用另外一个activity来打开文章详情，在横屏的时候使用two-pane的布局（直接显示在右边）
```java
public class ArticleActivity extends FragmentActivity {  
    int mCatIndex, mArtIndex;  
  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        mCatIndex = getIntent().getExtras().getInt("catIndex", 0);  
        mArtIndex = getIntent().getExtras().getInt("artIndex", 0);  
  
        // If should be in two-pane mode, finish to return to main activity  
        if (getResources().getBoolean(R.bool.has_two_panes)) {  
            finish();  
            return;  
        }  
        ...  
}  
```

*********************************
**学习自：[http://developer.android.com/training/multiscreen/adaptui.html](http://developer.android.com/training/multiscreen/adaptui.html)，请多指教，谢谢！**  
**转载请注明出自[http://kesenhoo.github.com](http://kesenhoo.github.com)，谢谢配合！**






