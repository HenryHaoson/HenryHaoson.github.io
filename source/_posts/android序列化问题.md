---
layout: post
title: android序列化
date: 2017-7-5
categories: blog
tags: [android]
description: android序列化问题，Serializable和Parcelable接口的对比

---
## 序列化

序列化是将java对象转换成二进制字节序列，在android内存中传输数据时会经常用到序列化。比如说用bundle传输数据，bundle在android组建通信中用的特别多，两种序列化操作都支持，Serializable和Parcelbale。

序列化有三种应用场景
* 数据持久化，将对象的字节流保存在本地
* 在网络传输中传递对象
* IPC中传输对象

## Serializable接口
Serializable接口是java自带的序列化接口，具体的序列化与反序列化操作是由ObjectOutputStream和ObjectInputStream完成的，然而android已经对这两个操作进行了封装。

### 序列化ID
在使用Serializable接口的时候，还需要注意一点，就是要标注序列化ID。下面说一下序列化ID的作用。

虚拟机是否允许反序列化，不仅取决于类路径和功能代码是否一致，一个非常重要的一点是两个类的序列化 ID 是否一致（一般是 `private static final long serialVersionUID = 1L`）。

举个例子，客户端A将对象C序列化为二进制字节流传输给客户端B，B接收到字节流并反序列化。 如果这时候在客户端A和客户端B中类C的序列化ID不一样，那么接收端一方将无法反序列化。

现在再想想，为什么要这样设计呢，搞不一样的ID这不是给自己找麻烦吗，那就来想想不一样的ID有什么应用场景。

下面是摘自网上的一个用例
![image](https://www.ibm.com/developerworks/cn/java/j-lo-serial/image003.gif)
Client 端通过 Façade Object 才可以与业务逻辑对象进行交互。而客户端的 Façade Object 不能直接由 Client 生成，而是需要 Server 端生成，然后序列化后通过网络将二进制对象数据传给 Client，Client 负责反序列化得到 Façade 对象。该模式可以使得 Client 端程序的使用需要服务器端的许可，同时 Client 端和服务器端的 Façade Object 类需要保持一致。当服务器端想要进行版本更新时，只要将服务器端的 Façade Object 类的序列化 ID 再次生成，当 Client 端反序列化 Façade Object 就会失败，也就是强制 Client 端从服务器端获取最新程序。

### 还有几点要注意的
* 静态变量不能序列化
* Transient标注的变量不会被序列化
* 父类不会被序列化
* 序列化存储时不会存储相同的对象，会存储以存储对戏那个的索引

## Parcelable接口
Parcelable接口是用来解决Serializable接口效率问题的（Serializable会产生大量的临时变量，引起频繁的GC）。Parcelable的使用场景也是因此而定的，序列化对象在网络中或者组建间，或者进程间传输的时候适合用Parcelable接口。

Parelable接口也有一个缺点，那就是不能使用在要将数据存储在磁盘上的情况(如：永久性保存对象，保存对象的字节序列到本地文件中)，因为Parcel本质上为了更好的实现对象在IPC间传递，并不是一个通用的序列化机制，当改变任何Parcel中数据的底层实现都可能导致之前的数据不可读取，所以此时还是建议使用Serializable 。


## 使用场景
引发我对序列化问题学习的原因是当我使用fragment的时候，使用newInstance方法实例化fragment（使用newInstance()方法的好处可以自行百度），需要对参数用bundle进行保存。

![image](http://upload-images.jianshu.io/upload_images/3351492-8e2372422a2eed2c.png)

![image](http://upload-images.jianshu.io/upload_images/3351492-0b4e740cb4d292ef.png)

可以看到bundle支持的是序列化对象，String类型也是继承Serializable接口的。

![image](http://upload-images.jianshu.io/upload_images/3351492-43f5068a57dd90c3.png)
在newInstance()方法里面进行保存以后，在onCreate方法中读取保存的数据。

```
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        if (getArguments() != null) {
            list = (ArrayList<DailyWordBean.DataBean>) getArguments().getSerializable("data");
            Log.e("study", list.size() + list.get(0).getWordContent());
        }
    }
```
这里使用的是Serializable接口，得说明一下，尽量使用Parcelable接口，速度比Serializable接口快了10倍左右。
