---
layout: post
title: kotlin语法-作用域函数
date: 2021-04-21
categories: blog
tags: [kotlin, 编程语言]
description: kotlin 作用域函数
---

## 概念

> Kotlin 标准库包含几个函数，它们的唯一目的是在对象的上下文中执行代码块。当对一个对象调用这样的函数并提供一个 lambda 表达式时，它会形成一个临时作用域。在此作用域中，可以访问该对象而无需其名称。这些函数称为作用域函数。共有以下五种：let、run、with、apply 以及 also。

官方从两个角度去区分他们。

- 引用上下文的方式
  - this
  - it
- 返回值
  - 上下文本身
  - lambda 表达式结果

## let
- 应用上下文方式：it
- 返回值：lambda 表达式结果

可以理解成 let it do something。
