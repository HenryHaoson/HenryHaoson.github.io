---
layout: post
title: 搭建自己的fragment库，管理自己的fragment栈
date: 2016-10-10
categories: blog
tags: [Android]
description: 搭建自己的fragment库，管理自己的fragment栈
---

想彻底了解 fragment 和避免踩坑的同学可以看看这三篇文章
http://www.jianshu.com/p/d9143a92ad94
http://www.jianshu.com/p/fd71d65f0ec6
http://www.jianshu.com/p/38f7994faa6b
看过此人写的三篇文章和所搭建的 fragment 库后我也决定搭一个自己的 fragment 库（特别适用于单 activity 多 fragment)。
我为了跳过不必要踩的坑，所以在搭建的时候使用自己管理的 fragment 栈（这不是造轮子！）。这样妈妈再也不用担心你的 fragment 栈出现嵌套问题啦。
首先想到搭一个库，肯定要用户友好型啊。那我们就定义几个方便的方法。

1.loadfragment（int containerId,SupportFragment fragment），在 activity 里面加载主 fragment（第一层 fragment）其中 containerId 是显示 fragment 布局资源 id，fragment 就是要加载的 fragment。

2.loadfragments(int containerId,int position,SupportFragment ...fragments)，大家应该都用过类似 qq 的底部导航栏布局吧，每个导航按钮对应的页面是用 fragment 来实现的，这样的函数就是用来一次性加载多个 fragment。其中 containerId 同上，position 是第一次加载时显示的是第几个 fragment，fragments 就是要加载的 fragment 们。

3.startfragment（int containerId,SupportFragment fragment),这个方法是用来加载一些使用频率较少的一些 fragment。由 activity 执行。

4.close(SupportFragment fragment),这个函数是用来删除嵌套的子 fragment。

5 .close（），这个函数是用来删除本身 fragment 的方法，其实就是调用父 fragment 的 close（SupportFragment fragment)方法。

6.show（SupportFragment fragment），如果有一些 fragment 你是要经常使用或者显示的，那么使用 hide（），show（）方法就可以实现 fragment 的切换，这里不知道的可以参考我上面分享的文章。这个 show（）方法就是封装了这一切换的操作。

写到这里我就想到如果我在这里公布源码那么还会有人去给我的 github star star star 吗。所以就不公布源码了。最重要的部分应该是 fragment 的嵌套和返回栈的管理吧。具体实现在我的 github 上，地址在文章最后。
这里我给大家贴上 SupportAction，和基本事务类的代码

```
public interface SupportAction {


    void loadfragment(@IdRes int containerId,@NonNull SupportFragment fragment);

    /**
     * 加载多个fragment使用场景类似于底部导航栏，懒加载
     * @param containerId
     * @param postion
     * @param fragments
     */
    void loadfragments(@IdRes int containerId,int postion,@NonNull SupportFragment ... fragments);
    /**
     * 打开新的页面
     * 默认操作：隐藏其余所有的Fragment
     */
    void startfragment(@IdRes int containerId, @NonNull SupportFragment fragment);

    /**
     * 显示指定的fragment隐藏其他的fragment
     *
     * @param fragment
     */
    void show(@NonNull SupportFragment fragment);


    /**
     * 关闭页面
     * 只有没有子Fragment的时候才执行该方法
     * 默认操作：显示上个页面
     */
    void close(@NonNull SupportFragment fragment);

    /**
     * 关闭当前页面
     * 默认操作交给上级处理自己
     */
    void close();

    void throwException(SupportException e);

</pre>


<pre>
public class FragmentBaseTransaction {
    private FragmentBaseTransaction() {

    }

    public static Builder start(FragmentManager fm) {
        return new Builder(fm);
    }

    public static class Builder {
        private FragmentManager fm;
        private FragmentTransaction ft;

        private Builder(FragmentManager fm) {
            this.fm = fm;
            ft = fm.beginTransaction();
        }

        public void commit() {
            ft.commitAllowingStateLoss();
            fm.executePendingTransactions();
            ft = null;
            fm = null;
        }

        public Builder add(@IdRes int containerId, @NonNull Fragment fragment) {
            ft.add(containerId, fragment,getFragmentTag(fragment));
            return this;
        }

        public Builder remove(Fragment fragment) {
            if (isFragmentAdded(fm, fragment)) {
                ft.detach(fragment);
                ft.remove(fragment);
            }
            return this;
        }

        public Builder removeAll() {
            List<Fragment> fragments = fm.getFragments();
            if (fragments == null || fragments.size() == 0) {
                return this;
            }
            for (Fragment fragment : fragments) {
                if (isFragmentAdded(fm, fragment)) {
                    ft.detach(fragment);
                    ft.remove(fragment);
                }
            }
            return this;
        }

        public Builder show(Fragment fragment) {
            ft.show(fragment);
            return this;
        }

        public Builder hide(Fragment fragment) {
            ft.hide(fragment);
            return this;
        }

        public Builder hideAll() {
            List<Fragment> fragments = fm.getFragments();
            if (fragments == null || fragments.size() == 0) {
                return this;
            }
            for (Fragment fragment : fragments) {
                if (fragment == null) {
                    continue;
                }
                hide(fragment);
            }
            return this;
        }

    }

    public static String getFragmentTag(@NonNull Fragment fragment) {
        if (fragment instanceof SupportFragment) {
            return ((SupportFragment) fragment).getFragmentTAG();
        }
        return fragment.getClass().getSimpleName();
    }

    public static Fragment getFragment(FragmentManager fm, String fragmentTag) {
        return fm.findFragmentByTag(fragmentTag);
    }

    public static Boolean isFragmentAdded(FragmentManager fm, Fragment fragment) {
        if (fragment == null) {
            return false;
        }
        Fragment fragmentByTag = fm.findFragmentByTag(getFragmentTag(fragment));
        return fragmentByTag != null && fragmentByTag.isAdded();
    }
}
```

最后，贴上 github 地址：https://github.com/HenryHaoson/EasyFragment
