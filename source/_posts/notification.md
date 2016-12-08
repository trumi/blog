---
title: Android 7.0 功能体验之通知中心
date: 2016-12-7 16:55:36
tags: [Android 7.0 , 通知中心]
---

在Android 7.0 中，我们重新设计了通知，使其更易于使用并且速度更快。部分变更包括：
<!--more-->
>* 模板更新：我们正在更新通知模板，新强调了英雄形象和化身。开发者将能够充分利用新模板，只需进行少量的代码调整。
* 消息传递样式自定义：您可以自定义更多与您的使用 MessagingStyle 类的通知相关的用户界面标签。您可以配置消息、会话标题和内容视图。
* 捆绑通知：系统可以将消息组合在一起（例如，按消息主题）并显示组。用户可以适当地进行拒绝或归档等操作。如果您已实现 Android Wear 的通知，那么您已经很熟悉此模型。
* 直接回复：对于实时通信应用，Android 系统支持内联回复，以便用户可以直接在通知界面中快速回复短信。
* 自定义视图：两个新的 API 让您在通知中使用自定义视图时可以充分利用系统装饰元素，如通知标题和操作。

>   ————摘自官网

可以看到，Android 7.0 在通知中心的变化大概可以总结为更新的UI样式，支持消息分组折叠，增加MessagingStyle 类以扩展通知栏可用性，支持通知栏直接回复

接下来就来对其中的几个特性进行解析

#### 消息分组
按照官方的说法，当notification的消息设置分组之后，将会对同组过多的通知折叠为一个通知，并可将其展开。

只有一个通知时
![one](/images/notification/Screenshot_1481007464.png)

出现少量通知时
![few](/images/notification/Screenshot_1481007474.png)

出现五条以上的同组通知时
![lot](/images/notification/Screenshot_1481007492.png)

此时在折叠的消息上面下拉就能展开所有通知
![open](/images/notification/Screenshot_1481007494.png)

这其中包含了两个部分，一个是最顶上的`There are %s ActivieNotifications`那一行，暂且称作为summary Notification，另一个就是下面的消息主体。这两个Notification是通过一个Groupkey的字符串相关联的
```
//定义关联组字符
private static final String NOTIFICATION_GROUP = "com.example.android.activenotifications.notification_type";

//定义顶部summary Notification的固定ID，以便后期进行更新消息数字等操作
private static final int NOTIFICATION_GROUP_SUMMARY_ID = -1;

NotificationManager mNotificationManager = (NotificationManager)getSystemService(Context.NOTIFICATION_SERVICE);

private void addNotificationAndUpdateSummaries() {

        final StatusBarNotification[]  activeNotifications = mNotificationManager
                    .getActiveNotifications();   //获取活动的Notification组，该API只有在6.0之后才可使用
        int numberOfNotifications = activeNotifications.length;
        String notificationContent = "There are "+numberOfNotifications +"ActivieNotifications";
           //构造最上方的提示性通知
        final NotificationCompat.Builder builder = new NotificationCompat.Builder(getContext())
                    .setSmallIcon(R.mipmap.ic_launcher)
                    .setStyle(new NotificationCompat.BigTextStyle()
                    .setSummaryText(notificationContent))
                    .setGroup(NOTIFICATION_GROUP)  //设置关联组
                    .setGroupSummary(true);       //如果想要折叠消息，必须设置为true，否则同组消息不会折叠
          final Notification notification = builder.build();
          mNotificationManager.notify(NOTIFICATION_GROUP_SUMMARY_ID, notification);       //NOTIFICATION_GROUP_SUMMARY_ID为顶部summary Notification的固定ID

       //构造底部Notification消息的内容
       final NotificationCompat.Builder builder = new NotificationCompat.Builder(getContext())
               .setSmallIcon(R.mipmap.ic_launcher)
               .setContentTitle(getString(R.string.app_name))
               .setContentText(getString(R.string.sample_notification_content))
               .setAutoCancel(true)
               .setGroup(NOTIFICATION_GROUP);   //必须与上面的summary Notification的NOTIFICATION_GROUP保持一致，且组内的Notification也要保持一致

       final Notification notification = builder.build();
       mNotificationManager.notify(getNewNotificationId(), notification);  //getNewNotificationId()返回一个int类型的数字，作为组内每个Notification的唯一ID

   }
```
反复调用`addNotificationAndUpdateSummaries()`,就可以看到组折叠的效果了。

有趣的是，只有一条Notification的时候，顶上的summary Notification并不会显示，开发者就不用去处理只有一条Notification时和多条Notification下summary Notification的变化了。


#### MessagingStyle

>您可以自定义更多与您的使用 MessagingStyle 类的通知相关的用户界面标签。您可以配置消息、会话标题和内容视图。

