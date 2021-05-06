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

## 总结

参考

https://www.yuque.com/arvinxx/hci-lab/50542017-40aa-4089-9b06-aa725258ed8f
