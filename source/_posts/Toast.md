title: Android Toast花式使用
date: 2016-08-11
tags: Android
---
![](http://upload-images.jianshu.io/upload_images/828721-e0fc168c7e6ce054.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

之前写过一篇[没时间解释了，快使用Snackbar!——Android Snackbar花式使用指南](http://www.jianshu.com/p/cd1e80e64311)。Toast的自定义使用原理与其类似。<!--more-->


## 1.Toast源码分析 ##
老规矩，我们先去看Toast的源码。

Toast有两种显示布局方式，一种最常见调用Toast.makeText(),看源码是这样写的

    public static Toast makeText(Context context, CharSequence text, @Duration int duration) {
    Toast result = new Toast(context);
    
    LayoutInflater inflate = (LayoutInflater)
    context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
    View v = inflate.inflate(com.android.internal.R.layout.transient_notification, null);
    TextView tv = (TextView)v.findViewById(com.android.internal.R.id.message);
    tv.setText(text);
    
    result.mNextView = v;
    result.mDuration = duration;
    
    return result;
    }

transient_notification这个布局文件代码是这样的

    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:background="?android:attr/toastFrameBackground">
    
    <TextView
    android:id="@android:id/message"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_weight="1"
    android:layout_gravity="center_horizontal"
    android:textAppearance="@style/TextAppearance.Toast"
    android:textColor="@color/bright_foreground_dark"
    android:shadowColor="#BB000000"
    android:shadowRadius="2.75"
    />
    
    </LinearLayout>

那么我们想要修改Toast的文字消息样式，其实就是修改Toast根布局和message这个TextView。

Toast的另外一种显示模式就是自定义布局显示。这个方法不调用Toast.makeText()方法，而是new一个Toast对象，然后调用setView()方法。当然自定义布局就不会加载transient_notification布局了。

## 2.实现花式Toast ##

先给大家看下我封装的工具类ToastUtil。

    import android.content.Context;
    import android.view.View;
    import android.widget.LinearLayout;
    import android.widget.TextView;
    import android.widget.Toast;

    /**
     * Created by 赵晨璞 on 2016/8/11.
     */
    public class ToastUtil {

    private  Toast toast;
    private LinearLayout toastView;

    /**
     * 修改原布局的Toast
     */
    public ToastUtil() {

    }

    /**
     * 完全自定义布局Toast
     * @param context
     * @param view
     */
    public ToastUtil(Context context, View view,int duration){
        toast=new Toast(context);
        toast.setView(view);
        toast.setDuration(duration);
    }

    /**
     * 向Toast中添加自定义view
     * @param view
     * @param postion
     * @return
     */
    public  ToastUtil addView(View view,int postion) {
        toastView = (LinearLayout) toast.getView();
        toastView.addView(view, postion);

        return this;
    }

    /**
     * 设置Toast字体及背景颜色
     * @param messageColor
     * @param backgroundColor
     * @return
     */
    public ToastUtil setToastColor(int messageColor, int backgroundColor) {
        View view = toast.getView();
        if(view!=null){
           TextView message=((TextView) view.findViewById(android.R.id.message));
            message.setBackgroundColor(backgroundColor);
            message.setTextColor(messageColor);
        }
        return this;
    }

    /**
     * 设置Toast字体及背景
     * @param messageColor
     * @param background
     * @return
     */
    public ToastUtil setToastBackground(int messageColor, int background) {
        View view = toast.getView();
        if(view!=null){
            TextView message=((TextView) view.findViewById(android.R.id.message));
            message.setBackgroundResource(background);
            message.setTextColor(messageColor);
        }
        return this;
    }

    /**
     * 短时间显示Toast
     */
    public  ToastUtil Short(Context context, CharSequence message){
        if(toast==null||(toastView!=null&&toastView.getChildCount()>1)){
            toast= Toast.makeText(context, message, Toast.LENGTH_SHORT);
            toastView=null;
        }else{
            toast.setText(message);
            toast.setDuration(Toast.LENGTH_SHORT);
        }
        return this;
    }

    /**
     * 短时间显示Toast
     */
    public ToastUtil Short(Context context, int message) {
        if(toast==null||(toastView!=null&&toastView.getChildCount()>1)){
            toast= Toast.makeText(context, message, Toast.LENGTH_SHORT);
            toastView=null;
        }else{
            toast.setText(message);
            toast.setDuration(Toast.LENGTH_SHORT);
        }
      return this;
    }

    /**
     * 长时间显示Toast
     */
    public ToastUtil Long(Context context, CharSequence message){
        if(toast==null||(toastView!=null&&toastView.getChildCount()>1)){
            toast= Toast.makeText(context, message, Toast.LENGTH_LONG);
            toastView=null;
        }else{
            toast.setText(message);
            toast.setDuration(Toast.LENGTH_LONG);
        }
        return this;
    }

    /**
     * 长时间显示Toast
     *
     * @param context
     * @param message
     */
    public ToastUtil Long(Context context, int message) {
        if(toast==null||(toastView!=null&&toastView.getChildCount()>1)){
            toast= Toast.makeText(context, message, Toast.LENGTH_LONG);
            toastView=null;
        }else{
            toast.setText(message);
            toast.setDuration(Toast.LENGTH_LONG);
        }
        return this;
    }

    /**
     * 自定义显示Toast时间
     *
     * @param context
     * @param message
     * @param duration
     */
    public ToastUtil Indefinite(Context context, CharSequence message, int duration) {
        if(toast==null||(toastView!=null&&toastView.getChildCount()>1)){
            toast= Toast.makeText(context, message,duration);
            toastView=null;
        }else{
            toast.setText(message);
            toast.setDuration(duration);
        }
         return this;
    }

    /**
     * 自定义显示Toast时间
     *
     * @param context
     * @param message
     * @param duration
     */
    public ToastUtil Indefinite(Context context, int message, int duration) {
        if(toast==null||(toastView!=null&&toastView.getChildCount()>1)){
            toast= Toast.makeText(context, message,duration);
            toastView=null;
        }else{
            toast.setText(message);
            toast.setDuration(duration);
        }
        return this;
    }

    /**
     * 显示Toast
     * @return
     */
    public ToastUtil show (){
        toast.show();

        return this;
    }

    /**
     * 获取Toast
     * @return
     */
    public Toast getToast(){
        return toast;
    }
    }

修改Toast背景色的使用法方法如下：

    ToastUtil toastUtil=new ToastUtil();
    toastUtil.Short(MainActivity.this,"自定义message字体、背景色").setToastColor(Color.WHITE, getResources().getColor(R.color.colorAccent)).show();

![修改Toast背景色](http://upload-images.jianshu.io/upload_images/828721-66195248a10c3434.gif?imageMogr2/auto-orient/strip)
方形的Toast看上去有些呆板，我自定义了一个名为toast_radius.xml的背景，代码如下：

    <?xml version="1.0" encoding="utf-8"?>
    <shape
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle">
    <!-- 填充的颜色 -->
    <solid android:color="#ffc107" />
    
    <!-- android:radius 弧形的半径 -->
    <corners android:radius="20dip" />
    
    </shape>

然后上面设置背景的代码改成:

    toastUtil.Short(MainActivity.this,"自定义message字体颜色和背景").setToastBackground(Color.WHITE,R.drawable.toast_radius).show();

![修改了背景的Toast](http://upload-images.jianshu.io/upload_images/828721-575d63beed215b00.gif?imageMogr2/auto-orient/strip)

虽然官方认为Toast和Snackbar都应该是短文本的形式，不能包含图标，但是个人感觉加上图标还是挺好玩的...

向Toast中添加图标可以这样:

     ImageView toastImage = new ImageView(getApplicationContext());
     toastImage.setImageResource(R.mipmap.ic_launcher);
     toastUtil.Short(MainActivity.this,"向Toast添加了一个ImageView").setToastBackground(Color.WHITE,R.drawable.toast_radius).addView(toastImage,0).show();

![添加图标的Toast](http://upload-images.jianshu.io/upload_images/828721-ada7364a24c915ae.gif?imageMogr2/auto-orient/strip)

如果你想要Toast显示自定义的布局，可以这样，我的image布局文件中中只有一个默认图标的ImageView

     View view = LayoutInflater.from(MainActivity.this).inflate(R.layout.image,null);
     new ToastUtil(MainActivity.this,view,Toast.LENGTH_SHORT).show();

![自定义布局Toast](http://upload-images.jianshu.io/upload_images/828721-f95a07f3fe9144af.gif?imageMogr2/auto-orient/strip)
大家都知道，连续触发Toast的show()方法的时候，Toast就会排着队连续展示，感觉上不太友好。所以我先判断了toast是否没被创建或者是否被添加了额外的view，如果是的话就重新生成一个toast对象；如果否的话就只修改message文字和显示时间。

![Toast布局修改，不排队显示](http://upload-images.jianshu.io/upload_images/828721-ccb92e45086fe98a.gif?imageMogr2/auto-orient/strip)

我这个工具类不是完全体，大家再根据自己项目的具体需求进行修改。 []~(￣▽￣)~*

***

谁说程序员不能文艺？欢迎关注微信订阅号《核桃42》。推送的内容关于科技、设计、心理、人文，用好文补脑。

![核桃42](http://upload-images.jianshu.io/upload_images/828721-2b9cde7b3a350c73.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我的简书主页[http://www.jianshu.com/users/990c16f1edc0/latest_articles](http://www.jianshu.com/users/990c16f1edc0/latest_articles)