---
layout: post
title: android性能优化笔记
date: 2017-8-8
categories: blog
tags: [Android, java]
description: 对内存方面，绘制渲染方面的性能优化问题的笔记总结。
---

## Activity 内存泄漏

activity 内存泄漏是内存泄漏的常见情况。而且 activity 泄漏会浪费巨大的内存空间，因为 activity 持有大量资源的引用

这里补充一下 GC Roots 对象的几种情况

-   虚拟机栈（本地变量表）中引用的对象
-   方法区（永久代）中类静态属性引用的对象
-   方法区（永久代）中常量引用的对象
-   本地方法栈汇中引用的对象

### activity 静态对象

如果声明一个 activity 静态变量，并且在引用一个当前运行的 activity 实例，那么这个 activity 实例永远不会在 GC 时被回收。

```
static LeakActivity activity;

void setStaticActivity(){
    activity = this;
}
```

这种情况可以再 activity 的 onDestroy 方法里面对静态变量进行解除引用

```
    @Override
    protected void onDestroy() {
        super.onDestroy();
        activity = null;
    }
```

### view 静态对象

> 记！！！刚刚写的东西被有道云笔记吞了！！！，在这重新开始写。垃圾软件，毁我青春！！！

深呼吸

有人说 activity 是重量级的，那我一个 view 总是轻量级的了吧，现在我有这样一个需求，我有一个 view，每次构造的时候比较耗时，那我把它设置成静态的吧，于是你就这样干了。

```
 static View view;

 void setStaticView(){
     view = this.findViewById(R.id.***);
 }
```

如果你不对其进行处理，那你就惨了。因为你没有注意到一点，view 一旦被添加到界面中，就会持有 context 的强引用，一个 Context 被一个静态变量引用，后果可想而知。但是也不是没有解决办法，你只要在 activity 快销毁的时候将这个 view 移出你的界面

```
@Override
    protected void onDestroy() {
        super.onDestroy();
        this.removeView(view);
    }
```

这样就万事大吉了。一旦被 remove 之后就会失去对 context 的引用。

### 非静态内部类

非静态内部类会导致内存泄漏，这其实就是反应一个事物的两面性，我们知道，内部类可以访问外部类的成员变量，为什么可以访问到呢？这是因为内部类会使用一个静态变量对外部类进行隐式引用，也正是这个原因，非静态内部类会导致内存泄漏。

其中 Handler 就是一个很好的例子

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

只要你的 handler 还活着，那么你的 activity 就不会回收，可能导致内存泄漏

当 handler 被实例化的时候利用 Looper 来实现当前线程的消息循环系统，handler 会发送 message，Looper 会持有 message 对象，而 message 会持有 handler 的引用，所以，当 handler 消息还没处理完时要销毁 activity，会因为 handler 持有 activity 的引用导致 activity 没法回收，造成内存泄漏。

这里可以使用静态内部类来解决这个问题，以为静态内部类不会持有外部类的静引用。那你又想问了，如果没有外部类的引用，怎么调用 activity 里面的成员变量和方法呢。这里可以定义一个 Handler 子类，并在里面实现对 activity 的弱引用，

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

像这样的匿名内部类还有 Thread TimerTask AsyncTask 等

### 如何避免 Activity 泄漏?

-   移除掉所有的静态引用。

-   考虑用 EventBus 或 RxBus 来解耦 Listener。

-   记着在不需要的时候，解除 Listener 的绑定。

-   尽量用静态内部类。

-   做 Code Review。个人经验：Code Review 能很早的发现内存泄漏。

-   了解你程序的结构。

-   用类似 MAT，Eclipse Analyzer，LeakCanary 这样的工具分析内存。

-   在 Callback 里打印 Log。

## 内存抖动

内存抖动是因为出现大量对象被创建出来，Eden 区到达阀值后会进行 GC，不停的有大量对象被创建这会导致频繁的 GC。这就是内存抖动现象。

这里仅仅说一下防治内存抖动一般要注意的编码习惯。

-   1.字符串拼接优化。
    减少字符串使用加号拼接，改为使用 StringBuilder。减少 StringBuilder.enlarge，初始化时设置 capacity；这里需要注意的是，若打开 Looper 中 Printer 回调,也会存在较多的字符串拼接。
-   2.读文件优化。 读文件使用 ByteArrayPool，初始设置 capacity，减少 expand
-   3.资源重用。
    建立全球缓存池，对频繁申请、释放的对象类型重用
-   4.减少不必要或不合理的对象。
    例如在 ondraw、getview 中应减少对象申请，尽量重用。更多是一些逻辑上的东西，例如循环中不断申请局部变量（尽量将对象内存分配放在循环体外），避免再自定义 View 的 onDraw 方法里分配过多对象。
