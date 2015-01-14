---
layout: post
title: "Android性能优化必须知道的16点"
date: 2015-01-13 21:57
comments: true
sidebar: false
categories: Android Android:Deeper
---

Google近期发布了关于[Android性能问题的专题](https://www.youtube.com/playlist?list=PLWz5rJ2EKKc9CBxr3BVjPTPoDPLdPIFCE)，一共16个短视频，帮助开发者创建更快更流畅的Android App。一方面介绍了Android系统的工作原理，同时也介绍了如何通过工具来找出性能问题以及处理建议。下面是对这些问题和建议的总结。

## Render Performance
大多数用户感知到的卡顿等性能问题的最主要根源都来是因为渲染问题。从设计师的角度，他们希望App能够有更多的动画，图片等流畅的体验。