---
layout: post
title: Broadcast机制
date: 2017-5-12
categories: blog
tags: [android]
description: Broadcast机制

---
 ### 一.BroadcastReceiver
虽然自从Rxjava出来以后，Broadcast已经用的很少了，但是Broadcast还是有自己的优势的，比如说接收系统广播。

#### 1.注册广播
```
public class MyReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        Log.d("tag","receiverGet");
        //...
    }
}
```

注册广播有两种方式，一种是静态注册，就是在AndroidManifest中注册
```
        <receiver android:name=".business.Broadcast.MyReceiver">
            <intent-filter>
                <action android:name="com.henry.call.LANCH"/>
            </intent-filter>
        </receiver>
```

还有一种是动态注册，通过动态注册的时候一定要及时的解除注册。采用`unregisterReceiver`方法。

```
        IntentFilter intentFilter = new IntentFilter();
        intentFilter.addAction("com.henry.call.LAUNCH");
        registerReceiver(new MyReceiver(),intentFilter);
```

注册完成之后就可以发送广播了。
```
 Intent intent =new Intent();
 intent.setAction("com.henry.call.LAUNCH");
 sendBroadcast(intent);
```

###### 广播注册的内部实现
静态广播的注册是由PMS（PackageManagerService）实现的。而动态的广播从上面代码可以看出是由`registerReceiver`方法开始的。而ContextWrapper把这一方法委托给ContextImpl，调用的是ContextImpl的`registerReceiver`方法
在这个方法里使用跨进程的方式来调用`ActivityManagerNative.getDefault().registerReceiver`方法。
所以说注册广播的真正实现是在AMS当中的。这个地方就是稍微了解一下内部实现过程，源码不深究。

### 二.发送广播

####发送广播的类型
普通广播（Normal Broadcast）
系统广播（System Broadcast）
有序广播（Ordered Broadcast）
粘性广播（Sticky Broadcast）（现在已经不推荐使用）
App应用内广播（Local Broadcast）

##### 1. 普通广播（Normal Broadcast）
若被注册了的广播接收者中注册时intentFilter的action与上述匹配，则会接收此广播（即进行回调onReceive()）

##### 2.有序广播（Ordered Broadcast）
有序广播和普通广播的区别就是可以通过优先级的不同，依次来接收广播，而且优先级高的Receiver可以拦截广播，那么优先级低的Receiver即使action匹配也不会接收到广播，优先级高的Receiver还可以添加信息，传到下一个Recevier。

##### 3.系统广播（System Broadcast）
这个就是接受系统级的广播，调用系统API，比如说拦截短信广播，
只要申请相关的权限就可以了。

##### 4.app应用内广播（Local Broadcast）
应用内广播就是是常用的避免回调的做法。

#### 广播的发送内部实现
其实发送和注册的实现很类似，通过ContextWrapper的`sendBroadcast`方法，还是交给ContextImpl中的 `sendBroadcast`方法去实现。ContextImpl依然是想AMS发送一个异步请求来发送广播。