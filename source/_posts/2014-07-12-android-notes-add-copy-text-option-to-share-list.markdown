---
layout: post
title: "Android Notes - 添加Copy to Clipboard的选项到分享列表中"
date: 2014-07-12 15:26
comments: true
sidebar: false
categories: Android Android:Notes
---

偶然被人问到如何添加复制到剪切板的选项到分享列表，如下图所示：

![copy_link_option_at_share_list](/images/copy_link_option_at_share_list.png)

一般情况下，分享一段文字或者图片，我们会使用如下Android默认的方式：

```java
public void shareText(Context context, String text) {
    Intent sendIntent = new Intent();
    sendIntent.setAction(Intent.ACTION_SEND);
    sendIntent.putExtra(Intent.EXTRA_TEXT, text);
    sendIntent.setType("text/plain");
    context.startActivity(Intent.createChooser(sendIntent, "Share via..."));
}
```

这样系统能够帮忙筛选出那些符合这个Intent的所有Activity，生成分享列表，呈现给用户。因为分享列表的信息是由系统过滤生成的，UI界面也是交给系统进行绘制的，我们的应用无法给这个分享列表设置点击监听器，那么如何才能实现添加一个**"Copy to Clipboard"**的选项到分享列表中，并在点击该选项之后执行对应的动作呢？当然，自己去实现这个分享列表的效果，UI完全交给自己的应用来控制，是可以轻松做到的，可是自己去过滤符合条件的应用，并绘制分享列表的代码量会大很多，实现起来更加复杂？下面介绍一个虽然写法有点奇怪却相对简便很多的方法。

<!-- More -->

实现步骤如下：

### 1)创建一个新的Intent

```java
Intent intent = new Intent(Intent.ACTION_SEND);
intent.setType("text/plain");
intent.putExtra(Intent.EXTRA_SUBJECT, title);
intent.putExtra(Intent.EXTRA_TEXT, body);

Intent clipboardIntent = new Intent("ACTION_COPY_TO_CLIPBOARD");
clipboardIntent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP);
clipboardIntent.putExtra("KEY_SHARE_TITLE", title);
clipboardIntent.putExtra("KEY_SHARE_BODY", body);

try {
    Intent chooserIntent = Intent.createChooser(intent, "Share via");
    chooserIntent.putExtra(Intent.EXTRA_INITIAL_INTENTS, new Intent[] {clipboardIntent});
    context.startActivity(chooserIntent);
} catch (android.content.ActivityNotFoundException ex) {
    Toast.makeText(context, "There are no share clients installed.", Toast.LENGTH_SHORT).show();
}
```

请注意：Action与Flag。   

### 2)在manifest文件中为当前的actiivty添加Intent Filter

```xml
<activity
    android:name=".TestActivity"
    android:label="Copy to clipboard"
    android:icon="@drawable/ic_action_copy"
    android:launchMode="singleTop"
    android:screenOrientation="portrait"
    android:windowSoftInputMode="adjustPan">
    <intent-filter>
        <action android:name="ACTION_COPY_TO_CLIPBOARD"/>
        <category android:name="android.intent.category.DEFAULT"/>
    </intent-filter>
</activity>
```

icon与label组成了分享列表中的"复制到剪切板"。launchMode定义为singleTop是因为当前activity已经在栈顶，没有必要因为intent的到来而重新创建一个，所以维持目前的activity，使得点击”复制到剪切板“之后，activity会直接执行onNewIntent()的回调，在这里获取到之前定义的intent，从这个intent获取后续操作的数据。

### 3)在onNewIntent()中执行把文字复制到剪切板的任务

```java
@Override
protected void onNewIntent(Intent intent) {
    super.onNewIntent(intent);
    Log.i(TAG, "[onNewIntent] intent = " + intent);
    if (intent != null && intent.getAction().equalsIgnoreCase("ACTION_COPY_TO_CLIPBOARD")) {
        String title = intent.getStringExtra("KEY_SHARE_TITLE");
        String body = intent.getStringExtra("KEY_SHARE_BODY");

        ClipboardManager clipboard = (ClipboardManager) getSystemService(Context.CLIPBOARD_SERVICE);
        // clipboard.setText(title + body);
        // Creates a new text clip to put on the clipboard
        ClipData clip = ClipData.newPlainText(title, body);
        clipboard.setPrimaryClip(clip);
        Log.d(TAG, "[onNewIntent] copy text title = " + title + ", body = " + body);
        Toast.makeText(this, "Copy Succussed!", Toast.LENGTH_SHORT).show();
    }
}
```

***
至此，这个功能就实现了，这种写法还给了我们更多的启发：可以使用类似的方式添加其他的选项到分享列表中，在activity的onNewIntent回调里面处理这个选项要求实现的任务。这种方式相比起自己去过滤并绘制分享列表要简单很多！欢迎有其他实现方法的同学留言交流！


