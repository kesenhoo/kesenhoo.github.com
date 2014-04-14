---
layout: post
title: "Android Training(08) - 云同步(Lesson 1 - 使用App Engine进行同步)"
date: 2012-04-22 13:56
comments: true
sidebar: false
categories: Android Android:Training
---

## Syncing with App Engine[使用App Engine进行同步]
写一个能够同步到云端的app是具有挑战性的。那存在许多细节需要处理，例如服务端身份验证，客户端身份验证，分享数据的模块，还有API。简化这些操作的一个方法是使用**Google Plugin for Eclipse**，这个插件帮你垂直整合处理了那些Android系统与App Engine程序交互的操作。这一课会介绍如何创建那样一个项目。

关于App Engine，请参考：https://developers.google.com/appengine/docs/whatisgoogleappengine?hl=zh-CN

下面会介绍:    
* 建立Android与App engine apps能够交互的程序。  
* 利用Cloud to Device Messaging(C2DM)的优势，这样app就不需要使用轮询机制去更新。[关于C2DM的详情请参考前面课程与http://code.google.com/intl/zh-CN/android/c2dm/)]

<!-- More -->

这一课仅仅专注于本地开发测试使用，并不涉及程序发布(例如，发布你的App Engine，发布你的Android程序到Market)，程序发布等知识点会在其他课程里涉及到。

## (1)Prepare Your Environment[准备你的开发环境]
如果你想继续下面的课程步骤，你必须依据下面所述搭建好你的开发环境:    
* 安装[Google Plugin for Eclipse](http://code.google.com/eclipse/)  
* 安装[GWT SDK](http://code.google.com/webtoolkit/download.html) 与 [Java App Engine SDK](http://code.google.com/appengine/). The [Quick Start Guide](http://code.google.com/eclipse/docs/getting_started.html)会演示如何安装那些组件.  
* 注册[C2DM access](http://code.google.com/android/c2dm/signup.html)账户. 我们强烈推荐[creating a new Google account](https://accounts.google.com/SignUp)来专门用来连接到C2DM. 在这一课Google服务器会重复使用这个账户来进行身份鉴定。

## (2)Create Your Projects[创建你的项目]
在你安装完Google Plugin for Eclipse之后，请注意在创建一个新的Eclipse项目的时候会存在一种新的Android项目选项：`App Engine Connected Android Project`(在Google项目分类下)。安装向导会提示你输入账户验证信息，这个账户就是之前提到的你在C2DM上注册的时候使用的。(请注意不要输错了账户类型)

一旦你创建好之后，你会在workspace看见两个有2个项目：一个Android程序与一个App Engine程序。好吧！那两个程序具备了需要实现的所有功能，安装向导创建了sample程序，它可以允许你使用AccountManager来验证Android设备与App Engine的交互。为了便于后续测试，请做下面的操作：

请确保你存在2.2以上的AVD,右击Eclipse中的Android项目，选择`Debug As`>`Local App Engine Connected Android Application`。这样程序就能够测试C2DM的功能(Google Play是这一类程序的典型代表).它也启动了一个App Engine的local instance，里面包含了你的程序。

## (3)Create the Data Layer[创建数据层]
因为上面已经创建了一个完整功能的sample程序。下面应该学习开始修改那些代码来创建你自己的程序。
首先，创建数据模块，它定义了在App Engine与Android app之间共享的数据。打开App Engine项目的文件夹，定位到 `(yourApp)`-`AppEngine` > `src` > `(yourapp)` > `server`。创建一个新的类，它包含了那些你想要存储到云端的数据。下面是一段sample code:
```java
package com.cloudtasks.server;  
  
import javax.persistence.*;  
  
@Entity  
public class Task {  
  
    private String emailAddress;  
    private String name;  
    private String userId;  
    private String note;  
  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    private Long id;  
  
    public Task() {  
    }  
  
    public String getEmailAddress() {  
        return this.emailAddress;  
    }  
  
    public Long getId() {  
        return this.id;  
    }  
    ...  
}
```  
请注意注释的使用：Entity, Id 与GeneratedValue 都是来自[Java Persistence API](http://www.oracle.com/technetwork/articles/javaee/jpa-137156.html). 这些注释都是必备的， Entity需要注释在类声明的上面， 这表明这个类在你的数据层代表了一个`Entity.Id` 与`GeneratedValue`分别表明了寻找这个类的id与这个id是如何形成的(在上面的示例中，`GenerationType.IDENTITY`意味这是由DB生成的). 你可以参考[Using JPA with App Engine](http://code.google.com/appengine/docs/java/datastore/jpa/overview.html)来查看更多关于资料。

一旦你完成了所有的数据实体类的创建, 你需要创建Android与App Engine程序之间交互的方法. 这种交互的方法可以通过创建一个`Remote Procedure Call (RPC)`服务来开启。通常的是，这包含了许多一成不变的单调的代码. 幸运的是, 有一种简单的办法来完成这个操作! 在你的App Engine源代码文件夹下右击，选择`New` > `Other` ，再选择`Google` > `RPC Service`. 这个时候会出现向导, 陈列出所有你在上一步骤创建的实例,它是通过在源代码文件夹中去查找具有`@Entity`的方法来实现的. 这样出来的代码非常整洁，之后点击Finish,向导会创建一个Service类，它包含了`Create`, `Retrieve`, `Update` and `Delete` (CRUD)实例的操作.

## (4)Create the Persistence Layer [创建持久层]
持久层是一个你的程序数据能够存放long-term的地方。为了编写你的持久层，你有一些选择，这取决于你想存储哪些类型的数据. 一些由Google管理的选择包含在[Google Storage for Developers](http://code.google.com/apis/storage/)与App Engine's built-in Datastore. 下面是一个sample code，使用DataStore的代码.

在你的com.cloudtasks.server下创建一个类用来处理持久层的输入与输出. 为了访问这些数据，使用[PersistenceManager](http://db.apache.org/jdo/api20/apidocs/javax/jdo/PersistenceManager.html). 你可以使用在com.google.android.c2dm.server.PMF下的PMF类生成这个类的的一个实例，然后使用它来执行基本的CRUD操作:
```java
/** 
* Remove this object from the data store. 
*/  
public void delete(Long id) {  
    PersistenceManager pm = PMF.get().getPersistenceManager();  
    try {  
        Task item = pm.getObjectById(Task.class, id);  
        pm.deletePersistent(item);  
    } finally {  
        pm.close();  
    }  
}
```  
你也可以使用Query对象从你的Datastore来retrieve数据。
```java
public Task find(Long id) {  
    if (id == null) {  
        return null;  
    }  
  
    PersistenceManager pm = PMF.get().getPersistenceManager();  
    try {  
        Query query = pm.newQuery("select from " + Task.class.getName()  
        + " where id==" + id.toString() + " && emailAddress=='" + getUserEmail() + "'");  
        List list = (List) query.execute();  
        return list.size() == 0 ? null : list.get(0);  
    } catch (RuntimeException e) {  
        System.out.println(e);  
        throw e;  
    } finally {  
        pm.close();  
    }  
}
```  
一个好的例子，帮你encapsulate了持久层，请参考Cloud Tasks app里面的DataStore类.

## (5)Query and Update from the Android App [从Android App查询与更新]
为了保持与App Engine程序的同步，你的Android程序需要知道下面两件事情：从云端拉取数据与发送数据到云端。大部分这类操作已经由上面提到的插件生成了，但是你需要自己编写UI来呈现那些操作.

插件生成的sample code显示了一些重要的特征：  
* 首先，我们需要删除样本里面的Activity.java中的setHelloWorldScreenContent()的方法，替换的是与实际程序有关的代码。  
* 其次，所有交互的操作都是包在AsyncTask中来完成的，这样不会因为网络操作而卡到UI thread.  
* 最后，它给出了一个简单的模板演示如何访问云端的数据, 使用RequestFactory 来操作，它由Eclipse plugin提供支持。

关于实例，如果你的云端数据模型包含了一个叫做Task的对象，这个对象会在你生成RPC layer的时候自动为你创建的一个TaskRequest的类, 还有一个TaskProxy来代表单独的Task. 在下面的代码中演示了请求一个所有task的列表:
```java
public void fetchTasks (Long id) {  
  // Request is wrapped in an AsyncTask to avoid making a network request  
  // on the UI thread.  
    new AsyncTask>() {  
        @Override  
        protected List doInBackground(Long... arguments) {  
            final List list = new ArrayList();  
            MyRequestFactory factory = Util.getRequestFactory(mContext,  
            MyRequestFactory.class);  
            TaskRequest taskRequest = factory.taskNinjaRequest();  
  
            if (arguments.length == 0 || arguments[0] == -1) {  
                factory.taskRequest().queryTasks().fire(new Receiver>() {  
                    @Override  
                    public void onSuccess(List arg0) {  
                      list.addAll(arg0);  
                    }  
                });  
            } else {  
                newTask = true;  
                factory.taskRequest().readTask(arguments[0]).fire(new Receiver() {  
                    @Override  
                    public void onSuccess(TaskProxy arg0) {  
                      list.add(arg0);  
                    }  
                });  
            }  
        return list;  
    }  
  
    @Override  
    protected void onPostExecute(List result) {  
        TaskNinjaActivity.this.dump(result);  
    }  
  
    }.execute(id);  
}  
...  
  
public void dump (List tasks) {  
    for (TaskProxy task : tasks) {  
        Log.i("Task output", task.getName() + "\n" + task.getNote());  
    }  
}
```  
为了创建一个新的任务并发送到云端，需要创建一个新的请求对象并使用它来创建一个proxy对象。然后proxy对象执行它的更新方法。重申，这些操作应该放在AsyncTask里面去执行，避免网络操作卡到UI Thread。下面是sample code:
```java
new AsyncTask() {  
    @Override  
    protected Void doInBackground(Void... arg0) {  
        MyRequestFactory factory = (MyRequestFactory)  
                Util.getRequestFactory(TasksActivity.this,  
                MyRequestFactory.class);  
        TaskRequest request = factory.taskRequest();  
  
        // Create your local proxy object, populate it  
        TaskProxy task = request.create(TaskProxy.class);  
        task.setName(taskName);  
        task.setNote(taskDetails);  
        task.setDueDate(dueDate);  
  
        // To the cloud!  
        request.updateTask(task).fire();  
        return null;  
    }  
}.execute();
```  

## (6)Configure the C2DM Server-Side[确认C2DM服务器端]
为了设置C2DM的消息能够被发送到你的Android设备，回到你的App Engine代码处，打开生成RPC层的时候创建的Service类. 如果你的项目名是Foo, 这个类的名字就叫做FooService. 为每一个方法都添加一些代码，允许做adding, deleting, or updating数据的操作，这样C2DM message才能发送到用户的设备上. 下面是一段sample code:
```java
public static Task updateTask(Task task) {  
    task.setEmailAddress(DataStore.getUserEmail());  
    task = db.update(task);  
    DataStore.sendC2DMUpdate(TaskChange.UPDATE + TaskChange.SEPARATOR + task.getId());  
    return task;  
}  
  
// Helper method.  Given a String, send it to the current user's device via C2DM.  
public static void sendC2DMUpdate(String message) {  
    UserService userService = UserServiceFactory.getUserService();  
    User user = userService.getCurrentUser();  
    ServletContext context = RequestFactoryServlet.getThreadLocalRequest().getSession().getServletContext();  
    SendMessage.sendMessage(context, user.getEmail(), message);  
}
```  
在下面的示例中，一个帮助类TaskChange，创建了一些常量. 这样一个帮助类能够使得App Engine与Android App直接的交互更简单。
```java
public class TaskChange {  
    public static String UPDATE = "Update";  
    public static String DELETE = "Delete";  
    public static String SEPARATOR = ":";  
}

## (7)Configure the C2DM Client-Side [确认C2DM的客户端]
为了定义Android程序在接受到C2DM的消息的行为，打开C2DMReceiver类, 找到onMessage() 方法. 根据接受到的消息类型进行修改这个方法.
```java
//In your C2DMReceiver class  
  
public void notifyListener(Intent intent) {  
    if (listener != null) {  
        Bundle extras = intent.getExtras();  
        if (extras != null) {  
            String message = (String) extras.get("message");  
            String[] messages = message.split(Pattern.quote(TaskChange.SEPARATOR));  
            listener.onTaskUpdated(messages[0], Long.parseLong(messages[1]));  
        }  
    }  
}  
// Elsewhere in your code, wherever it makes sense to perform local updates  
public void onTasksUpdated(String messageType, Long id) {  
    if (messageType.equals(TaskChange.DELETE)) {  
        // Delete this task from your local data store  
        ...  
    } else {  
        // Call that monstrous Asynctask defined earlier.  
        fetchTasks(id);  
    }  
}
```  
一旦C2DM消息触发了本地进行更新，那么说明已经设置成功。

**Ps:不知何故，官方后来移除了这篇文章，原有的[链接](http://developer.android.com/training/cloudsync/aesync.html))已经失效,云备份我也一直没有接触过，上面的例子也没有自己实践过，当作是知识储备了，大家也可以参考学习下。**

*** 
**转载请注明出自[http://kesenhoo.github.com](http://kesenhoo.github.com)，谢谢配合！**
