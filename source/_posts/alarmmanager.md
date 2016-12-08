---
title: AlarmManager实现上课静音
date: 2016-02-08 22:40:08
tags: [Android, AlarmManager]
thumbnail: images/alarmmanager/thumbnail.jpg
---
前段时间，把玩别人家app的时候，发现某课程表类app带有上课静音的功能:上课时自动将手机调至震动或静音状态，课间或放学恢复铃声。这简直就是我们这种上课时不时忘记静音的人的福音啊！于是，我也打算在正在做的一个app上加入该功能。
<!--more-->

### 奇怪的解决方案

一开始，准备采用Service+Thread. sleep()的解决方案，sleep到每个上课、下课的时间点，简单粗暴。不过很快就打消了这种念头，太不优雅了。

### 最终的方案
查资料后，发现有个叫AlarmManager的东西。如果想要深入了解，建议可以看看[深入学习android之AlarmManager](http://jinguo.iteye.com/blog/799778)这篇博文。AlarmManager的作用文档中的解释是：在特定的时刻为我们广播一个指定的Intent。简单的说就是我们设定一个时间，然后在该时间到来时，AlarmManager为我们广播一个我们设定的Intent。看上去符合要求，接下来开始解决问题


<!--more-->

### 了解

##### android提供了四种类型的AlarmManager：

* LAPSED_REALTIME
在指定的延时过后，发送广播，但不唤醒设备。

* LAPSED_REALTIME_WAKEUP
在指定的演示后，发送广播，并唤醒设备
延时是要把系统启动的时间SystemClock.elapsedRealtime()算进去的。

* RTC
在指定的时刻，发送广播，但不唤醒设备

* RTC_WAKEUP
在指定的时刻，发送广播，并唤醒设备（本次主要使用这种）

##### AlarmManager提供的方法：
* void set(int type, long triggerAtTime, PendingIntent operation)
设置一个闹钟

* void setRepeating(int type, long triggerAtTime, long interval, PendingIntent operation)
设置一个会重复的闹钟

* void setInexactRepeating(int type, long triggerAtTime, long interval, PendingIntent operation)
设置一个重复闹钟的不精确版本，它相对而言更节能（power-efficient）一些，因为系统可能会将几个差不多的闹钟合并为一个来执行，减少设备的唤醒次数。
内置的几个interval为：
INTERVAL_FIFTEEN_MINUTES
INTERVAL_HALF_HOUR
INTERVAL_HOUR
INTERVAL_HALF_DAY
INTERVAL_DAY
如果你将其设为DAY，那么可能这一天中的所有闹钟都会被合并掉。

* void cancel(PendingIntent operation)
取消一个设置的闹钟

* void setTimeZone(String timeZone)

* LAPSED_REALTIME
在指定的延时过后，发送广播，但不唤醒设备。

* LAPSED_REALTIME_WAKEUP
在指定的演示后，发送广播，并唤醒设备
延时是要把系统启动的时间SystemClock.elapsedRealtime()算进去的。

* RTC
在指定的时刻，发送广播，但不唤醒设备

* RTC_WAKEUP
在指定的时刻，发送广播，并唤醒设备（本次主要使用这种）

##### AlarmManager提供的方法：
* void set(int type, long triggerAtTime, PendingIntent operation)
设置一个闹钟

* void setRepeating(int type, long triggerAtTime, long interval, PendingIntent operation)
设置一个会重复的闹钟

* void setInexactRepeating(int type, long triggerAtTime, long interval, PendingIntent operation)
设置一个重复闹钟的不精确版本，它相对而言更节能（power-efficient）一些，因为系统可能会将几个差不多的闹钟合并为一个来执行，减少设备的唤醒次数。
内置的几个interval为：
INTERVAL_FIFTEEN_MINUTES
INTERVAL_HALF_HOUR
INTERVAL_HOUR
INTERVAL_HALF_DAY
INTERVAL_DAY
如果你将其设为DAY，那么可能这一天中的所有闹钟都会被合并掉。

* void cancel(PendingIntent operation)
取消一个设置的闹钟

* void setTimeZone(String timeZone)
设置系统的默认时区。需要android.permission.SET_TIME_ZONE权限

---
### 思路

* 将每天上午，下午，晚上的课分为三个模块，分别以一个以天为单位进行循环计划任务为其第一节小课的上课时间定时。

* 在每个模块里，再细分一次，化为第一节小课上课，第一节小课下课，第二节小课上课，第二节小课下课。当以天为单位进行循环计划任务唤醒第一节小课的上课时间时，设定一个距离此时45分钟后开始的计划任务，也就是上课时间。当此计划任务唤醒第一节小课下课时间时，设定一个距离此时10分钟后开始的计划任务，也就是下课时间。当此计划任务唤醒第二节小课上课时间时，设定一个距离此时45分钟后开始的计划任务。当此计划任务唤醒第二节小课下课时间时，完成一个周期。

* 在最后一个小节，也就是第二节小课下课时间触发后，判断当前是上午，下午，还是晚上。如果是上午或下午，再设定一个距离此时20分钟后开始的计划任务，也就是大课下课时间，用来唤醒第二节大课的第一节小课上课时间，从而再进行一个周期。如果是晚上，则不进行操作，从而完成了一天的周期

---

### 开始

#### 定义两种唤醒方式

以天为单位进行循环计划任务

{% codeblock lang:java %}
    public void startSheduleEveryday(Intent intent, int requestCode, int hour, int minuts) {
        Calendar calendar = Calendar.getInstance();

        calendar.set(Calendar.HOUR_OF_DAY, hour);
        calendar.set(Calendar.MINUTE, minuts);
        calendar.set(Calendar.SECOND, 0);
        calendar.set(Calendar.MILLISECOND, 0);

        PendingIntent sender = PendingIntent.getBroadcast(mcontext, requestCode, intent, 0);
        AlarmManager am = (AlarmManager) mcontext.getSystemService(Context.ALARM_SERVICE);
        am.setRepeating(AlarmManager.RTC_WAKEUP,
                calendar.getTimeInMillis(), AlarmManager.INTERVAL_DAY, sender);
    }
{% endcodeblock %}

距离此时后多少秒开始计划任务

{% codeblock  lang:java %}
    public void startSheduleDelayTime(Intent intent, int requestCode, int delaySecond) {
        Calendar calendar = Calendar.getInstance();
        calendar.setTimeInMillis(System.currentTimeMillis());
        calendar.add(Calendar.SECOND, delaySecond);
        PendingIntent sender = PendingIntent.getBroadcast(mcontext, requestCode, intent, 0);
        AlarmManager am = (AlarmManager) mcontext.getSystemService(Context.ALARM_SERVICE);
        am.set(AlarmManager.RTC_WAKEUP, calendar.getTimeInMillis(), sender);
    }
    {% endcodeblock %}

#### 初始化计划任务
主要是初始化上午，下午，晚上开始上课的计划任务
{% codeblock lang:java %}
                        Intent muteIntent = new Intent();
                        muteIntent.setAction(BackGroundScheduleTaskReceiver.ACTION);
                           //上午
                        muteIntent.putExtra("time", DateManager.MORNING);
                        muteIntent.putExtra("hour",8);
                        muteIntent.putExtra("minute",0);
                        startSheduleEveryday(muteIntent, 101, 8, 0);
                        //夏季作息时间( isSummer为我自定义一个类里的方法，判断是否为夏季作息时间，下面的isWinter()同理 )
                        if (dateManager.isSummer()) {
                            //下午
                            muteIntent.putExtra("time", DateManager.AFTERNOON);
                            muteIntent.putExtra("hour",14);
                            muteIntent.putExtra("minute",30);
                            startSheduleEveryday(muteIntent, 102, 14, 30);
                            //晚上
                            muteIntent.putExtra("time", DateManager.NIGHT);
                            muteIntent.putExtra("hour", 19);
                            muteIntent.putExtra("minute",30);
                            startSheduleEveryday(muteIntent, 103, 19, 30);
                            //冬季作息时间
                        } else if (dateManager.isWinter()) {
                            //下午
                            muteIntent.putExtra("time", DateManager.AFTERNOON);
                            muteIntent.putExtra("hour",14);
                            muteIntent.putExtra("minute",0);
                            startSheduleEveryday(muteIntent, 102, 14, 0);
                            //晚上
                            muteIntent.putExtra("time", DateManager.NIGHT);
                            muteIntent.putExtra("hour", 19);
                            muteIntent.putExtra("minute",0);
                            startSheduleEveryday(muteIntent, 103, 19, 0);
                        }
{% endcodeblock %}

#### 定义BroadcastReceiver接收
由于AlarmManager通过发送广播传送Intent，所以要定义一个BroadcastReceiver接收Intent

{% codeblock lang:java %}
public class BackGroundScheduleTaskReceiver extends BroadcastReceiver {

    public static final String ACTION = "scheduleTask";

    private final String TAG="Mute_In_Class";

    private AudioManager audioManager;
    //loopMark用来记录每个小周期内的阶段
    //classTime用来记录重复周期的次数
    private int loopMark = 1,classTime=0;

    private ScheduleTask scheduleTask;

    @Override
    public void onReceive(Context context, Intent intent) {
        if (audioManager==null||scheduleTask==null) {
            audioManager = (AudioManager) context.getSystemService(Context.AUDIO_SERVICE);
            scheduleTask = new ScheduleTask(context);
        }
        String action = intent.getAction();
        switch (action) {
            case ACTION:
                String time = intent.getStringExtra("time");
                int hour = intent.getIntExtra("hour", 8);
                int minute = intent.getIntExtra("minute", 0);
                switch (time) {
                    case DateManager.MORNING:
                        muteHandler(DateManager.MORNING);
                        break;
                    case DateManager.AFTERNOON:
                        muteHandler(DateManager.AFTERNOON);
                        break;
                    case DateManager.NIGHT:
                        muteHandler(DateManager.NIGHT);
                        break;
                    default:
                        break;
                }
                break;
            default:
                break;
        }
    }

    private void muteHandler(String time) {
        Intent muteIntent = new Intent();
        //向自身发送Broadcast
        muteIntent.setAction(BackGroundScheduleTaskReceiver.ACTION);
        switch (loopMark) {
            //第一节小课上课
            case 1:
            //静音且震动
                setRingMode(true, true);
                scheduleTask.startSheduleDelayTime(muteIntent, 1, 45 * 60 * 1000);
                loopMark++;
                Log.i(TAG,"第一节小课上课");
                break;
            //第一节小课下课
            case 2:
            //恢复铃声
                setRingMode(false, true);
                scheduleTask.startSheduleDelayTime(muteIntent, 2, 60 * 1000);
                loopMark++;
                Log.i(TAG,"第一节小课下课");
                break;
            //第二节小课上课
            case 3:
            //静音且震动
                setRingMode(true, true);
                scheduleTask.startSheduleDelayTime(muteIntent, 3, 45 * 60 * 1000);
                loopMark++;
                Log.i(TAG,"第二节小课上课");
                break;
            //第二节小课下课
            case 4:
            //恢复铃声
                setRingMode(false, true);
                if (classTime == 1 && !DateManager.NIGHT.equals(time)) {
                    scheduleTask.startSheduleDelayTime(muteIntent, 4, 20 * 60 * 1000);
                    classTime++;
                }else{
                    classTime=1;
                }
                loopMark = 1;
                Log.i(TAG,"第二节小课下课");
                break;
            default:
                break;
        }
    }
     /**
     * 静音管理
     *
     * @param slient    是否静音
     * @param vibrate    是否震动
     *
     */
    private void setRingMode(boolean slient, boolean vibrate) {
        if (slient) {
            audioManager.setRingerMode(vibrate ? AudioManager.RINGER_MODE_VIBRATE :
                    AudioManager.RINGER_MODE_SILENT);
        } else {
            audioManager.setRingerMode(AudioManager.RINGER_MODE_NORMAL);
        }
    }
}
{% endcodeblock %}

至此，上课静音功能已实现

---

### 小结

虽然这种解决方法可能不是最好的，但也花费了一点心思。AlarmManager用在长时间定时启动方面非常合适，毕竟系统闹钟也是用的AlarmManager，比线程休眠来得更加优雅。
