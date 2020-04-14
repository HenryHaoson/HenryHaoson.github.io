---
layout: post
title: searchview
date: 2017-7-7
categories: blog
tags: [Android]
description: searchview的三种显示状态
---

-   1.在 toolbar 中使用 searchview

```
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <item
        android:id="@+id/action_search"
        android:icon="@drawable/ic_search_white_24dp"
        android:title="搜索"
        //使用上面这种的时候在代码中动态设置状态会失效，而是采用这里设置的状态
        <!--app:showAsAction="always|collapse>ActionView"
        app:showAsAction="always"
        app:actionViewClass="android.support.v7.widget.SearchView"/>
</menu>
```

-   2.我不会使用什么 oncreateOptionMenu 这个方法。直接使用 toolbar，在设置 toolbar 默认状态的时候我只会通过代码改。

    > /_------------------ SearchView 有三种默认展开搜索框的设置方式，区别如下： ------------------_/

-   //设置搜索框直接展开显示。左侧有放大镜(在搜索框中) 右侧有叉叉 可以关闭搜索框

> mSearchView.setIconified(false);

-   //设置搜索框直接展开显示。左侧有放大镜(在搜索框外) 右侧无叉叉 有输入内容后有叉叉 不能关闭搜索框

> mSearchView.setIconifiedByDefault(false);

-   //设置搜索框直接展开显示。左侧有无放大镜(在搜索框中) 右侧无叉叉 有输入内容后有叉叉 不能关闭搜索框

> mSearchView.onActionViewExpanded();

```
        setHasOptionsMenu(true);
        toolbar=view.findViewById(R.id.main_toolbar);
        toolbar.inflateMenu(R.menu.toolbar_menu);
        Menu menu=toolbar.getMenu();
        MenuItem searchAction=menu.findItem(R.id.action_search);
        search= (SearchView) MenuItemCompat.getActionView(searchAction);
        search.setQueryHint("请输入要翻译的单词");
        //设置状态默认为展开的形式，总共有三种方式
        search.onActionViewExpanded();
        search.setMaxWidth(4000);
```
