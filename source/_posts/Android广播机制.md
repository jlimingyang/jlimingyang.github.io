---
title: Android广播机制
date: 2017-01-16 10:00:13
categories: Android
tags: [广播机制,动态监听]
---

# 详解广播机制
Android广播主要分为类型:标准广播和有序广播。
- 标准广播

标准广播是完全异步的，在广播发出后所有的广播接收器机会都会在同一时刻接收到广播，因此他们之间没有任何先后顺序。这种广播效率会比较高，但是这种广播也是无法被截取的。

<img src="https://yuml.me/diagram/scruffy/class/[发出一条广播]-^[广播接收器3],[发出一条广播]-^[广播接收器2],[发出一条广播]-^[广播接收器1]">

- 有序广播

有序广播是一种同步执行的广播，在广播发出后，同一时刻只有一个广播接收器可以接收到这条广播。当这个广播接收器的逻辑执行完毕之后广播才会继续传递。此时的广播接收器是有先后顺序的，优先级越高的广播接收器就可以越先接收到广播消息。而且前面的广播接收器还可以截断正在传递的广播，这样后面的广播接收器就无法接收到该条广播了。

<img src="https://yuml.me/diagram/scruffy/class/[发出一条广播]-^[广播接收器1],[广播接收器1]-^[广播接收器2]">

# 接收系统广播
Android内置了很多级别的广播，我们可以通过监听这些广播来得到各种系统信息。
## 动态监听网路变化
广播接收器可以自由的对自己感兴趣的广播进行注册，这样在该广播接收器就可以只接收自己感兴趣的广播，并在内部处理响应的逻辑。广播接收器的注册一般有两种方式，第一种是在代码中进行注册，这种方式称为动态注册，另一种是在Androidmanifest.xml文件中进行注册，这种注册方式称为静态注册。

首先定义一个类并集成自BroadcastReceiver，然后实现其onReceive()方法。

示例代码:
```Java
class NetWorkChangedReceiver extends BroadcastReceiver{

     @Override
     public void onReceive(Context context, Intent intent) {
         Log.d("NetWorkChangedReceiver", "网路状态改变了");
     }
 }
```
首先创建一个IntentFilter的实例并添加相应的action。然后创建一个NetWorkChangedReceiver实例并调用registerReceiver()方法动态注册。

示例代码:
```Java
        intentFilter=new IntentFilter();
        intentFilter.addAction("android.net.conn.CONNECTIVITY_CHANGE");
        netWorkChangedReceiver=new NetWorkChangedReceiver();
        registerReceiver(netWorkChangedReceiver,intentFilter);
        
```

这样就实现了一个监听网络状态发生变化的广播接收器。

最后不要忘记了动态注册的广播一定要取消注册才行，可以通过调用unregisterReceiver()方法实现。

## 静态注册实现开机自动启动

使用Android Studio自带模版创建一个BootCompleteReceiver，此时就可以在AndroidMainfest.xml文件中就可以看到这里面已经有Android Studio帮我们把BootCompleteReceiver注册好了。此时我们只需要在BootCompleteReceiver中添加如下代码就可以了
示例代码:
```xml
      <intent-filter>
          <action android:name="android.intent.action.BOOT_COMPLETED"></action>
      </intent-filter>
```

当然，最后仍然需要区BootCompleteReceiver的onReceive方法中去实现需要实现的具体逻辑。

# 发送自定义广播
## 发送标准广播

使用静态方法注册广播，具体步骤和做开机广播是一样的。值得注意的是intent-filter中的action就应该是我们自己定义的action。

示例代码：
```Java
Intent intent=new intent("com.ixiongyu.broadcasttest.MY_BROSTCAST");
sendBroadCast(intent);
```
因为是通过intent发送的，因此还可以通过intent携带其他参数传给广播接收器。

## 发送有序广播
广播是一种可以跨进程的通信方式。因此我们程序中发送的广播其他程序应该也是可以接收到的。

示例代码:
```Java
Intent intent=new intent("com.ixiongyu.broadcasttest.MY_BROSTCAST");
sendOrderBroadCast(intent,null);
```
可以看到，发送有序广播只需要在之前的基础上修改一行代码。可以通过在AndroidMainfest文件中注册的相应recevier的intent-filter添加android:priority="100",这里的100就表示优先级。

示例代码:
```xml
<intent-filter android:priority="100">
              <action android:name="android.intent.action.BOOT_COMPLETED"></action>
</intent-filter>
```
在接收到了广播的广播接收器中可以通过abortBroadcast()将这条广播截断，这样后面的广播接收器就无法在接收到这条广播了。

## 使用本地广播
前面我们发出和接受的都是全局广播，即使说发出的广播可以被其他应用应用程序接收到，而且我们也可以接收到来自其他程序的广播。这样就会引起很多安全问题，那么有没有办法解决呢？答案是肯定的，那就是本地广播。使用这个机制的发出的广播只能在应用程序的内部传递，而且广播接收器也只能接收来自本应用程序发出的广播。

本地广播的用法并不不复杂，主要就是使用了一个LocalBroatcastManger()对广播进行管理并提供发送广播和注册广播接收器的方法。
具体实现时先通过LocalBroadcastManager的getInstance()方法得到他的一个实例。然后在注册广播接收器的时候调用LocalBroatcastManger的registerReceiver()方法，发送广播的时候调用的是LocalBroatcastManger的sendBroadCast()方法。

示例代码:
```Java
        localBroadcastManager=LocalBroadcastManager.getInstance(this);
        Intent intent=new Intent("com.ixiongyu.broadcast.LOACL_BROADCAST");
        localBroadcastManager.sendBroadcast(intent);
        intentFilter=new IntentFilter();
        intentFilter.addAction("com.ixiongyu.broadcast.LOACL_BROADCAST");
        localReceiver=new LocalReceiver();
        localBroadcastManager.registerReceiver(localReceiver,intentFilter);
```

## 本地广播的优势
- 可以明确的直到正在发送的广播不会离开自己的程序，一次不用担心数据泄露。
- 其他的程序无法将广播发送到我们程序的内部，因此不需要担心安全漏洞的隐患。
- 本地广播比全局广播更高效。
