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
****************
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
*****************
更多通用的常量定义，请查看[Intent](http://developer.android.com/reference/android/content/Intent.html)类的描述。  
你可以定义自己的action string来激活你程序中的其他组件。那些action请遵循把包名作为前缀的规则：例如"com.example.project.SHOW_COLOR".

* Data数据
不同的action会有不同类型的数据。例如，如果你的action是ACTION_EDIT,那么数据应该包含想要编辑的文件URI。如果action是ACTION_CALL,数据应该是tel: URI的格式。同样的，如果action是ACTION_VIEW，那么数据应该是http: URI。接受的action的activity会负责去处理那些指定的数据。  
数据的MIME类型也是非常重要的，例如，一个可以显示图片数据的组件不应该被叫起来做播放音频的动作。  
在许多情况下，数据类型可以从URI中读取到。但是数据类型同样可以显示的定义在Intent对象中。setData()方法指定了数据的URL，setType()方法指定了MIME类型, 同时setDataAndType()方法则可以同时指定URI与MIME。URI可以通过getData()来读取，数据类型则可以通过getType()来读取.

* Category分类
指定哪一类的组件应该处理这个intent。Intent有定义一些分类，如下:  
*************
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
****************
请查看[Intent](http://developer.android.com/reference/android/content/Intent.html)获取完整的分类列表.  
addCategory()方法可以把某个intent对象加入到指定的分类中，也可以通过removeCategory()来移除，使用getCategories()来获取当前对象中的所有分类。

* Extras额外的值
Intent对象有一系列put...() 方法来插入不同类型的数据，get...() 方法来读取数据. 那些方法可以对数据进行序列化处理成一个bundle对象。实际上，我们也可以new一个bundle对象，通过putExtras() 与 getExtras() 方法来进行处理.

* Flags标志
Flags用来表示，如何启动一个activity(例如，指定activity应该属于哪一个task) 并且在activity被启动后，这个activity在系统中的状态是怎么样的 (例如，是否应该属于最近使用的activity列表中). 所有的flag都在Intent类中有定义。

想了解如何启动Android系统内的组件，请查看[list of intents](http://developer.android.com/guide/appendix/g-app-intents.html).

## Intent Resolution[Intent详解]
Intents应该可以分成两类:  
* **Explicit**显式intents通过组件名来指定目标组件。因为组件名不一定可以被其他应用程序的开发者所了解，显式intent通常用来程序内部传递消息：例如activity启动一个下属级别的service或者是启动一个同级的activity。Android系统会根据组件名来把intent送到指定的组件上。  
* **Implicit**隐式intents并不指定特定的组件名。Implicit intents通常用来触发其他应用程序里面的组件。Android系统通过intentFilter来筛选出合适的组件来接收这个intent。没有定义IntentFilter的组件只能接受显式的intent，而定义了intentFilter的组件则可以接受显式与隐式两种类型的intent。

### Intent filters[Intent过滤器]
* Activities, services 与 broadcast receivers可以有一个或者多个intent filter.每一个filter都描述了一种可以通过筛选的intent。一个intent要像被这个组件所接收到，必须满足这个组件的所有filter中的一个。  
* 接收intent的组件会根据接收到的数据来决定做不同的动作。  
**注意**:intent filter并不能用来做安全性的检查。因为如果某个显式的intent通过组合多个action与data,指定发送到组件，这样就没有办法起到filter的作用了。  
* intent filter是[IntentFilter](http://developer.android.com/reference/android/content/IntentFilter.html)的实例。因为Android系统需要在组件被启动之前知道它的过滤值，所以一般都是直接写在AndroidManifest.xml里面。其中一个例外是broadcast receiver可以通过Context.registerReceiver()来动态进行注册。
* 一个filter会对action,data与category进行序列化处理。一个隐式的intent均需要通过前面三方面的过滤。如果其中某一项不符合，Android系统將过滤掉这个事件。另外，如果组件有定义多个filter，那么只要满足其中一个filter就可以让其通过。

* Action test
```xml
<intent-filter . . . >
    <action android:name="com.example.project.SHOW_CURRENT" />
    <action android:name="com.example.project.SHOW_RECENT" />
    <action android:name="com.example.project.SHOW_PENDING" />
    . . .
</intent-filter>
```
如果某个intent有定义action，那么想要通过上面的测试，必须满足其中一条action.同时，如果intent没有定义任何action，则默认是可以通过的。

* Category test
```xml
<intent-filter . . . >
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    . . .
</intent-filter>
```
与action类似，如果有定义category，想要通过，则必须满足filter中的任意一条，否则，如果没有定义category，则默认可以通过。  
然而，activity如果想要接收隐式的intent，那么在activity的filter里面必须包含"android.intent.category.DEFAULT"。

* Data test
```xml
<intent-filter . . . >
    <data android:mimeType="video/mpeg" android:scheme="http" . . . /> 
    <data android:mimeType="audio/mpeg" android:scheme="http" . . . />
    . . .
</intent-filter>
```
每一个<data>标签都可以指定一个URI与一个数据类型(MIME media type). 格式像下面定义的那样：
```xml
scheme://host:port/path
```
例如：content://com.example.project:200/folder/subfolder/etc  
那些属性是可选的，但是他们并不是相互独立的。例如，一个路径想要有意义，必须指定scheme与authority.  
他们有着下面一些规则需要遵守：

* 一个既没有URI也没有数据类型的Intent对象，只有在filter没有指定任何URI或者数据类型的情况下才能通过。
* 一个包含URI但是没有数据类型的Intent对象，符合filter定义的URI,同时filter并没有定义数据类型，这样这个对象能够通过。这种情况仅仅发生在类似**mailto:**与**tel:**，他们并没有指定任何实际的数据。
* 一个包含数据类型但是没有定义URI的Intent对象，仅仅在fliter拥有同样的数据类型而且也没有定义URI的情况下，才能通过。
* 一个即包含URI又包含数据类型的(或者数据类型可以从URI中获取到)Intent对象。如果与filter中的URI相匹配，这URI部分可以通过。如果数据类型也匹配，则数据部分也可以通过。有个例外，**如果intent的URI是content: 或者 file: ，然而filter却没有定义任何URI(只定义了数据类型)，这样的话，同样能够通过。也就是说content:与file:是特例。**
**********
如果一个intent可以通过多个组件的filter，那么系统会提示用户选择启动哪一个组件。如果没有一个组件满足，则会发生异常。

### Common cases[常见情况]
上面最后一条规则说明组件可以从文件或者content provider中获取本地的数据。因此，他们的filter可以仅仅需要指定数据类型，而不用显式指定content: 与 file: 的URI. 这是一个很常见的Case.

* 下面的filter将使得符合条件的组件从content provider获取图片数据并进行显示
```xml
<data android:mimeType="image/*" />
```
* 从网络获取video数据并进行显示
```xml
<data android:scheme="http" android:type="video/*" />
```

### Using intent matching[intent匹配模式]
* Intents与intent filters进行匹配，不仅仅是为了寻找待激活的某个组件，而且是为了寻找符合条件的所有的组件。例如，Android系统会把那些intent filter中包含android.intent.action.MAIN" action与"android.intent.category.LAUNCHER" category的组件作为程序的入口(icon launcher).同样的，它通过"android.intent.category.HOME"来寻找桌面组件。

* [PackageManager](http://developer.android.com/reference/android/content/pm/PackageManager.html)有一组query...()方法来查询那些符合intent条件的组件，还提供了一组resolve...() 方法来决定最佳的组件对intent进行响应。例如，[queryIntentActivities()](http://developer.android.com/reference/android/content/pm/PackageManager.html#queryIntentActivities(android.content.Intent, int))返回所有符合参数intent描述的activity。[queryIntentServices()](http://developer.android.com/reference/android/content/pm/PackageManager.html#queryIntentServices(android.content.Intent, int))则返回符合条件的sevice。同样的，对[queryBroadcastReceivers()](http://developer.android.com/reference/android/content/pm/PackageManager.html#queryBroadcastReceivers(android.content.Intent, int))也是一样.


*****************
**文章学习自[http://developer.android.com/guide/components/processes-and-threads.html](http://developer.android.com/guide/components/processes-and-threads.html)**  
**转载请注明出自[http://kesenhoo.github.com](http:://kesenhoo.github.com)，谢谢**

