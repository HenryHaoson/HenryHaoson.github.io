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

那么kotlin是如何实现这种写法的呢？ 其实就是这次要研究的 kotlin 的高阶函数。

先直接贴实现

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

这里涉及到了 kotlin 的高阶函数

## 定义
维基百科关于高阶函数的定义：
> 在函数式编程中，折叠（fold），也称为归约（reduce）、积累（accumulate）、聚集（aggregate）、压缩（compress）或注入（inject），指称一组高阶函数，它们分析递归数据结构并通过使用给定组合运算，将递归的处理它的构成部件、建造一个返回值的结果重组起来。典型的，要向折叠提供一个组合函数，一个数据结构的顶端节点，和可能的在特定条件下使用的某些缺省值。折叠接着以系统性方式使用这个函数，进行组合这个数据结构的层级中的元素。折叠在某种意义上是展开（unfold）的对偶，它接受一个种子值并共递归的应用一个函数，来确定如何进一步的构造一个共递归的数据结构。折叠递归的分解这个数据结构，在每个节点应用一个组合函数于它的终结值和递归结果之上，用得到这个结果替代它。折叠是catamorphism，而展开是anamorphism。



