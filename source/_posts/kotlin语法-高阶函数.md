---
layout: post
title: kotlin语法-高阶函数
date: 2021-04-21
categories: blog
tags: [kotlin, 编程语言]
description: kotlin 高阶函数
---

## 背景
之前在项目里看到一种回调写法，相比于 java 看上去十分简洁

```java

view_wordlist_controller.setListener(new Listener {
    @override
    public void onFilterClicked(T it) {
        showFilterPop(it)
    }

    @override
    public void onCalendarClicked() {
        mListener.onCalendarClicked()
    }
})

```

```kotlin
view_wordlist_controller.setListener {
    onFilterClicked {
        showFilterPop(it)
    }

    onCalendarClicked {
        mListener.onCalendarClicked()
    }
}

```

这样会少很多模版代码，看上去十分清晰，也没有丢失可读性。

那么kotlin是如何实现这种写法的呢？ 其实就是这次要研究的 kotlin 的高阶函数(和 lambda 表达式)。

先直接贴实现，接下来一步一步分析。

```kotlin
    private var mListener: ListenerBuilder? = null

    fun setListener(listenerBuilder: ListenerBuilder.() -> Unit) {
        mListener = ListenerBuilder().also(listenerBuilder)
    }

    inner class ListenerBuilder {
        internal var mOnFilterClickedAction: ((View) -> Unit)? = null

        internal var mOnCalendarClickedAction: (() -> Unit)? = null

        fun onFilterClicked(action: (View) -> Unit) {
            mOnFilterClickedAction = action
        }

        fun onCalendarClicked(action: () -> Unit) {
            mOnCalendarClickedAction = action
        }
    }
```

## 定义
维基百科关于高阶函数的定义：
> 在函数式编程中，折叠（fold），也称为归约（reduce）、积累（accumulate）、聚集（aggregate）、压缩（compress）或注入（inject），指称一组高阶函数，它们分析递归数据结构并通过使用给定组合运算，将递归的处理它的构成部件、建造一个返回值的结果重组起来。典型的，要向折叠提供一个组合函数，一个数据结构的顶端节点，和可能的在特定条件下使用的某些缺省值。折叠接着以系统性方式使用这个函数，进行组合这个数据结构的层级中的元素。折叠在某种意义上是展开（unfold）的对偶，它接受一个种子值并共递归的应用一个函数，来确定如何进一步的构造一个共递归的数据结构。折叠递归的分解这个数据结构，在每个节点应用一个组合函数于它的终结值和递归结果之上，用得到这个结果替代它。折叠是catamorphism，而展开是anamorphism。


kotlin 官网是这样定义高阶函数的：
> 高阶函数是将函数用作参数或返回值的函数。

像上边的 `setListener` 、 `onFilterClicked` 、 `onCalendarClicked` 都是高阶函数。

## 实现

除了告诫函数，实现上边的写法还要两个特性

### 1.lambda

关于lambda，在 Kotlin 中有一个约定：如果函数的最后一个参数是函数，那么作为相应参数传入的 lambda 表达式可以放在圆括号之外：

```kotlin
fun foo((Int,Boolean) -> Unit){
    ...
}

// 可以这样调
foo({num, isHide->{
        //
    }
}

// 也可以这样
foo(){num, isHide->{
        //
    }
}

```

### 2.带有接收者的函数字面值
官方定义：
> 带有接收者的函数类型，例如 A.(B) -> C，可以用特殊形式的函数字面值实例化—— 带有接收者的函数字面值。如上所述，Kotlin 提供了调用带有接收者（提供接收者对象）的函数类型实例的能力。
> 在这样的函数字面值内部，传给调用的接收者对象成为隐式的this，以便访问接收者对象的成员而无需任何额外的限定符，亦可使用 this 表达式 访问接收者对象。
>这种行为与扩展函数类似，扩展函数也允许在函数体内部访问接收者对象的成员。

就像上述的 `setListener` 方法，入参是 `ListenerBuilder.() -> Unit`,他就是带有接收者的，接收者是ListenerBuilder的实例。


结合lambda表达式，就可以实现上边的写法。

不用lambda表达式就是这样（this可以省略）

```kotlin
view_wordlist_controller.setListener({
    this.onFilterClicked({it -> {
        showFilterPop(it)
    }})

    this.onCalendarClicked({
        mListener.onCalendarClicked()
    })
})
```