>   ————摘自官网

要想使用MessagingStyle非常简单，首先创建一个MessagingStyle对象，然后在`NotificationCompat.Builder`中有个方法叫`setStyle()`(上文的代码中出现过)，直接`setstyle(style)`就行。MessagingStyle是一个强大的通知模板，下面来看看它的大致使用方法。

```
NotificationCompat.MessagingStyle style = new NotificationCompat.MessagingStyle("我");
    style.setConversationTitle("Team lunch")
    style.addMessage("Hi", 100, null)
    style.addMessage("What's up?", 200, "Coworker")
    style.addMessage("Not much", 300, null)
    style.addMessage("How about lunch?", 400, "Coworker"));
NotificationCompat.Builder builder.setStyle(style);
```
呈现出来的效果是这样的
![MessagingStyle](/images/notification/Screenshot_1481016793.png)

先看这一句
```
NotificationCompat.MessagingStyle style = new NotificationCompat.MessagingStyle("我");
```
向MessagingStyle的构造方法传递了一个String类型的参数，它的意义是指定使用者(也就是自己)出现在对话中的称谓

再看这句
```
style.setConversationTitle("Team lunch")
```
这行代码设置了对话的标题

最后就是这若干的addMessage了
```
style.addMessage("Hi", 100, null)
style.addMessage("What's up?", 200, "Coworker")
```
一个style对象可以多次调用`addMessage()`,经过实验，最多可以显示6条对话，如果大于了6条，那么出现时间最久的那条会被挤走。


addMessage共有3个参数，`addMessage(CharSequence text, long timestamp, CharSequence sender)` 。第一个是消息文本内容，第二个是消息添加的时间戳，第三个是这条消息的发话者的称谓，如果传递了null过去，将会被认为这条消息是有使用者发出的，称谓显示为构造方法传递的那个字符串。


#### 直接回复
>对于实时通信应用，Android 系统支持内联回复，以便用户可以直接在通知界面中快速回复短信。


这个特性，搭配MessagingStyle一起来用，简直好用有木有！

![directreply](/images/notification/Screenshot_1481094801.png)

直接回复，是靠Android7.0引入的`RemoteInput`来实现的
```
RemoteInput remoteInput = new RemoteInput.Builder(KEY_TEXT_REPLY)
                .setLabel("回复此消息")
                .build();
```
这样就构造了一个可用的remoteInput。
`setLabel()`方法可设置预显示的文字，有点像EditText里的hint
至于这个KEY_TEXT_REPLY，是个可自定义字符串，相当于ID，在提取RemoteInput输入的文本时会用到

构建好了`RemoteInput`对象，就要靠`NotificationCompat.Action`来绑定到Notification中
```
NotificationCompat.Action action = new NotificationCompat.Action.Builder(android.R.drawable.ic_menu_send, "回复此消息", pendingIntent)
                    .addRemoteInput(remoteInput)
                    .setAllowGeneratedReplies(true)
                    .build();
```
它的构造方法`NotificationCompat.Action.Builder()`三个参数分别为action图标，显示在Notification中的文字，触发后发出的PendingIntent。


接下来
```
NotificationCompat.Builder builder.addAction(action);
```
这样，Notification的最下方就会出现一个输入框了，这个输入框还自带发送按钮。

那么，如何接收输入的文字呢？
在Intent的目标class中
```
Intent intent=getIntent();  //获取传递的Intent
Bundle remoteInput = RemoteInput.getResultsFromIntent(intent);
String text=(String) remoteInput.getCharSequence(KEY_TEXT_REPLY);  //这个KEY_TEXT_REPLY与构造时的对应
```
当然，`RemoteInput`还可以获取`byte`，`float`，`char`等类型数据，根据自己的实际情况选择，如`remoteInput.getFloat(String key)`


`这里着重说明一下`，当点击完发送按钮后，发送按钮会变成一个progressBar，就这么一直转啊转，它不会自己停下来，让它停下来的办法，是将这个Notification再`notify`一次，notify的ID必须要与最初显示时notify的ID一致，否则不但圆圈停不下来，还会新建一个相同的Notification

#### 其他
可能你也发现了，我的这份demo通知栏是有主题色（绿色）的，这个可以通过NotificationCompat.Builder类中的setColor()方法设置，可以改变标题、action button，以及RemoteInput的底色。

跟着RemoteInput来的，还有NotificationCompat.Builder类的`setRemoteInputHistory(String[] history)`方法
![remotehistory](/images/notification/Screenshot_1481100092.png)
与MessagingStyle不一样的是，setRemoteInputHistory之后显示在Notification中的仅有文字，没有所属者等附加信息，且最多只能显示三条，多余的不会显示
