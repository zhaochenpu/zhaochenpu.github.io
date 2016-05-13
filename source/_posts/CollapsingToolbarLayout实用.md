title: 看，这个工具栏能折叠!——Android CollapsingToolbarLayout使用介绍及B站app视频折叠工具栏模仿
date: 2016-05-14
tags: Android
---

我非常喜欢Material Design里可折叠工具栏的效果，bilibili客户端视频详情页就是采用的这种设计。

我们就通过模仿其实现来深入了解下CollapsingToolbarLayout的使用。

## 一、相关基础属性介绍 ##
Android studio中有一个Activity模板叫ScrollingActivity，它实现的就是简单的可折叠工具栏，我们将此模板添加到项目中。

ScrollingActivity的布局代码如下

    <?xml version="1.0" encoding="utf-8"?>
	<android.support.design.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true">

    <android.support.design.widget.AppBarLayout
        android:id="@+id/app_bar"
        android:layout_width="match_parent"
        android:layout_height="@dimen/app_bar_height"
        android:fitsSystemWindows="true"
        android:theme="@style/AppTheme.AppBarOverlay">

        <android.support.design.widget.CollapsingToolbarLayout
            android:id="@+id/toolbar_layout"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:fitsSystemWindows="true"
            app:contentScrim="?attr/colorPrimary"
            app:layout_scrollFlags="scroll|exitUntilCollapsed">

            <android.support.v7.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                app:layout_collapseMode="pin"
                app:popupTheme="@style/AppTheme.PopupOverlay" />

        </android.support.design.widget.CollapsingToolbarLayout>
    </android.support.design.widget.AppBarLayout>

    <include layout="@layout/content_scrolling" />

    <android.support.design.widget.FloatingActionButton
        android:id="@+id/fab"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="@dimen/fab_margin"
        android:src="@android:drawable/ic_dialog_email"
        app:layout_anchor="@id/app_bar"
        app:layout_anchorGravity="bottom|end" />

	</android.support.design.widget.CoordinatorLayout>

AppBarLayout是一种支持响应滚动手势的app bar布局（比如工具栏滚出或滚入屏幕），CollapsingToolbarLayout则是专门用来实现子布局内不同元素响应滚动细节的布局。

AppBarLayout的子布局有5中滚动标识(就是上面代码CollapsingToolbarLayout中配置的app:layout_scrollFlags属性)：

1. scroll:将此布局和滚动时间关联。这个标识要设置在其他标识之前，没有这个标识则布局不会滚动且其他标识设置无效。
1. enterAlways:任何向下滚动操作都会使此布局可见。这个标识通常被称为“快速返回”模式。
1. enterAlwaysCollapsed：假设你定义了一个最小高度（minHeight）同时enterAlways也定义了，那么view将在到达这个最小高度的时候开始显示，并且从这个时候开始慢慢展开，当滚动到顶部的时候展开完。
1. exitUntilCollapsed：当你定义了一个minHeight，此布局将在滚动到达这个最小高度的时候折叠。
2. snap:当一个滚动事件结束，如果视图是部分可见的，那么它将被
滚动到收缩或展开。例如，如果视图只有底部25%
显示，它将折叠。相反，如果它的底部75%
可见，那么它将完全展开。

CollapsingToolbarLayout可以通过app:contentScrim设置折叠时布局的颜色，通过app:statusBarScrim设置折叠时状态栏的颜色。默认contentScrim是colorPrimary的色值，statusBarScrim是colorPrimaryDark的色值。这个后面会用到。

CollapsingToolbarLayout的子布局有3种折叠模式（Toolbar中设置的app:layout_collapseMode）

1. off：这个是默认属性，布局将正常显示，没有折叠的行为。
1. pin：CollapsingToolbarLayout折叠后，此布局将固定在顶部。
1. parallax：CollapsingToolbarLayout折叠时，此布局也会有视差折叠效果。

当CollapsingToolbarLayout的子布局设置了parallax模式时，我们还可以通过app:layout_collapseParallaxMultiplier设置视差滚动因子，值为：0~1。

FloatingActionButton这个控件通过app:layout_anchor这个设置锚定在了AppBarLayout下方。FloatingActionButton源码中有一个Behavior方法，当AppBarLayout收缩时，FloatingActionButton就会跟着做出相应变化。关于CoordinatorLayout和Behavior，我下一篇文章会和大家一起学习。

注意，CollapsingToolbarLayout和ScrollView一起使用会有滑动bug，应该使用NestedScrollView来替代ScrollView。

## 二、模仿bilibili客户端视频详情页 ##

我们先对原界面分析一下。

界面初始，CollapsingToolbarLayout是展开状态，显示的是视频封面。我们向上滚动界面，CollapsingToolbarLayout收缩。当AppBarLayout完全收缩的时候视频av号隐藏，显示出来一个小电视图标和“立即播放”，点击则使AppBarLayout完全展开，CollapsingToolbarLayout子布局由ImageView切换为视频弹幕播放器。

