---
layout: post
title: android性能优化笔记
date: 2017-8-8
categories: blog
tags: [android java]
description: 对内存方面，绘制渲染方面的性能优化问题的笔记总结。

---

## Activity内存泄漏
activity内存泄漏是内存泄漏的常见情况。而且activity泄漏会浪费巨大的内存空间，因为activity持有大量资源的引用

这里补充一下GC Roots对象的几种情况
* 虚拟机栈（本地变量表）中引用的对象
* 方法区（永久代）中类静态属性引用的对象
* 方法区（永久代）中常量引用的对象
* 本地方法栈汇中引用的对象

### activity静态对象
如果声明一个activity静态变量，并且在引用一个当前运行的activity实例，那么这个activity实例永远不会在GC时被回收。
```
static LeakActivity activity;

void setStaticActivity(){
    activity = this;
}
```

这种情况可以再activity的onDestroy方法里面对静态变量进行解除引用

```
    @Override
    protected void onDestroy() {
        super.onDestroy();
        activity = null;
    }
```
### view静态对象
> 记！！！刚刚写的东西被有道云笔记吞了！！！，在这重新开始写。垃圾软件，毁我青春！！！

深呼吸

有人说activity是重量级的，那我一个view总是轻量级的了吧，现在我有这样一个需求，我有一个view，每次构造的时候比较耗时，那我把它设置成静态的吧，于是你就这样干了。
```
 static View view;
 
 void setStaticView(){
     view = this.findViewById(R.id.***);
 }
```

如果你不对其进行处理，那你就惨了。因为你没有注意到一点，view一旦被添加到界面中，就会持有context的强引用，一个Context被一个静态变量引用，后果可想而知。但是也不是没有解决办法，你只要在activity快销毁的时候将这个view移出你的界面
```
@Override
    protected void onDestroy() {
        super.onDestroy();
        this.removeView(view);
    }
```
这样就万事大吉了。一旦被remove之后就会失去对context的引用。

### 非静态内部类
非静态内部类会导致内存泄漏，这其实就是反应一个事物的两面性，我们知道，内部类可以访问外部类的成员变量，为什么可以访问到呢？这是因为内部类会使用一个静态变量对外部类进行隐式引用，也正是这个原因，非静态内部类会导致内存泄漏。

其中Handler就是一个很好的例子

```
public class LeakActivity extends Activity {
  //...
  Handler handler;
  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    //...
    handler = new Handler() {
      @Override
      public void handleMessage(Message msg) {
              }
  }
}

```
只要你的handler还活着，那么你的activity就不会回收，可能导致内存泄漏

当handler被实例化的时候利用Looper来实现当前线程的消息循环系统，handler会发送message，Looper会持有message对象，而message会持有handler的引用，所以，当handler消息还没处理完时要销毁activity，会因为handler持有activity的引用导致activity没法回收，造成内存泄漏。

这里可以使用静态内部类来解决这个问题，以为静态内部类不会持有外部类的静引用。那你又想问了，如果没有外部类的引用，怎么调用activity里面的成员变量和方法呢。这里可以定义一个Handler子类，并在里面实现对activity的弱引用，

```
private static class MyHandler extends Handler { 
  private final WeakReference<MainActivity> mActivity; 
  // ...
  public MyHandler(MainActivity activity) {
    mActivity = new WeakReference<MainActivity>(activity);
    //... 
  }

  @Override
  public void handleMessage(Message msg) { 
  }
  //...
}
```

### 匿名类
匿名类其实和内部类类似，它们会持有外部类的静态引用。这里的解决办法是使用静态实例，因为匿名类的静态实例不会隐式持有他们外部类的引用。
```
 private static final Runnable mRunnable = new Runnable() {
            @Override
            public void run() {
            //操作
            }
        };
```
像这样的匿名内部类还有Thread TimerTask AsyncTask等



### 如何避免 Activity 泄漏?

* 移除掉所有的静态引用。

* 考虑用 EventBus或RxBus 来解耦 Listener。

* 记着在不需要的时候，解除 Listener 的绑定。

* 尽量用静态内部类。

* 做 Code Review。个人经验：Code Review 能很早的发现内存泄漏。

* 了解你程序的结构。

* 用类似 MAT，Eclipse Analyzer，LeakCanary 这样的工具分析内存。

* 在 Callback 里打印 Log。



## 内存抖动
内存抖动是因为出现大量对象被创建出来，Eden区到达阀值后会进行GC，不停的有大量对象被创建这会导致频繁的GC。这就是内存抖动现象。

这里仅仅说一下防治内存抖动一般要注意的编码习惯。

* 1.字符串拼接优化。
减少字符串使用加号拼接，改为使用StringBuilder。减少StringBuilder.enlarge，初始化时设置capacity；这里需要注意的是，若打开Looper中Printer回调,也会存在较多的字符串拼接。
* 2.读文件优化。 读文件使用ByteArrayPool，初始设置capacity，减少expand
* 3.资源重用。
建立全球缓存池，对频繁申请、释放的对象类型重用
* 4.减少不必要或不合理的对象。
例如在ondraw、getview中应减少对象申请，尽量重用。更多是一些逻辑上的东西，例如循环中不断申请局部变量（尽量将对象内存分配放在循环体外），避免再自定义View的onDraw方法里分配过多对象。



