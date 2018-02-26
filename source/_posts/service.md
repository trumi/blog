---
title: 关于Service绑定期间对其操作的处理
date: 2017-10-16 14:45:04
tags: [Andorid, Service]
---
我们知道，`bindService()`回调`ServiceConnection()`是异步的，所以如果在绑定服务时调用了`service`的一些方法，此时极有可能`onServiceConnected()`还没有回调，`IBinder`也没有传递过来，导致方法调用失败。

<!--more-->
举个例子：
```java
private boolean mConnected = false;
private Messenger mService = null;
private ServiceConnection mServiceConnection;

mServiceConnection = new ServiceConnection() {
    @Override
     public void onServiceConnected(ComponentName name, IBinder service) {
         mService = new Messenger(service);
         mConnected = true;
    }

    @Override
    public void onServiceDisconnected(ComponentName name) {
        mService = null;
        mConnected = false;
    }
};

bindService(intent, mServiceConnection, Context.BIND_AUTO_CREATE);
if(mConnected){
     mService.send(message);
}
```
上面举的例子，执行的是绑定`service`后，通过`Messenger`进程间通信的操作。可以看到，在执行`bindService`操作后，立即执行了对绑定服务方法的调用，但是极有可能服务绑定操作还没有回调，`mConnected = false`，方法也不会被执行。

那么如果真的有这个需求，在绑定服务的时候就会发起对服务方法的调用，该怎样优雅地处理呢？

### CountDownLatch
#### 简介
`CountDownLatch`的本质是一个"共享锁"，在完成一组正在其他线程中执行的操作之前，它允许一个或多个线程一直等待。

`CountDownLatch`的函数列表
```java
CountDownLatch(int count)
构造一个用给定计数初始化的 CountDownLatch。

// 使当前线程在锁存器倒计数至零之前一直等待，除非线程被中断。
void await()
// 使当前线程在锁存器倒计数至零之前一直等待，除非线程被中断或超出了指定的等待时间。
boolean await(long timeout, TimeUnit unit)
// 递减锁存器的计数，如果计数到达零，则释放所有等待的线程。
void countDown()
// 返回当前计数。
long getCount()
// 返回标识此锁存器及其状态的字符串。
String toString()
```
看到它的功能，你应该可以猜到要拿它来干吗了。既然服务绑定的回调是异步的，那就手动把它变成同步不就好了！

#### 使用

```java
private boolean mConnected = false;
private Messenger mService = null;
private ServiceConnection mServiceConnection;
private CountDownLatch mCountDownLatch = new CountDownLatch(1);

mServiceConnection = new ServiceConnection() {
     @Override
     public void onServiceConnected(ComponentName name, IBinder service) {
         mService = new Messenger(service);
         mConnected = true;
        //计数减1，初始化时传入的计数为1，执行完这步后计数变为0，所有等待的线程被释放
         mCountDownLatch.countDown();
    }

     @Override
    public void onServiceDisconnected(ComponentName name) {
          mService = null;
          mConnected = false;
    }
 };

bindService(intent, mServiceConnection, Context.BIND_AUTO_CREATE);
try {
    //等待，直到CountDownLatch计数变为0后释放
     mCountDownLatch.await();
 } catch (InterruptedException e) {
    Logger.e(TAG, "countDownLatch interrupted failed", e);
}

mService.send(message);
```
在执行完`bindService`后，便调用`CountDownLatch`的`await()`方法，使得接下来的操作处于等待状态。当`onServiceConnected`回调被执行的时候，调用`countDown()`方法，释放处于等待状态的线程。此时`IBinder`已经返回，所以可以对被绑定的服务进行方法调用。

等等！这很优雅吗？！？！不不不！都是骗人的！

### Vector
#### 思路
在`onServiceConnected`回调被执行前，所有的操作都被放入一个`List`，等到回调执行时，遍历`List`，执行所有的操作，遍历完后，清空`List`。之后所有的操作全部直接对`IBinder`进行操作。
```java
private boolean mConnected = false;
private Messenger mService = null;
private ServiceConnection mServiceConnection;
private List<Message> mPendingTasks = new Vector<>();

mServiceConnection = new ServiceConnection() {
     @Override
     public void onServiceConnected(ComponentName name, IBinder service) {
        mService = new Messenger(service);
        mConnected = true;
        for (Message task: mPendingTasks){
            mService.send(task);
        }
        mPendingTasks.clear();
    }

     @Override
    public void onServiceDisconnected(ComponentName name) {
        mService = null;
        mConnected = false;
    }
 };

bindService(intent, mServiceConnection, Context.BIND_AUTO_CREATE);

if(!mConnected){
    mPendingTasks.add(message);
}else{
    mService.send(message);
}
```