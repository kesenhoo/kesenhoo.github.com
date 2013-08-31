---
layout: post
title: "Android Training UserInfo - 记住用户的信息(Lesson 2 - 使用OAuth2来进行身份鉴定)"
date: 2012-03-28 18:53
comments: true
sidebar: false
categories: Android Android:Training Android:Training:UserInfo
---

## Authenticating to OAuth2 Services[使用OAuth2来进行鉴权]
为了安全的访问线上服务，用户需要在service上进行鉴定，他们需要提供身份的证明。对于一个程序来说，如果是访问第三方的服务，那么这个安全问题就更加复杂。
【比如，你有个资料在A服务器上，但是你需要在B上面对A里面的数据进行操作，这个时候如果把登入A的帐号与密码给B去直接操作就不够安全，简单的方法是先把A上的资料拿下来，再弄到B上面去，然后B再做操作，这又显得很不方便，就是为了解决这样类似的问题，才诞生了OAuth。关于OAuth2的详情，请参考：[http://oauth.net/2/](http://oauth.net/2/)，谢谢！】
目前行业内解决这种第三方服务身份鉴定的方法是使用OAuth2协议。OAuth2提供了auth token，它代表用户身份与用户对于程序的授权。这一课演示了如何使用OAuth2连接到Google Server。同样在其他支持OAuth2协议的服务上都可以使用类似的方法。

使用OAuth2有利于:

* 从用户那得到授权，使用账户信息来访问online service。  
* 对online service进行鉴定，保护用户利益。  
* 处理认证错误。  

<!-- More -->

## (1)Gather Information[收集信息]
在开始使用OAuth2之前，你需要获取到下面一些信息:

* 你想访问的服务地址。  
* auth scope，app获取到用来表示操作的权限范围的字串。例如，Google Tasks的read-only的auth scope是View your tasks，但是read-write的auth scope是Manage Your Tasks。  
* client id与client secret。用来表示身份的字串。你需要直接从service提供者那边获取那些字串。文章[Getting Started with the Tasks API and OAuth 2.0 on Android](http://code.google.com/apis/tasks/articles/oauth-and-tasks-on-android.html)解释了如何使用Google Tasks API来获取那些需要的字串。 

## (2)Request an Auth Token[请求一个授权口令]
现在你准备请求一个auth token，下面是请求流程图。

![oauth_dance.png](/images/articles/oauth_dance.png "Figure 1")

为了获取到auth token，你首先需要在manifest文件中增加ACCOUNT_MANAGER与INTERNET的权限。
```xml
<manifest ... >  
    <uses-permission android:name="android.permission.ACCOUNT_MANAGER" />  
    <uses-permission android:name="android.permission.INTERNET" />  
    ...  
</manifest>  
```
一旦设置好了上面的permissions，你可以使用AccountManager.getAuthToken() 来获取到token。
值得注意的是：Call [AccountManager](http://developer.android.com/reference/android/accounts/AccountManager.html)里面的方法can be tricky(狡猾的，棘手的)！因为account操作可能包括了网络通信，大多数方法是asynchronous，这意味着不应该把所有的auth操作放在一个方法里面，你需要使用callback机制来实现它。例如：
```java
AccountManager am = AccountManager.get(this);  
Bundle options = new Bundle();  
  
am.getAuthToken(  
    myAccount_,                     // Account retrieved using getAccountsByType()  
    "Manage your tasks",            // Auth scope  
    options,                        // Authenticator-specific options  
    this,                           // Your activity  
    new OnTokenAcquired(),          // Callback called when a token is successfully acquired  
    new Handler(new OnError()));    // Callback called if an error occurs  
```
在上面的例子中，OnTokenAcquired是AccountManagerCallback的子类。在OnTokenAcquired类里面AccountManager会执行run(AccountManagerFuture<Bundle> arg0)方法。如果获取成功，那么token就在Bundle里面。

下面是如何从Bundle中获取token的示例：
```java
private class OnTokenAcquired implements AccountManagerCallback<Bundle> {  
    @Override  
    public void run(AccountManagerFuture<Bundle> result) {  
        // Get the result of the operation from the AccountManagerFuture.  
        Bundle bundle = result.getResult();  
      
        // The token is a named value in the bundle. The name of the value  
        // is stored in the constant AccountManager.KEY_AUTHTOKEN.  
        token = bundle.getString(AccountManager.KEY_AUTHTOKEN);  
        ...  
    }  
}
```  
如果一切正常，那么Bundle里面会包含KEY_AUTHTOKEN的字段，但是通常事情没有那么顺利。

## (3)Request an Auth Token... Again[再次请求Auth Token]
你的第一次有可能由于下面的某个原因而导致失败:

* 设备上的某个错误或者是网络错误导致AccuntManager操作失败。  
* 用户不授权你的app访问account。  
* 存储的account证书信息不足以让你访问account。  
* 在Cache里面的auth token已经过期。  

程序能简单的处理前两种情况，通常仅仅是显示一个错误信息给用户。如果网络异常或者用户不授权，程序就没有必要接下去操作了。对于后面两种情况稍微有点复杂，通常对于好的程序都应该自动处理那些错误。

第三种情况，没有足够的证书，这些证书是通过前面提到的回调函数返回在bunde里面，是一个使用KEY_INTENT关键字的intent。这是获取token的前提。
之所以鉴定者返回一个intent是有很多原因的。也许是用户的account过期或者他们存储的证书出错，这个时候可以使用intent来让用户重新登入。也许account需要两个证书，或者他需要激活camera来做某个扫描的动作进而验证。不管到底是因为什么，如果你想要一个有效的token，你需要启动intent来获取token。
```java
private class OnTokenAcquired implements AccountManagerCallback<Bundle> {  
    @Override  
    public void run(AccountManagerFuture<Bundle> result) {  
        ...  
        Intent launch = (Intent) result.get(AccountManager.KEY_INTENT);  
        if (launch != null) {  
            startActivityForResult(launch, 0);  
            return;  
        }  
    }  
}  
```
请注意例子中使用的是startActivityForResult()，这样我们可以在自己的activity里面通过实现onActivityResult()的方法来获取start intent的结果。这是非常重要的，如果你没有获取返回的结果，那么就无法区分出用户是否成功获得了鉴定。如果result是RRSULT_OK，然后认证者就会更新存储的证书，这样就可以获取到足够的证书，你也可以再次执行AccountManage.getAuthToken()方法来请求新的token。

最后一种情况，token过期，实际上这不是AccountManager的错误。唯一判断token是否过期的方法是把token告诉server，通过server来告知已经过期，但是不断的去线上检查是否过期明显是比较浪费资源的。

## Connect to the Online Service[连接到Online Service]
下面的例子显示了如何连接到Google server。因为Google使用了行业标准的OAuth2协议，所以这个例子很具有代表性。请记住，尽管每一个Server是不一样的，你仍然可以对特殊的情形进行调整。
Google APIs需要你为每个请求提供4个值，分别是API key,client ID,client secret与auth key。前面三个可以从Google API的网站上找到，最后一个字段需要你通过执行AccountManager.getAuthToken()方法来获取。当都拿到之后，通过HTTP request来传递那些值给Google Server。
```java
URL url = new URL("https://www.googleapis.com/tasks/v1/users/@me/lists?key=" + your_api_key);  
URLConnection conn = (HttpURLConnection) url.openConnection();  
conn.addRequestProperty("client_id", your client id);  
conn.addRequestProperty("client_secret", your client secret);  
conn.setRequestProperty("Authorization", "OAuth " + token);  
```
如果上面的请求返回HTTP错误代码401，表明你的token被否定了。在最后一部分我们有提到，最通常的错误原因是token过期了，解决这个问题的方法很简单，执行AccountManager.invalidateAuthToken()方法并且在需要的时候重复执行token的请求操作。

因为token过期是如此的常见，并且修复它是那么的简单，许多程序甚至在获取token之前就假定它是过期的，如果server重新生成一个token的花费并不大，我们可以直接刚开始就执行AccountManager.invalidateAuthToken()，这样就省得刚开始需要请求两次。

**Ps:oauth2很常见:目前开放平台似乎都是用这个规则来开放自己的账户系统，使用QQ登入，Sina微博帐号登入等等**

***
**学习自：[http://developer.android.com/training/id-auth/authenticate.html](http://developer.android.com/training/id-auth/authenticate.html)，请多指教，谢谢！**  
**转载请注明出自[http://kesenhoo.github.com](http://kesenhoo.github.com)，谢谢配合！**
