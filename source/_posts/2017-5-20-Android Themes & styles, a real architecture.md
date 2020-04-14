---
layout: post
title: theme
date: 2017-5-20
categories: blog
tags: [Android]
description: Theme
---

# Android Themes & styles, a real architecture（译）

在当前的 android 开发中，有很多框架十分流行，比如说一些架构 MVP,MVVM 或者 clean 架构，又比如说 Rxjava，Dagger,还有 google 二儿子 Kotlin。
但是我们是否是注意到，当我们构建项目时，我们对 Theme 和 Style 的构建时还是用的老一套的方法，我们并没有考虑过如何构建他们，因为我们往往更注重我们的 java 代码，那些 xml 代码经常被我们忽略。

## 一个经常犯的错误

当新建一个项目时，androidStudio 会帮我们构建一个初始的 AppTheme 的 style.xml 文件。当你写项目不断的进行开发迭代，你的 style.xml 文件里会被你塞满许多属性配置。
但是有一点，当你的项目的最低 sdk 为 16，而你又想使用 api19 里面的特性时，比如说`windowTransluscentStatus`,你不能在文件里直接配置，这样会报错。

![image](http://blog.octo.com/wp-content/uploads/2017/03/capture-decran-2017-03-31-a-10-33-05-2.png)

在 android 开发的时候，你可以创建特定的 API 文件夹，用来存放特定 API 的配置。比如说你可以创建一个`res/values-v19/`文件夹用来存放一个新的 sytle.xml 文件，当运行在 API 为 19 的手机上，`res/values-v19/`这个文件夹比`res/values/`有更高的优先级，也就是说首先会加载`res/values-v19/`

![image](http://blog.octo.com/wp-content/uploads/2017/03/capture-decran-2017-03-31-a-10-36-56.png)

所有说现在当我们遇到上面的问题的时候，我们可以按照下面的步骤来。

-   复制原来 sytle.xml 里面的所有属性的配置。
-   粘贴到新建的-v19 文件夹里的 style.sml 里。
-   在新建的 style.xml 文件里添加新的属性配置。

按照上面的方法，你可以实现对应的 API 加载对应的 sytle,你确实可以添加你的新特性。但是！！！请你不要这么做，因为当你的配置越来越多的时候，你会疯掉的，你必须每次添加新的配置的时候，还要记得复制到你的-v19 文件夹里。
还有一种解决方法

-   在`res/values/styles.xml`里建一个 BaseAppTheme，里面添加除了特定 API 性质的所有属性
-   在相同文件下改变 AppTheme 让他继承 BaseAppTheme
-   在你的-v19 里，你只要也让你的 AppTheme 继承 BaseAppTheme，
-   在-v19 里面添加你需要使用的新特性。

如果你使用上面的方法，你就可以自由自在的添加新特新而不需要担心拷贝的问题了。这多好。

`values/styles.xml`

```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <!-- Base application theme. -->
    <style name="BaseAppTheme" parent="Theme.AppCompat.Light.NoActionBar">
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
        <item name="android:windowBackground">@color/windowBackground</item>
    </style>
    <style name="AppTheme" parent="BaseAppTheme"/>
</resources>
```

`values-v19/styles.xml`

```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <style name="AppTheme" parent="BaseAppTheme">
        <item name="android:windowTranslucentStatus">true</item>
    </style>
</resources>
```

## 更高 API 出现的问题

假如说你现在使用的是更高级的 API21，你想要使用里面的新特性，比如说共享元素（这个东西确实很炫酷）。
你现在学了上面的只是肯定会新建一个-v21 文件夹。在里面加入你的新特性。

```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <style name="AppTheme" parent="BaseAppTheme">
        <item name="android:windowSharedElementEnterTransition">@android:animator/fade_in</item>
    </style>
</resources>
```

现在当 app 运行在 android5.0 系统的手机上时，你会发现你的状态栏变了，因为你的手机使用的 AppTheme 是-v21 的属性，他会首先加载-v21 的属性配置，但是-v21 并没有继承-v19，所以并不会加载-v19d 属性配置。

## theme 继承链

为了解决上面的问题，我们来重写 themes,让不同的 API 采用链式继承方式来实现所有的属性配置

`values/styles.xml`

```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <style name="AppTheme" parent="Base.V0.AppTheme"/>

    <style name="Base.V0.AppTheme" parent="Theme.AppCompat.Light.NoActionBar">
        <!-- Generic, non-specific attributes -->
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
        <item name="android:windowBackground">@color/windowBackground</item>
    </style>
</resources>
```

`values-v19/styles.xml`

```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <style name="AppTheme" parent="Base.V19.AppTheme"/>

    <style name="Base.V19.AppTheme" parent="Base.V0.AppTheme">
        <!-- API 19 specific attributes -->
        <item name="android:windowTranslucentStatus">true</item>
    </style>
</resources>
```

`values-v21/styles.xml`

```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <style name="AppTheme" parent="Base.V21.AppTheme"/>

    <style name="Base.V21.AppTheme" parent="Base.V19.AppTheme">
        <!-- API 21 specific attributes -->
        <item name="android:windowSharedElementEnterTransition">@android:animator/fade_in</item>
    </style>
</resources>
```

使用这种主题架构，对于每个 API 级别，AppTheme 将具有来自所有 API 级别的所有属性，并且以后很容易扩展和添加新的 api 级别特定属性。
