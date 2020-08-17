---
layout: post
title: Android 消息机制
date: 2020-6-17
categories: blog
tags: [Android]
description: Android 消息机制研究摸索。
---

### 基本概念，使用方法介绍

基本使用方法：

```java

private void testHandler() {
		new Thread(new Runnable() {
			@Override
			public void run() {
				Log.d("handler", "333");
				Looper.prepare();

				Handler handler = new Handler(Looper.myLooper()){
					@Override
					public void handleMessage(Message msg) {
						super.handleMessage(msg);
						Log.d("handler", "444");
					}
				};
				handler.post(new Runnable() {
					@Override
					public void run() {
						Log.d("handler", "222");
					}
				});
				handler.dispatchMessage(new Message());
				Looper.loop();

			}
		}).start();
	}

```

这里需要注意一点，一定要在 sendMessage 之后再开始 loop();因为 loop 之后会阻塞线程，消息就发不出去了。 具体为什么会阻塞下面会分析。

### MessageQueue

#### enqueue 排序插入

#### next()

### Looper

### Handler

#### deprecated 的两个构造方法

1.`public Handler ()`

2.`public Handler(Handler.Callback callback)`

这两个构造方法不建议使用的原因如下

```
Implicitly choosing a Looper during Handler construction can lead to bugs where operations are silently lost (if the Handler is not expecting new tasks and quits), crashes (if a handler is sometimes created on a thread without a Looper active), or race conditions, where the thread a handler is associated with is not what the author anticipated. Instead, use an Executor or specify the Looper explicitly, using Looper#getMainLooper, {link android.view.View#getHandler}, or similar. If the implicit thread local behavior is required for compatibility, use new Handler(Looper.myLooper()) to make it clear to readers.

```
