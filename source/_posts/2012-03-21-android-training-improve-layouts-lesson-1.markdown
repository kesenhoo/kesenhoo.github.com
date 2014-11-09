---
layout: post
title: "Android Training - 提升布局文件的性能(Lesson 1 - 优化布局的层级)"
date: 2012-03-21 17:14
comments: true
sidebar: false
categories: Android Android:Training
---

## Optimizing Layout Hierarchies
Layout是Android程序影响用户体验最关键的一部分。如果布局文件不好会使得程序比较卡。SDK里面包含了一些工具用来帮助我们发现布局文件的性能问题

使用基本的Layout结构是最有效的。但是，每一个添加到系统的组件都需要初始化，进行布局，绘制的过程。比如，使用在LinearLayout里面使用子组件会导致一个过于deep的层级结构。而且内嵌使用包含layout_weight属性的LinearLayout会在绘制时花费昂贵的系统资源，因为每一个子组件都需要被测量两次。在使用ListView与GridView的时候，这个问题显的尤其重要，因为子组件会重复被创建

这一课我们会学习使用Hierarchy Viewer and Lint 来检查并最优化布局文件。

<!-- More -->

## 1)Inspect Your Layout
Android SDK里面包含了一个叫做[Hierarchy Viewer](http://developer.android.com/tools/help/hierarchy-viewer.html)的工具，在程序运行的时候分析布局文件，从而找到性能瓶颈

连接上设备，打开Hierarchy Viewer(定位到tools/目录下，直接执行hierarchyviewer的命令，选定需要查看的Process，再点击Load View Hierarchy会显示出当前界面的布局Tree。在每个模块的Traffic light上有三个灯，分别代表了Measure, Layout and Draw三个步骤的性能。

![layout-listitem.png](/images/articles/layout-listitem.png "Figure 1. ListView每个Item的常见布局.")

![hierarchy-linearlayout.png](/images/articles/hierarchy-linearlayout.png "Figure 2. 上面显示了对应与图片1的布局层级信息.")

可以看到中间LinearLayout的Measure的灯是红色的，这就是因为上面说到的：使用内嵌layout_weight的属性的LinearLayout会导致测量时花费了双倍的时间。

![hierarchy-layouttimes.png](/images/articles/hierarchy-layouttimes.png "Figure 3. 点击某个模块会显示具体每个步骤所花费的时间。")

## 2)Revise Your Layout
使得Layout宽而浅，而不是窄而深（在Hierarchy Viewer的Tree视图里面体现）

针对上面的布局我们可以使用RelativeLayout来替代LinearLayout，从而实现shallow and wide.

![hierarchy-relativelayout.png](/images/articles/hierarchy-relativelayout.png "Figure 4. 改用RelativeLayout来实现图片1的布局。")

可以看到这是一个小的优化，可是这带来的效果是明显的，因为在ListView里面会出现很多这样的布局。
导致前面的case会出现花费时间比较多的愿意是使用了layout_weight在LinearLayout。我们需要仔细评估到底是否需要使用那样的布局，尽量避免使用layout_weight。

## 3)Use Lint
[Lint](http://tools.android.com/tips/lint)是一款在ADT 16才出现用来替代layoutopt的新型工具，具有更强大的功能。

Lint的会提示的一些建议规则如下：

* 使用compound drawables - 一个包含了ImageView与TextView的LinearLayout可以被当作一个compound drawable来处理。
* 使用merge根框架 - 如果FramLayout仅仅是一个纯粹的（没有设置背景，间距等）布局根元素，我们可以使用merge标签来当作根标签。
* 无用的分支 - 如果一个layout并没有任何子组件，那么可以被移除，这样可以提高效率。
* 无用的父控件 - 如果一个layout只有子控件，没有兄弟控件，并且不是一个ScrollView或者根节点，而且没有设置背景，那么我们可以移除这个父控件，直接把子控件提升为父控件】
* 深层次的layout - 尽量减少内嵌的层级，考虑使用更多平级的组件 RelativeLayout or GridLayout来提升布局性能，默认最大的深度是10

![lint_icon.png](/images/articles/lint_icon.png)

Eclipse会自动运行Lint的工具，并给出相应的提醒，不管是在导出APK，编辑，保存XML还是在使用layout编辑器的时候。如果想驱动一次运行，请参看上面的图标，点击运行。如果没有安装ADT 16,需要在命令行中执行。

***
**文章学习自[http://developer.android.com/training/improving-layouts/optimizing-layout.html](http://developer.android.com/training/improving-layouts/optimizing-layout.html)**
**转载请注明出自[http://kesenhoo.github.com](http:://kesenhoo.github.com)，谢谢**

