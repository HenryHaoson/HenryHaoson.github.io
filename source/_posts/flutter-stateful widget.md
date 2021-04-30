---
layout: post
title: flutter stateful widget
date: 2021-04-30
categories: blog
tags: [flutter, 组件, 跨平台]
description: flutter stateful widget
---

## 概念
官方定义：
> A widget is either stateful or stateless. If a widget can change—when a user interacts with it, for example—it’s stateful.

> A stateless widget never changes. Icon, IconButton, and Text are examples of stateless widgets. Stateless widgets subclass StatelessWidget.

> A stateful widget is dynamic: for example, it can change its appearance in response to events triggered by user interactions or when it receives data. Checkbox, Radio, Slider, InkWell, Form, and TextField are examples of stateful widgets. Stateful widgets subclass StatefulWidget.

> A widget’s state is stored in a State object, separating the widget’s state from its appearance. The state consists of values that can change, like a slider’s current value or whether a checkbox is checked. When the widget’s state changes, the state object calls setState(), telling the framework to redraw the widget.

Stateful Widget 是动态的，会根据内部的状态去渲染，他的状态存储在 State 对象中。

这里可以看到视图和数据分离的原则。 State 理论上只应该存储和视图相关的状态。 这个外部使用的时候就可以直接构造 state 就可以了，实现解耦复用。

## 使用

下边根据官方文档来实现一个 StateFulWidget。

