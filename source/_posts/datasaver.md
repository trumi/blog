---
title: Android全局变量的存储
date: 2016-05-10 16:55:36
tags: [Android , SharedPreferences]
thumbnail: /images/datasaver/thumbnail.jpg
---

### 问题描述
由于最近把项目的返回逻辑修改了一下，重写了onKeyDown()的KEYCODE_BACK事件，从点返回键直接finishi()，改为常驻，也就是模拟home键。
<!--more-->
好吧，我承认这样有点流氓。先不说这个，由此带来了一个问题：项目的全局变量是用一个静态的JavaBean保存的，在登录时将获取的数据保存，供其他界面使用。当App挂在后台时间过久，或者点开了其他的App，导致App申请的内存被回收，但主Activity并未走入onDestroy()，因此App并未退出，无法从Launcher Activity开始周期，而此时全局变量会由于内存释放而丢失。当重新进入App的主界面时，会重新从onCreate()开始生命周期，如果全局变量丢失，直接报NullPointerException


### 了解
Android保存全局变量的方式有几种
* Application
官方推荐继承Application类，在继承的类中保存数据。Application生命周期与Activity同步，也就是说一旦Activity被回收，Application也很难幸免
* 静态类
使用静态类存储比较灵活，Activity、Service都能从中存取数据，但是也会出现长时间后数据丢失的情况，同时也存在着内存泄露的风险
* SharedPreferences
Android数据存储方式的一种，文件将以xml的格式保存在 data/data/包名/shared_prefs 目录下。使用这种方式存储全局变量虽然数据不会丢失，也不存在内存泄露的问题，但涉及文件读写，使用也不及静态类来得方便

### 思路

既然Sharedpreferences使用不如静态类来得方便，静态类的数据保存的又没有SharedPreferences来得持久，于是采用SharedPreferences+静态类的解决方案：在一个静态类中定义两个方法，onCreate()和onStop()，从名字上大概可以猜出，这两个方法是要在Activity的相应生命周期中执行的。onCreate方法中执行对静态数据完整度的检查，如果发现数据为null，则从SharedPreferences中拿数据。onStop()方法中执行全局变量写入至SharedPreferences操作。两个方法分别在主Activity的onCreate()和onStop()中调用

### 实现

再次说明一下，由于在我的项目中，主Activity之前有个LoginActivity，登录成功后，一次性将所有数据全部保存至一个静态的JavaBean中，然后主Activity启动，读取数据进行操作，所以在调用onCreate方法的时候，变量已全部赋好值，不会存在为null的情况，也就不会出现默认只从SharedPreferences中读取的情况

在写onCreate方法的时候就遇到了一个问题，String类型的丢失很好判断，直接看是否为null就行，而int，特别是boolean，不太好处理，因为这两个类型数据一旦丢失，int取值时将是默认值0，而boolean将是默认值false，所以我用了一个JsonObject作为缓冲。

```java
    public static void onCreate() {
        if (sharedPreferences == null) {
        //获取SharedPreferences
            sharedPreferences = mContext.getSharedPreferences(fileName, Context.MODE_PRIVATE);
        }
        //获取SharedPreferences中保存的数据Json
        String data = sharedPreferences.getString("data", "");
        JSONObject temp = null;
        try {
            temp = new JSONObject(data);
        } catch (JSONException e) {
            e.printStackTrace();
        }
        //判断数据是否丢失，通过判断现json的长度与保存在SharedPreferences中的Json长度进行比较
        if (temp != null && (temp.length() ！= jsonObject.length() ) && !"".equals(data)) {
            jsonObject = temp;
            /**
            *变量赋值操作
            *
            */
        }
    }
```

这里可能会有疑问，为什么是通过判断Json长度来确定数据是否有丢失，而不是直接判断Json长度为0或者Json为null。
经过几次调试，发现一个问题，Json中的数据可能不会全部丢失，而是丢失一部分，比如下面这段Json

丢失前
```json
{
    "develop_version": true,
    "token": "token=mqsittvzqr6fw4ahh62q5j62rccrwak4",
    "name": "刘卓旻",
    "studentId": "1305010117",
    "SCOREREMIND": "",
    "rank": 9,
    "department": "计算机科学与工程学院",
    "major": "计算机科学与技术",
    "class": "13计算机1班",
    "week": "11",
    "term": "2015-2016-2",
    "HAVE_NEW_VERSION": false
}
```
丢失后
```json
{
    "token": "token=mqsittvzqr6fw4ahh62q5j62rccrwak4"
}
```
只留下了token！！！
经过分析，原因是App的Service一直在跑，token也一直被Service所持有，所以没有丢失，而其他的字段由于持有它们的类被释放了，所以它们也跟着被释放

接下来是onStop方法
```java
    public static void onStop() {
        editor.putString("data", jsonObject.toString());
        editor.apply();
    }
```
这里只进行了将JsonObject转为String保存在SharedPreferences中的操作

既然出现了SharedPreference.Editor，那就顺便说说SharedPreferences.Editor中commit和apply的区别
* apply没有返回值而commit返回boolean表明修改是否提交成功
* apply是将修改数据提交到内存, 而后异步真正提交到硬件磁盘, 而commit是同步的提交到硬件磁盘，因此，在多个并发的提交commit的时候，他们会等待正在处理的commit保存到磁盘后在操作，从而降低了效率。而apply只是原子的提交到内容，后面有调用apply的函数的将会直接覆盖前面的内存数据，这样从一定程度上提高了很多效率
* 由于在一个进程中，SharedPreference是单实例，一般不会出现并发冲突，如果对提交的结果不关心的话，建议使用apply，当然需要确保提交成功且有后续操作的话，还是需要用commit的

剩下的，就只剩下构建JavaBean了

```java
    public static void setToken(String token) {
        Token = token;
        mputData(TOKEN, token);
    }
    .
    .
    .
    public static String getToken() {
        return Token;
    }
    .
    .
    .
```

mputData方法
```java
private static void mputData(String name, String value) {
        try {
            jsonObject.put(name, value);
        } catch (JSONException e) {
            e.printStackTrace();
        }
    }
```
这个方法进行操作的就是将参数保存在Json中，方便写入SharedPreferences

整个数据保存类就构建完了，接下来就是调用了

由于采用静态类，所以只需要在Launcher实例化一次就行，接下来在LoginActivity'完成所有数据的初始化，进入MainActivity后，在onCreate方法中添加

```java
 @Override
    protected void onCreate(Bundle savedInstanceState) {
          DataSaver.onCreate();   //DataSaver是数据保存类的类名
}
```

重写onStop()，添加
```java
    @Override
    protected void onStop() {
        super.onStop();
        DataSaver.onStop();
    }
```

至此，所有步骤结束。采用这种模式的存储后，数据操作方便，也不会再丢失了，兼顾了静态类和SharedPreferences的特点，降低了文件读写次数的同时保证了数据的完整性。本人亲测，自从采用这种方案后，App再也没有出现过由于丢失数据而Crash的情况了
