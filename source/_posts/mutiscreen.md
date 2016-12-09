---
title: Android 7.0 功能体验之多窗口支持
date: 2016-12-6 16:55:36
tags: [Android 7.0 , 多窗口]
thumbnail: /images/mutiscreen/mw-splitscreen.png
---

### 概览
Android N 允许多个应用同时共享屏幕。例如，用户可以分屏显示应用，在左边查看网页，同时在右边写邮件。 用户体验取决于设备：
<!--more-->
运行 Android N 的手持设备具有分屏模式。 在此模式中，系统以左右并排或上下并排的方式分屏显示两个应用。 用户可以拖动两个应用之间的分界线，放大其中一个应用，同时缩小另一个。
在运行 Android N 的 Nexus Player 上，应用能以画中画模式显示，即在用户浏览网页或与其他应用交互的同时继续显示内容。
较大设备的制造商可选择启用自由形状模式，在该模式中，用户可以自由调整各 Activity 的尺寸。 若制造商启用此功能，设备将同时具有自由形状模式和分屏模式。

![asd](/images/mutiscreen/mw-splitscreen.png)

用户可以通过以下方式切换到多窗口模式：

若用户打开 Overview 屏幕并长按 Activity 标题，则可以拖动该 Activity 至屏幕突出显示的区域，使 Activity 进入多窗口模式。
若用户长按 Overview 按钮，设备上的当前 Activity 将进入多窗口模式，同时将打开 Overview 屏幕，用户可在该屏幕中选择要共享屏幕的另一个 Activity。
用户可以在两个 Activity 共享屏幕的同时在这两个 Activity 之间拖放数据 （在此之前，用户只能在一个 Activity 内部拖放数据）。

————摘自官网

### 解读
若要将app调整为可供分屏形式，需要在AndroidManifest.xml中的`<activity>`或 `<application>`节点中添加：

```xml
android:resizeableActivity=["true" | "false"]
```
如果该属性设置为 true，Activity 将能以分屏和自由形状模式启动。 如果此属性设置为 false，Activity 将不支持多窗口模式。 如果该值为 false，且用户尝试在多窗口模式下启动 Activity，该 Activity 将全屏显示。

如果应用targetSdkVersion为 Android N，如果但未对该属性指定值，则该属性的值默认设为 true。
若要设置为可画中画模式，则在AndroidManifest.xml中的`<activity>`或 `<application>`节点中添加：
```xml
android:supportsPictureInPicture=["true" | "false"]
```
目前画中画模式还只是　Andorid TV　可用

