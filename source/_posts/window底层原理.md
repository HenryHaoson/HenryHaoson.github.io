---
layout: post
title: WMS Window工作原理
date: 2017-8-23
categories: blog
tags: [Android]
description: 最近在研究framework层的代码，之前听说过一些，神马AMS，PMS，WMS 啥啥啥的。所以就来看一看WMS（Window Manager Service）。
---

最近在研究 framework 层的代码，之前听说过一些，神马 AMS，PMS，WMS 啥啥啥的。所以就来看一看 WMS（Window Manager Service）。

## Window 对象的创建

在 android 机制里面，view 是视图呈现的载体，然而 view 也不能够单独存在，它也必须依赖 window 才能存在。

打开源码可以发现 window 是一个抽象类。

```java
public abstract class Window {
    public static final int DECOR_CAPTION_SHADE_AUTO = 0;
    public static final int DECOR_CAPTION_SHADE_DARK = 2;
    public static final int DECOR_CAPTION_SHADE_LIGHT = 1;
    /** @deprecated */
    @Deprecated
    protected static final int DEFAULT_FEATURES = 65;
    public static final int FEATURE_ACTION_BAR = 8;
    //...
    }
```

Window 的具体实现类是 PhoneWindow.那 PhoneWindow 又是通过什么、什么时候创建的呢，毕竟我们没有在开发中直接创建过 PhoneWindow。根据上面的结论，activity 也是依赖一个 window 的，我们以 activity 为例，来看一看到底这个 Window 是什么时候创建的。（以下会涉及到一些 AMS 的知识，可能会比较难以下咽(;´༎ຶД༎ຶ`)）。

### activity 中 window 的创建

这里会涉及一点 Activity 的启动流程。。。因为我们要知道 acitvity 的创建过程。

我们知道应用程序的入口是 ActivityThread，activity 的创建也是也是交给 ActivityThread 实现的。这里不啰嗦，反正就是 ActivityThread 和 AMS（ActicityManagerService）的频繁 IPC 交互。最终会在 ActivityThread 的 performLaunchActivity()方法中进行 activity 的加载。

```java
    java.lang.ClassLoader cl = r.packageInfo.getClassLoader();
    //Instrumentation的作用是监听应用程序和系统化交互的
    activity = mInstrumentation.newActivity(cl, component.getClassName(), r.intent);
    if (acticity != null) {
    //context的创建，其实activity本质上也是一个context
     Context appContext = createBaseContextForActivity(r, activity);
     //...
     activity.attach(appContext, this, getInstrumentation(), r.token, r.ident,
     app, r.intent, 此处省略一堆参数)；

    }
```

关键就在这个 attach 方法，这个方法会传入一堆参数，将 activity 关联其运行时所依赖的上下文环境变量。Window 对象就是在这个方法里面创建的，并且 activity 实现了 window 的 Callback 接口，所以党 window 的状态发生改变会回调 activity 的相应方法。

```java
    //window对象的创建
    mWindow = PolicyManager.makeNewWindow(this);
    //window的一些配置
    mWindow.setCallback(this);
    mWindow.setOnWindowDismissedCallback(this);
    //....
```

可以看到 Window 的创建是通过 PolicyManager 来实现的。那我们就来看看 PolicyManager 是一个神马东西。

```java
public final class PolicyManager{
    //Policy实现类
    private static final String POLICY_IMPL_CLASS_NAME = "com.android.internal.policy.impl,Policy";
    private static final IPolicy sPolicy;

    static {
        try{
            Class policyClass = Class.forName(POLICY_IMPL_CALSS_NAME);
            sPolicy = (IPolicy)policyClass.new Instance();
        }
        catch{
            //...
        }
    }

    private PolicyManager(){}

    //这里就是创建window的地方
    public static Window makeNewWindow(Context context){
        return sPolicy.makeNewWindow(context);
    }

    public static LayoutInflater makeNewLayoutInflater(Context context){
        return sPolicy.makeNewLayoutInflater(context);
    }


}
```

从上面的代码可以看出 Policy 也只是一个代理类，但是他的一个巧妙的地方就是通过反射来构造 Policy 对象，实现了对外隐藏实现。没办法，还要再来看一看 Policy 的代码。

```java
Pulic class Policy implements IPolicy {
    //...
    //哇！终于等到你。。。
    public Window makeNewWindow(Context context){
        return new PhoneWindow(context);
    }

    //这里还有一个重要的对象的创建,LayoutInflater对象的实例化所在
    public LayoutInflater makeNewLayoutInflater(Context context){
        return new PhoneLayoutInflater(context);
    }
}
```

终于等到你，终于等到你，终于等到你。到这里 window 对象就就已经创建好了（到这里也只是创建好了 window 对象。接下来我们还要看看 window 是怎么工作的）。

## window 的工作原理

在这里先说明一下，window 是一个比较抽象的东西，每一个 window 都对应着一个 View 和一个 ViewRootImpl(这个也可以说是 view 工作的入口)，ViewRootImpl 将 Window 和 View 联系起来。

### WindowManager

书上说 WindowManager 是 Window 外界访问入口。我觉得这样说是不合适的，其实我们可以很方便的获得 view 的 window 对象，我觉得 WindowManager 主要是用来和 WindowManagerService 交互的。

WindowManager 提供了三个操作 Window 的方法，`addView`,`updateViewLayout`,`removeView`。从方法名就可以看出来对 Window 操作其实就是对 View 的操作。

WindowManager 是一个接口，他的实现类是 WindowManagerImpl，这个类又是在哪实现的呢？

其实 WindowManager 也是 ContextImpl 中注册点服务之一。代码如下:

```java
registerService(WINDOW_SERVICE, new ServiceFetcher(){
    Display mDefaultDisplay;
    public Object getService(ContextImpl ctx){
        Display display = ctx.mDisplay;
        if(display == null){
            if(mDefaultDisplay == null){
                DisplayManager dm = (DisplayManager)ctx.getOuterContext().getSystemService(Context.DISPLAY_SERVICE);
                mDefaultDisplay = dm.getDisplay(Dispaly.DEFAULT_DISPLAY);
            }
            display = mDefaultDisplayl;
        }
        return new WindowManagerImpl(disolay);
    }
});
```

下面又是一些 WindowManager 操作的源码分析了，这里就不重新造轮子了，直接按照《人体艺术探索》上的思路和源码来分析( ^ \_ ^ )v

这里就说一个 addView 吧，其他的可以去找资料，网上很多都是直接抄的《人体艺术探索》的。

### Window 添加过程

window 的添加就是通过 WindowManager 的`addView`方法,这里又是一样的套路，WindowManager 只是一个接口，他需要一个实现类，那就是 WindowManager+Impl(~ \_ ~;)，下面看看 WindowManagerImpl 的这三个方法的实现。

```java
@Override
public void addView(View view, ViewGroup.Layoutparams params) {
    mGlobal.addView(view, params, mDisplay, mParentWindow);
}

@Override
public void updateViewLayout(View view, ViewGroup.Layoutparams params) {
    mGlobal.updateViewLayout(view, params);
}

@Override
public void removeView(View view) {
    mGlobal.removeView(view);
}
```

可以看到 WindowManagerImpl 也没有真正实现 window 的操作，而是交给 WindowManagerGlobal 来处理，由此就可以看出 WindowManagerImpl 就是一个代理类(书上说这是桥接模式，我一直没有研究过侨界模式，但就是看着像代理模式）。所以说真正的大头还是 WindowManagerGlobal。

WindowManagerGlobal 以工厂形式对外提供自己的实例。

```java
private final WindowManagerGlobal mGlobal = WindowManagerGlobal.getInstance();
```

WindowManagerGlobal 中 addView 方法分为下面几个步骤

-   检查参数是否合法，如果是子 Window 那么还需要调整一些布局参数
-   创建 ViewRootImpl 并将 View 添加到列表中。
    ```java
    private final ArrayList<View> mViews = new ArrayList<View>();

        private final ArrayList<ViewRootImpl> mRoots = new ArrayList<ViewRootImpl>();

        private final ArrayList<WindowManager.Layoutparams> mParams = new ArrayList<WindowManager.Layoutparams>();

        private final ArraySet<View> mDyingViews = new ArraySet<View>();
        ```
        上面mViews存储的是所有Window对应的View；我们上面说过，每一个Window都对应一个View ，并且每个Window都有一个ViewRootImpl对象，就是这边的mRoots,ViewRootImpl继承了Handler类，是native与java层的View系统通信的桥梁。

        ```java

        public ViewRootImpl(Context context, Display display){
            mContext = context;
        //获取windowSession，与WMS建立连接
        mWindowSession = WindowManagerGlobal.getWindowSeesion();
        //保存当前线程，更新UI只能在创建ViewRootImpl的线程中执行
        //创建ViewRootImpl的线程就是UI线程，也就是主线程。
        mThread = Thread.currentThread();

        }

        ```

    而且也说过 ViewRootImpl 会将 Window 和 View 关联，是怎样关联的呢，关联的一方 面我们可以看看下面代码
    ```
    //这也算是关联的一方面
    root = new ViewRootImpl(view.getContext(), display);
    view.setLayoutParams(wParams);

        mView.add(view);
        mRoots.add(root);
        mParams.add(wParams);
        ```

-   通过 ViewRootImpl 来更新界面并完成 Window 的添加

    这个步骤由 ViewRootImpl 的 setView 方法完成(这个方法完成了关联)。

    ```java
    root.setView(view, wparams, panelParentView());

    public void setView(View view, WindowManager.LayoutParams attrs, View panelParentView){
        synchronized(this){
           requestLayout();
           try{
             res = sWindowSession.add(mWindow, mWindowAttributes,getHostVisibility(), mAttachInfo.mContentInsets);
           }
        }
    }
    ```

    都知道 View 的绘制过程是由 ViewRootImpl 发起的。setView 内部会通过 requestLayout 来完成异步刷新。下面的代码中，scheduleTraversals 实际上是 View 绘制的入口。ScheduleTraversals 最终会调用 ViewRootImpl 的 performTraverslas 方法。

    ```java

    public void requestLayout() {
        if (!mHandingLayoutInLayoutRequest) {
            checkThread ();
            mLayoutRequested = true;
            ScheduleTraversals();
        }
    }

    public void ScheduleTraversals(){
       if(!mTraversalScheduled){
          mTraversalScheduled = true;
          sendEmptyMessage(DO_TRAVERSAL);
       }
    }
    ```

    `sendEmptyMessage(DO_TRAVERSAL);`这个函数向 Handler 发送一个消息,出发整个视图树的绘制，也就是最终执行 performTraversals 函数。

    ```java

    private void performTraversals(){
     //获取Surface对象，用于图形绘制
     //三大过程

    }

    ```

    这里就不分析 performTraversals 方法了，因为这就属于 View 绘制范畴了，不在本文的讨论范围之内。可以看我之前写的这篇文章[view 工作原理](https://henryhaoson.github.io/2017/06/15/2017-6-5-View%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86/)。

    接着会通过 WindowSession 最终完成 Window 的添加过程。在下面代码中，mWindowSession 的类型是 IWindowSession，这是一个 Binder 对象，实现类是 Session。这就说明这是一个 IPC 过程，也就是离 WMS 不远了。

    ```java

    try {
        mOrigWindowType = mWindowAttributes.type;
        mAttachInfo.mRecomputeClobalAttributes =true;
        collectViewAttributes();
        res = mWindowSession.addToDisplay(mWindow, mSeq, mWindowAttributes,
              getHostVisibility(), mDisplay.getDisplayId(),
              mAttachinfo.mContentInsets, mInputChannel);
    } catch {
        mAdded = false;
        mView = null;
        //...对异常的处理
    }

    ```

    在 Session 内部，会将添加操作最终交给 WMS（WindowManagerService）。

    ```java

    public int addToDisplay(IWindow window,...省略若干参数) {
        return mService.addWindow(this, window, seq, attrs,
        viewVisibility, displayId, outContentInsets, outInputChannel);
    }

    ```

    终于看到了神秘的 WMS(就是上面的 mService)。在这先不分析 WMS 的内部操作了(还没看(; ^ \_ ^A )。

这里看完了 Window 添加到 WMS 中的过程，再回到 Activity

### activity 中 window 的工作

上面已经分析了 acticity 的 window 对象创建的过程，那么 window 是证明工作的呢，又是在何时绑定的呢（其实就是找 WindowManager 何时执行 addView 方法）。

我们从 activity 的生命周期角度来分析。

我们知道在 activity 的 onCreate 方法里会执行 setContentView 方法，这个方法最终也是由 window 来执行的。

```java
 @Override
public void setContentView(@LayoutRes int layoutResID) {
    getDelegate().setContentView(layoutResID);
}
```

在这你可能会发现，不对啊，说好的 window 呢，getDelegate()是个什么鬼。其实我这个看的是 AppCompatActivity 的源码，如果是 acticity 的远源码，那就是 getWIndow(),呢我们就来看一看这个 getDelegate()说到底是啥。

```java
    public AppCompatDelegate getDelegate() {
        if (mDelegate == null) {
            mDelegate = AppCompatDelegate.create(this, this);
        }
        return mDelegate;
    }
```

发现这是获取一个类型为 AppCompatDelegate 的 mDelegate 对象。我们再来看一看他的 create 方法

```java
    /**
     * Create a {@link android.support.v7.app.AppCompatDelegate} to use with {@code activity}.
     *
     * @param callback An optional callback for AppCompat specific events
     */
    public static AppCompatDelegate create(Activity activity, AppCompatCallback callback) {
        return create(activity, activity.getWindow(), callback);
    }

    /**
     * Create a {@link android.support.v7.app.AppCompatDelegate} to use with {@code dialog}.
     *
     * @param callback An optional callback for AppCompat specific events
     */
    public static AppCompatDelegate create(Dialog dialog, AppCompatCallback callback) {
        return create(dialog.getContext(), dialog.getWindow(), callback);
    }

    private static AppCompatDelegate create(Context context, Window window,
            AppCompatCallback callback) {
        final int sdk = Build.VERSION.SDK_INT;
        if (BuildCompat.isAtLeastN()) {
            return new AppCompatDelegateImplN(context, window, callback);
        } else if (sdk >= 23) {
            return new AppCompatDelegateImplV23(context, window, callback);
        } else if (sdk >= 14) {
            return new AppCompatDelegateImplV14(context, window, callback);
        } else if (sdk >= 11) {
            return new AppCompatDelegateImplV11(context, window, callback);
        } else {
            return new AppCompatDelegateImplV9(context, window, callback);
        }
    }
```

可以看到它这边其实是传了 window 对象的。其实最终的 setContent 方法还是交给 Window 的。

我们来看 Window(实际上是 PhoneWindow)的 setContentView 方法。

window 的 setContentView 其实可以分为 3 个步骤

-   1.木有 DecorView 就创建。

    DecorView 是 Activity 的顶级 View，关于 DecorView 的知识可以自己去查。DecorView 的创建实在 installDecor 方法中实现的，在 DecorView 中会调用 generateDecor 方法。

```java
protected DecorView generateDecor() {
    return new DecorView(getContext, -1);
}
```

-   2.这样实例化之后还只是一个空白的 View，之后会通过 LayoutInflater 来将布局文件中的布局添加到 DecorView 的 mContentParent 中。
    ```java
    mLayoutInflater.inflate(layoutResID, mContentParent);
    ```
-   3.回调 activity 的 onContentChanged 方法通知 activity 视图已经发生改变。
    这个方法其实是空实现，可以自己定义处理策略。

经过上面的过程，setContentView 实现的 DecorView 的创建。但是还没有分析到 window 是如何添加到 WMS 中的,只有执行 WindowManager 的 addView 方法，window 才能真正的工作。

在 ActivityThread 的 handleResumeActivity 方法中，会调用 activity 的 onResume 方法，接着会调用 activity 的 makeVisible 方法。在这个方法里会实现 addView 操作。

```java
void makeVisible () {
    if (!mWindowAdded){
        //获取WM对象
        ViewManager wm =getWindowManager();
        //执行addView方法。添加window的入口
        wm.addView(mDecor, getWindow().getAttributes());
        mWindowAdded = true;
    }
    mDecorr.setVisibility(View,VISIBLE);
}
```

这样 activiy 的 window 就成功添加了。
