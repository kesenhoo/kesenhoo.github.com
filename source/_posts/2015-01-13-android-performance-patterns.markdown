---
layout: post
title: "Android性能优化必知16点"
date: 2015-01-13 21:57
comments: true
sidebar: false
categories: Android Android:Deeper
---

Google近期发布了关于[Android性能问题的专题](https://www.youtube.com/playlist?list=PLWz5rJ2EKKc9CBxr3BVjPTPoDPLdPIFCE)，一共16个短视频，帮助开发者创建更快更流畅的Android App。一方面介绍了Android系统的工作原理，同时也介绍了如何通过工具来找出性能问题以及处理建议。下面是对这些问题和建议的总结。

## 0)Render Performance
大多数用户感知到的卡顿等性能问题的最主要根源都来是因为渲染问题。从设计师的角度，他们希望App能够有更多的动画，图片等流畅的体验。但是Android系统会每隔16ms发出VSYNC信号，触发对UI进行一次渲染，如果每次渲染都成功，这样就能够达到60fps，为了能够实现60fps，这意味着你的大多数操作都必须在16ms内完成。

![](/images/draw_per_16ms.png)

<!-- More -->

如果你的某个操作花费时间是24ms，系统在得到VSYNC信号的时候就无法进行正常渲染，这样就发生了丢帧现象。那么用户在32ms内看到的会是同一帧画面。

![](/images/vsync_over_draw.png)

用户容易在UI执行动画或者滑动ListView的时候感知到卡顿不流畅，是因为这里容易发生丢帧的现象。有很多原因可以导致丢帧，也许是因为你的layout太过复杂，无法在16ms内完成渲染，有可能是因为你的UI上有层叠太多的绘制单元，还有可能是因为动画执行的次数过多。这些都会导致CPU或者GPU负载过重。

我们可以通过一些工具来定位问题，比如可以使用HierarchyViewer来查找Activity中的布局是否过于复杂，也可以使用手机设置里面的开发者选项，打开Show GPU Overdraw等选项进行观察。你还可以使用TraceView来观察CPU的执行情况，更加快捷的找到性能瓶颈。

## 1)Understanding Overdraw
如果我们曾经粉刷过家里的墙壁，可能会遇到之前刷过的区域再重刷一遍的情况，这就是重绘。同样的，在App里面也会有这样的问题。

Overdraw(重绘)描述的是屏幕上的某个像素在同一帧的时间内被绘制了多次。在多层次的UI结构里面，如果不可见的UI也在做绘制的操作，这就会导致某些像素区域被重绘了多次。这也浪费大量的CPU/GPU资源。

![](/images/overdraw_hidden_view.png)

当设计上追求更华丽的视觉效果的时候，我们就容易陷入采用越来越多的层叠组件来实现效果的怪圈。这很容易导致大量的性能问题，为了获得最佳的性能，我们必须尽量减少重绘的情况发生。

幸运的是，我们可以通过手机设置的开发者选项，打开显示GPU重绘的选项，可以观察UI上的重绘情况。

![](/images/overdraw_options_view.png)

蓝色，淡绿，淡红，深红代表了4种不同程度的重绘情况，我们的目标就是尽量减少红色重绘，看到更多的蓝色区域。

重绘有时候是因为你的UI布局存在大量重叠的部分，还有的时候是因为非必须的重叠背景。例如某个Activity有一个背景，然后里面的Layout又有自己的背景，同时子View又分别有自己的背景。仅仅是通过移除非必须的背景图片，这能够减少大量的红色重绘区域，增加蓝色区域的占比。这能够提示不少的程序性能。

## 2)Understanding VSYNC
为了理解App是如何进行渲染的，我们必须了解手机硬件是如何工作，这样的话，我们就必须理解什么是*VSYNC*。

在理解VSYNC之前，我们需要了解两个相关的概念：

*	Refresh Rate：代表了屏幕能够在一秒内刷新屏幕的次数，这取决于硬件的固定参数，例如60Hz。 
*	Frame Rate：代表了GPU能够在一秒内绘制多少帧，例如30fps，60fps。

GPU会获取图形数据进行渲染，然后硬件负责把渲染后的内容呈现到屏幕上，他们两者不停的进行协作。

![](/images/vsync_gpu_hardware.png)

不幸的是，刷新频率和帧率并不是总能够保持相同的节奏。如果发生帧率与刷新频率不一致的情况，就会容易出现**Tearing**的现象(画面上下两部分显示内容发生断裂，来自不同的两帧数据发生重叠)。

![](/images/vsync_gpu_hardware_not_sync.png)

理解图像渲染里面的双重与三重缓存机制，请移步查看这里：<http://source.android.com/devices/graphics/index.html>，这个概念比较复杂。

![](/images/vsync_buffer.png)

通常来说，帧率超过刷新频率是一种理想的状况，在超过60fps的情况下，GPU所产生的帧数据会因为等待VSYNC的刷新信息而被Hold住，这样能够保持每次刷新都有实际的新的数据可以显示。但是我们遇到更多的情况是帧率小于刷新频率。

![](/images/vsync_gpu_hardware_not_sync2.png)

在这种情况下，某些帧显示的画面内容就会与上一帧的画面相同。糟糕的事情是，帧率从超过60fps突然掉到60fps以下，这样就会发生**LAG**，**JANK**，**HITCHING**等卡顿掉帧的不顺滑的情况。这也是用户感受不好的原因所在。

## 3)Tool:Profile GPU Rendering
性能问题如此的麻烦，辛好我们可以有工具来进行调试。打开手机里面的开发者选项，选择Profile GPU Rendering，选中On screen as bars的选项。

![](/images/tools_gpu_profile_rendering.png)

选择了这样以后，我们可以在手机画面上看到丰富的GPU绘制图形信息，分别关于StatusBar，NavBar，Activity区域的信息。

![](/images/tools_gpu_profile_rendering_graphic_activity.png)



























