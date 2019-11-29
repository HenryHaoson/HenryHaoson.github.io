---
layout: post
title: 静态构造器的问题
date: 2017-7-28
categories: blog
tags: [android java]
description: 静态构造器的问题，结合fragment的使用场景。

---
 不要让客户端去调用默认的构造函数，然后手动地设置fragment的参数。我们直接为它们提供一个静态工厂方法。这样做比调用默认构造方法好，有两个原因：一个是，它方便别人的调用。另一个是，保证了fragment的构建过程不会出错。通过提供一个静态工厂方法，我们避免了自己犯错--我们再也不用担心不小心忘记初始化fragmnet的参数或者没正确设置参数。
 
总的来说，虽然两者的区别只在于设计，但是他们之间的差别非常大。因为提供静态工厂方法有向上抽象了一个级别，让代码更容易懂。

注：其实提供静态工厂而不是使用默认构造函数或者自己定义一个有参的构造函数还有至关重要一点。fragmnet经常会被销毁重新实例化，Android framework只会调用fragment无参的构造函数。在系统自动实例化fragment的过程中，你没有办法干预。一些需要外部传入的参数来决定的初始化就没有办法完成。使用静态工厂方法，将外部传入的参数可以通过Fragment.setArgument保存在它自己身上，这样我们可以在Fragment.onCreate(...)调用的时候将这些参数取出来。

注意：fragmentnew出来不会立即执行生命周期，通过FM调用后会执行生命周期。

> 更新

后来发现这是effective java 中的一则思想，用静态构造方法来代替原有对公共构造方法。下面来copy网上总结的这种静态构造方法对优势：
* 1.**静态工厂方法在方法命名上更具有可读性（这条看看就行**）

    在使用构造函数去构造对象的时候，我们传递不同的参数构造不同类型的对象。如果不看文档的话，我们很难记住传递什么参数能够构造什么样子的对象。用静态的工厂方法就不一样啦，我们可以不同的工厂方法不同名字，我们就可以很容易的记住什么方法名可以构造什么样的对象。关于这一点，在 JDK 中最有说服力的一个例子就是 java.util.concurrent.Executors，里面N多的静态工厂方法：newFixedThreadPool、newSingleThreadExecutor、newCachedThreadPool、newScheduledThreadPool等等
    
* 2.**静态工厂方法不需要每次在被调用的时候都构造一个新的对象（这一点还要继续做一下研究，看看是否JVM会对静态构造方法实现对对象进行缓存）**

    也就是说我们调用静态工厂方法返回的可能是缓存的一个对象，而不是一个新的对象。这可以减少创建新的对象，从来提高性能，前面提到的 Boolean 就是一个例子。这对于 immutable 的对象特别有用，读到这里，我翻看了一下 JDK 的源代码，才发现原来 JDK 中原始数据类型（Primitive Data Types）的包装类都是 immutable 的。也就是说只要创建 Boolean、Integer、Long 等的对象，它的值就是不能改变的。我们再看 Integer 提供的静态工厂函数：
    ```
    public static Integer valueOf(int i) {
    assert IntegerCache.high >= 127;
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
    }
    ```
    上面的代码把出现概率高的 int 做了一个 cache，这样每次只要返回 cache 里的对象，而不用新建一个对象。
    
* 3.**静态工厂方法还可以返回该类型的子类对象(水平尚浅，还没用到)**
    
    这个特性让静态工厂方法的可扩展性大大的优于构造函数。在 JDK 中最典型的应该是 java.util.EnumSet。EnumSet 本身是 absract，我们无法直接调用它的构造函数。不过我们可以调用它的静态方法 noneOf 来创建对象，RegularEnumSet/JumboEnumSet 都继承至 EnumSet，noneOf 根据参数返回合适的对象。

    ```
    public abstract class EnumSet<E extends Enum<E>> extends AbstractSet<E>
    implements Cloneable, java.io.Serializable {

    EnumSet(Class<E>elementType, Enum[] universe) {
    }

    public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
        if (universe.length <= 64)
            return new RegularEnumSet<>(elementType, universe);
        else
            return new JumboEnumSet<>(elementType, universe);
             }
    }
    ```
    
* 4.静态工厂方法还可以简化参数化类型的对象创建
这个优点优点语法糖的味道，不过语法糖人人都喜欢啦。

```
Map<String, List<String>> m =
    new HashMap<String, List<String>>();

public static <K, V> HashMap<K, V> newInstance() {
    return new HashMap<K, V>();
}
Map<String, List<String>> m = HashMap.newInstance();
```
第一行冗长的代码我们就可以简化成第三行的代码。比较典型的例子就是 guava 中的 Maps （Google collections library）


以上优点摘自：[这个人对静态构造器对看法（应该也是看了effective java之后的总结吧）](http://hellojinjie.com/2014/04/03/effective-java%EF%BC%9A%E4%BD%BF%E7%94%A8%E9%9D%99%E6%80%81%E5%B7%A5%E5%8E%82%E6%96%B9%E6%B3%95/)