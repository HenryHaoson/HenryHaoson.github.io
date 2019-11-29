---

layout: post
title: singleton
date: 2017-12-5
categories: blog
tags: [设计模式]
description: 设计模式学习第一篇，最常见的最常用的单例模式

---

## 单例模式介绍

> 一个由单例模式引出了技术链：java内存模型，乱序执行

单例模式是真的很常用。java和安卓里面都有大量的实例，比如说Math工具类，Collections，LayoutInflater等等。从这里就可以看出单例模式的作用，那就是作为服务整个程序的工具类。

那什么样的服务需要使用单例模式呢。那就是整个程序只能有一个存在的实例。只存在一个那就说明这个实例是消耗资源的，比如说使用线程池，访问网络或者数据库文件操作等。

## 单例模式实现的关键点

- 构造函数一般设为private

- 通过一个静态方法或者是枚举来返回实例

- 确保实例只有一个，尤其在多线程的情况下

- 确保在反序列化的时候不会重新构造对象。

## 实现单例模式的几种方式

### 懒汉式

```java

public class Singleton {
	private static Singleton instance;
	//构造方法私有化
	private static Singleton () {}

	public static sychronized Singleton getInstance(){
		if(instance == null){
			instance = new Singleton();
		}
		return instance;
	}
}

```

优点：使用的时候才会被实例化

缺点：每次调用的时候都会执行getInstance方法，而这个方法使用了sychronized关键字，所以每次调用都会消耗不必要的资源。

### DCL(Double Check Lock)

也叫做双重锁检测

```java

public class Singleton {
	
	private Singleton instance;
	
	private Singleton () {}

	public static Singleton getInstance(){
		
		if(instance == null){
		sychronized (Singleton.class){
			if(instance == null){
				instance = new Singleton();
			}
		}	
		}
	}

}

```

这里对instance进行了两次判断，可以通过第一次判断来决定是否要进行同步。

优点：比起懒汉式，避免了同步的资源浪费。

缺点：由于java的内存模型原因，有可能会失效。俗称DCL失效。

> DCL失效

语句执行的过程:instance  = new Singleton();

- 给instance分配内存。
- 初始化这个类，调用Singleton的构造函数。
- 将instance指向分配的内存空间。

由于编译器的原因，没有办法保证2，3两步的执行顺序。所以当第三步先于第二步执行的话，此时另一个线程使用了instance对象，然而这个对象还没初始化，就会出错。

在JDK1.5后，可以用volatile来避免这个问题，instance的声明方式改成`private volatile static Singleton instance = null;` ，这样保证了instance对象每次都是从主存中读取的。但是volatile在性能上有一定的消耗。性能换取正确率。

### 静态内部类

```java

	public class Singleton {

		private Singleton();
		
		public static Singleton getInstance(){
			
			return SingletonHolder.instance;

		}
		
		private static class SingletonHolder {

			private static final Singleton instance = new Singleton();

		}
	}

```

//java静态代码块是线程安全的。使用内部类是用的时候才会实例化。（知识点，类加载机制）
优点：保证了线程安全，对象唯一，延迟了单例的实例化。所以推荐使用。

### 枚举

```java

	public enum Singleton {

	INSTANCE;
	public void doSomething(){
		//...
	}
	
	}

```

枚举的实例创建默认是线程安全的，而且是在*任何情况下都是单例*，在其他几种单例中，在反序列化的时候都会重新创建新的对象，而枚举并不会。

而其他单例想要避免这个问题可以重写反序列化提供的一个钩子函数，readResolve();

```java

private Object readResolve() throws ObjectStreamExcetion {

	return instance;

}

```
