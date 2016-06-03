title: 看，这个工具栏能伸缩折叠!——Android CollapsingToolbarLayout使用介绍
date: 2016-05-14
tags: Android
---

我非常喜欢Material Design里折叠工具栏的效果，bilibili Android客户端视频详情页就是采用的这种设计。这篇文章的第二部分我们就通过简单的模仿bilibili视频详情页的实现来了解下CollapsingToolbarLayout的使用。文章的第三部分介绍了CollapsingToolbarLayout与TabLayout的组合使用。
<!--more-->

有基础的朋友可以直接跳过第一部分。

## 一、相关基础属性介绍 ##
Android studio中有一个Activity模板叫ScrollingActivity，它实现的就是简单的可折叠工具栏，我们将此模板添加到项目中。
![ScrollingActivity.gif](http://upload-images.jianshu.io/upload_images/828721-30f5bb60fee93ae4.gif?imageMogr2/auto-orient/strip)

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
    <android.support.v4.widget.NestedScrollView                    
            android:layout_width="match_parent"        
            android:layout_height="match_parent"              
            app:layout_behavior="@string/appbar_scrolling_view_behavior"
            >    
        <TextView        
                android:layout_width="wrap_content"        
                android:layout_height="wrap_content"        
                android:layout_margin="@dimen/text_margin"         
                android:text="@string/large_text" />     
    </android.support.v4.widget.NestedScrollView>

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

与AppBarLayout组合的滚动布局（Recyclerview、NestedScrollView等）需要设置app:layout_behavior="@string/appbar_scrolling_view_behavior"（上面代码中NestedScrollView控件所设置的）。没有设置的话，AppBarLayout将不会响应滚动布局的滚动事件。

CollapsingToolbarLayout和ScrollView一起使用会有滑动bug，注意要使用NestedScrollView来替代ScrollView。

AppBarLayout的子布局有5种滚动标识(就是上面代码CollapsingToolbarLayout中配置的app:layout_scrollFlags属性)：

1. scroll:将此布局和滚动时间关联。这个标识要设置在其他标识之前，没有这个标识则布局不会滚动且其他标识设置无效。
1. enterAlways:任何向下滚动操作都会使此布局可见。这个标识通常被称为“快速返回”模式。
1. enterAlwaysCollapsed：假设你定义了一个最小高度（minHeight）同时enterAlways也定义了，那么view将在到达这个最小高度的时候开始显示，并且从这个时候开始慢慢展开，当滚动到顶部的时候展开完。
1. exitUntilCollapsed：当你定义了一个minHeight，此布局将在滚动到达这个最小高度的时候折叠。
2. snap:当一个滚动事件结束，如果视图是部分可见的，那么它将被滚动到收缩或展开。例如，如果视图只有底部25%显示，它将折叠。相反，如果它的底部75%可见，那么它将完全展开。

CollapsingToolbarLayout可以通过app:contentScrim设置折叠时工具栏布局的颜色，通过app:statusBarScrim设置折叠时状态栏的颜色。默认contentScrim是colorPrimary的色值，statusBarScrim是colorPrimaryDark的色值。这个后面会用到。

CollapsingToolbarLayout的子布局有3种折叠模式（Toolbar中设置的app:layout_collapseMode）

1. off：这个是默认属性，布局将正常显示，没有折叠的行为。
1. pin：CollapsingToolbarLayout折叠后，此布局将固定在顶部。
1. parallax：CollapsingToolbarLayout折叠时，此布局也会有视差折叠效果。

当CollapsingToolbarLayout的子布局设置了parallax模式时，我们还可以通过app:layout_collapseParallaxMultiplier设置视差滚动因子，值为：0~1。

FloatingActionButton这个控件通过app:layout_anchor这个设置锚定在了AppBarLayout下方。FloatingActionButton源码中有一个Behavior方法，当AppBarLayout收缩时，FloatingActionButton就会跟着做出相应变化。关于CoordinatorLayout和Behavior，我下一篇文章会和大家一起学习。

这一堆属性看着有点烦，大家可以新建一个ScrollingActivity模板去实验一下玩玩。

## 二、模仿bilibili客户端视频详情页 ##

我们先对原界面分析一下。
![哔哩哔哩Android客户端视频详情页.gif](http://upload-images.jianshu.io/upload_images/828721-61defc66bb3e9fb8.gif?imageMogr2/auto-orient/strip)

界面初始，CollapsingToolbarLayout是展开状态，显示的是视频封面。我们向上滚动界面，CollapsingToolbarLayout收缩。当AppBarLayout完全折叠的时候视频av号隐藏，显示出来一个小电视图标和“立即播放”，点击则使AppBarLayout完全展开，CollapsingToolbarLayout子布局由ImageView切换为视频弹幕播放器。

额...弹幕播放器...

B站很早就开源了一个弹幕引擎，还起了个狂拽酷炫吊炸天的名字叫“烈焰弹幕使 ”（一看就是二次元程序猿们的作品→_→），源码在github上，项目名叫[DanmakuFlameMaster](https://github.com/Bilibili/DanmakuFlameMaster)。

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

            <!--视频及弹幕控件-->
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
                    android:id="@+id/playButton"
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

因为我们要实现沉浸式状态栏，所以就需要先把整个activity设置成状态栏透明模式。然后在布局文件中，把CollapsingToolbarLayout里要实现沉浸式的控件设置上android:fitsSystemWindows="true"，如果没有设置，则子布局会位于状态栏下方，未延伸至状态栏。

布局并不算复杂，接下来先实现无弹幕播放时的功能,。

我们需要监听CollapsingToolbarLayout的折叠、展开状态。唉我去，官方并没有提供现成的方法（⊙＿⊙？）。

查看源码，可以看到CollapsingToolbarLayout是通过实现AppBarLayout的OnOffsetChangedListener接口，根据AppBarLayout的偏移来实现子布局和title的视差移动以及ContentScrim和StatusBarScrim的显示。那么我们也可以通过调用AppBarLayout的addOnOffsetChangedListener方法监听AppBarLayout的位移，判断CollapsingToolbarLayout的状态。

先写一个枚举定义出CollapsingToolbarLayout展开、折叠、中间，这三种状态。

     private CollapsingToolbarLayoutState state;

     private enum CollapsingToolbarLayoutState {
        EXPANDED,
        COLLAPSED,
        INTERNEDIATE
    }
接下来对AppBarLayout进行监听，判断CollapsingToolbarLayout的状态并实现相应的逻辑。

为了让大家对状态看着更直观，我在修改状态值的时候把title一起进行了修改。

使用CollapsingToolbarLayout的时候要注意，在完成CollapsingToolbarLayout设置之后再调用Toolbar的setTitle()等方法将没有效果，我们需要改为调用CollapsingToolbarLayout的setTitle()等方法来对工具栏进行修改。（具体原因各位亲去看下CollapsingToolbarLayout源码就知道了　( ˙-˙ )　）

        AppBarLayout  app_bar=(AppBarLayout)findViewById(R.id.app_bar);
        app_bar.addOnOffsetChangedListener(new AppBarLayout.OnOffsetChangedListener() {
            @Override
            public void onOffsetChanged(AppBarLayout appBarLayout, int verticalOffset) {

                if (verticalOffset == 0) {
                    if (state != CollapsingToolbarLayoutState.EXPANDED) {
                        state = CollapsingToolbarLayoutState.EXPANDED;//修改状态标记为展开
                        collapsingToolbarLayout.setTitle("EXPANDED");//设置title为EXPANDED
                    }
                } else if (Math.abs(verticalOffset) >= appBarLayout.getTotalScrollRange()) {
                    if (state != CollapsingToolbarLayoutState.COLLAPSED) {
                        collapsingToolbarLayout.setTitle("");//设置title不显示
                        playButton.setVisibility(View.VISIBLE);//隐藏播放按钮
                        state = CollapsingToolbarLayoutState.COLLAPSED;//修改状态标记为折叠
                    }
                } else {
                    if (state != CollapsingToolbarLayoutState.INTERNEDIATE) {
                        if(state == CollapsingToolbarLayoutState.COLLAPSED){
                            playButton.setVisibility(View.GONE);//由折叠变为中间状态时隐藏播放按钮
                        }
                        collapsingToolbarLayout.setTitle("INTERNEDIATE");//设置title为INTERNEDIATE
                        state = CollapsingToolbarLayoutState.INTERNEDIATE;//修改状态标记为中间
                    }
                }
            }
        });
然后对播放按钮设置监听，点击则调用AppBarLayout的setExpanded(true)方法使工具栏展开。
![CollapsingToolbarLayout状态监听演示.gif](http://upload-images.jianshu.io/upload_images/828721-50b668639326f465.gif?imageMogr2/auto-orient/strip)

哔哩哔哩客户端的title是固定不动的，可以调用CollapsingToolbarLayout的setTitleEnabled(false)方法实现。

细心的朋友可能发现了哔哩哔哩客户端为了避免视频封面图片颜色过浅影响状态栏信息的显示，加了一个渐变的不透明层。

实现渐变遮罩层很简单。先在res/drawable文件夹下新建了一个名为gradient的xml文件，其中代码如下：

    <?xml version="1.0" encoding="utf-8"?>
    <shape xmlns:android="http://schemas.android.com/apk/res/android">

        <gradient
            android:startColor="#33000000"
            android:endColor="#00000000"
            android:angle="270" />

    </shape>

shape节点中，可以通过android:shape来设置形状，默认是矩形。gradient节点中angle的值270是从上到下，0是从左到右，90是从下到上。起始颜色#33000000是20%不透明度的黑色，#00000000表示全透明。

然后在CollapsingToolbarLayout里的ImageView代码下面加上一个自定义view，背景设置为上面的渐变效果。

    <View
       android:layout_width="match_parent"
       android:layout_height="40dp"
       android:background="@drawable/gradient"
       android:fitsSystemWindows="true"
    />    
一般状态栏的高度大概在20dp左右，我为了让渐变效果比较自然，并且不过多影响图（mei）片（zi）,把高度设置成了40dp。（状态栏能看清了，妹子脸也没黑，挺好　(๑• . •๑)　）

![有无渐变遮罩层的对比.jpg](http://upload-images.jianshu.io/upload_images/828721-a613e0814823a935.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我省略了弹幕播放的相关实现，接下来只要在播放按钮监听中写出封面图片的隐藏、视频和弹幕弹幕控件的显示初始化及播放逻辑，在AppBarLayout的三种状态监听中根据是否视频在播放写出其他相应逻辑就好了，感兴趣的朋友可以下载哔哩哔哩的“烈焰弹幕使”源码[DanmakuFlameMaster](https://github.com/Bilibili/DanmakuFlameMaster)玩玩。

B站点击追番或投硬币后会出现一个类似Snackbar的提示控件，可以通过我上一篇文章[没时间解释了，快使用Snackbar!——Android Snackbar花式使用指南](http://www.jianshu.com/p/cd1e80e64311)来实现，欢迎感兴趣的朋友去看看。
![模仿哔哩哔哩视频详情页.gif](http://upload-images.jianshu.io/upload_images/828721-02aad67d7a942ea6.gif?imageMogr2/auto-orient/strip)

真的不是我懒得上代码了，真的…（基友：赶紧的，开黑了。 我：等等我，马上来！＼(≧▽≦)／）

## 三.CollapsingToolbarLayout与TabLayout ##

CollapsingToolbarLayout与TabLayout组合使用的效果也不错。
![CollapsingToolbarLayout与TabLayout.gif](http://upload-images.jianshu.io/upload_images/828721-a68d0e424c1ab044.gif?imageMogr2/auto-orient/strip)

来看下CollapsingToolbarLayout里的代码

        <android.support.design.widget.CollapsingToolbarLayout
            android:id="@+id/collapsingtoolbar"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            app:titleEnabled="false"
            app:contentScrim="@color/colorPrimary"
            app:statusBarScrim="@android:color/transparent"
            app:layout_scrollFlags="scroll|exitUntilCollapsed|snap">
            <ImageView
                android:id="@+id/imageview"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:scaleType="centerCrop"
                android:adjustViewBounds="true"
                app:layout_collapseMode="parallax"
                app:layout_collapseParallaxMultiplier="0.7"
                android:fitsSystemWindows="true"/>
             <View
                android:layout_width="match_parent"
                android:layout_height="40dp"
                android:background="@drawable/gradient"
                android:fitsSystemWindows="true" />
            <android.support.v7.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="96dp"
                app:titleMarginTop="15dp"
                android:gravity="top"
                app:layout_collapseMode="pin"
                app:title="hello"
                app:popupTheme="@style/AppTheme.PopupOverlay"
               />
            <android.support.design.widget.TabLayout
                android:id="@+id/tablayout"
                android:layout_width="match_parent"
                android:layout_height="45dp"
                android:layout_gravity="bottom"/>
        </android.support.design.widget.CollapsingToolbarLayout>

TabLayout没有设置app:layout_collapseMode，在CollapsingToolbarLayout收缩时就不会消失。

CollapsingToolbarLayout收缩时的高度是Toolbar的高度，所以我们需要把Toolbar的高度增加，给TabLayout留出位置，这样收缩后TabLayout就不会和Toolbar重叠。

Toolbar的高度增加，title会相应下移。android:gravity="top"方法使Toolbar的title位于Toolbar的上方，然后通过app:titleMarginTop调整下title距顶部高度，这样Toolbar就和原来显示的一样了。

---
CollapsingToolbarLayout还可以和Palette搭配使用，但是我感觉在实际使用中有些坑，因为CollapsingToolbarLayout中的图片不确定，Palette从图片中获取到的色彩很可能不是你想要的。

感兴趣的朋友可以自己查下Palette的用法。

就是这些。　［］～（￣▽￣）～＊

我的简书主页[http://www.jianshu.com/users/990c16f1edc0/latest_articles](http://www.jianshu.com/users/990c16f1edc0/latest_articles)