---
title: Android 7.0 功能体验之Quick Setting Tile
date: 2016-12-09 15:21:52
tags: [Android 7.0 , Quick Setting Tile]
---
>“快速设置”通常用于直接从通知栏显示关键设置和操作，非常简单。在 Android 7.0 中，我们已扩展“快速设置”的范围，使其更加有用更方便。
对于开发者，Android 7.0 还添加了一个新的 API，从而让您可以定义自己的“快速设置”图块，使用户可以轻松访问您应用中的关键控件和操作。
<!--more-->

![quicksetting](/images/quicksettingtile/quicksettings.png)
Quick Setting Tile 可以被赋予很多功能，比如作为单纯的开关，启动一个Activity，显示一个Dialog供用户选择。

## 了解
每一个Tile都是一个`android.service.quicksettings.Tile`对象，它继承自`Object`，实现了`Parcelable`接口

Quick Setting Tile的响应动作要靠TileService来完成，它继承自`android.app.Service`

Tile会在用户第一次启动App后出现在通知中心的快捷方式中，并将在App被删除之后消失。它默认是不显示的，用户需要自定义快捷图标时自己拖拽添加

## 使用
首先要在AndroidMenifast.xml文件中注册
```xml
<service
    android:name=".MyQSTileService"
    android:label="@string/my_default_tile_label"
    android:icon="@drawable/my_default_icon_label"
    android:permission="android.permission.BIND_QUICK_SETTINGS_TILE">
    <intent-filter>
        <action android:name="android.service.quicksettings.action.QS_TILE" />
    </intent-filter>
</service>
```
`android:label`和`android：icon`将初始化Tile的提示文字和图标

TileService的运行需要`android.permission.BIND_QUICK_SETTINGS_TILE`权限

先来看一个作为开关的demo
```java
public class QuickSettingsService extends TileService {

    private static final String SERVICE_STATUS_FLAG = "serviceStatus";
    private static final String PREFERENCES_KEY = "com.google.android_quick_settings";

    /**
     * 当Tile被添加至快速设置区域时调用，可进行初始化操作
     */

    @Override
    public void onTileAdded() {
        Log.d("QS", "Tile added");
    }

    /**
     * 当用户下拉通知中心，Tile可见时调用，可初始化Tile的状态
     */
    @Override
    public void onStartListening() {
        Log.d("QS", "Start listening");
    }

    /**
     * 当用户点击该Tile时调用，进行相关操作
     */
    @Override
    public void onClick() {
        Log.d("QS", "Tile tapped");

        updateTile();
    }

    /**
     * 当通知中心关闭，Tile变为不可见时调用
     */
    @Override
    public void onStopListening() {
        Log.d("QS", "Stop Listening");
    }

    /**
     * 当Tile从快速设置区域被移除至时调用
     */
    @Override
    public void onTileRemoved() {
        Log.d("QS", "Tile removed");
    }

    // 更改Tile的状态
    private void updateTile() {

        Tile tile = this.getQsTile(); //获取Tile对象
        boolean isActive = getServiceStatus();

        Icon newIcon;
        String newLabel;
        int newState;

        // 将Tile更改为与Service同步的状态
        if (isActive) {

            newLabel = String.format(Locale.US,
                    "%s %s",
                    getString(R.string.tile_label),
                    getString(R.string.service_active));

            newIcon = Icon.createWithResource(getApplicationContext(),
                    ic_android_black_24dp);

            newState = Tile.STATE_ACTIVE;

        } else {
            newLabel = String.format(Locale.US,
                    "%s %s",
                    getString(R.string.tile_label),
                    getString(R.string.service_inactive));

            newIcon =
                    Icon.createWithResource(getApplicationContext(),
                            android.R.drawable.ic_dialog_alert);

            newState = Tile.STATE_INACTIVE;
        }

        // 更改Tile的UI状态
        tile.setLabel(newLabel);
        tile.setIcon(newIcon);
        tile.setState(newState);
        tile.updateTile();   // 通过updateTile()提交Tile状态更新
    }

    private boolean getServiceStatus() {

        SharedPreferences prefs =
                getApplicationContext()
                        .getSharedPreferences(PREFERENCES_KEY,
                                MODE_PRIVATE);

        boolean isActive = prefs.getBoolean(SERVICE_STATUS_FLAG, false);
        isActive = !isActive;

        prefs.edit().putBoolean(SERVICE_STATUS_FLAG, isActive).apply();

        return isActive;
    }
}
```

## 提供的方法

先来看Tile
* describeContents()   描述此Parcelable实例的封送表示中包含的特殊对象的类型
* getContentDescription()   获取当前Tile的描述文本内容
* getIcon()   获取当前Tile的Icon
* getLabel()  获取当前Tile的Label
* getState()  返回当前Tile的状态，内置有三种`STATE_ACTIVE`、`STATE_INACTIVE`、`STATE_UNAVAILABLE`
* setContentDescription(CharSequence contentDescription)   设置当前Tile的描述文本内容
* setIcon(Icon icon)  设置当前Tile的Icon
* setLabel(CharSequence label)   设置当前Tile的Label
* setState(int state)   设置当前Tile的状态
* updateTile()  提交当前更改的状态从而更新Tile的显示状态
* writeToParcel(Parcel dest, int flags)   将对象写入Parcel


再来看TileService的
* getQsTile() 获取service中的Tile
* isLocked()  检查设备是否处于锁屏状态
* isSecure()  检查设备是否处于安全模式
* onBind(Intent intent)
* onClick()
* onDestroy()
* onStartListening()
* onStopListening()
* onTileAdded()
* onTileRemoved()
* requestListeningState(Context context, ComponentName component)   请求Tile变为监听模式以发送状态更新
* showDialog(Dialog dialog)   关闭通知栏并弹出Dialog
* startActivityAndCollapse(Intent intent)   关闭通知栏并启动一个Activity
* unlockAndRun(Runnable runnable)   在执行Runnable之前提示用户解锁设备，如果用户解锁，则Runnable执行，反之不执行
