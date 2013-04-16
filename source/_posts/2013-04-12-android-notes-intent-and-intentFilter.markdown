---
layout: post
title: "Android Notes - Intents and Intent Filters"
date: 2013-04-12 18:27
comments: true
sidebar: false
categories: Android
---

## Intent与Intent Filter

Android其中的三大组件，Activity,Service与broadcast receivers是通过Intent来触发彼此的。Intent本身会携带一些信息，它是一种组件内部或者组件间进行交互的中介。  
对于不同的组件之间有着不同的机制：

* 一个Intent对象通过Context.startActivity()或者Activity.startActivityForResult()来启动一个activity或者让已经存在的activity做一些更新的操作。
* 一个Intent对象通过Context.startService()来初始化一个Service或者给已经在运行的Service传递新的指令。同样的，对于Context.bindService()是一样的道理。
* Intent对象通过broadcast的方法（例如Context.sendBroadcast(), Context.sendOrderedBroadcast()或者Context.sendStickyBroadcast())传递给那些感兴趣的broadcast receivers.

<!-- more -->

每一种不同类型的intent都会由系统传递到对应的组件上，下面会介绍Android系统是如何根据Intent里面的参数进行分类处理并传递到符合要求的组件上的。

### Intent对象
一个Intent对象是许多信息的一个bundle.通常来说，它可以包含下面的内容：  
* Component name组件名  
	* 指定处理这个intent的组件。包名与包中的组件名。组件名是可选的。如果设置了组件名，这个intent就会传递给对应的类。如果没有设置组件名，Android会利用Intent对象中的其他信息来判断传递的对象。
	* 可以通过setComponent(), setClass()或者setClassName()来设置组件名，并通过getComponent()来读取组件名.
* Action动作意图  
一个String名称来表示执行Intent的意图是什么，Intent类定义了一些action常量，如下：  

<table>
   <tr>
      <td>Constant</td>
      <td>Target component</td>
      <td>Action</td>
   </tr>
   <tr>
      <td>ACTION_CALL</td>
      <td>activity</td>
      <td>Initiate a phone call.</td>
   </tr>
   <tr>
      <td>ACTION_EDIT</td>
      <td>activity</td>
      <td>Display data for the user to edit.</td>
   </tr>
   <tr>
      <td>ACTION_MAIN</td>
      <td>activity</td>
      <td>Start up as the initial activity of a task, with no data input and no returned output.</td>
   </tr>
   <tr>
      <td>ACTION_SYNC</td>
      <td>activity</td>
      <td>Synchronize data on a server with data on the mobile device.</td>
   </tr>
   <tr>
      <td>ACTION_BATTERY_LOW</td>
      <td>broadcast receiver</td>
      <td>A warning that the battery is low.</td>
   </tr>
   <tr>
      <td>ACTION_HEADSET_PLUG</td>
      <td>broadcast receiver</td>
      <td>A headset has been plugged into the device, or unplugged from it.</td>
   </tr>
   <tr>
      <td>ACTION_SCREEN_ON</td>
      <td>broadcast receiver</td>
      <td>The screen has been turned on.</td>
   </tr>
   <tr>
      <td>ACTION_TIMEZONE_CHANGED</td>
      <td>broadcast receiver</td>
      <td>The setting for the time zone has changed.</td>
   </tr>
</table>

更多通用的常量定义，请查看[Intent](http://developer.android.com/reference/android/content/Intent.html)类的描述。  
你可以定义自己的action string来激活你程序中的其他组件。那些action请遵循把包名作为前缀的规则：例如"com.example.project.SHOW_COLOR".

* Data数据
不同的action会有不同类型的数据。例如，如果你的action是ACTION_EDIT,那么数据应该包含想要编辑的文件URI。如果action是ACTION_CALL,数据应该是tel: URI的格式。同样的，如果action是ACTION_VIEW，那么数据应该是http: URI。接受的action的activity会负责去处理那些指定的数据。  
数据的MIME类型也是非常重要的，例如，一个可以显示图片数据的组件不应该被叫起来做播放音频的动作。  
在许多情况下，数据类型可以从URI中读取到。但是数据类型同样可以显示的定义在Intent对象中。setData()方法指定了数据的URL，setType()方法指定了MIME类型, 同时setDataAndType()方法则可以同时指定URI与MIME。URI可以通过getData()来读取，数据类型则可以通过getType()来读取.

* Category分类
指定哪一类的组件应该处理这个intent。Intent有定义一些分类，如下:  

<table>
   <tr>
      <td>Constant</td>
      <td>Meaning</td>
   </tr>
   <tr>
      <td>CATEGORY_BROWSABLE</td>
      <td>The target activity can be safely invoked by the browser to display data referenced by a link — for example, an image or an e-mail message.</td>
   </tr>
   <tr>
      <td>CATEGORY_GADGET</td>
      <td>The activity can be embedded inside of another activity that hosts gadgets.</td>
   </tr>
   <tr>
      <td>CATEGORY_HOME</td>
      <td>The activity displays the home screen, the first screen the user sees when the device is turned on or when the Home button is pressed.</td>
   </tr>
   <tr>
      <td>CATEGORY_LAUNCHER</td>
      <td>The activity can be the initial activity of a task and is listed in the top-level application launcher.</td>
   </tr>
   <tr>
      <td>CATEGORY_PREFERENCE</td>
      <td>The target activity is a preference panel.</td>
   </tr>
</table>

请查看[Intent](http://developer.android.com/reference/android/content/Intent.html)获取完整的分类列表.  
addCategory()方法可以把某个intent对象加入到指定的分类中，也可以通过removeCategory()来移除，使用getCategories()来获取当前对象中的所有分类。

* Extras额外的值
Intent对象有一系列put...() 方法来插入不同类型的数据，get...() 方法来读取数据. 那些方法可以对数据进行序列化处理成一个bundle对象。实际上，我们也可以new一个bundle对象，通过putExtras() 与 getExtras() 方法来进行处理.

* Flags标志
Flags用来表示，如何启动一个activity(例如，指定activity应该属于哪一个task) 并且在activity被启动后，这个activity在系统中的状态是怎么样的 (例如，是否应该属于最近使用的activity列表中). 所有的flag都在Intent类中有定义。

想了解如何启动Android系统内的组件，请查看[list of intents](http://developer.android.com/guide/appendix/g-app-intents.html).

## Intent Resolution[Intent详解]



*****************
**文章学习自http://developer.android.com/guide/components/processes-and-threads.html**  
**转载请注明出自[http://kesenhoo.github.com](http:://kesenhoo.github.com)，谢谢**

