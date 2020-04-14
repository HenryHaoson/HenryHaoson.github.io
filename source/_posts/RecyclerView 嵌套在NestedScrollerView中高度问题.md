---
layout: post
title: ScrollView嵌套RecyclerView导致的Recycler的问题。
date: 2017-8-16
categories: blog
tags: [Android]
description: 这个现象的原因还有待看源码才好具体分析。这里只是做一个测试。
---

测试:

-   wrap_content : 会加载所有的 item。高度打印 0
-   match_parent : 加载所有 item。高度打印 0
-   450 固定 dp : 加载正常，高度打印 1350

测试结果也应了一句话

这个是看到的一个关于 RecyclerView 的使用 Tip

When RecyclerView is wrapping its content, it’s not recycling anymore. Every record in the dataset has an item View kept in memory for as long as the RecyclerView is in the layout hierarchy.

结论，尽量不要在 ScrollView 中使用 RecyclerView，如果要使用，请设置固定的宽高，否则将一下子加载所有 item。(ScrollView 的 MeasureSpecMode 是 UNSPECIFIED,子 view 要多大有多大(;´༎ຶД༎ຶ`))

测试:

当 recyclerView 没有嵌套在 ScrollView 中的时候，

-   wrap_content : 加载正常，高度打印是 0；
-   match_parent : 加载正常，高度显示 336;
-   450 固定 dp : 加载正常，显示不全（是因为高度问题）。
