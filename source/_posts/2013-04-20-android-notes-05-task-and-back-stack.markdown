---
layout: post
title: "Android Notes 05 - Tasks and Back Stack"
date: 2013-04-20 20:42
comments: true
sidebar: false
categories: Android Android:Note
---

* 所有的activities都归属于一个task。
* 一个task包含了一些activities，这些activities以用户与他们的交互前后为顺序存放在task中。
* Tasks可以把activies放置在background并且保存每一个activites的状态，以便于用户可以切换到其他task而不至于丢失之前的活动状态。

# 要点概述
一个程序通常包含了多个actvities。每一个activity都应该围绕用户需要执行的一个特定功能而进行设计，并且可以启动其他的activites。例如:一个邮件程序应该拥有一个显示新邮件的列表。当用户选择其中一封邮件，一个新的activity将打开用来显示邮件内容。

<!-- more -->

一个activity甚至可以启动设备上其他程序的activites。例如：如果你的程序想要发送邮件，你可以定义一个intent来表达你的请求。其他可以接收这个请求的程序则会对这个请求作出响应。当使用其他程序的组件把邮件发送出去之后，你自己的activity将会resume，这样另外一个程序的发邮件组件看起来像是你的程序中的一部分。尽管真正发邮件的activites也许是来自其他不同的程序，Android会通过保存那些activites到同一个task中的方式让用户感觉到无缝的体验。

一个task是用户执行一个确定的动作所会用到的activites的集合。那些activities被安置到一个stack中 (也就是"back stack"), 他们以每个activity被打开的顺序方式存放在stack中。

设备Home界面是大多数tasks的起始点。当用户点击程序在launcher或者Home上的启动图标时，这个程序的task则变成了foreground的。如果程序之前不存在task(程序最近并没有被使用过), 那么一个新的task将被创建并且程序的"main" activity将被打开并作为stack的root activity。

当目前的activity启动另外一个activity时，新的activity被压入栈中作为栈顶并且获取到了focus。前面的那个activity则以stopped的状态继续保留在stack中。当一个activity stops时，系统会保留它的UI状态。当用户点击back按钮时，当前的activity从栈顶退出并被destroyed，之前的activity则resume(之前保存的UI状态得到恢复). 在栈中的Activities的是不会被重新排序的，仅仅是做入栈与出栈的动作。当被启动时入栈，用户点击后退时出栈。下图演示了上面提到的情况：

![diagram_backstack.png](/images/articles/diagram_backstack.png "Figure 1. A representation of how each new activity in a task adds an item to the back stack. When the user presses the Back button, the current activity is destroyed and the previous activity resumes.")

如果用户持续点击back按钮，那么在栈中的每一个activity都会做退栈并显示之前activity的动作, 直到用户退回到Home界面(或者是用户开始task的地方).当所有的activities都从栈中被移除之后，这个task也就消失了。

一个task是一个紧密结合的单元，当用户开始一个新的task或者通过点击Home按钮回到Home桌面时，之前的task会整体移动到background。当在background时，在task中的所有的activity都是stopped状态的。但是这个task的回退栈仍然保留了与系统交互的特质，仅仅是失去了focus而已。如下图所示：

![diagram_multitasking.png](/images/articles/diagram_multitasking.png "Figure 2. Two tasks: Task B receives user interaction in the foreground, while Task A is in the background, waiting to be resumed.")

一个task可以返回到"foreground"，这样用户可以从他们离开的地方重新开始操作。例如，当前task(Task A)的栈中有三个activities. 用户点击Home按钮，然后从桌面启动一个新的程序. 当Home界面出现时，Task A变成了background的. 当新的程序启动，系统为它启动一个新的task，里面包含了它自己的activites. 在用户与那个程序交互完之后，重新退回桌面并选择之前启动的task A. 这个时候，Task A重新回到foreground, 在stack中的那三个activities被重新选中，栈顶的那个activity被resume. 此时，用户仍然可以选择通过切回Home，再次启动task B.( 或者通过长按Home按钮触发最近使用的task界面进行选择). 这便是Android中的多任务的一个例子.

**Note:** 众多tasks都可以一并在后台被Hold住。然而，如果用户同时执行了多个后台任务，系统便会为了恢复内存而开始销毁后台activities. 这会导致activity状态丢失. 关于着部分内容，请看下面的Activity state部分.

因为在back stack中的activities是不会被重新调整顺序的。如果你的程序中的某个特定的activity可以被多个activity所叫起，那么将会为那个特定activity创建一个新的实例并压入栈中(而不是把那个activity之前的实例移动到栈顶). 如下图所示:你程序中的一个activity可以被多次实例化(即使是在不同的task中)。如果用户通过点击Back按钮回退，每一个实例都将以他们被打开的顺序依次被重新呈现。然而，如果你不想一个activity被重复实例化，你可以修改这种行为。关于如何修改这种行为，将在下面的Managing Tasks中讲到.

