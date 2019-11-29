---
layout: post
title: AsyncTask工作原理
date: 2017-7-15
categories: blog
tags: [android]
description: AsyncTask工作原理

---
AsyncTask使用限制
* 1.AsyncTask的对象必须在主线程中创建
* 2.execute方法必须在主线程中调用
* 3.不要再程序中直接调用onPreExecute(),onPostExecute(),doInBackground(),onProgressUpdate()方法。
* 4.一个AsyncTask对象只能执行一次。

## AsyncTask工作原理
首先执行暴露的方法execute(),

```
public final AsyncTask<Params , Progress , Result> execute(Params ...params){
    return executeOnExecutor(sDefaultExecutor, params);
}
```
会调用executeOnExecutor()方法，sDefaultExecutor是一个串型的线程池，一个进程所有的AsyncTask全部在这个串型的线程池中排队执行。
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

首先执行onPreExecute()方法，然后执行线程池。

线程池的执行过程：
> SerialExecutor是用于任务队列的排序，THREAD_POOL_EXECUTOR用于任务的执行

### SerialExecutor的工作原理

在SerialExecutor的execute方法执行之前，系统首先会吧AsyncTask的Params参数封装成FutureTask对象，这是一个并发类，在这里充当Runnable的作用，接着这个参数会传给SerialExecutor的execute方法去处理。首先会判断当前有没有正在活动的任务，如果没有就调用SerialExecutor的scheduleNext方法执行下一个AsyncTask任务。执行完会继续调用scheduleNext方法，知道任务全部执行完，所以这是一个串型的执行方式。

上面说到会把AsyncTask的参数封装成FutureTask的类型，在FutureTask执行run方法的时候会调用mWorker的call方法。所以说mWorker的call方法是在线程池中执行的。mWorker会在AsyncTask的构造方法中实例化。

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
在call方法中首先将mTaskInvoked设置成true，表示该任务已经调用过了，然后执行doInBackground方法，接着将返回值传给postResult。

postResult会将结果通过InternalHandler传给主线程。handler接收消息的时候灰调用AsyncTask的finish方法。finish方法中，任务如果被取消，会调用onCancelled方法，否则调用onPostExecute方法，这样整个流程就结束了。
