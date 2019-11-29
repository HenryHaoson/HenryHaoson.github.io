---
layout: post
title: WMS Window工作原理
date: 2017-8-23
categories: blog
tags: [android]
description: 最近在研究framework层的代码，之前听说过一些，神马AMS，PMS，WMS 啥啥啥的。所以就来看一看WMS（Window Manager Service）。

---
最近在研究framework层的代码，之前听说过一些，神马AMS，PMS，WMS 啥啥啥的。所以就来看一看WMS（Window Manager Service）。

## Window对象的创建

在android机制里面，view是视图呈现的载体，然而view也不能够单独存在，它也必须依赖window才能存在。

打开源码可以发现window是一个抽象类。
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
Window的具体实现类是PhoneWindow.那PhoneWindow又是通过什么、什么时候创建的呢，毕竟我们没有在开发中直接创建过PhoneWindow。根据上面的结论，activity也是依赖一个window的，我们以activity为例，来看一看到底这个Window是什么时候创建的。（以下会涉及到一些AMS的知识，可能会比较难以下咽(;´༎ຶД༎ຶ`)）。

### activity中window的创建

这里会涉及一点Activity的启动流程。。。因为我们要知道acitvity的创建过程。

我们知道应用程序的入口是ActivityThread，activity的创建也是也是交给ActivityThread实现的。这里不啰嗦，反正就是ActivityThread和AMS（ActicityManagerService）的频繁IPC交互。最终会在ActivityThread的performLaunchActivity()方法中进行activity的加载。

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
关键就在这个attach方法，这个方法会传入一堆参数，将activity关联其运行时所依赖的上下文环境变量。Window对象就是在这个方法里面创建的，并且activity实现了window的Callback接口，所以党window的状态发生改变会回调activity的相应方法。

```java
    //window对象的创建
    mWindow = PolicyManager.makeNewWindow(this);
    //window的一些配置
    mWindow.setCallback(this);
    mWindow.setOnWindowDismissedCallback(this);
    //....
```
可以看到Window的创建是通过PolicyManager来实现的。那我们就来看看PolicyManager是一个神马东西。
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
从上面的代码可以看出Policy也只是一个代理类，但是他的一个巧妙的地方就是通过反射来构造Policy对象，实现了对外隐藏实现。没办法，还要再来看一看Policy的代码。
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

终于等到你，终于等到你，终于等到你。到这里window对象就就已经创建好了（到这里也只是创建好了window对象。接下来我们还要看看window是怎么工作的）。

## window的工作原理
在这里先说明一下，window是一个比较抽象的东西，每一个window都对应着一个View和一个ViewRootImpl(这个也可以说是view工作的入口)，ViewRootImpl将Window和View联系起来。

### WindowManager
书上说WindowManager是Window外界访问入口。我觉得这样说是不合适的，其实我们可以很方便的获得view的window对象，我觉得WindowManager主要是用来和WindowManagerService交互的。

WindowManager提供了三个操作Window的方法，`addView`,`updateViewLayout`,`removeView`。从方法名就可以看出来对Window操作其实就是对View的操作。

WindowManager是一个接口，他的实现类是WindowManagerImpl，这个类又是在哪实现的呢？

其实WindowManager也是ContextImpl中注册点服务之一。代码如下:

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

下面又是一些WindowManager操作的源码分析了，这里就不重新造轮子了，直接按照《人体艺术探索》上的思路和源码来分析( ^ _ ^ )v

这里就说一个addView吧，其他的可以去找资料，网上很多都是直接抄的《人体艺术探索》的。

### Window添加过程

window的添加就是通过WindowManager的`addView`方法,这里又是一样的套路，WindowManager只是一个接口，他需要一个实现类，那就是WindowManager+Impl(~ _ ~;)，下面看看WindowManagerImpl的这三个方法的实现。

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

可以看到WindowManagerImpl也没有真正实现window的操作，而是交给WindowManagerGlobal来处理，由此就可以看出WindowManagerImpl就是一个代理类(书上说这是桥接模式，我一直没有研究过侨界模式，但就是看着像代理模式）。所以说真正的大头还是WindowManagerGlobal。

WindowManagerGlobal以工厂形式对外提供自己的实例。

```java
private final WindowManagerGlobal mGlobal = WindowManagerGlobal.getInstance();
```

WindowManagerGlobal中addView方法分为下面几个步骤

- 检查参数是否合法，如果是子Window那么还需要调整一些布局参数
- 创建ViewRootImpl并将View添加到列表中。
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
而且也说过ViewRootImpl会将Window和View关联，是怎样关联的呢，关联的一方 面我们可以看看下面代码
    ```
    //这也算是关联的一方面
    root = new ViewRootImpl(view.getContext(), display);
    view.setLayoutParams(wParams);
    
    mView.add(view);
    mRoots.add(root);
    mParams.add(wParams);
    ```
    
- 通过ViewRootImpl来更新界面并完成Window的添加

    这个步骤由ViewRootImpl的setView方法完成(这个方法完成了关联)。
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
    都知道View的绘制过程是由ViewRootImpl发起的。setView内部会通过requestLayout来完成异步刷新。下面的代码中，scheduleTraversals实际上是View绘制的入口。ScheduleTraversals最终会调用ViewRootImpl的performTraverslas方法。
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

    `sendEmptyMessage(DO_TRAVERSAL);`这个函数向Handler发送一个消息,出发整个视图树的绘制，也就是最终执行performTraversals函数。
    
    ```java

    private void performTraversals(){
     //获取Surface对象，用于图形绘制
     //三大过程
  
    }

    ```

    这里就不分析performTraversals方法了，因为这就属于View绘制范畴了，不在本文的讨论范围之内。可以看我之前写的这篇文章[view工作原理](https://henryhaoson.github.io/2017/06/15/2017-6-5-View%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86/)。
    
    接着会通过 WindowSession 最终完成 Window 的添加过程。在下面代码中，mWindowSession的类型是IWindowSession，这是一个Binder对象，实现类是Session。这就说明这是一个IPC过程，也就是离WMS不远了。
    
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
    
    在Session内部，会将添加操作最终交给WMS（WindowManagerService）。

    ```java

    public int addToDisplay(IWindow window,...省略若干参数) {
        return mService.addWindow(this, window, seq, attrs, 
        viewVisibility, displayId, outContentInsets, outInputChannel);
    }

    ```

    终于看到了神秘的WMS(就是上面的mService)。在这先不分析WMS的内部操作了(还没看(; ^ _ ^A  )。
    

这里看完了Window添加到WMS中的过程，再回到Activity

### activity中window的工作
上面已经分析了acticity的window对象创建的过程，那么window是证明工作的呢，又是在何时绑定的呢（其实就是找WindowManager何时执行addView方法）。

我们从activity的生命周期角度来分析。

我们知道在activity的onCreate方法里会执行setContentView方法，这个方法最终也是由window来执行的。
```java
 @Override
public void setContentView(@LayoutRes int layoutResID) {
    getDelegate().setContentView(layoutResID);
}
```
在这你可能会发现，不对啊，说好的window呢，getDelegate()是个什么鬼。其实我这个看的是AppCompatActivity的源码，如果是acticity的远源码，那就是getWIndow(),呢我们就来看一看这个getDelegate()说到底是啥。

```java
    public AppCompatDelegate getDelegate() {
        if (mDelegate == null) {
            mDelegate = AppCompatDelegate.create(this, this);
        }
        return mDelegate;
    }
```
发现这是获取一个类型为AppCompatDelegate的mDelegate对象。我们再来看一看他的create方法

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
可以看到它这边其实是传了window对象的。其实最终的setContent方法还是交给Window的。

我们来看Window(实际上是PhoneWindow)的setContentView方法。

window的setContentView其实可以分为3个步骤

- 1.木有DecorView就创建。
    
    DecorView是Activity的顶级View，关于DecorView的知识可以自己去查。DecorView的创建实在installDecor方法中实现的，在DecorView中会调用generateDecor方法。

```java
protected DecorView generateDecor() {
    return new DecorView(getContext, -1);
}
```
- 2.这样实例化之后还只是一个空白的View，之后会通过LayoutInflater来将布局文件中的布局添加到DecorView的mContentParent中。
    ```java
    mLayoutInflater.inflate(layoutResID, mContentParent);
    ```
- 3.回调activity的onContentChanged方法通知activity视图已经发生改变。
    这个方法其实是空实现，可以自己定义处理策略。
    
经过上面的过程，setContentView实现的DecorView的创建。但是还没有分析到window是如何添加到WMS中的,只有执行WindowManager的addView方法，window才能真正的工作。

在ActivityThread的handleResumeActivity方法中，会调用activity的onResume方法，接着会调用activity的makeVisible方法。在这个方法里会实现addView操作。
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

这样activiy的window就成功添加了。









