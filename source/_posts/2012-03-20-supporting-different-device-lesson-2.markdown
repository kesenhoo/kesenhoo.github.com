---
layout: post
title: "Android Training 02 - 适配不同的屏幕[Lesson 2 - 适配不同屏幕密度]"
date: 2012-03-20 20:32
comments: true
sidebar: false
categories: Android Android:Training
---

# Supporting Different Densities
上一篇文章和大家分享了如何适配不同大小的屏幕，有个概念需要提前弄清楚，屏幕大的不一定就分辨率高，详细请看下面的内容。  
This lesson shows you how to support different screen densities by providing different resources and using resolution-independent units of measurements.  
我们需要通过提供不同的resources来support不同的屏幕密度，使用一种独立与分辨率的测量单元来表示（也就是dp）  

## Use Density-independent Pixels[使用设备独立像素dp/sp]
前一节，我们就提到过，一定要避免使用绝对pixels(像素)的方式来设计layout的大小或者距离，因为不同的的屏幕有不同的像素密度，所以相同的像素在不同的设备上物理大小会有区别。通常我们都会在需要的时候使用dp，或者sp来表示大小

<!-- more -->

* dp：A dp is a density-independent pixel that corresponds to the physical size of a pixel at 160 dpi(dots per inch:每英寸点数). 
* dp也就是dip:device independent pixels(设备独立像素)
* dp是一种与密度无关的像素单位，在每英寸160点的屏幕上，1dp = 1px

不同设备有不同的显示效果,这个和设备硬件有关，一般我们为了支持WVGA、HVGA和QVGA 推荐使用这个，不依赖像素
```xml
<Button android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@string/clickme"
    android:layout_marginTop="20dp" />
```

* sp：An sp is the same base unit, but is scaled by the user's preferred text size (it’s a scale-independent pixel), so you should use this measurement unit when defining text size (but never for layout sizes).
* scaled pixels(刻度像素). 主要用于定义字体的大小，而从来不再layout上使用
```xml
<TextView android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:textSize="20sp" />
```
* px：pixels(像素). 不同设备显示效果相同，一般我们HVGA代表320x480像素，这个用的比较多

**总结**：dp也就是dip。这个和sp基本类似。如果设置表示长度、高度等属性时可以使用dp或sp。但如果设置字体，需要使用sp。dp是与密度无关，sp除了与密度无关外，还与scale无关。如果屏幕密度为160，这时dp和sp和px是一样的。1dp=1sp=1px，但如果使用px作单位，如果屏幕大小不变（假设还是3.2寸），而屏幕密度变成了320。那么原来TextView的宽度设成160px，在密度为320的3.2寸屏幕里看要比在密度为160的3.2寸屏幕上看短了一半。但如果设置成160dp或160sp的话。系统会自动将width属性值设置成320px的。也就是160 * 320 / 160。其中320 / 160可称为密度比例因子。也就是说，如果使用dp和sp，系统会根据屏幕密度的变化自动进行转换.(百度百科：http://baike.baidu.com/view/416780.htm#sub5084586)

## Provide Alternative Bitmaps [提供可选择的图片]
因为需要适配不同屏幕，我们需要提供不同的图片来适配，这样才能带来更好的用户体验。通常我们需要提供下面的资源图片来适配

* xhdpi: 2.0
* hdpi: 1.5
* mdpi: 1.0 (baseline)
* ldpi: 0.75

这意味着如果我们为xhdpi的设备生成了一张200x200的图片，同时也需要为hdpi的设备生成150x150的图片，为mdpi的设备生成100x100的图片，最后为ldpi的设备生成75x75的图片。需要像下面一样来放置那些特殊适配的文件

res/
    drawable-xhdpi/
        awesomeimage.png
    drawable-hdpi/
        awesomeimage.png
    drawable-mdpi/
        awesomeimage.png
    drawable-ldpi/
        awesomeimage.png

这样之后，在任何地方引用@drawable/awesomeimage，系统都会自动根据当前设备的dpi来选择合适的图片进行显示。


*********************************
**学习自：[http://developer.android.com/training/multiscreen/screendensities.html](http://developer.android.com/training/multiscreen/screendensities.html)，请多指教，谢谢！**  
**转载请注明出自[http://kesenhoo.github.com](http://kesenhoo.github.com)，谢谢配合！**