额...弹幕播放器...

B站很早就开源了一个弹幕引擎，还起了个狂拽酷炫的名字叫“烈焰弹幕使 ”，源码在github上，项目名叫[DanmakuFlameMaster](https://github.com/Bilibili/DanmakuFlameMaster)。

来我们先看修改完成的布局。

    <?xml version="1.0" encoding="utf-8"?>
    <android.support.design.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/coordinatorLayout"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <android.support.design.widget.AppBarLayout
        android:id="@+id/app_bar"
        android:layout_width="match_parent"
        android:layout_height="@dimen/app_bar_height"
        android:fitsSystemWindows="true"
        android:theme="@style/AppTheme.AppBarOverlay">

        <android.support.design.widget.CollapsingToolbarLayout
            android:id="@+id/toolbar_layout"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            app:contentScrim="?attr/colorPrimary"
            app:statusBarScrim="@android:color/transparent"
            app:layout_scrollFlags="scroll|exitUntilCollapsed|snap">

            <!--封面图片-->
            <ImageView
                android:id="@+id/imageview"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:scaleType="centerCrop"
                android:src="@drawable/diqiu"
                app:layout_collapseMode="parallax"
                app:layout_collapseParallaxMultiplier="0.7"
                android:fitsSystemWindows="true"/>

            <!--视频及弹幕-->
            <FrameLayout
                android:id="@+id/video_danmu"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                app:layout_collapseMode="parallax"
                app:layout_collapseParallaxMultiplier="0.7"
                android:fitsSystemWindows="true"
                android:visibility="gone">
                <VideoView
                    android:id="@+id/videoview"
                    android:layout_width="match_parent"
                    android:layout_height="match_parent" />

                <!--哔哩哔哩开源的弹幕控件-->
                <master.flame.danmaku.ui.widget.DanmakuView
                    android:id="@+id/danmaku"
                    android:layout_width="match_parent"
                    android:layout_height="match_parent" />
            </FrameLayout>

            <android.support.v7.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                app:layout_collapseMode="pin"
                app:popupTheme="@style/AppTheme.PopupOverlay" >

                <!--自定义带图片的立即播放按钮-->
                <android.support.v7.widget.ButtonBarLayout
                    android:id="@+id/buttonlayout"
                    android:layout_width="match_parent"
                    android:layout_height="match_parent"
                    android:gravity="center"
                    android:visibility="gone">
                    <ImageView
                        android:layout_width="wrap_content"
                        android:layout_height="match_parent"
                        android:layout_gravity="center_horizontal"
                        android:src="@mipmap/ic_play_circle_filled_white_48dp"/>

                    <TextView
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:textColor="#ffffff"
                        android:text="立即播放"
                        android:layout_gravity="center_vertical"
                       />
                </android.support.v7.widget.ButtonBarLayout>

            </android.support.v7.widget.Toolbar>
        </android.support.design.widget.CollapsingToolbarLayout>
    </android.support.design.widget.AppBarLayout>

    <include layout="@layout/content_scrolling" />

    <android.support.design.widget.FloatingActionButton
        android:id="@+id/fab"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="@dimen/fab_margin"
        android:src="@mipmap/ic_play_circle_filled_white_48dp"
        app:layout_anchor="@id/app_bar"
        app:layout_anchorGravity="bottom|end" />

    </android.support.design.widget.CoordinatorLayout>

我把colorPrimary的色值修改成了B站的“少女粉”，播放的图标是从网上找的。

    <color name="colorPrimary">#FA7199</color>

因为我们要实现沉浸式状态栏，所以就需要先把整个activity设置成状态栏透明模式。然后布局文件中，CollapsingToolbarLayout中最上方的子布局要设置android:fitsSystemWindows="true"，如果没有设置，则子布局会位于状态栏下方，未延伸至状态栏。


布局并不算复杂，接下来我们实现具体功能。

我们需要判断CollapsingToolbarLayout的折叠、展开状态，但是官方并没有提供现成的方法。查看源码，我们看到CollapsingToolbarLayout是通过实现AppBarLayout的OnOffsetChangedListener接口，根据AppBarLayout的偏移来实现子布局和title的视差移动，以及ContentScrim和StatusBarScrim的显示。那么我们也可以通过调用AppBarLayout的addOnOffsetChangedListener方法获取AppBarLayout的位移，判断CollapsingToolbarLayout的状态。

先定义了一个枚举定义出CollapsingToolbarLayout展开、折叠、中间过渡这三种状态。

     private enum CollapsingToolbarLayoutState {
        EXPANDED,
        COLLAPSED,
        INTERNEDIATE
    }

