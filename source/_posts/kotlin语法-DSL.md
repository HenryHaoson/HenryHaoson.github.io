---
layout: post
title: kotlin语法-DSL
date: 2021-04-21
categories: blog
tags: [kotlin, dsl]
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

kotlin 日期处理

```
val yesterday = 1.days().ago()
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


#### 操作符重载

kotlin 对很多操作符进行了重载

比如说 ==
``` kotlin
// 使用
a == b

实现
a?.equals(b) ?: (b === null)
```

## 原理

其实现原理就是使用的 kotlin 的四个语法知识
- 扩展函数
- lambda
- 中缀调用
- invoke 约定
### 扩展函数

>Kotlin 能够扩展一个类的新功能而无需继承该类或者使用像装饰者这样的设计模式。 这通过叫做 扩展 的特殊声明完成。 例如，你可以为一个你不能修改的、来自第三方库中的类编写一个新的函数。 这个新增的函数就像那个原始类本来就有的函数一样，可以用普通的方法调用。 这种机制称为 扩展函数 。

``` kotlin
class Entity(){
    //
}

// 扩展函数
fun Entity.ability(){

}

// 调用
Entity.ability();

```

扩展函数可以给类增加能力，这个对于贫血模型来说是非常重要的一个能力。当一个模型的定义不在你的控制范围之内，你依然可以通过扩展函数去编写他的职责（补充领域层）。

利用扩展函数可以实现一套和日期有关的 DSL，就像上边一样。

```kotlin
fun Int.days() = Period.ofDays(this)
fun Period.ago() = LocalDate.now() - this

// 调用
val result = 3.days().ago()

```

如果使用扩展属性，可以省略括号，看上去和酷一点

```kotlin
val Int.days:Period
    get() = Period.ofDays(this)

val Period.ago:LocalDate
    get() = LocalDate.now() - this

val result = 3.days.ago
```

### lambda
lambda 可以使我们的代码看上去更加简洁。

目前java 在 Android 中使用 lambda 没有那么方便。kotlin 可以满足我们使用 lambda 的需求。

在kotlin 中，lambda 配合高阶函数就可以实现上边的类似 html 的 DSL（这个是实现嵌套的本职）

#### 高阶函数概念
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

val result = html {
    head{

    }
    body{

    }
}

```

kotlin 外部可以这样调用主要依赖了一个规则
在 Kotlin 中有一个约定：如果函数的最后一个参数是函数，那么作为相应参数传入的 lambda 表达式可以放在圆括号之外

所以写法原来是这样的
``` kotlin

html(head = {

})

```

### 中缀调用

Kotlin 中有种特殊的函数可以使用中缀调用。

标有 infix 关键字的函数也可以使用中缀表示法（忽略该调用的点与圆括号）调用。中缀函数必须满足以下要求：

- 它们必须是成员函数或扩展函数；
- 它们必须只有一个参数；
- 其参数不得接受可变数量的参数且不能有默认值。

中缀调用可以帮助我们实现类似自然语言的调用方式

比如说之前的日期表示法

 ``` kotlin
 val yesterday = 1 days ago

 // 实现
 infix fun Int.days():Period = {/**/}
 infix fun Period.ago():LocalDate = {/**/}
 ```

这样看上去就更舒适了。

### invoke 约定

Kotlin 提供了 invoke 约定，可以让对象向函数一样直接调用，比如 gradle dsl：

```kotlin
dependencies {
    compile("com.android.support:appcompat-v7:27.0.1")
    compile("com.android.support.constraint:constraint-layout:1.0.2")
}

// 等价于：
dependencies.compile("com.android.support:appcompat-v7:27.0.1")
dependencies.compile("com.android.support.constraint:constraint-layout:1.0.2")

class Dependencies{

    fun compile(coordinate:String){
        println("add $coordinate")
    }

    operator fun invoke(block:Dependencies.()->Unit){
        block()
    }
}

>>>val dependencies = Dependencies()
>>>// 以两种方式分别调用 compile()
```

## 总结
其中比较好玩的我觉得是中缀函数，这样代码就像普通文本一样。
还有一个 高阶函数也还行（lambda），嵌套风格看上去也很简洁（UI）

kotlin 的 DSL 是语法糖的融合。 可以玩一玩，体验一下创造一个语言的乐趣。但是如果想要建立一个DSL 还是需要花很多成本的（领域的认知成本）。（目前还不建议在生产随意创建 DSL）

参考

https://www.yuque.com/arvinxx/hci-lab/50542017-40aa-4089-9b06-aa725258ed8f
