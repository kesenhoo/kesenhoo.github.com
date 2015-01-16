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
如果我们曾经粉刷过家里的墙壁，可能会遇到之前刷过的区域再重刷一遍的情况，这就是Overdraw。同样的，在App里面也会有这样的问题。

Overdraw(过度绘制)描述的是屏幕上的某个像素在同一帧的时间内被绘制了多次。在多层次的UI结构里面，如果不可见的UI也在做绘制的操作，这就会导致某些像素区域被绘制了多次。这也浪费大量的CPU/GPU资源。

![](/images/overdraw_hidden_view.png)

当设计上追求更华丽的视觉效果的时候，我们就容易陷入采用越来越多的层叠组件来实现效果的怪圈。这很容易导致大量的性能问题，为了获得最佳的性能，我们必须尽量减少Overdraw的情况发生。

幸运的是，我们可以通过手机设置的开发者选项，打开Show GPU Overdraw的选项，可以观察UI上的Overdraw情况。

![](/images/overdraw_options_view.png)

蓝色，淡绿，淡红，深红代表了4种不同程度的Overdraw情况，我们的目标就是尽量减少红色Overdraw，看到更多的蓝色区域。

Overdraw有时候是因为你的UI布局存在大量重叠的部分，还有的时候是因为非必须的重叠背景。例如某个Activity有一个背景，然后里面的Layout又有自己的背景，同时子View又分别有自己的背景。仅仅是通过移除非必须的背景图片，这能够减少大量的红色Overdraw区域，增加蓝色区域的占比。这能够提示不少的程序性能。

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

选择了这样以后，我们可以在手机画面上看到丰富的GPU绘制图形信息，分别关于StatusBar，NavBar，激活的程序Activity区域的信息。

![](/images/tools_gpu_profile_rendering_graphic_activity.png)

随着界面的刷新，界面上会滚动显示垂直的柱状图来表示每帧画面所需要渲染的时间，柱状图越高表示花费的渲染时间越长。

![](/images/tools_gpu_rendering_bar.png)

中间有一根绿色的横线，代表16ms，我们需要确保每一帧花费的总时间都低于这条横线，这样才能够避免出现卡顿的问题。

![](/images/tools_gpu_profile_three_color.png)

每一条柱状线都包含三部分，蓝色代表测量绘制Display List的时间，红色代表OpenGL渲染Display List所需要的时间，黄色代表CPU等待GPU处理的时间。

## 4)Why 60fps?
我们通常都会提到60fps与16ms，可是知道为何会是以程序是否达到60fps来作为App性能的衡量标准吗？这是因为人眼与大脑之间的协作无法感知超过60fps的画面更新。

12fps大概类似手动快速翻动书籍的帧率，这明显是可以感知到不够顺滑的。24fps会使得人眼感到的是流体运动，这其实是归功于运动模糊的效果。24fps是电影胶圈通常使用的帧率，因为这个帧率已经足够支撑大部分电影画面需要表达的内容，同时能够最大的减少费用支出。但是低于30fps是无法顺畅表现绚丽的画面内容的，此时就需要用到60fps来达到想要的效果，当然超过60fps是没有必要的。

开发app的性能目标就是保持60fps，这意味着每一帧你只有16ms=1000/60的时间来处理所有的任务。

## 5)Android, UI and the GPU
了解Android是如何利用GPU运作有助于我们更好的理解性能问题。那么一个最实际的问题是：activity的画面是如何绘制到屏幕上的？那些复杂的XML布局文件又是如何能够被识别并绘制出来的？

![](/images/gpu_rasterization.png)

栅格化是绘制那些Button，Shape，Path，String，Bitmap等组件最基础的操作。它把那些组件拆分到不同的像素上进行显示。栅格化是一个很费时的操作，GPU的引入就是为了加快栅格化的操作。

CPU负责把界面组件计算成Texture纹理，然后交给GPU进行栅格化渲染。

![](/images/gpu_cpu_rasterization.png)

然而每次从CPU转移到GPU是一件很麻烦的事情，所幸的是OpenGL ES可以把那些需要渲染的纹理Hold在GPU Memory里面，在下次需要渲染的时候直接进行操作。所以如果你更新了GPU所hold住的纹理内容，那么之前保存的状态就丢失了。