现在我们来借用多窗口的官方[demo](https://github.com/googlesamples/android-MultiWindowPlayground)来进行解读

官方的demo中，包含了６种不同的Activity启动方式，分别为
* 不做任何处理启动的Activity
* 将resizeableActivity属性手动设置为`false`的Activity
* 在启动Activity的Intent中同时添加了`Intent.FLAG_ACTIVITY_LAUNCH_ADJACENT`和`Intent.FLAG_ACTIVITY_NEW_TASK`属性
* 注册了屏幕状态改变事件的Activity
* 指定了最小宽高的Activity
* 设置了创建时所在屏幕的位置与自身大小的Activity

第一个，什么都不做处理的Activity，在分屏模式下，会直接覆盖在原有Activity的位置   

第二个，将resizeableActivity属性手动设置为false的Activity，如果原Activity处于分屏模式中的话，会退出分屏模式，变为正常模式，并弹出提示Toast“应用不支持分屏。”，此时按返回键，原有的Activity也将会处于全屏模式      

第三个比较有意思，同时添加了这两种属性的话，假设已经进入了分屏模式，主Activity位于整个屏幕的一侧，那么加上这个属性启动的Activity将会被放置在主Activity的另一侧，变为这样
![adjacent](/images/mutiscreen/adjacent.png)
上层Activity为主Activity,下层的Activity则为新建的Activity    
如果不处于分屏模式下，就只是在新的任务栈中启动一个新的Activity，不会变成分屏模式

第四个，在AndroidManifest清单文件下的对应`<activity>`节点中添加了
```xml
android:configChanges="screenSize|smallestScreenSize|screenLayout|orientation"
```
这样的话，在Activity中重写
```java
onConfigurationChanged(Configuration newConfig)
```
可以获取到Activity窗口变化的大小等数据

第五个，在`<activity>`节点下加入`<layout>`节点。`<layout>`节点可以自定义的属性有
```
android:defaultHeight　　 配置自由模式下默认的高度

android:defaultWidth　　　配置自由模式下默认的宽度

android:gravity　　　　　　配置自由模式下activity的初始位置

android:minHeight　　　 　配置分屏和自由形状模式下最小的高度

android:minWidth 　　　 　配置分屏和自由形状模式下最小的宽度
```
如
```xml
<activity android:name=".MyActivity">
    <layout android:defaultHeight="500dp"
          android:defaultWidth="600dp"
          android:gravity="top|end"
          android:minHeight="450dp"
          android:minWidth="300dp" />
</activity>

```
注意，这个地方有两个坑。首先，是在自由模式下，前三个属性才有作用，分屏模式和普通模式没有作用。第二，官方将最后两个属性写为`android:minimalHeight`和`android:minimalWidth`，本人表示编译过不了，查阅资料后发现已更名，坑啊．．．     

这里还有一个坑，官方文档里这样描述自由模式
>较大设备的制造商可选择启用自由形状模式，在该模式中，用户可以自由调整各 Activity 的尺寸。

我下载的原生7.0镜像中并没有开启自由模式的地方，只有分屏，这让我一度怀疑人参＝＝，幸好还有万能的google哈哈哈，只要输入
```shell
adb shell settings put global enable_freeform_support 1
adb reboot
```
等到重启完成，进入后台切换卡片界面
![recentActivity](/images/mutiscreen/recentActivity.png)
可以看到在右上角的×旁边多了个按钮，点击即可进入自由模式
![freeform](/images/mutiscreen/freeform.png)

第六个，在启动Activity的Intent中加了个Rect，
```java
    Rect bounds = new Rect(500, 300, 100, 0);
    ActivityOptions options = ActivityOptions.makeBasic();
    options.setLaunchBounds(bounds);
    intent = new Intent(getActivity(), LaunchBoundsActivity.class);
    startActivity(intent, options.toBundle());
```
即可在自由模式下设置Activity大小

### 生命周期
在多窗口模式中，在指定时间只有最近与用户交互过的 Activity 为活动状态。 该 Activity 将被视为顶级 Activity。 所有其他 Activity 虽然可见，但均处于暂停状态。 但是，这些已暂停但可见的 Activity 在系统中享有比不可见 Activity 更高的优先级。 如果用户与其中一个暂停的 Activity 交互，该 Activity 将恢复，而之前的顶级 Activity 将暂停。

注：在多窗口模式中，用户仍可以看到处于暂停状态的应用。 应用在暂停状态下可能仍需要继续其操作。 例如，处于暂停模式但可见的视频播放应用应继续显示视频。 因此，我们建议播放视频的 Activity 不要暂停其 `onPause()` 处理程序中的视频。 应暂停 `onStop()` 中的视频，并恢复 `onStart()` 中的视频播放。

————摘自官网

### 其他
新增了几个api
* Activity.isInMultiWindowMode() 　　　　　　查询是否处于多窗口模式
* Activity.isInPictureInPictureMode() 　　　查询是否处于画中画模式
* Activity.onMultiWindowModeChanged() 　　　多窗口模式变化时进行通知(进入或退出多窗口)
* Activity.onPictureInPictureModeChanged() 画中画模式变化时进行通知（进入或退出画中画模式）
* Activity.enterPictureInPictureMode()　　　调用这个接口进入画中画模式，如果系统不支持，这个调用无效
* ActivityOptions.setLaunchBounds()　　　　　在系统已经处于Freeform模式时，可以通过这个参数来控制新启动的Activity大小，如果系统不支持，这个调用无效

fragment也有自己对应的方法，如`getActivity().isInMultiWindowMode()`。

从Android N开始，系统支持在多个Activity之间进行拖拽，对应的api有
* DragAndDropPermissions
* View.startDragAndDrop()
* View.cancelDragAndDrop()
* View.updateDragShadow()
* Activity.requestDragAndDropPermissions()

这里不做过多的阐述，有兴趣的话可以参考官方文档


### 最后
如果你对分屏是怎么实现的，推荐看下这篇博文[Android 7.0中的多窗口实现解析](http://www.codeceo.com/article/android-multi-window.html)
