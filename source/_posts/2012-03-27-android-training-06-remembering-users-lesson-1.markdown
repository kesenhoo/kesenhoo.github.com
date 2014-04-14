---
layout: post
title: "Android Training(06) - 记住用户的信息(Lesson 1 - 使用AccountManager来记录用户)"
date: 2012-03-27 11:12
comments: true
sidebar: false
categories: Android Android:Training
---

**章节概览:Remembering Users**

Android用户希望把自己的信息绑定到喜欢的app与设备上，那么使得你的程序更加令人喜爱的一个方法是使得它更加的人性化。Android设备知道你的使用者是谁，他们都使用过哪些服务，在哪里存储了你的数据。在得到你的用户授权的前提下，你可以使用那些信息来使得你的app更加丰富，更加人性化。

在这一章节，你将学习到鉴定用户信息的多种技术，你可以:  

* 通过记住用户账户名来人性化你的appd。
* 鉴定用户以确保是否是那个人。
* 通过service来获取访问用户线上数据的权限，例如Google APIs。
* 增加一个自定义的账户来鉴定自己后台的服务。

<!-- More -->

**Requirements and prerequisites**

* Android 2.0 (API level 5) or higher
* Experience with [Services](http://developer.android.com/guide/topics/fundamentals/services.html)
* Experience with [OAuth 2.0](http://oauth.net/2/)

**You should also read**

[SampleSyncAdapter app](http://developer.android.com/resources/samples/SampleSyncAdapter/index.html)

每个人都很喜欢自己的名字能被人记住。其中最简单，最有效的使得你的app让人喜欢的方法是记住你的用户是谁，特别是当用户升级到一台新的设备或者是在tablet希望能够像在手机上一样使用(存有同样的数据，比如书签等)。但是如何知道用户是谁，如何在新的设备上识别出他们。

对于许多程序来说，可以使用AccountManager APIs来处理上面的问题。在用户授权下，你可以使用AccountManager来获取用户存储在设备上的账户名。

整合用户的账户，这样可以使得你可以做许多事情，例如:

* 自动填写用户的email地址。  
* 获取绑定到用户的ID，而不是绑定到设备的。

## (1)Determine if AccountManager for You
程序通常使用下面三个方法之一来尝试记住用户:

* (a)通知用户输入用户名. 
* (b)取得一个唯一的ID来记住设备.
* (c)从AccountManager取得一个嵌入的账户.

选项(a):是有问题的。第一，在进入app之前通知用户来输入些什么，这会使得app不受欢迎[当然需要排除首次登入]，第二，那不能保证用户名的唯一性[可能的前提是说某个app固定显示某个用户的信息，而不需要进行切换。这个理解起来有点怪怪的]。

选项(b):对于用户来说稍微简单点，但是有点投机取巧的味道。更重要的是，这仅仅使得用户只能在某个设备上被识别，当用户升级到新的设备上时，会导致app不再记得那些用户。

选项(c):是比较好的。Account Manager允许你获取存储在用户设备上的账户信息。下面我们会学习到使用AccountManager来记住用户，不管用户有多少的设备，仅仅需要几步额外的操作就可以达到同步目的。
【老外写文章习惯就是这样，讲某个技术之前，说一大堆为什么选择这个技术，而不是选择其他的方法。这种精神很值得我们学习，先问WHY？而不只立马拿来灌输】

## (2)Decide What Type of Account to Use
Android设备可以根据许多不同的提供者来存储多个不同类型的账户。当你为了某个账户名而使用AcccountManager进行查询的时候，可以选择使用Account Type来filter。账户类型是一个唯一标识已经发布账户的String。例如，Google账户使用“com.google”，Twitter使用“com.twitter.android.auth.login”。

## (3)Request GET_ACCOUNT permission
为了获得在设备上所有的账户列表，你的app需要有GET_ACCOUNTS权限，使用<uses-permission>标签在manifest文件中来添加请求权限。
```xml
<manifest ... >  
    <uses-permission android:name="android.permission.GET_ACCOUNTS" />  
    ...  
</manifest>  
```

## (4)Query AccountManager for a List of Accounts
一旦你决定需要查询哪些账户了，可以像下面的例子一样来获得一个Account的数组，里面均是与类型符合的账户信息。
```java
AccountManager am = AccountManager.get(this); // "this" references the current Context  
  
Account[] accounts = am.getAccountsByType("com.google");  
```
如果在数组里面不止一个账户，你需要先呈现出一个对话框来让用户选择其中一个。

## (5)Use the Account Object to Personalize Your App
Account对象里面包含了账户名(对于Google账户来说是一个邮件地址)。你可以使用这个信息来做不同的事情，例如：
  
* 在填写表格的时候给出对应的提示，这样的话用户就不用手动输入完整的账户信息。
* 作为你自己线上数据库的使用与个性化信息的关键字。

## (6)Decide Whether an Account Name is Enough
账户名是记住用户的一个好方法，但是Account对象本身并不会保护你的数据或者让你访问除账户名本身之外的任何东西。
如果你的app需要允许用户到线上访问私人数据，你需要一些更加强大的东西：authentication。
下一节课会解释如何通过线上服务来鉴定当前用户，如何自定义的一个认证机制，这样使得可以安装自定义的账户。[也就是OAuth2的使用]。

***
**学习自：[http://developer.android.com/training/id-auth/identify.html](http://developer.android.com/training/id-auth/identify.html)，请多指教，谢谢！**  
**转载请注明出自[http://kesenhoo.github.com](http://kesenhoo.github.com)，谢谢配合！**
