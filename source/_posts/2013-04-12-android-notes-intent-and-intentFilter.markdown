---
layout: post
title: "Android Notes - Intents and Intent Filters"
date: 2013-04-12 18:27
comments: true
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


*****************
** 文章学习自http://developer.android.com/guide/components/processes-and-threads.html**  
** 转载请注明出自[四方城](http:://kesenhoo.github.com)，谢谢**

