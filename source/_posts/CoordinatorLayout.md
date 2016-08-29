title: 一个神奇的控件——Android CoordinatorLayout与Behavior使用指南 

date: 2016-07-6
tags: Android
---
CoordinatorLayout是support.design包中的控件，它可以说是Design库中最重要的控件。本文通过模仿知乎介绍了自定义Behavior，通过模仿百度地图介绍了BottomSheetBehavior的使用。
<!--more-->

## 1.CoordinatorLayout介绍 ##

官方对CoordinatorLayout的描述是这样的：

> CoordinatorLayout is a super-powered FrameLayout.
> 
  CoordinatorLayout is intended for two primary use cases:
>
- As a top-level application decor or chrome layout
- As a container for a specific interaction with one or more child views

官方对Behavior的描述是这样的：

> Interaction behavior plugin for child views of CoordinatorLayout.

简单来说，CoordinatorLayout是用来协调其子view们之间动作的一个父view，而Behavior就是用来给CoordinatorLayout的子view们实现交互的。

CoordinatorLayout在我之前的文章中都有出镜：

[没时间解释了，快使用Snackbar!——Android Snackbar花式使用指南](http://www.jianshu.com/p/cd1e80e64311)中Snackbar显示时FloatingActionButton相应上移为其留出位置

![Snackbar与FloatingActionButton的互动](http://upload-images.jianshu.io/upload_images/828721-9640c1cf8ad5dbeb.gif?imageMogr2/auto-orient/strip)

[看，这个工具栏能伸缩折叠——Android CollapsingToolbarLayout使用介绍](http://www.jianshu.com/p/06c0ae8d9a96)中CollapsingToolbarLayout折叠或展开时，FloatingActionButton跟随运动并且大小相应变化：

![AppBarLayout与FloatingActionButton的互动](http://upload-images.jianshu.io/upload_images/828721-50b668639326f465.gif?imageMogr2/auto-orient/strip)

看下FloatingActionButton的源码就能发现其中有一个Behavior方法继承自CoordinatorLayout.Behavior，并在其中实现了与Snackbar互动时的逻辑。

我本文使用的support:design版本是23.4.0

本文图片接口来自干货集中营http://gank.io/api

## 2.自定义Behavior模仿知乎 ##
![知乎的效果.gif](http://upload-images.jianshu.io/upload_images/828721-7a0b4ec704d59138.gif?imageMogr2/auto-orient/strip)


![本文实现的效果.gif](http://upload-images.jianshu.io/upload_images/828721-4542ebe5198fa443.gif?imageMogr2/auto-orient/strip)

先看下布局
    
    <?xml version="1.0" encoding="utf-8"?>
    <android.support.design.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/behavior_demo_coordinatorLayout"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <android.support.design.widget.AppBarLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:theme="@style/AppTheme.AppBarOverlay">

        <android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            app:layout_scrollFlags="scroll|enterAlways|snap"
            android:background="?attr/colorPrimary" />
    </android.support.design.widget.AppBarLayout>

    <android.support.v4.widget.SwipeRefreshLayout
        android:id="@+id/behavior_demo_swipe_refresh"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior">

        <android.support.v7.widget.RecyclerView
            android:id="@+id/behavior_demo_recycler"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            />
    </android.support.v4.widget.SwipeRefreshLayout>

    <android.support.design.widget.FloatingActionButton
        android:id="@+id/fab"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginRight="16dp"
        android:layout_marginBottom="72dp"
        android:src="@android:drawable/ic_dialog_email"
        app:layout_behavior="com.example.zcp.coordinatorlayoutdemo.behavior.MyFabBehavior"
        android:layout_gravity="bottom|right" />

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize"
        android:layout_gravity="bottom"
        android:background="@color/colorPrimary"
        android:gravity="center"
        app:layout_behavior="com.example.zcp.coordinatorlayoutdemo.behavior.MyBottomBarBehavior">
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:textColor="#ffffff"
            android:text="这是一个底栏"/>
    </LinearLayout>

    </android.support.design.widget.CoordinatorLayout>

SwipeRefreshLayout、FloatingActionButton和当做底栏的LinearLayout上有一个app:layout_behavior配置。

SwipeRefreshLayout配置的"@string/appbar_scrolling_view_behavior"是系统提供的，用来使滑动控件与AppBarLayout互动。

FloatingActionButton和底栏上配置的是我们接下来要自定义的Behavior。

先看FloatingActionButton的Behavior。
    
    public class MyFabBehavior extends CoordinatorLayout.Behavior<View> {

    private static final Interpolator INTERPOLATOR = new FastOutSlowInInterpolator();

    private float viewY;//控件距离coordinatorLayout底部距离
    private boolean isAnimate;//动画是否在进行

    public MyFabBehavior(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    //在嵌套滑动开始前回调
    @Override
    public boolean onStartNestedScroll(CoordinatorLayout coordinatorLayout, View child, View directTargetChild, View target, int nestedScrollAxes) {

        if(child.getVisibility() == View.VISIBLE&&viewY==0){
            //获取控件距离父布局（coordinatorLayout）底部距离
            viewY=coordinatorLayout.getHeight()-child.getY();
        }

        return (nestedScrollAxes & ViewCompat.SCROLL_AXIS_VERTICAL) != 0;//判断是否竖直滚动
    }

    //在嵌套滑动进行时，对象消费滚动距离前回调
    @Override
    public void onNestedPreScroll(CoordinatorLayout coordinatorLayout, View child, View target, int dx, int dy, int[] consumed) {
        //dy大于0是向上滚动 小于0是向下滚动

        if (dy >=0&&!isAnimate&&child.getVisibility()==View.VISIBLE) {
            hide(child);
        } else if (dy <0&&!isAnimate&&child.getVisibility()==View.GONE) {
            show(child);
        }
    }

    //隐藏时的动画
    private void hide(final View view) {
        ViewPropertyAnimator animator = view.animate().translationY(viewY).setInterpolator(INTERPOLATOR).setDuration(200);

        animator.setListener(new Animator.AnimatorListener() {
            @Override
            public void onAnimationStart(Animator animator) {
                isAnimate=true;
            }

            @Override
            public void onAnimationEnd(Animator animator) {
                view.setVisibility(View.GONE);
                isAnimate=false;
            }

            @Override
            public void onAnimationCancel(Animator animator) {
                show(view);
            }

            @Override
            public void onAnimationRepeat(Animator animator) {
            }
        });
        animator.start();
    }

    //显示时的动画
    private void show(final View view) {
        ViewPropertyAnimator animator = view.animate().translationY(0).setInterpolator(INTERPOLATOR).setDuration(200);
        animator.setListener(new Animator.AnimatorListener() {
            @Override
            public void onAnimationStart(Animator animator) {
                view.setVisibility(View.VISIBLE);
                isAnimate=true;
            }

            @Override
            public void onAnimationEnd(Animator animator) {
                isAnimate=false;
            }

            @Override
            public void onAnimationCancel(Animator animator) {
                hide(view);
            }

            @Override
            public void onAnimationRepeat(Animator animator) {
            }
        });
        animator.start();
    }
    }

逻辑并不复杂，我们通过重写Behavior中关于嵌套滑动的两个回调完成了FloatingActionButton的隐藏和显示判断及操作。

单独出场的底栏也可以利用上面一样的方法来设置隐藏或显示，我的底栏是和AppBarLayout一起出场，所以我就让底栏从属于AppBarLayout活动。代码如下：

    public class MyBottomBarBehavior extends CoordinatorLayout.Behavior<View> {

    public MyBottomBarBehavior(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    //确定所提供的子视图是否有另一个特定的同级视图作为布局从属。
    @Override
    public boolean layoutDependsOn(CoordinatorLayout parent, View child, View dependency) {
    //这个方法是说明这个子控件是依赖AppBarLayout的
        return dependency instanceof AppBarLayout;
    }

    //用于响应从属布局的变化
    @Override
    public boolean onDependentViewChanged(CoordinatorLayout parent, View child, View dependency) {

        float translationY = Math.abs(dependency.getTop());//获取更随布局的顶部位置

        child.setTranslationY(translationY);
        return true;
    }

    }

代码量比上个还少。我们还可以通过重写onMeasureChild()来使控件响应从属控件的大小变化。

Behavior提供的很多，我这里用到的只是一部分，大家可以看看源码，根据具体需求去使用。

## 3.BottomSheetBehavior使用（仿百度地图） ##

BottomSheetBehavior是23.2版本后提供的控件，用于实现从底部滑出一个折叠布局的操作，就是谷歌地图中的效果。

别问为什么我不模仿谷歌地图，当然是因为中国特色国情→_→（雾）。


![百度地图BottomSheet.gif](http://upload-images.jianshu.io/upload_images/828721-79452c4674b7c0f3.gif?imageMogr2/auto-orient/strip)

![本文BottomSheet.gif](http://upload-images.jianshu.io/upload_images/828721-ea0115f7071f1620.gif?imageMogr2/auto-orient/strip)

来看下布局
    
    <?xml version="1.0" encoding="utf-8"?>
    <android.support.design.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/bottom_sheet_demo_coordinatorLayout"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <android.support.v4.widget.SwipeRefreshLayout
        android:id="@+id/bottom_sheet_demo_swipe_refresh"
        android:layout_width="match_parent"
        android:layout_height="match_parent">
        <android.support.v7.widget.RecyclerView
            android:id="@+id/bottom_sheet_demo_recycler"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            />
    </android.support.v4.widget.SwipeRefreshLayout>

    <RelativeLayout
        android:id="@+id/design_bottom_sheet_bar"
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize"
        android:background="@color/colorAccent"
        app:layout_anchor="@+id/design_bottom_sheet"
        app:layout_anchorGravity="top"
        android:layout_gravity="bottom"
        android:visibility="gone"
        >
        <ImageView
            android:layout_width="23dp"
            android:layout_height="23dp"
            android:layout_marginLeft="23dp"
            android:src="@mipmap/ic_arrow_back_white"
            android:layout_centerVertical="true"/>
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="点击收起BottomSheet"
            android:textColor="#ffffff"
            android:layout_centerInParent="true"/>
    </RelativeLayout>

    <RelativeLayout
        android:id="@+id/design_bottom_sheet"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:minHeight="100dp"
        app:behavior_peekHeight="56dp"
        app:behavior_hideable="false"
        app:layout_behavior="@string/bottom_sheet_behavior"
        android:background="#ffffff"
        >
        <TextView
            android:id="@+id/bottom_sheet_tv"
            android:layout_width="wrap_content"
            android:layout_height="56dp"
            android:layout_centerHorizontal="true"
            android:gravity="center"
            android:text="这是一个BottomSheet"/>
        <ImageView
            android:id="@+id/bottom_sheet_iv"
            android:layout_width="wrap_content"
            android:layout_height="match_parent"
            android:layout_centerInParent="true"
            android:padding="10dp"
            android:minHeight="100dp"
            android:adjustViewBounds="true"
            android:scaleType="centerInside"
            android:layout_gravity="center"/>
    </RelativeLayout>

    </android.support.design.widget.CoordinatorLayout>

名为design_bottom_sheet的RelativeLayout是一个BottomSheet（这个到底叫什么，底纸？底板？底部折叠纸片？ (⊙_⊙?)），因为它设置了

    app:layout_behavior="@string/bottom_sheet_behavior"

 app:behavior_hideable="false"说明这个BottomSheet不可以被手动滑动隐藏，设置为true则可以滑到屏幕最底部隐藏。

app:behavior_peekHeight设置的是折叠状态时的高度。

名为design_bottom_sheet_bar的RelativeLayout是用来实现BottomSheet完全展开时的自定义顶部工具条，

     app:layout_anchor="@+id/design_bottom_sheet"
     app:layout_anchorGravity="top"
     android:layout_gravity="bottom"

上面这段设置是使design_bottom_sheet_bar的位置与design_bottom_sheet锚定，然后调整了下位置。

    design_bottom_sheet_bar=(RelativeLayout) findViewById(R.id.design_bottom_sheet_bar);

    design_bottom_sheet=(RelativeLayout) findViewById(R.id.design_bottom_sheet);
    bottom_sheet_iv=(ImageView) findViewById(R.id.bottom_sheet_iv);
    bottom_sheet_tv=(TextView) findViewById(R.id.bottom_sheet_tv);

    BottomSheetBehavior behavior = BottomSheetBehavior.from(design_bottom_sheet);

这段是BottomSheet相关控件初始化，我省略了RecyclerView相关配置。

design_bottom_sheet设置的高度是充满父布局，我们需要其给design_bottom_sheet_bar留出位置，所以要修改一下design_bottom_sheet的高度，我把这部分操作放在了onWindowFocusChanged（）里。

        public void onWindowFocusChanged(boolean hasFocus) {
        // TODO Auto-generated method stub
        super.onWindowFocusChanged(hasFocus);

        //修改SetBottomSheet的高度 留出顶部工具栏的位置
        if(!isSetBottomSheetHeight){
            CoordinatorLayout.LayoutParams linearParams =(CoordinatorLayout.LayoutParams) design_bottom_sheet.getLayoutParams();
            linearParams.height=coordinatorLayout.getHeight()-design_bottom_sheet_bar.getHeight();
            design_bottom_sheet.setLayoutParams(linearParams);
            isSetBottomSheetHeight=true;//设置标记 只执行一次
        }

    }

然后我们设置design_bottom_sheet_bar点击后design_bottom_sheet变为折叠状态

    design_bottom_sheet_bar.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                behavior.setState(BottomSheetBehavior.STATE_COLLAPSED);
            }
        });

BottomSheetBehavior设置为折叠状态的话，BottomSheet显示出来的就是peekHeight设置的高度，展开状态显示的是完整的布局，隐藏状态就是滑到屏幕最底部隐藏。

最后为BottomSheetBehavior设置监听

    behavior.setBottomSheetCallback(new BottomSheetBehavior.BottomSheetCallback() {

            //BottomSheet状态改变时的回调
            @Override
            public void onStateChanged(@NonNull View bottomSheet, int newState) {
                
                if(newState!=BottomSheetBehavior.STATE_COLLAPSED&&bottom_sheet_tv.getVisibility()==View.VISIBLE){
                    bottom_sheet_tv.setVisibility(View.GONE);
                    bottom_sheet_iv.setVisibility(View.VISIBLE);                
                }else if(newState==BottomSheetBehavior.STATE_COLLAPSED&&bottom_sheet_tv.getVisibility()==View.GONE){
                    bottom_sheet_tv.setVisibility(View.VISIBLE);
                    bottom_sheet_iv.setVisibility(View.GONE);              
                }
            }

            //BottomSheet滑动时的回调
            @Override
            public void onSlide(@NonNull View bottomSheet, float slideOffset) {
                
                if(bottomSheet.getTop()<2*design_bottom_sheet_bar.getHeight()){
                    design_bottom_sheet_bar.setVisibility(View.VISIBLE);
                    design_bottom_sheet_bar.setAlpha(slideOffset);
                    design_bottom_sheet_bar.setTranslationY(bottomSheet.getTop()-2*design_bottom_sheet_bar.getHeight());
                }
                else{
                    design_bottom_sheet_bar.setVisibility(View.INVISIBLE);
                }
            }
        });


***

源码 **[CoordinatorLayoutDemo](https://github.com/zhaochenpu/CoordinatorLayoutDemo)**

说实话，知乎中控件跟随用户滑动操作的隐藏和显示这种设计挺普遍的，但是有的时候感觉这个动画挺烦的，尤其是控件色彩明亮的时候，不自觉的就会吸引用户注意力。建议大家在设计交互的时候注意。

就是这些[]~(￣▽￣)~*


我的简书主页[http://www.jianshu.com/users/990c16f1edc0/latest_articles](http://www.jianshu.com/users/990c16f1edc0/latest_articles)
