title: 没时间解释了，快使用Snackbar!
date: 2016-05-04
tags: Android
---
Snackbar是Android Support Design Library库中的一个控件，可以在屏幕底部快速弹出消息，比Toast更加好用。本文对原生Snackbar进行了修改，使其更加灵活。<!--more-->

本文是在[《Design Support Library第三部分：Snackbar样式》](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0714/3186.html)和[《Snackbar使用及其注意事项》](http://blog.csdn.net/jywangkeep_/article/details/46405301)两篇文章的启发下而来，首先对两篇文章的作者表示感谢。

Snackbar是Android Support Design Library库中的一个控件，可以在屏幕底部快速弹出消息，比Toast更加好用。本文对原生Snackbar进行了修改，使其更加灵活。

## 1.Snackbar基本介绍 ##

使用Snackbar要导入com.android.support:design库。

Snackbar显示在所有屏幕其它元素之上(屏幕最顶层)，同一时间只能显示一个snackbar。

Snackbar的基本使用很简单，与Toast类似。

    Snackbar.make(view, message_text, duration)
       .setAction(action_text, click_listener)
       .show();

make()方法是生成Snackbar的。Snackbar需要一个控件容器view用来容纳，官方推荐使用CoordinatorLayout来确保Snackbar和其他组件的交互，比如滑动取消Snackbar、Snackbar出现时FloatingActionButton上移。显示时间duration有三种类型LENGTH_SHORT、LENGTH_LONG和LENGTH_INDEFINITE。

setAction()方法可设置Snackbar右侧按钮，增加进行交互事件。如果不使用setAction()则只显示左侧message。

    Snackbar.make(coordinatorLayout,"这是massage", Snackbar.LENGTH_LONG).setAction("这是action", new View.OnClickListener() {
    	@Override
    	public void onClick(View v) {
        	Toast.makeText(MainActivity.this,"你点击了action",Toast.LENGTH_SHORT).show();
         }
     }).show();

下面这张图演示了上面代码所实现的效果：Snackbar长显示、点击Action弹出toast提示以及Snackbar在CoordinatorLayout中滑动取消。

![基础演示.gif](http://upload-images.jianshu.io/upload_images/828721-476620e85b863aa6.gif?imageMogr2/auto-orient/strip)


如果你想在Snackbar的显示时或消失时做些什么，可以调用Snackbar的setCallback()方法。

## 2.多彩Snackbar ##

Snackbar和Toast的默认样式都很单一，但是有时我们希望把不同类型信息区别显示，从而使用户更容易注意到提示信息。所以使Snackbar变色是一个好主意。

Snackbar的官方API只提供了setActionTextColor()这个方法修改Action的文字颜色，这怎么办？查源码吧，哪里不会点哪里。(><)

在源码中我们看到Snackbar中定义了一个继承自LinearLayout的内部类SnackbarLayout，Snackbar的样子就是由这个SnackbarLayout实现的。

SnackbarLayout中加载了R.layout.design_layout_snackbar_include布局文件，打开后看到下面这段代码（我把padding、margin的具体数值也打了出来）：

    <merge xmlns:android="http://schemas.android.com/apk/res/android">
    <TextView
            android:id="@+id/snackbar_text"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:paddingTop="14dp"
            android:paddingBottom="14dp"
            android:paddingLeft="12dp"
            android:paddingRight="12dp"
            android:textAppearance="@style/TextAppearance.Design.Snackbar.Message"
            android:maxLines="2"
            android:layout_gravity="center_vertical|left|start"
            android:ellipsize="end"
            android:textAlignment="viewStart"/>

    <Button
            android:id="@+id/snackbar_action"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginLeft="0dp"
            android:layout_marginStart="0dp"
            android:layout_gravity="center_vertical|right|end"
            android:paddingTop="14dp"
            android:paddingBottom="14dp"
            android:paddingLeft="12dp"
            android:paddingRight="12dp"
            android:visibility="gone"
            android:textColor="?attr/colorAccent"
            style="?attr/borderlessButtonStyle"/>
    </merge>

由命名可知，以snackbar_text为名的TextView就是Snackbar左侧的message。

好了，我们开始修改Snackbar的背景颜色和message字体颜色吧。

    public static void setSnackbarColor(Snackbar snackbar, int messageColor, int backgroundColor) {
        View view = snackbar.getView();//获取Snackbar的view
        if(view!=null){
            view.setBackgroundColor(backgroundColor);//修改view的背景色
            ((TextView) view.findViewById(R.id.snackbar_text)).setTextColor(messageColor);//获取Snackbar的message控件，修改字体颜色
        }
    }

很简单，没有几行代码。

本文最后提供的Snackbar封装类代码中定义了4种不同类型的信息：Info(妹子向你发来一条消息)、Confirm(妹子已收到你发出的消息)、Warning(妹子删除了你发出的消息)、Alert(妹子已将你拉黑)，分别用蓝色、绿色、橙色、红色来表示。

![消息信息.png](http://upload-images.jianshu.io/upload_images/828721-19a81f6031b04caa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![警告和错误信息演示.gif](http://upload-images.jianshu.io/upload_images/828721-d0268574ad9f793c.gif?imageMogr2/auto-orient/strip)


## 3.在Snackbar中增加图标 ##
> 
> **短文本**
> 
> 通常 Snackbar 的高度应该仅仅用于容纳所有的文本，而文本应该与执行的操作相关。Snackbar 中不能包含图标，操作只能以文本的形式存在。
> 
> **最多0-1个操作，不包含取消按钮**
> 
> 当一个动作发生的时候，应当符合提示框和可用性规则。当有2个或者2个以上的操作出现时，应该使用提示框而不是 Snackbar，即使其中的一个是取消操作。如果 Snackbar 中提示的操作重要到需要打断屏幕上正在进行的操作，那么理当使用提示框而非 Snackbar。

上面这段是谷歌 [Material Design设计规范](http://wiki.jikexueyuan.com/project/material-design/components/snackbars-and-toasts.html "Material Design中文版")中的话。

但是我就是想在Snackbar中加图标增加趣味性，引起用户注意怎么办？我就是想在Snackbar中放两个按钮进行可选非必要操作怎么办？我就是想整幺蛾子。︿(￣︶￣)︿

设计规范中的说法是有道理的，因为官方认为“Snackbar是一种针对操作的轻量级反馈机制”，做的麻烦了影响视觉感受。但是对于上述任性的开发者（或者是接了奇葩需求的苦逼开发者）我们也有解决方法。

前面我们提到过Snackbar的view是由SnackbarLayout实现的，而SnackbarLayout是继承自LinearLayout，那么我们新建一个布局添加进去不就行了么。(～o￣￣)～o...

    public static void SnackbarAddView(Snackbar snackbar,int layoutId,int index) {
        View snackbarview = snackbar.getView();//获取snackbar的View(其实就是SnackbarLayout)

        Snackbar.SnackbarLayout snackbarLayout=(Snackbar.SnackbarLayout)snackbarview;//将获取的View转换成SnackbarLayout

        View add_view = LayoutInflater.from(snackbarview.getContext()).inflate(layoutId,null);//加载布局文件新建View

        LinearLayout.LayoutParams p = new LinearLayout.LayoutParams(LinearLayout.LayoutParams.WRAP_CONTENT,LinearLayout.LayoutParams.WRAP_CONTENT);//设置新建布局参数

        p.gravity= Gravity.CENTER_VERTICAL;//设置新建布局在Snackbar内垂直居中显示

        snackbarLayout.addView(add_view,index,p);//将新建布局添加进snackbarLayout相应位置
    }

上面的代码中，如果我们不设置向Snackbar中添加的布局文件的布局参数，新布局会显示在Snackbar内的顶部。使用上述任性方法的时候要注意新加布局的大小和Snackbar内文字长度，Snackbar过大或过于花哨了可不好看。

下面是使用示例。我们先新建一个布局，暂时命名为snackbar_addview.xml,简单的放进了一个ImageView，图片就是android默认图标。

    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_gravity="center_vertical"
    >
    <ImageView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_vertical"
        android:src="@mipmap/ic_launcher"/>
    </LinearLayout>

然后在activity中写下snackbar的设置：

     Snackbar snackbar= Snackbar.make(coordinatorLayout,"这是massage", Snackbar.LENGTH_LONG);

     SnackbarUtil.setSnackbarColor(snackbar,SnackbarUtil.blue);

     SnackbarUtil.SnackbarAddView(snackbar,R.layout.snackbar_addview,0);

      snackbar.show();

![添加图标演示.gif](http://upload-images.jianshu.io/upload_images/828721-cc7664d0feb9f724.gif?imageMogr2/auto-orient/strip)


## 4.SnackbarUtil ##

我将我常用的Snackbar相关设置封装成了一个类，大家可以根据自己的需求使用。

    /**
	 * Created by 赵晨璞 on 2016/5/1.
	 */
    public class SnackbarUtil {

    public static final   int Info = 1;
    public static final  int Confirm = 2;
    public static final  int Warning = 3;
    public static final  int Alert = 4;


    public static  int red = 0xfff44336;
    public static  int green = 0xff4caf50;
    public static  int blue = 0xff2195f3;
    public static  int orange = 0xffffc107;

    /**
     * 短显示Snackbar，自定义颜色
     * @param view
     * @param message
     * @param messageColor
     * @param backgroundColor
     * @return
     */
    public static Snackbar ShortSnackbar(View view, String message, int messageColor, int backgroundColor){
        Snackbar snackbar = Snackbar.make(view,message, Snackbar.LENGTH_SHORT);
        setSnackbarColor(snackbar,messageColor,backgroundColor);
        return snackbar;
    }

    /**
     * 长显示Snackbar，自定义颜色
     * @param view
     * @param message
     * @param messageColor
     * @param backgroundColor
     * @return
     */
    public static Snackbar LongSnackbar(View view, String message, int messageColor, int backgroundColor){
        Snackbar snackbar = Snackbar.make(view,message, Snackbar.LENGTH_LONG);
        setSnackbarColor(snackbar,messageColor,backgroundColor);
        return snackbar;
    }

    /**
     * 自定义时常显示Snackbar，自定义颜色
     * @param view
     * @param message
     * @param messageColor
     * @param backgroundColor
     * @return
     */
    public static Snackbar IndefiniteSnackbar(View view, String message,int duration,int messageColor, int backgroundColor){
        Snackbar snackbar = Snackbar.make(view,message, Snackbar.LENGTH_INDEFINITE).setDuration(duration);
        setSnackbarColor(snackbar,messageColor,backgroundColor);
        return snackbar;
    }

    /**
     * 短显示Snackbar，可选预设类型
     * @param view
     * @param message
     * @param type
     * @return
     */
    public static Snackbar ShortSnackbar(View view, String message, int type){
        Snackbar snackbar = Snackbar.make(view,message, Snackbar.LENGTH_SHORT);
        switchType(snackbar,type);
        return snackbar;
    }

    /**
     * 长显示Snackbar，可选预设类型
     * @param view
     * @param message
     * @param type
     * @return
     */
    public static Snackbar LongSnackbar(View view, String message,int type){
        Snackbar snackbar = Snackbar.make(view,message, Snackbar.LENGTH_LONG);
        switchType(snackbar,type);
        return snackbar;
    }

    /**
     * 自定义时常显示Snackbar，可选预设类型
     * @param view
     * @param message
     * @param type
     * @return
     */
    public static Snackbar IndefiniteSnackbar(View view, String message,int duration,int type){
        Snackbar snackbar = Snackbar.make(view,message, Snackbar.LENGTH_INDEFINITE).setDuration(duration);
        switchType(snackbar,type);
        return snackbar;
    }

    //选择预设类型
    private static void switchType(Snackbar snackbar,int type){
        switch (type){
            case Info:
                setSnackbarColor(snackbar,blue);
                break;
            case Confirm:
                setSnackbarColor(snackbar,green);
                break;
            case Warning:
                setSnackbarColor(snackbar,orange);
                break;
            case Alert:
                setSnackbarColor(snackbar,Color.YELLOW,red);
                break;
        }
    }

    /**
     * 设置Snackbar背景颜色
     * @param snackbar
     * @param backgroundColor
     */
    public static void setSnackbarColor(Snackbar snackbar, int backgroundColor) {
        View view = snackbar.getView();
        if(view!=null){
            view.setBackgroundColor(backgroundColor);
        }
    }

    /**
     * 设置Snackbar文字和背景颜色
     * @param snackbar
     * @param messageColor
     * @param backgroundColor
     */
    public static void setSnackbarColor(Snackbar snackbar, int messageColor, int backgroundColor) {
        View view = snackbar.getView();
        if(view!=null){
            view.setBackgroundColor(backgroundColor);
            ((TextView) view.findViewById(R.id.snackbar_text)).setTextColor(messageColor);
        }
    }

    /**
     * 向Snackbar中添加view
     * @param snackbar
     * @param layoutId
     * @param index 新加布局在Snackbar中的位置
     */
    public static void SnackbarAddView( Snackbar snackbar,int layoutId,int index) {
        View snackbarview = snackbar.getView();
        Snackbar.SnackbarLayout snackbarLayout=(Snackbar.SnackbarLayout)snackbarview;

        View add_view = LayoutInflater.from(snackbarview.getContext()).inflate(layoutId,null);

        LinearLayout.LayoutParams p = new LinearLayout.LayoutParams( LinearLayout.LayoutParams.WRAP_CONTENT,LinearLayout.LayoutParams.WRAP_CONTENT);
        p.gravity= Gravity.CENTER_VERTICAL;

        snackbarLayout.addView(add_view,index,p);
    }

    }


简单的使用示例：

    SnackbarUtil.ShortSnackbar(coordinator,"妹子向你发来一条消息",SnackbarUtil.Info).show();

![消息演示.gif](http://upload-images.jianshu.io/upload_images/828721-9640c1cf8ad5dbeb.gif?imageMogr2/auto-orient/strip)


整出幺蛾子的使用示例：

     Snackbar snackbar= SnackbarUtil.ShortSnackbar(coordinator,"妹子删了你发出的消息",SnackbarUtil.Warning).setActionTextColor(Color.RED).setAction("再次发送", new View.OnClickListener() {
        @Override
     	public void onClick(View v) {
        	SnackbarUtil.LongSnackbar(coordinator,"妹子已将你拉黑",SnackbarUtil.Alert).setActionTextColor(Color.WHITE).show();
        }
     });

     SnackbarUtil.SnackbarAddView(snackbar,R.layout.snackbar_addview,0);

     SnackbarUtil.SnackbarAddView(snackbar,R.layout.snackbar_addview2,2);

      snackbar.show();
这个示例中调用了两次SnackbarAddView()方法向Snackbar中添加了两个不同的自定义布局，效果如下（不建议大家这么玩 _(:з」∠)＿ ）：

![添加多布局.gif](http://upload-images.jianshu.io/upload_images/828721-19e2a53bbe1b1c10.gif?imageMogr2/auto-orient/strip)


暂时就是这些。

***
谁说程序员不能文艺？欢迎关注微信订阅号《核桃42》。推送的内容关于科技、设计、心理、人文，用好文补脑。

![核桃42](http://upload-images.jianshu.io/upload_images/828721-2b9cde7b3a350c73.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我的简书主页[http://www.jianshu.com/users/990c16f1edc0/latest_articles](http://www.jianshu.com/users/990c16f1edc0/latest_articles)