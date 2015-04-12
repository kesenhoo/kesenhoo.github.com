---
layout: post
title: "Android性能优化之渲染篇"
date: 2015-04-11 22:16
comments: true
sidebar: false
categories: Android Android:Deeper
---

![](/images/android_performance_course_udacity.jpg)

Google近期在Udacity上发布了[Android性能优化的在线课程](https://www.udacity.com/course/ud825)，目前有三个篇章，分别从渲染，运算与内存，电量三个方面介绍了如何去优化性能，这些课程是Google之前在Youtube上发布的[Android性能优化典范](http://hukai.me/android-performance-patterns/)专题课程的细化与补充。

下面是渲染篇章的学习笔记，部分内容和前面的性能优化典范有重合，欢迎大家一起学习交流！

### 1)Why Rendering Performance Matters
现在有不少App为了达到很华丽的视觉效果，会需要在界面上层叠很多的视图组件，但是这会很容易引起性能问题。如何平衡Design与Performance就很需要智慧了。

### 2)Defining 'Jank'
大多数手机的屏幕刷新频率是60hz，如果在1000/60=16.67ms内没有办法把这一帧的任务执行完毕，就会发生丢帧的现象。丢帧越多，用户感受到的卡顿情况就越严重。

![](/images/android_performance_course_drop_frame.png)

### 3)Rendering Pipeline: Common Problems
渲染操作通常依赖于两个核心组件：CPU与GPU。CPU负责包括Measure，Layout，Record，Execute的计算操作，GPU负责Rasterization(栅格化)操作。CPU通常存在的问题的原因是存在非必需的视图组件，它不仅仅会带来重复的计算操作，而且还会占用额外的GPU资源。

<!-- More -->

![](/images/android_performance_course_render_problems.jpg)

### 4)Android UI and the GPU
了解Android是如何利用GPU进行画面渲染有助于我们更好的理解性能问题。一个很直接的问题是：activity的画面是如何绘制到屏幕上的？那些复杂的XML布局文件又是如何能够被识别并绘制出来的？

![](/images/gpu_rasterization.png)

**Resterization栅格化**是绘制那些Button，Shape，Path，String，Bitmap等组件最基础的操作。它把那些组件拆分到不同的像素上进行显示。这是一个很费时的操作，GPU的引入就是为了加快栅格化的操作。

CPU负责把UI组件计算成Polygons，Texture纹理，然后交给GPU进行栅格化渲染。

![](/images/gpu_cpu_rasterization.png)

然而每次从CPU转移到GPU是一件很麻烦的事情，所幸的是OpenGL ES可以把那些需要渲染的纹理Hold在GPU Memory里面，在下次需要渲染的时候直接进行操作。所以如果你更新了GPU所hold住的纹理内容，那么之前保存的状态就丢失了。

在Android里面那些由主题所提供的资源，例如Bitmaps，Drawables都是一起打包到统一的Texture纹理当中，然后再传递到GPU里面，这意味着每次你需要使用这些资源的时候，都是直接从纹理里面进行获取渲染的。当然随着UI组件的越来越丰富，有了更多演变的形态。例如显示图片的时候，需要先经过CPU的计算加载到内存中，然后传递给GPU进行渲染。文字的显示比较复杂，需要先经过CPU换算成纹理，然后交给GPU进行渲染，返回到CPU绘制单个字符的时候，再重新引用经过GPU渲染的内容。动画则存在一个更加复杂的操作流程。

为了能够使得App流畅，我们需要在每帧16ms以内处理完所有的CPU与GPU的计算，绘制，渲染等等操作。

### 5)GPU Problem: Overdraw
Overdraw(过度绘制)描述的是屏幕上的某个像素在同一帧的时间内被绘制了多次。在多层次重叠的UI结构里面，如果不可见的UI也在做绘制的操作，会导致某些像素区域被绘制了多次。这样就会浪费大量的CPU以及GPU资源。

![](/images/overdraw_hidden_view.png)

当设计上追求更华丽的视觉效果的时候，我们就容易陷入采用复杂的多层次重叠视图来实现这种视觉效果的怪圈。这很容易导致大量的性能问题，为了获得最佳的性能，我们必须尽量减少Overdraw的情况发生。

幸运的是，我们可以通过手机设置里面的开发者选项，打开Show GPU Overdraw的选项，观察UI上的Overdraw情况。

![](/images/overdraw_options_view.png)

蓝色，淡绿，淡红，深红代表了4种不同程度的Overdraw情况，我们的目标就是尽量减少红色Overdraw，看到更多的蓝色区域。

### 6)Visualize and Fix Overdraw - Quiz & Solution
这里举了一个例子，通过XML文件可以看到有好几处非必需的background。通过把XML中非必需的background移除之后，可以显著减少布局的过度绘制。其中一个比较有意思的地方是：针对ListView中的Avatar ImageView的设置，在getView的代码里面，判断是否获取到对应的Bitmap，在获取到Avatar的图像之后，把ImageView的Background设置为Transparent，只有当图像没有获取到的时候才设置对应的Background占位图片，这样可以避免因为给Avatar设置背景图而导致的过度渲染。

![](/images/android_perf_course_overdraw_compare.png)

总结一下，优化步骤如下：

* 移除Window默认的Background
* 移除XML布局文件中非必需的Background
* 按需显示占位背景图片

### 7)ClipRect & QuickReject
前面有提到过，对不可见的UI组件进行绘制更新会导致Overdraw。例如Nav Drawer从前置可见的Activity滑出之后，如果还继续绘制那些在Nav Drawer里面不可见的UI组件，这就导致了Overdraw。为了解决这个问题，Android系统会通过避免绘制那些完全不可见的组件来尽量减少Overdraw。那些Nav Drawer里面不可见的View就不会被执行浪费资源。

![](/images/overdraw_invisible.png)

但是不幸的是，对于那些过于复杂的自定义的View(通常重写了onDraw方法)，Android系统无法检测在onDraw里面具体会执行什么操作，系统无法监控并自动优化，也就无法避免Overdraw了。但是我们可以通过[canvas.clipRect()](http://developer.android.com/reference/android/graphics/Canvas.html)来帮助系统识别那些可见的区域。这个方法可以指定一块矩形区域，只有在这个区域内才会被绘制，其他的区域会被忽视。这个API可以很好的帮助那些有多组重叠组件的自定义View来控制显示的区域。同时clipRect方法还可以帮助节约CPU与GPU资源，在clipRect区域之外的绘制指令都不会被执行，那些部分内容在矩形区域内的组件，仍然会得到绘制。

![](/images/overdraw_reduce_cpu_gpu.png)

除了clipRect方法之外，我们还可以使用[canvas.quickreject()](http://developer.android.com/reference/android/graphics/Canvas.html)来判断是否没和某个矩形相交，从而跳过那些非矩形区域内的绘制操作。

### 8)Apply clipRect and quickReject - Quiz & Solution

![](/images/android_perf_course_clip_1.png)

上面的示例图中显示了一个自定义的View，主要效果是呈现多张重叠的卡片。这个View的onDraw方法如下图所示：

![](/images/android_perf_course_clip_3.png)

打开开发者选项中的显示过度渲染，可以看到我们这个自定义的View部分区域存在着过度绘制。那么是什么原因导致过度绘制的呢？

![](/images/android_perf_course_clip_2.png)

### 9)Fixing Overdraw with Canvas API
下面的代码显示了如何通过clipRect来解决自定义View的过度绘制，提高自定义View的绘制性能：

![](/images/android_perf_course_clip_code_compare.png)

下面是优化过后的效果：

![](/images/android_perf_course_clip_result.png)

### 10)Layouts, Invalidations and Perf
Android需要把XML布局文件转换成GPU能够识别并绘制的对象。这个操作是在**DisplayList**的帮助下完成的。DisplayList持有所有将要交给GPU绘制到屏幕上的数据信息。

在某个View第一次需要被渲染时，Display List会因此被创建，当这个View要显示到屏幕上时，我们会执行GPU的绘制指令来进行渲染。

如果View的Property属性发生了改变（例如移动位置），我们就仅仅需要Execute Display List就够了。

![](/images/android_perf_course_displaylist_execute.png)

然而如果你修改了View中的某些可见组件的内容，那么之前的DisplayList就无法继续使用了，我们需要重新创建一个DisplayList并重新执行渲染指令更新到屏幕上。

![](/images/android_perf_course_displaylist_invalidation.png)

请注意：任何时候View中的绘制内容发生变化时，都会需要重新创建DisplayList，渲染DisplayList，更新到屏幕上等一系列操作。这个流程的表现性能取决于你的View的复杂程度，View的状态变化以及渲染管道的执行性能。举个例子，假设某个Button的大小需要增大到目前的两倍，在增大Button大小之前，需要通过父View重新计算并摆放其他子View的位置。修改View的大小会触发整个HierarcyView的重新计算大小的操作。如果是修改View的位置则会触发HierarchView重新计算其他View的位置。如果布局很复杂，这就会很容易导致严重的性能问题。

![](/images/android_perf_course_displaylist_kick_off.png)

### 11)Hierarchy Viewer: Walkthrough
Hierarchy Viewer可以很直接的呈现布局的层次关系，视图组件的各种属性。
我们可以通过红，黄，绿三种不同的颜色来区分布局的Measure，Layout，Executive的相对性能表现如何。

### 12)Nested Hierarchies and Performance
提升布局性能的关键点是尽量保持布局层级的扁平化，避免出现重复的嵌套布局。例如下面的例子，有2行显示相同内容的视图，分别用两种不同的写法来实现，他们有着不同的层级。

![](/images/android_perf_course_hierarchy_1.png)

![](/images/android_perf_course_hierarchy_2.png)

下图显示了使用2种不同的写法，在Hierarchy Viewer上呈现出来的性能测试差异：

![](/images/android_perf_course_hierarchy_3.png)

### 13)Optimizing Your Layout
下图举例演示了如何优化ListItem的布局，通过RelativeLayout替代旧方案中的嵌套LinearLayout来优化布局。

![](/images/android_perf_course_hierarchy_4.png)








