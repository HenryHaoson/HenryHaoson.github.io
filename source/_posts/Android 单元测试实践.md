---
layout: post
title: Android 单元测试实践
date: 2021-1-27
categories: blog
tags: [Android, 单元测试]
description: 着重看用于表示模型的3种模型元素模式。细品讲的有道理的话。作为以后建模的参考原则。
---

## 概念
在计算机编程中，单元测试（英语：Unit Testing）又称为模块测试，是针对程序模块（软件设计的最小单位）来进行正确性检验的测试工作。程序单元是应用的最小可测试部件。在过程化编程中，一个单元就是单个程序、函数、过程等；对于面向对象编程，最小单元就是方法，包括基类（超类）、抽象类、或者派生类（子类）中的方法。
[维基百科]("https://zh.wikipedia.org/wiki/%E5%8D%95%E5%85%83%E6%B5%8B%E8%AF%95")

## 单元测试作用
    - 提高代码质量
    - 适应变更
      - 在重构的时候可以检验程序的正确性
    - 降低测试、开发成本
      - 单元测试覆盖度高的程序往往证明程序出错的概率低，会减少测试打回的次数，减少开发修复的成本。

## 框架

### Junit
Junit 是 Android 官方提供的单元测试框架，是 Android 单元测试的基础。

```gradle
implementation 'junit:junit:4.12'
```

默认的 Android Project 下面会生成 src/test 目录，在这里可以编写单元测试。AS还提供了快捷方式，可以在源文件中使用快捷键`command` `shift` `t` 在 src/test 目录下生成这个文件的单元测试。




### Mockito

### Powermock

### Robolectric
