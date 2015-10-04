---
layout: post
title: "Android开发最佳实践"
date: 2015-10-02 15:38
comments: true
sidebar: false
categories: Android
---

![android_dev_patterns_logo](/images/android_dev_patterns_logo.png)

前段时间，Google公布了[Android开发最佳实践](https://www.youtube.com/playlist?list=PLWz5rJ2EKKc-lJo_RGGXL2Psr8vVCTWjM)的一系列课程，涉及到一些平时开发过程中应该保持的良好习惯以及如何使用最新的[Android Design Support Library](http://android-developers.blogspot.com/2015/05/android-design-support-library.html)来快速实现官方推荐的Material Design样式的应用。下面是个人的学习摘要总结，不对的地方请多多交流指点，谢谢！

## 1）注意对隐式Intent的运行时检查保护
类似打开相机，发送图片等隐式Intent，是并不一定能够在所有的Android设备上都正常运行。例如打开相机的隐式Intent，如果系统相机应用被关闭或者不存在相机应用，又或者是相机应用的某些权限被关闭等等情况都可能导致这个隐式的Intent无法正常工作。一旦发生隐式Intent找不到合适的调用组件的情况，系统就会抛出`ActivityNotFoundException`的异常，如果我们的应用没有对这个异常做任何处理，那应用就会发生Crash。

预防这个问题的最佳解决方案是在发出这个隐式Intent之前调用`resolveActivity`做检查，关于这个API的解释以及用法如下：

<!-- More -->

![android_dev_patterns_implicit_intent](/images/android_dev_patterns_implicit_intent.png)

然后这个API的使用范例如下：

```java
Intent intent = new Intent(Intent.ACTION_XXX);
ComponentName componentName = intent.resolveActivity(getPackageManager());
if(componentName != null) {
    String className = componentName.getClassName();
}
```

## 2）使用NotificationCompat兼容包来处理消息通知
为了解决Android系统版本差异导致的Notification兼容性问题，Android官方提供了`NotificationCompat`兼容类来帮助开发实现体验统一的Notification。通常来说，建立一个Notification至少会有三种元素：图标，标题，文本。我们通常会使用如下的代码来实现一个基础的Notification功能：

![android_dev_patterns_notification_base](/images/android_dev_patterns_notification_base.png)

上面那段代码，运行时候的效果应该如下所示：

![android_dev_patterns_notification_base_show](/images/android_dev_patterns_notification_base_show.png)

为了给上面的Notification添加点击之后的响应效果，我们还需要构造一个`PendingIntent`作为contentIntent，例如：

```java
PendingIntent intent = xxx;
builder.setContentIntent(intent);
```

为了使得Notification更加的具有辨识度，我们还有可能做如下的设置：

![android_dev_patterns_notification_set_large](/images/android_dev_patterns_notification_set_large.png)

从Android 4.1开始，Notification可以支持展开显示的模式，这样一来，Notification就演变出了下面4种不同的风格样式：

![android_dev_patterns_notification_4_styles](/images/android_dev_patterns_notification_4_styles.png)

Notification还提供了快捷操作的功能，如下图所示：

![android_dev_patterns_notification_set_action_show](/images/android_dev_patterns_notification_set_action_show.png)
![android_dev_patterns_notification_set_action](/images/android_dev_patterns_notification_set_action.png)

除了显示在手机上的Notification，我们还可以给Notification分别设置在Wearable，Auto上的不同表现行为，例如针对可穿戴设备上显示Notification，我们可以如下的设置：

![android_dev_patterns_notification_wearable](/images/android_dev_patterns_notification_wearable.png)

关于更多的Wearable上的Notification相关的知识，还可以参考[Pages of Content](https://www.youtube.com/watch?v=N7aJPyvHPgs&feature=iv&src_vid=-iog_fmm6mE&annotation_id=annotation_347477041)与[Stackable Notifications](https://www.youtube.com/watch?v=L4LvKOTkZ7Q&feature=iv&src_vid=-iog_fmm6mE&annotation_id=annotation_2485794965)

## 3）Android 6.0 Marshmallow的运行时权限
Android 6.0开始引入了新的运行时权限检查授权机制，替代了之前安装应用的时候对权限进行授权的方案。为了避免6.0及以上的机器运行发生运行时异常，我们需要做到至少以下5个步骤：

- **检查系统版本号**：针对6.0以下的系统版本，默认权限在安装的时候已经获取到了，对于6.0开始的版本，才需要做运行时的权限检查。
- **检查申请的权限**：在使用某个权限之前，需要检查权限是否已经获取到了。
- **解释申请的权限**：在权限没有获取到的情况下，需要通过`shouldShowRequestPermissionRationable()`的判断来决定如何给用户进行提示。
- **执行申请权限操作**：前面判断没有获取到权限，为了能够让功能顺利执行，我们会需要在代码里面再次执行申请此权限的操作。

![android_dev_patterns_permission_check](/images/android_dev_patterns_permission_check.png)

- **处理权限申请的结果**：申请权限之后，我们需要处理申请的响应结果，分别处理权限申请成功与失败的情况

![android_dev_patterns_permission_response](/images/android_dev_patterns_permission_response.png)

## 4）使用MediaSessionCompat操作音乐的播放
MediaSessionCompat来自Android官方的兼容包，通过它可以告诉Android系统与其他的应用，自己正在播放的内容是什么以及自己支持哪些类型的播放控制：

![android_dev_patterns_media_session](/images/android_dev_patterns_media_session.png)

在Android的官方培训课程中有介绍过关于[Media Button Receiver](http://developer.android.com/training/managing-audio/volume-playback.html)的概念，Android系统会把来自蓝牙控制器或者是耳机等其他设备的操作事件转换成Media Button事件传递出来，如果我们的应用程序需要监听这些事件并做出相应的响应，就需要注册MEDIA_BUTTON的action，接收到这些事件之后，再传递给音乐播放模块进行控制处理。

![android_dev_patterns_media_receiver](/images/android_dev_patterns_media_receiver.png)

基于上面的认知，我们现在演示如何使用MediaSessionCompat，下面演示了如何构造一个MediaSessionCompat以及构造完之后通常需要做的三件事情：设置合理的flag，设置回调（在5.0开始会响应onPlay，onPause等等回调），设置激活。

![android_dev_patterns_media_usage](/images/android_dev_patterns_media_usage.png)

搭建好了MediaSessionCompat之后，还需要通过`MediaMetadataCompat`来传递播放的资料信息，通过`PlaybackStateCompat`来传递播放的状态信息。

![android_dev_patterns_media_set](/images/android_dev_patterns_media_set.png)

做了上面那些操作之后，MediaSessionCompat的任务就算是完成了。

![android_dev_patterns_media_done](/images/android_dev_patterns_media_done.png)

## 5）使用Toolbar替代ActionBar
自从MaterialDesign开始，Android官方就开始使用Toolbar替代了原来的ActionBar，现在Toolbar已经加入Support兼容包。Toolbar是一个相比起ActionBar更加丰富，更加灵活的组件，另外它的布局本身还是View Hierarchy的一部分，这就意味着可以对Toolbar执行动画操作，增加点击滑动事件等等，甚至我们还可以在一个页面里面加入两个Toolbar。

为了启用Toolbar，首先要做的事情就是关闭当前Activity的ActionBar。我们可以通过使得Activity的主题继承`Theme.AppCompat.NoActionBar`，然后在对应的XML布局文件中，添加类似下面的toolbar布局信息：

```xml
<!--Toolbar-->
<android.support.v7.widget.Toolbar
    android:id="@+id/toolbar_layout"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_alignParentTop="true"
    android:minHeight="?attr/actionBarSize">
</android.support.v7.widget.Toolbar>
```

在布局文件中添加Toolbar的信息之后，需要启动Toolbar替代ActionBar，需要像下面一样做设置：

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
	super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar_layout);
    setSupportActionBar(toolbar);
}
```

关于更多Toolbar的细节，请参考官方文档<https://developer.android.com/reference/android/support/v7/widget/Toolbar.html>

## 6）使用AppBarLayout并处理滑动手势
AppBarLayout是一个在android.support.design兼容包（这里有关于该兼容包的官方博客介绍<http://android-developers.blogspot.com/2015/05/android-design-support-library.html>）里面的新推出的组件，它是一个垂直方向的LinearLayout，包装了很多Material Design的设计元素，例如滑动手势的处理。我们可以使用`app:layout_scrollFlags`这样的标签来设置滑动的行为表现。关于App Bar，官方还有这样一段描述：

![android_dev_patterns_appbar_des](/images/android_dev_patterns_appbar_des.png)

使用AppBarLayout需要注意下面几个要点：

* 首先，AppBarLayout必须作为`CoordinatorLayout`的直接子View
* 其次，在AppBarLayout里面必须包含一个ToolBar
* 最后，在CoordinatorLayout里面可以添加那些可以滑动的组件，例如RecyclerView

一个标准的布局文件应该是类似下面结构的：

![android_dev_patterns_appbar_scroll_xml](/images/android_dev_patterns_appbar_scroll_xml.png)

我们需要注意，在Toolbar里面设置的layout_scrollFlags会影响到滑动之后的显示效果，请看下面的具体解释：

![android_dev_patterns_appbar_scroll_flag](/images/android_dev_patterns_appbar_scroll_flag.png)

## 7）使用SearchView来实现搜索功能
关于在使用搜索的时候及时显示给用户的候选词，会需要根据数据的类型以及具体的情况做不同的处理，这里先暂时不讨论那一块的内容。

![android_dev_patterns_searchview_show](/images/android_dev_patterns_searchview_show.png)
![android_dev_patterns_searchview_tips](/images/android_dev_patterns_searchview_tips.png)

上面的图示已经清楚的演示了使用SearchView处理搜索的通常情况，关于如何实现这个功能，需要做到以下几个步骤：

* 在Menu的XML文件中，声明使用SearchView

![android_dev_patterns_searchview_xml](/images/android_dev_patterns_searchview_xml.png)

* 在onCreateOptionsMenu的回调函数里面获取到SearchView，并设置监听（请注意使用MenuItemCompat的那行代码，否者会出现很多兼容性问题，获取不到这个View等等奇怪的BUG），在监听回调里面处理业务逻辑

![android_dev_patterns_searchview_code](/images/android_dev_patterns_searchview_code.png)

至此，其实就已经实现了一个基础的搜索功能。但是，如果为了能够让自己的应用的某些功能被Android系统的Search功能检索到，我们就需要做更进一步的操作，例如定义Searchable，实现一个SearchableActivity，响应系统的Search行为等等。国内的应用很少会去关注这个功能，这里就不展开了，感兴趣点击下面的链接进一步学习：<https://developer.android.com/guide/topics/search/index.html>

## 8）Navigation Drawer, DrawerLayout, NavigationView
Navigation Drawer是Material Design当中很重要的一种设计元素，为了能够快速的实现这种设计，Android在新的design support包里面为我们提供了DrawerLayout与NavigationView。

![android_dev_patterns_navigation_show](/images/android_dev_patterns_navigation_show.png)

实现这样的一个功能，我们需要在XML中写下类似下面的布局文件

![android_dev_patterns_navigation_xml](/images/android_dev_patterns_navigation_xml.png)

在NavigationView中有两个重要的标签，app:headerLayout与app:menu，分别代表了拉出菜单的顶部布局与下面的菜单项列表。创建菜单项列表，可以使用类似下面的MenuItem文件：

![android_dev_patterns_navigation_menu](/images/android_dev_patterns_navigation_menu.png)

`android:checked`表示当前选中的Item，会被系统Highlight出来。除了上面简单的平铺的菜单，还可以使用菜单嵌套的方式实现多级的菜单。关于点击具体某个菜单的时候的监听与响应，需要做如下的设置：

![android_dev_patterns_navigation_click](/images/android_dev_patterns_navigation_click.png)

除了点击菜单事件，我们还需要处理整个侧滑菜单的打开与关闭事件，当我们给DrawerLayout设置了setDrawerListener之后，可以得到下面两个回调：

![android_dev_patterns_navigation_drawer_event](/images/android_dev_patterns_navigation_drawer_event.png)

大多数时候，侧滑菜单都是从左到右滑出的，但是我们也可以做到从右往左滑出，只需要在DrawerLayout的菜单布局LinearLayout里面修改一下margin的相关属性即可：

![android_dev_patterns_navigation_right](/images/android_dev_patterns_navigation_right.png)

## 9）Tabs and ViewPager
![android_dev_patterns_viewpager_show](/images/android_dev_patterns_viewpager_show.png)

ViewPager是Android上面实现横向滑动的基础组件，能够帮助大家迅速搭建类似上面图示一样的左右滑动交互设计。ViewPager需要使用PagerAdapter来提供内容，除了PagerAdpater，Android还提供了FragmentPagerAdpater与FragmentStatePagerAdapter，前者会把所有的fragment都保存在内存中，以便提高切换速度，后者仅仅保留了fragment状态信息，fragment还是会进行正常的重建与销毁。一个典型的使用demo代码如下：

![android_dev_patterns_viewpager_code](/images/android_dev_patterns_viewpager_code.png)

为了实现前面图示的Tab与ViewPager的绑定，我们可以使用[Android Design Support Library](http://android-developers.blogspot.com/2015/05/android-design-support-library.html)提供的`TabLayout`，仅仅需要按照下面的代码示例一样把TabLayout与ViewPager做一个绑定，就能够实现左右滑动与点击Tab快速切换的功能：

![android_dev_patterns_viewpager_tablayout](/images/android_dev_patterns_viewpager_tablayout.png)

关于Material Design里面的Tabs设计，请再参考<http://www.google.com/design/spec/components/tabs.html>以及官方Training课程里面的<http://developer.android.com/training/implementing-navigation/lateral.html>

## 10）Making Apps Accessible
为了照顾部分视力或者听觉不好的用户，我们需要做一定的处理使得自己的应用能够被每一个可用。Android系统为了帮组应用实现辅助功能，提供了诸如文本朗读，触感反馈，指向炳导航，手势导航等等功能来更好的帮助用户使用这些应用。

![android_dev_patterns_accessible_feature](/images/android_dev_patterns_accessible_feature.png)

为了确保你的应用能够被Android系统提供的辅助功能正常使用，需要做以下三个步骤的检查：

* **Content Description**：确保类似ImageView，ImageButton，CheckBox等组件都包含了content descrption。

![android_dev_patterns_accessible_content](/images/android_dev_patterns_accessible_content.png)

* **Focus Order**：确保给布局里面的关键元素增加了Focus的指示顺序，只有这样，辅助功能才能够在指向导航的时候帮助用户按照指定的顺序来聚焦界面元素。

![android_dev_patterns_accessible_focus_order](/images/android_dev_patterns_accessible_focus_order.png)
![android_dev_patterns_accessible_focus_code](/images/android_dev_patterns_accessible_focus_code.png)

* **Feedback Mechanisms**：确保部分关键的操作有多个反馈，例如当短信来的时候，既有声音也有震动，这样才能够确保听力不好的用户可以通过震动的反馈来感知到响应。

更多关于辅助功能的知识，请参考<http://developer.android.com/guide/topics/ui/accessibility/checklist.html>
