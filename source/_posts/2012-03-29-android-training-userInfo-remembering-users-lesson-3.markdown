---
layout: post
title: "Android Training[UserInfo] - 记住用户的信息(Lesson 3 - 创建自定义的账户类型)"
date: 2012-03-29 12:37
comments: true
sidebar: false
categories: Android Android:Training
---

## Creating a Custom Account Type
到目前为止，我们讨论了如何使用Google APIs。但是我们应该不仅仅是只需要Google的服务而已，比如增加QQ账户，Sina账户等。那么这一课会讲述如何创建一个自定义的账户，并且像内置的账户那样进行工作。

## (1)Implement Your Custom Account Code
首先需要做的是从用户那获取证书[输入账户与密码后进行验证],这个过程也许只是简单的显示一个对话框来输入用户名与密码，或者是比较复杂的操作来获取证书，比如一次性的密码口令或者精密的扫描。不管怎么样，你需要实现下面的操作:

<!-- More -->

* 从用户那收集账户与密码。  
* 连接到server进行验证。  
* 把获得的证书存储到设备上。  

上面三个请求通常能够在一个Activity上实现，我们叫这个Acitivity为Authenticator activity。
因为需要与AccountManager系统进行交互，authenticator activity需要比通常的activity多做一些特定的请求。为了使得这个过程简单化，Android framework提供了一个[AccountAuthenticatorActivity](http://developer.android.com/reference/android/accounts/AccountAuthenticatorActivity.html)来给用户进行扩展并创建自定义的authenticator。

前面两个操作，都需要你来完成操作[邀请用户输入信息并进行验证]，第三个步骤通常像下面一样
```java
final Account account = new Account(mUsername, your_account_type);  
mAccountManager.addAccountExplicitly(account, mPassword, null);  
```

## (2)Be Smart About Security!
请注意，AccountManager里面的账户信息是没有加密的。他仅仅是使用plain text的方式[明文方式]来存储那些账户信息。在大多数设备上，那不是特别的严重，因为他存储那些信息在数据库中，而这些数据只能是有ROOT权限的才能访问。但是在已经有root权限的设备上，证书信息可以通过adb来被任何人进行访问。

请注意这个问题，你不应该使用AccountManager.addAccountExplicitly()方法来传递真实的密码。你应该使用暗文加密的方式来存储账户信息。[显然安全性是很重要的，最近爆发的大量密码泄露事件又重新提醒了开发者需要注意这些问题，听说之前CSDN就是明文存储了用户的账户信息]

请注意：当面对安全性问题的时候，请遵守**"Mythbusters**(流言终结者:美国某个科普节目)”守则：Don't Try This at Home(请不要在家里模仿尝试)。【这是一个习语：a phrase used in the media to advise against imitating a dangerous activity,详细请参考：http://baike.baidu.com/view/725048.htm】，在现实任何一个自定义账户之前请向专业的安全顾问进行咨询。
[最后一句话，一直不知道怎么理解比较好，还是贴上原文] Now that the security disclaimers are out of the way, it's time to get back to work. You've already implemented the meat of your custom account code; what's left is plumbing.(大概意思应该是说这只是实现密码安全的一小部分，真正如何确保安全好需要做更多深入的研究)

## (3)Extend AbstractAccountAuthenticator
为了使得AccountManager能够和你的自定义账户进行操作，你需要一个实现AccountManager期待的界面类来处理这个问题。这个类叫做authenticator类。
最简单的方法是创建一个authenticator类去继承AbstractAccountAuthenticator，并且实现它的抽象方法。如果你学习过前面一课的内容，AbstractAccountAuthenticator的抽象方法与获取account与token的方法有一定的关系。
实现这个类需要下面一些操作：

* 首先，需要重写AbstractAccountAuthenticator的7个抽象方法。
* 其次，你需要为"android.accounts.AccountAuthenticator"在manifest中增加一个intent filiter。
* 最后你需要有两套资源文件，自定义的账户名与图标。

你可以在AbstractAccountAuthenticator 文档中找到按部就班的实现方法。在SampleSyncAdapter sample app里面也有一个案例。
如果你查阅了SampleSyncAdapter的代码，你会发现一些方法里面返回了带有bundle的intent。这个intent是用来启动你自定义authenticator activity的。如果你的authenticator activity需要任何特定初始化的参数，你可以使用intent.putExtra()方法来附带参数。

## (4)Create an Authenticator Service
现在你已经有了一个authenticator的类，你需要一个让它发挥作用的地方。Account authenticator需要在多个程序中可用，并且在后台程序中运行。因此它需要在运行在一个Service里面。我们叫这个为authenticator service.
你的authenticator Service可以是非常简单的，它需要做的仅仅是在authenticator的onCreate()里面创建一个instance，并且在onBind()里面调用getIBinder()。在SampleSyncAdapter里面有一个比较好的例子。
别忘记在manifest里面增加<serivce>标签，并添加intent filiter。
```xml
<service ...>  
   <intent-filter>  
      <action android:name="android.accounts.AccountAuthenticator" />  
   </intent-filter>  
   <meta-data android:name="android.accounts.AccountAuthenticator"  
             android:resource="@xml/authenticator" />  
</service>  
```
## Distribute Your Service
你可以到Account & Sync的设置页面去增加一个account。

如果仅仅只有一个app会使用到这个account,这样做就不太好了，你只需要把这个account捆绑到自己app的service里面就可以了。但是如果你想要把自己的account被多个app使用，增加account到Account & Sync里面会是比较好的做法，避免了重复多个绑定到app。

一个解决方案是，把这个Service放在一个特定功能的APK里面。当一个app想要使用你自定义的账户的时候，你可以检查设备看是否你的自定义账户可用。如果不可用，它会引导用户到Google Play上面去下载这种服务。增加账户到Account & Sync，这个操作可能刚开始觉得会比较麻烦，但是比起为每个app重新输入账户与密码显得要要简单些。 

***
**学习自：[http://developer.android.com/training/id-auth/custom_auth.html](http://developer.android.com/training/id-auth/custom_auth.html)，请多指教，谢谢！**  
**转载请注明出自[http://kesenhoo.github.com](http://kesenhoo.github.com)，谢谢配合！**
