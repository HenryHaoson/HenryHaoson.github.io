---
layout: post
title: GARSP概念
date: 2020-4-27
categories: blog
tags: [design, OO, GRASP]
description: 在学习DDD 的时候看到了这样一种设计概念，于是了解一下。最近看DDD，执着于事物的定义，职责的划分。这个模式应该比GoF（那23种设计模式更适合目前的我）
---

### 概念

General Responsibility Assignment Software Patterns（通用职责分配软件模式）

GRASP 专注于职责的划分。所以他更适合 DDD [](https://www.jdon.com/53047)

GRASP 和 GoF 没有关系，但是和 GoF 的 SOLID 一样也是属于 OOD 的原则。

分配的职责分为两种：

-   knowing 知道
-   doing 做

对象的 doing 职责包括：

-   自己做某事，比如创建一个对象或进行计算
-   在其他对象中启动操作
-   控制和协调其他对象的活动

对象的 knowing 职责包括：

-   了解私有封装数据
-   了解相关对象
-   知道它可以导出或计算的东西

GRASP 包含 9 个原则：

-   controller(控制器)
-   creator(创建者)
-   indirection(间接)
-   information expert(信息专家)
-   high cohesion(搞内聚)
-   low coupling(低耦合)
-   polymorphism(多态)
-   protected variations(受保护的变化)
-   pure fabrication(纯粹制造)

后续补充各个原则的概念～

### low coupling(低耦合)

模块之间不应该有太多依赖关系，有的话也应该通过接口依赖，接口依赖也要保持最小原则。

IoC(控制反转)经常用来做松散耦合，DI(依赖注入)就是 IoC 的一种.

### information expert(信息专家)（DDD 聚合）

是谁的信息就由谁提供

比如说有单词这个类`Word`，那么这个单词的内容信息应该由这个类提供。

```java
class Word{
    public String getContent(){
        return mContent;
    }
}


class View{
    private void render(){
        mTextView.setText(mWord.getContent());
    }
}

```
