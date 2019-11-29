---
layout: post
title: ScrollView嵌套RecyclerView导致的Recycler的问题。
date: 2017-8-16
categories: blog
tags: [android]
description: 这个现象的原因还有待看源码才好具体分析。这里只是做一个测试。
---
测试:
   - wrap_content : 会加载所有的item。高度打印0
   - match_parent : 加载所有item。高度打印0
   - 450固定dp : 加载正常，高度打印1350
   
   
   测试结果也应了一句话
   
   这个是看到的一个关于RecyclerView的使用Tip
   
   When RecyclerView is wrapping its content, it’s not recycling anymore. Every record in the dataset has an item View kept in memory for as long as the RecyclerView is in the layout hierarchy.
   
  结论，尽量不要在ScrollView中使用RecyclerView，如果要使用，请设置固定的宽高，否则将一下子加载所有item。(ScrollView的MeasureSpecMode是UNSPECIFIED,子view要多大有多大(;´༎ຶД༎ຶ`))
  
  测试:

  当recyclerView没有嵌套在ScrollView中的时候，
  
  - wrap_content : 加载正常，高度打印是0；
  - match_parent : 加载正常，高度显示336;
  - 450固定dp : 加载正常，显示不全（是因为高度问题）。
  
