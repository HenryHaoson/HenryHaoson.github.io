---
layout: post
title: kotlin语法-DSL
date: 2021-04-21
categories: blog
tags: [kotlin, 编程语言]
description: kotlin DSL
---

## DSL 概念
维基百科定义
> 领域特定语言（英语：domain-specific language、DSL）指的是专注于某个应用程序领域的计算机语言。又译作领域专用语言。

DSL 主要分三类
- 内部DSL
- 外部DSL
- 语言工作台

### 内部 DSL

内部 DSL 指的是通过编程语言的语法形成的针对某一领域的特定用法。比如说前端的 jQuery ，Android 的 JetPack Compose 。

Jetpack Compose 示例：
``` kotlin

 Column(
        Modifier.background(backgroundColor.copy(alpha = 0.95f))
    ) {
        AppBar(
           title = "DSL Sample"
        )
        Divider()
        Content()
    }
```

### 外部 DSL

外部 DSL 一般指的是独立的编程语言。 比如说前端的 HTML、 CSS 还有 Android 的 xml 布局文件。

Android xml 布局文件示例：
``` xml

<LinearLayout
	xmlns:android="http://schemas.android.com/apk/res/android"
	android:id="@+id/switch_container"
	android:layout_width="wrap_content"
	android:layout_height="wrap_content"
	android:layout_gravity="right"
	android:orientation="horizontal">
	<TextView
		android:layout_width="wrap_content"
		android:layout_height="wrap_content"
		android:text="单词间停顿"
		android:textColor="@color/biz_words_color_b0b4be"
		android:textSize="@dimen/textsize12"
		android:layout_marginRight="@dimen/margin5"/>

	<ImageView
		android:id="@+id/iv_interval_mode"
		android:layout_width="wrap_content"
		android:layout_height="wrap_content"
		android:layout_marginRight="@dimen/margin9"
		android:src="@drawable/biz_words_icon_listen_mode_switch_short" />

</LinearLayout>

```
### kotlin DSL

kotlin 的 DSL 属于内部 DSL。

目前官方文档里描述了两个实践

- 类型安全构造器（Type-safe builders）
  - html
  - 图形构建 anko
- 操作符重载

#### 类型安全构造器

类型安全的构建器可以创建基于 Kotlin 的适用于采用半声明方式构建复杂层次数据结构领域专用语言。

其中一个很好的例子是[针对前端领域的一个DSL](https://github.com/Kotlin/kotlinx.html)

``` kotlin
li {
    classes = setOf("dropdown")

    a("#", null) {
        classes = setOf("dropdown-toggle")
        attributes["data-toggle"] = "dropdown"
        role = "button"
        attributes["aria-expanded"] = "false"

        ul {
            classes = setOf("dropdown-menu")
            role = "menu"

            li { a("#") { +"Action" } }
            li { a("#") { +"Another action" } }
            li { a("#") { +"Something else here" } }
            li { classes = setOf("divider")}
            li { classes = setOf("dropdown-header"); +"Nav header" }
            li { a("#") { +"Separated link" } }
            li { a("#") { +"One more separated link" } }
        }

        span {
            classes = setOf("caret")
        }
    }
}

```
## 原理

其实现原理就是使用的 kotlin 的高阶函数特性

### 高阶函数概念
维基百科：
> 在函数式编程中，折叠（fold），也称为归约（reduce）、积累（accumulate）、聚集（aggregate）、压缩（compress）或注入（inject），指称一组高阶函数，它们分析递归数据结构并通过使用给定组合运算，将递归的处理它的构成部件、建造一个返回值的结果重组起来。典型的，要向折叠提供一个组合函数，一个数据结构的顶端节点，和可能的在特定条件下使用的某些缺省值。折叠接着以系统性方式使用这个函数，进行组合这个数据结构的层级中的元素。折叠在某种意义上是展开（unfold）的对偶，它接受一个种子值并共递归的应用一个函数，来确定如何进一步的构造一个共递归的数据结构。折叠递归的分解这个数据结构，在每个节点应用一个组合函数于它的终结值和递归结果之上，用得到这个结果替代它。折叠是catamorphism，而展开是anamorphism。

kotlin 官方
> 高阶函数是将函数用作参数或返回值的函数。

高阶函数在外部调用的时候很简单。

高阶函数示例：
``` kotlin

// 声明
fun html(html:(Html.()->Unit)):Html{
    return Html().alse(html)
}

class Html{
    var mHead : Head? = null
    fun head(head:(Head.()->Unit)){
        mHead = Head().also(head)
    }

    // body 类似
}

// 使用

html {
    head{

    }
    body{

    }
}

```

kotlin 外部可以这样调用主要依赖了两个语法
- lambda 表达式
  在 Kotlin 中有一个约定：如果函数的最后一个参数是函数，那么作为相应参数传入的 lambda 表达式可以放在圆括号之外

所以写法原来是这样的
``` kotlin

html(head = {

})

```


## 总结

参考

https://www.yuque.com/arvinxx/hci-lab/50542017-40aa-4089-9b06-aa725258ed8f