在Android里面那些由主题所提供的资源，例如Bitmaps，Drawables都是一起打包成统一的Texture纹理当中，然后再传递到GPU里面，这意味着每次你需要使用这些资源的时候，都是直接从纹理里面进行获取渲染的。当然随着UI组件的越来越丰富，有了更多演变的形态。例如显示图片的时候，需要先经过CPU的计算加载到内存中，然后传递给GPU进行渲染。文字的显示更加复杂，需要先经过CPU换算成纹理，然后再交给GPU进行渲染，回到CPU绘制单个字符的时候，再重新引用经过GPU渲染的内容。动画则是一个更加复杂的操作流程。

为了能够使得App流畅，我们需要在每一帧16ms以内处理完所有的CPU与GPU计算，绘制，渲染等等操作。

## 6)Invalidations, Layouts, and Performance
顺滑精妙的动画是app设计里面最重要的元素之一，这些动画能够显著提升用户体验。下面会讲解Android系统是如何处理UI组件的更新操作的。

通常来说，Android需要把XML布局文件转换成GPU能够识别并绘制的对象。这个操作是在**DisplayList**的帮助下完成的。DisplayList持有所有将要交给GPU绘制到屏幕上的数据信息。

在某个View第一次需要被渲染时，DisplayList会因此而被创建，当这个View要显示到屏幕上时，我们会执行GPU的绘制指令来进行渲染。如果你在后续有执行类似移动这个View的位置等操作而需要再次渲染这个View时，我们就仅仅需要额外操作一次渲染指令就够了。然而如果你修改了View中的某些可见组件，那么之前的DisplayList就无法继续使用了，我们需要回头重新创建一个DisplayList并且重新执行渲染指令并更新到屏幕上。需要注意的是：任何时候View中的绘制内容发生变化时，都会重新执行创建DisplayList，渲染DisplayList，更新到屏幕上的一系列操作。这个流程的表现性能取决于你的View的复杂程度，View的状态变化以及渲染管道的执行性能。举个例子，假设某个Button的大小需要增大到目前的两倍，在增大Button大小之前，需要通过父View重新计算并摆放其他子View的位置。修改View的大小会触发整个HierarcyView的重新计算大小的操作。如果是修改View的位置则会触发HierarchView重新计算其他View的位置。如果布局很复杂，这就会很容易导致严重的性能问题。我们需要尽量减少不必要的Overdraw。

![](/images/layout_three_steps.png)

我们可以通过前面介绍的Monitor GPU Rendering来查看渲染的表现性能如何，另外也可以通过开发者选项里面的Show GPU view updates来查看视图更新的操作，最后我们还可以通过HierarchyViewer这个工具来查看布局，使得布局尽量扁平化，移除非必需的UI组件，这些操作能够减少Measure，Layout的计算时间。

## 7)Overdraw, Cliprect, QuickReject
引起性能问题的一个很重要的方面是因为过多复杂的绘制操作。我们可以通过工具来检测并修复标准UI组件的Overdraw问题，但是针对高度自定义的UI组件则显得有些力不从心。

有一个窍门是我们可以通过执行几个APIs方法来显著提升绘制操作的性能。前面有提到过，非可见的UI组件进行绘制更新会导致Overdraw。例如Nav Drawer从前置可见的Activity滑出之后，如果还继续绘制那些在Nav Drawer里面不可见的UI组件，这就导致了Overdraw。为了解决这个问题，Android系统会通过避免绘制那些完全不可见的组件来尽量减少Overdraw。那些Nav Drawer里面不可见的View就不会被执行浪费资源。

![](/images/overdraw_invisible.png)

但是不幸的是，对于那些过于复杂的自定义的View(重写了onDraw方法)，Android系统无法检测具体在onDraw里面会操作的事情，这样就无法监控，也就无法避免Overdraw了。但是我们可以通过[canvas.clipRect()](http://developer.android.com/reference/android/graphics/Canvas.html)来帮助系统识别那些可见的区域。这个方法可以指定一块矩形区域，只有在这个区域内才会被绘制，其他的区域会被忽视。这个API可以很好的帮助那些有多组重叠组件的自定义View来控制显示的区域。同时clipRect方法还可以帮助节约CPU与GPU资源，在clipRect区域之外的绘制指令都不会被执行，那些部分内容在矩形区域内的组件，仍然会得到绘制。

![](/images/overdraw_reduce_cpu_gpu.png)

除了clipRect方法之外，我们还可以使用[canvas.quickreject()](http://developer.android.com/reference/android/graphics/Canvas.html)来判断是否没和某个矩形相交，从而跳过那些非矩形区域内的绘制操作。做了那些优化之后，我们可以通过上面介绍的Show GPU Overdraw来查看效果。

## 8)Memory Churn and performance











































