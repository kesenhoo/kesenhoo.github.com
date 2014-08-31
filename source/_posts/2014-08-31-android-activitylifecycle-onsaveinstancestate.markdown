---
layout: post
title: "Android Notes - Activity生命周期中的onSaveInstanceState"
date: 2014-08-31 17:01
comments: true
sidebar: false
categories: Android Android:Notes
---

记录下Activity生命周期中的[onSaveInstanceState(Bundle outState)](http://developer.android.com/reference/android/app/Activity.html)

## onSaveInstanceState与onRestoreInstanceState的作用：
在资源紧张的情况下，系统会选择杀死一些处于非栈顶的Activity来回收资源。
为了能够让这些可能被杀死的Activity能够在恢复显示的时候状态不丢失，所以需要在Activity从栈顶往下压的时候提供onSaveInstanceState的回调用来提前保存状态信息。

而onRestoreInstanceState则是在这个Activity真的回收掉之后的恢复显示阶段用来恢复之前保存的数据。

## onSaveInstanceState与onRestoreInstanceState的调用时机：
只要某个Activity是做入栈并且非栈顶时（启动跳转其他Activity或者点击Home按钮），此Activity是需要调用onSaveInstanceState的，
如果Activity是做出栈的动作（点击back或者执行finish），是不会调用onSaveInstanceState的。

只有在Activity真的被系统非正常杀死过，恢复显示Activity的时候，就会调用onRestoreInstanceState。

## [Sample Code](https://github.com/kesenhoo/ActivityLifeCycle)
* 从ActivityA启动ActivityB执行顺序是：A：onCreate -> A：onStart -> A：onResume -> B：onCreate -> B：onStart -> B：onResume -> A：onSaveInstanceState –> A：onStop。
* 正常流程从ActivityB点击Back按钮或者是触发finish方法回退到ActivityA，执行顺序是：B：finish –> B：onPause –> A： onRestart –> A：onStart  -> A：onResume -> B： onStop –> B：onDestroy。
* 若启动ActivityB之后，选择点击Home按钮，程序退到后台，那么执行顺序是：B：onPause -> B：onSaveInstanceState -> B：onStop。
* 程序在后台的时候，选择主动杀死程序进程，然后再从桌面点击应用启动，会显示之前的ActivityB，执行顺序是：B：onCreate -> B：onStart –> B：onRestoreInstanceState - > B：onResume。
* 点击Back按钮或者是执行Activity B里面提供的finish方法：B：finish –> B：onPause –> A：onCreate  -> A：onStart -> A：onRestoreInstanceState -> A：onResume -> B：onStop -> B：onDestory。
* 最后再点击Back按钮或是执行Activity A里面的finish方法退出程序：A：finish -> A：onPause –> A：onStop -> A：onDestory。

