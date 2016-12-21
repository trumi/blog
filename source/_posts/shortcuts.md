---
title: Android 7.1功能体验之Shortcuts
date: 2016-12-20 10:04:18
tags: [Android 7.1, Shortcuts]
thumbnail: /images/shortcuts/shortcuts.png
---
## 简介

如果你的App构建的target是Android 7.1，你就可以使用`Shortcuts`为用户提供一些常用功能的入口，`Shortcuts`可以显示在那些支持`Shortcuts`的launcher上，如Pixel launcher。`Shortcuts`的出现可以让用户更快捷的使用一些常用的功能，而不用从launcher activity一步步点击。
<!--more-->

![Shortcuts](/images/shortcuts/shortcuts.png)

`Shortcuts`通过长按launcher上的App图标激活，激活后，会弹出条状可点击的快捷方式。它的触发方式与App图标位置的移动并不冲突。当长按图标后，首先会弹出`Shortcuts`快捷方式，当手指按住不放，并向任意方向拖动图标时，`Shortcuts`快捷方式将会消失，App图标进入悬浮状态。点击每个Shortcut后面的双横杠时，还可将此shortcut添加至桌面快捷方式。

`Shortcuts`可以通过两种方式添加：
* 静态：在`AndroidManifest.xml`中注册，在xml文件中添加`shortcut`的各项属性
* 动态：由[ShortcutManager](https://developer.android.com/reference/android/content/pm/ShortcutManager.html)管理，完成Shortcuts的添加，更新，删除等各项操作

静态的`Shortcuts`和动态的可以共存，它们之间也有优先级之分，稍后会说明。一个App最多可添加5个`Shortcuts`（静态+动态）

看起来是不是和`Broadcast`的注册有点类似。接下来就逐一介绍

## 静态Shortcuts
由于静态Shortcuts的特性，在下一次应用更新前，由静态方式添加的Shortcuts的内容无法改变。所以建议通过静态方式添加一些固定且常用的功能，比如扫码

#### 静态添加的主要步骤
* 在`AndroidManifest.xml`中找到launcher activity，在这个activity的节点下添加`<meta-data>`
```xml
<activity android:name=".MainActivity">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>

    <meta-data
        android:name="android.app.shortcuts"
        android:resource="@xml/shortcuts" />
</activity>
```
* 在`res/xml/`目录下创建`shortcuts.xml`(文件名不一定叫这个名字，只要与`<meta-data>`中的`android:resource`能对应就行)，在这个文件中填写所有静态`Shortcuts`的属性

```xml
<?xml version="1.0" encoding="utf-8"?>
<shortcuts xmlns:android="http://schemas.android.com/apk/res/android">
    <shortcut
        android:shortcutId="adjacentactivity"
        android:enabled="true"
        android:icon="@drawable/ic_menu_camera"
        android:shortcutShortLabel="@string/adjacentactivity"
        android:shortcutLongLabel="@string/adjacentactivity"
        android:shortcutDisabledMessage="@string/adjacentactivity">
        <intent
            android:action="android.intent.action.VIEW"
            android:targetPackage="com.hnust.lzm.newfuncdemo"
            android:targetClass="com.hnust.lzm.newfuncdemo.activity.multiWindowActivities.AdjacentActivity" />
        <categories android:name="android.shortcut.conversation" />
    </shortcut>

    <shortcut
        android:shortcutId="LaunchBoundsActivity"
        android:enabled="true"
        android:icon="@drawable/ic_menu_send"
        android:shortcutShortLabel="@string/LaunchBoundsActivity"
        android:shortcutLongLabel="@string/LaunchBoundsActivity"
        android:shortcutDisabledMessage="@string/LaunchBoundsActivity">
        <intent
            android:action="android.intent.action.VIEW"
            android:targetPackage="com.hnust.lzm.newfuncdemo"
            android:targetClass="com.hnust.lzm.newfuncdemo.activity.multiWindowActivities.LaunchBoundsActivity" />

        <intent
            android:action="android.intent.action.VIEW"
            android:targetPackage="com.hnust.lzm.newfuncdemo"
            android:targetClass="com.hnust.lzm.newfuncdemo.activity.multiWindowActivities.AdjacentActivity" />
        <categories android:name="android.shortcut.conversation" />
    </shortcut>
</shortcuts>
```

#### Shortcut的主要属性
* `android:shortcutId`  每个Shortcut的唯一ID
* `android:enabled`   该Shortcut是否可用，不可用时图标会变灰，点击后无响应或弹出自定义的提示
* `android:icon`   Shortcut显示的图标，如果添加到了桌面快捷方式，则此图标会被设置成桌面快捷方式图标
* `android:shortcutShortLabel`  添加至桌面快捷方式后快捷方式的名称
* `android:shortcutLongLabel`   长按图标后显示在Shortcuts快捷组内的名称
* `android:shortcutDisabledMessage`  当快捷方式被禁用时显示的提示语

在第一个`shortcut`里，可以看到一个`<intent>`标签，这个`intent`就是用户选中快捷方式之后将会触发的intent。

在第二个`shortcut`里，可以看到两个`<intent>`标签。在同一个`shortcut`里，允许存在多个Intent，多个Intent的触发方式是：所有Intent都会被触发，然后依次入`Activity`栈，最终显示给用户交互的`Activity`是位于栈顶，也就是最后一个Intent所调起的`Activity`。当用户点击back键后，`Activity`会依次出栈，用户也将会看到所有由Intent调起的`Activity`。


## 动态Shortcuts
动态的Shortcuts比较灵活，可以在用户使用app的过程中进行添加, 更新, 删除等操作。
```java
ShortcutManager shortcutManager = getSystemService(Context.SHORTCUT_SERVICE);

ShortcutInfo shortcut = new ShortcutInfo.Builder(this, "id1")
    .setShortLabel("Web site")
    .setLongLabel("Open the web site")
    .setIcon(Icon.createWithResource(context, R.drawable.icon_website))
    .setIntent(new Intent(Intent.ACTION_VIEW,
                   Uri.parse("https://www.baidu.com/")))
    .build();

shortcutManager.setDynamicShortcuts(Arrays.asList(shortcut));
```
上面的代码就添加了一个`Web Site`的Shortcut。每一个shortcut在代码中以`ShortcutInfo`体现，然后通过`ShortcutManager`进行相应的操作。应用被关闭以后，动态Shortcuts依旧存在。

#### ShortcutManager的主要方法
* 添加：`setDynamicShortcuts(List)`设置整个的动态快捷方式，` addDynamicShortcuts(List)`向已有快捷方式列表中新的快捷方式添加
* 更新：`updateShortcuts(List)`更新已存在的快捷方式
* 删除：`removeDynamicShortcuts(List)`移除指定的动态快捷方式，`removeAllDynamicShortcuts()`移除所有的动态快捷方式
* 禁用：`disableShortcuts(List)`禁用任何已存在的快捷方式，`disableShortcuts(List, Charsquence)` 点击时会给出错误提示
* `getMaxShortcutCountPerActivity()`返回当前launcher最多支持的Shortcuts个数
* `getDynamicShortcuts()`   返回动态快捷方式的`List<ShortcutInfo>`
* `getManifestShortcuts()`   返回静态快捷方式的`List<ShortcutInfo>`

与静态快捷方式相似，动态快捷方式同样支持多个Intent构建back stack，只需要将`setIntent()`改为`setIntents()`
```java
ShortcutInfo shortcut = new ShortcutInfo.Builder(this, "id1")
    .setShortLabel("Web site")
    .setLongLabel("Open the web site")
    .setIcon(Icon.createWithResource(context, R.drawable.icon_website))
    .setIntents(new Intent[]{
      new Intent(Intent.ACTION_MAIN, Uri.EMPTY, this, targetActivity.class),
      new Intent(Intent.ACTION_VIEW,Uri.parse("https://www.baidu.com/"))
               })
    .build();
```

#### 注意点
* 禁用和删除指定的快捷方式时，传入的`List`参数只需要带上指定快捷方式的ID即可，如
```java
//对ID为“id1”的快捷方式进行操作
shortcutManager.disableShortcuts(Arrays.asList("id1"));
shortcutManager.removeDynamicShortcuts(Arrays.asList("id1"));
```

* 更新操作时，保持待更新的Shortcut的ID不变，其余的变化即可
```java
//对ID为“id1”的快捷方式进行更新操作
ShortcutInfo shortcut = new ShortcutInfo.Builder(this, "id1") //ID保持不变
    .setShortLabel("A new web site")
    .setLongLabel("Open the new web site")
    .setIcon(Icon.createWithResource(context, R.drawable.icon_website))
    .setIntent(new Intent(Intent.ACTION_VIEW,
                   Uri.parse("https://www.google.com/")))
    .build();
shortcutManager.updateShortcuts(Arrays.asList(shortcut));
```

* 不可以通过`ShortcutManager`对静态快捷方式进行修改操作，若执行了修改会抛出`IllegalArgumentException`

## 优先级
优先级主要体现在快捷方式弹出时与App图标的之间距离，在快捷方式组中，越靠近App图标的快捷方式优先级越高
* 静态快捷方式的优先级永远高于动态快捷方式
* 静态快捷方式的优先级按照`shortcuts.xml`文件中的顺序依次排列
* 动态快捷方式可通过`setRank（int rank）`来手动设置优先级
```java
ShortcutInfo shortcut = new ShortcutInfo.Builder(MainActivity.this, "shortcut")
            .setRank(1)
            .build();
```
  `rank`值越大，优先度越低。静态快捷方式地`rank`值为0
* `setRank()`不能传入负值，否则会抛出异常