![diagram_multiple_instances.png](/images/articles/diagram_multiple_instances.png "Figure 3. A single activity is instantiated multiple times.")

**总结一下系统默认的activities与tasks的行为:**

* 当Activity A启动Activity B时, Activity A会是stopped, 但是系统会保存它的状态(例如scroll的位置与填表格的文字). 如果用户在activity B时点击Back按钮，那么activity A会恢复它之前保存的状态信息.
* 当用户通过点击Home按钮离开一个task时，当前的activity会是stopped并且它的task会退到background. 系统保留task中的每一个activity. 如果用户之后通过点击启动图标来重新叫起它,task会成为foreground的并且栈顶的activity会得到恢复.
* 如果用户点击Back按钮, 当前activity会从栈中退出并被destroyed. 在栈中的前一个activity得到resumed. 当一个activity被destroyed, 系统不会再保留activity的状态信息.
* Activities可以被多次实例化，即使是请求来自其他tasks.

# Saving Activity State(保存Activity状态)
正如上面提到的，当activity stopped时，系统默认会保存它的状态. 这样的话, 当用户回退到之前的activity, 它的UI将和离开时一致. 然而, 如果activity被destoryed并且需要recreated时，你可以并且应该主动使用activity的callback方法来保存它的状态信息.

当系统stop你的某个activity时, 系统可能会在需要恢复内存时destory那个stop状态的activity. 当发生这件事时，关于activity的状态信息则会丢失. 即使真的发生那样的情况, 系统仍然为那个activity在back stack中保留了位置, 但是当这个activity成为栈顶activity时, 系统必须recreate它(而不是resume它). 为了避免丢失用户的工作内容, 你应该主动通过实现[onSaveInstanceState()](http://developer.android.com/reference/android/app/Activity.html#onSaveInstanceState(android.os.Bundle))回调方法来保存那些信息.

想了解更多关于如何保存activity的状态，请参考[Activities](http://developer.android.com/guide/components/activities.html#SavingActivityState).

# Managing Tasks(管理Tasks)
Android系统默认的管理tasks与back stack的方法使用于大部分情况, 对于大多数程序来说, 你不需要担心你的activites是如何与task进行联系的，它们又是如何存放在back stack中的. 但是你可能想要打断通常的行为, 可能你想要在你的程序中的某个activity被启动时会创建一个新的task而不是放置到旧的task中; 或者, 当你启动一个activity，你想要把已经存在的某个实例放置到栈顶(而不是在栈顶创建一个新的实例); 又或者说, 你想要在用户退出task时除了根activity之外清除栈中其它的activities.

你可以通过在manifest的<activity>标签下设置属性，并且在把intent传递给startActivity()方法之前为intent设置flag. 

关于此事, 在<activity>标签下可以设置的主要属性为:  
* taskAffinity
* launchMode
* allowTaskReparenting
* clearTaskOnLaunch
* alwaysRetainTaskState
* finishOnTaskLaunch

你可以为intent设置flag的主要值有:  
* FLAG_ACTIVITY_NEW_TASK
* FLAG_ACTIVITY_CLEAR_TOP
* FLAG_ACTIVITY_SINGLE_TOP

下面将会讲解如何使用上面定义的方法  
**Caution:** 大多数程序不应该打断系统默认的activities与tasks之间的行为. 如果你确定有必要修改默认的行为, 请谨慎使用并确保用户切换与回退的行为是可用正常的. 请确保你的设计行为不会对用户期待的体验有冲突.

## (0)Defining launch modes(定义启动模式)
启动模式允许你定义一个activity的实例与当前task是如何结合的. 你可以用下面两种方式来定义不同的启动模式:

* **Using the manifest file**  
当你在manifest文件中定义一个activity时，你可以指定当它启动时与task的结合方式.

* **Using Intent flags**  
当你调用startActivity(), 你可以对intent进行设置flag来表示这个activity将与task如何进行结合.

像这样, 如果Activity A 启动 Activity B, 那么Activity B 可以在它的manifest中定义它应该如何与当前的task进行联合，同样Activity A 也可以请求Activity B 应该如何与当前task进行结合. **如果两个都有定义，那么A的请求(定义在Intent中的)的优先级比B的请求(定义在manifest中)要更高**.

**Note:**某些可以定义在manifest中的启动模式并不一定在intent flag中可以找到对应的方式. 同样,某些在intent flag中可以找到的并不一定可以在manifest中进行设置.

### Using the manifest file
在manifest的<activity>标签下设置**launchMode**属性，一共可以设置下面4种类型的启动模式:
***
* **"standard"** (the default mode)

系统默认的启动模式. 系统在task中创建一个新的实例，activity可以被多次实例化. 每一个实例可以属于不同的task, 同样一个task可以有多个实例.

* **"singleTop"**

如果activity的实例已经存在于当前task的栈顶, 系统会通过调用它的onNewIntent()方法把intent导向那个实例, 而不是为那个activity创建新的实例. activity可以被多次实例化，每一个实例可以属于不同的tasks, 同样一个task可以有多个实例 (仅仅当目前栈顶activity不是那个activity时才行，否则栈顶元素就是想要创建的activity的实例的话，则不会重复创建).

例如, 某个栈中的元素由root至top依次为activity A-B-C-D; Acivity D是栈顶. 一个intent来到activity D中. 如果D是默认的"standard"启动模式, 那么stack则会变成A-B-C-D-D. 然而, 如果D的启动模式是"singleTop", 那么D将通过onNewIntent()来接受那个intent, 栈结构仍然是A-B-C-D. 然而, 如果Activity B接受到一个intent, 那么即使现在是single top模式，仍然会创建一个新的B加入到栈中.(A-B-C-D-B)

**Note:** 当一个activity的实例被创建时，用户可以点击Back按钮来回退到之前的activity. **但是当已经存在的某个实例需要处理一个新的intent时，用户在这个intent还没有到达onNewIntent()方法之前是没有办法通过点击back按钮来回退的.**

* **"singleTask"**

系统会创建一个新的task并把实例化的activity作为task的root元素(这个task还可以继续添加其他activity的实例). **然而, 如果这个activity的实例已经存在于另外一个task中，系统会通过调用onNewIntent()方法把intent导向已经存在的实例中, 而不是创建一个新的实例. 某一时刻，activity的实例只能存在一个.**

**Note:**尽管activity是在一个新的task中被启动的, Back按钮仍然是可以回退到上一个activity的.

* **"singleInstance".**

除了系统不会再对拥有的那个activity的task添加新的其他实例之外，与"singleTask"是类似的. activity的实例总是唯一的，并且是task中唯一的一个元素. 并且task是新创建的.
***
另外一个例子, Android的Browser程序通过为web browser activity在manifest中指定singleTask模式定义了activity总是在它自己的taks中打开. 这意味着，如果你的程序定义了一个intent来打开Android Browser, 它的activity并不是在你的程序task中被打开的. 要么为browser启动一个新的task，要么把browser正在后台允运行的task带到forgound来处理那个Intent.  
**???**(*奇怪，前面不是说intent请求的优先级比manifest中的要高吗？这里怎么会这样？*)

不管是启动一个新的task还是在原来的task中，Back按钮都可以回退到前面的activity. 然而，如果你启动的activity的launch mode定义为singleTask, 然后如果那个activity的某个实例已经存在于一个background的task中，那么这个background的task的所有activites会**整个全部**被带到foreground的. 下图Figure 4 演示了这种情况：

![diagram_backstack_singletask_multiactivity.png](/images/articles/diagram_backstack_singletask_multiactivity.png "Figure 4. A representation of how an activity with launch mode "singleTask" is added to the back stack. If the activity is already a part of a background task with its own back stack, then the entire back stack also comes forward, on top of the current task.")

**Note:** 通过manifest中launchMode属性设置的方式定义的启动模式可以被intent中的flag所覆盖，请看下面的解释.

### Using Intent flags
* FLAG_ACTIVITY_NEW_TASK  
在新的task中启动activity. 如果你要启动的activity已经存在于某个运行的task中, 那么这个task会整个被提升为foregound的，activity会通过onNewIntent()来接收intent.这部分的行为与singleTask启动模式是一致的.

* FLAG_ACTIVITY_SINGLE_TOP  
如果在栈顶的activity接受到这样的intent, 那么已经存在于栈顶的实例会通过onNewIntent()来获取intent, 而不是创建一个新的实例. 这部门的行为与singleTop启动模式是一致的.

* FLAG_ACTIVITY_CLEAR_TOP  
如果被启动的activity已经存在于当前运行的task中, 不是为那个activity启动一个新的实例，而是在这个activity实例之上的activities都被destoryed，intent被通过onNewIntent()传递到这个实例中，并且这个时候，它就成了栈顶元素. 这个行为在launchMode的属性中并没有对应的值.  
FLAG_ACTIVITY_CLEAR_TOP通常与FLAG_ACTIVITY_NEW_TASK同时使用. 当他们结合一起时，那意味着把已经存在的activity放置到另外一个task中并且把它作为栈顶元素开始与用户进行交互.

**Note:**如果被启动的activity的launch mode是"standard", 那么这个activity同样会被先清除掉，然后创建一个新的实例来接受intent. 那是因为当launch mode是standard时，新的实例总是为了新的intent而被创建.

## (1)Handling affinities(处理联姻关系)
Affinity(联姻)意味着activity更倾向归属于哪一个task. 默认的，来自同一程序的所有的activities会拥有同样的联姻关系. 因此，他们更倾向于处于同一个task中. 然而, 你可以修改这种默认的行为. 在不同程序中的activites可以有同样的联姻关系，或者同一程序中的不同activites可以分配到不同的task联姻关系中

可以在manifest的activity中设置taskAffinity属性，一共有下面两种情况：

* When the intent that launches an activity contains the FLAG_ACTIVITY_NEW_TASK flag.

A new activity is, by default, launched into the task of the activity that called startActivity(). It's pushed onto the same back stack as the caller. However, if the intent passed to startActivity() contains the FLAG_ACTIVITY_NEW_TASK flag, the system looks for a different task to house the new activity. Often, it's a new task. However, it doesn't have to be. If there's already an existing task with the same affinity as the new activity, the activity is launched into that task. If not, it begins a new task.  
If this flag causes an activity to begin a new task and the user presses the Home button to leave it, there must be some way for the user to navigate back to the task. Some entities (such as the notification manager) always start activities in an external task, never as part of their own, so they always put FLAG_ACTIVITY_NEW_TASK in the intents they pass to startActivity(). If you have an activity that can be invoked by an external entity that might use this flag, take care that the user has a independent way to get back to the task that's started, such as with a launcher icon (the root activity of the task has a CATEGORY_LAUNCHER intent filter; see the Starting a task section below).

* When an activity has its allowTaskReparenting attribute set to "true".

In this case, the activity can move from the task it starts to the task it has an affinity for, when that task comes to the foreground.  
For example, suppose that an activity that reports weather conditions in selected cities is defined as part of a travel application. It has the same affinity as other activities in the same application (the default application affinity) and it allows re-parenting with this attribute. When one of your activities starts the weather reporter activity, it initially belongs to the same task as your activity. However, when the travel application's task comes to the foreground, the weather reporter activity is reassigned to that task and displayed within it.

**Tip:** 如果一个.apk文件从用户的角度看包含了不止一个"application"的话，你可能会想要使用taskAffinity属性来为那些activites设置不同的联姻关系.

## (2)Clearing the back stack(清除回退栈)
如果用户长时间离开一个task, 系统会清除task中root activity之外的所有的activites. 当用户回退到task时，仅仅只有root activity会被恢复.

有下面一些值可以用来设置，如果你想修改系统默认的行为的话:  

* **alwaysRetainTaskState**  
设置了这个属性为true之后，即使很长时间，也不会被destory的

* **clearTaskOnLaunch**  
如果这个属性设置为true，那么就是与alwaysRetainTaskState相反的. 每次启动都会清除root以上的activities.

* **finishOnTaskLaunch**  
这个属性与clearTaskOnLaunch类似, 但是它仅仅是对单个activity进行操作，而不是整个task. It can also cause any activity to go away, including the root activity. When it's set to "true", the activity remains part of the task only for the current session. If the user leaves and then returns to the task, it is no longer present.

## (3)Starting a task(启动一个任务)
你可以启动一个activity作为一个task的起点，通过为它的intent filter设置"android.intent.action.MAIN"作为特定的action，设置"android.intent.category.LAUNCHER"作为特定的category. 例如：
```xml
<activity ... >
    <intent-filter ... >
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
    ...
</activity>
```
上面的设置方式不仅仅是使得这个activity作为launcher的图标，允许用户通过点击它启动activity并且随时通过点击它来触发恢复之前的task.

第二点非常重要: 用户必须能够在离开task之后通过activity的launcher来回到task. 因为这点, "singleTask" 与 ""singleInstance" 这两种启动模式总是会初始化一个task, 需要有ACTION_MAIN 与 CATEGORY_LAUNCHER 配合. 例如，用户启动了一个task做了一些动作，然后点击Home回到桌面，这个时候想要回到刚才的task，需要有launcher作为入口.

对于那些你不想用户可以回退的情况，请设置<activity>标签下的finishOnTaskLaunch属性为"true" (参考上面的章节).

*****************
**文章学习自[http://developer.android.com/guide/components/tasks-and-back-stack.html](http://developer.android.com/guide/components/tasks-and-back-stack.html)**  
**转载请注明出自[http://kesenhoo.github.com](http:://kesenhoo.github.com)，谢谢**

