---
layout: post
title: "Android Training 03 - 使用Fragments建立动态的UI(Lesson 0 - 章节概览)"
date: 2012-11-27 14:38
comments: true
sidebar: false
categories: Android Android:Training
---

# Building a Dynamic UI with Fragments

* 学习这章节的先决条件:
	* 了解Activity的基本概念 (see [Managing the Activity Lifecycle](http://developer.android.com/training/basics/activity-lifecycle/index.html))
	* 有使用 XML layouts 的经验
* 你还需要看的有:
	* [Fragments](http://developer.android.com/guide/components/fragments.html)
	* [Supporting Tablets and Handsets](http://developer.android.com/guide/practices/tablets-and-handsets.html)
* DEMO:
[FragmentBasics.zip](http://developer.android.com/shareables/training/FragmentBasics.zip)

<!-- more -->
**********************************
为了在Android上创建一个动态并适配multi-pane的用户界面，你需要封装（encapsulate）界面的组件与activity的行为到一个模块中，这样你可以用这个模块与父activity进行交互：载入与换出。你可以使用Fragment创建那些模块。Fragment像是一个嵌套的activity，你可以定义它的layout并且管理它自己的生命周期。

当一个fragment定义了他自己的layout,它可以在一个activity内与其他有关系的fragment一起，为不同的屏幕大小显示不同的布局或者行为。(一个小的屏幕可能会一次显示一个fragment，但是在大的屏幕上可以一次显示两个或者更多个）。

这一章会演示如何使用fragment创建一个动态可适配的用户体验，并且学习如何为不同的屏幕大小设备来优化你的app的用户体验。

## Lessons
* Using the Android Support Library(使用Android Support Library)  
学习如何通过绑定Android Support Library到你的app的方式把最近的API使用到比较早的Android系统上。

* Creating a Fragment(创建一个Fragment)  
学习如何建立一个fragment并通过实现它的回调方法来实现一些基础的行为。

* Building a Flexible UI(建立一个灵活的UI)  
学习如何为不同的屏幕创建不同的fragment机制来进行适配。

* Communicating with Other Fragments(与其他的fragment进行交互)  
学习如何建立fragment与activity之间，还有fragments之间的交流。


*********************************
**学习自：[http://developer.android.com/training/basics/fragments/index.html](http://developer.android.com/training/basics/fragments/index.html)，请多指教，谢谢！**  
**转载请注明出自[http://kesenhoo.github.com](http://kesenhoo.github.com)，谢谢配合！**






