---
layout: post
title: AsyncTask工作原理
date: 2017-7-15
categories: blog
tags: [Android]
description: AsyncTask工作原理
---

AsyncTask 使用限制

-   1.AsyncTask 的对象必须在主线程中创建
-   2.execute 方法必须在主线程中调用
-   3.不要再程序中直接调用 onPreExecute(),onPostExecute(),doInBackground(),onProgressUpdate()方法。
-   4.一个 AsyncTask 对象只能执行一次。

## AsyncTask 工作原理

首先执行暴露的方法 execute(),

```
public final AsyncTask<Params , Progress , Result> execute(Params ...params){
    return executeOnExecutor(sDefaultExecutor, params);
}
```

会调用 executeOnExecutor()方法，sDefaultExecutor 是一个串型的线程池，一个进程所有的 AsyncTask 全部在这个串型的线程池中排队执行。

```
public final AsyncETask<Params , Progress , Result> executeOnExecutor(Executor exec, Params ...params){
    if(mStatus != Status.PENDING){
        switch(mStatus){
            ...错误异常，主要是一个AsyncTask只能执行一次的错误
        }
    }
    mStatus = Status.RUNNING;
    onPreExecute();
    mWorker.mPararms =params;
    exec.execute(mFuture);
    return this;
}
```

首先执行 onPreExecute()方法，然后执行线程池。

线程池的执行过程：

> SerialExecutor 是用于任务队列的排序，THREAD_POOL_EXECUTOR 用于任务的执行

### SerialExecutor 的工作原理

在 SerialExecutor 的 execute 方法执行之前，系统首先会吧 AsyncTask 的 Params 参数封装成 FutureTask 对象，这是一个并发类，在这里充当 Runnable 的作用，接着这个参数会传给 SerialExecutor 的 execute 方法去处理。首先会判断当前有没有正在活动的任务，如果没有就调用 SerialExecutor 的 scheduleNext 方法执行下一个 AsyncTask 任务。执行完会继续调用 scheduleNext 方法，知道任务全部执行完，所以这是一个串型的执行方式。

上面说到会把 AsyncTask 的参数封装成 FutureTask 的类型，在 FutureTask 执行 run 方法的时候会调用 mWorker 的 call 方法。所以说 mWorker 的 call 方法是在线程池中执行的。mWorker 会在 AsyncTask 的构造方法中实例化。

```
mWorker = new WorkerRunnable<Params, Results>(){
    public Result call() throws Exception{
        mTaskInvoked.set(true);

        Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
        //noinspection unchecked
        return postResult(doInBackground(mParams));
    }
}
```

在 call 方法中首先将 mTaskInvoked 设置成 true，表示该任务已经调用过了，然后执行 doInBackground 方法，接着将返回值传给 postResult。

postResult 会将结果通过 InternalHandler 传给主线程。handler 接收消息的时候灰调用 AsyncTask 的 finish 方法。finish 方法中，任务如果被取消，会调用 onCancelled 方法，否则调用 onPostExecute 方法，这样整个流程就结束了。
