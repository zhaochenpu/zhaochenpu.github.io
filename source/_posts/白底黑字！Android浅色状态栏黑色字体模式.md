title: 白底黑字！Android浅色状态栏黑色字体模式
date: 2016-05-07
tags: Android
---
MIUI、Flyme、android6.0状态栏字体图标深色模式。
<!--more-->

公司项目UI改版的时候，把大部分标题栏都设计成了白色，给我们的设计图是这个样子的：

![设计图.png](http://upload-images.jianshu.io/upload_images/828721-5c6558a077f83f3e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这状态栏布局和图标挺像Android,但是这白底黑字Android设计规范里可没有啊，于是我们开发的时候果断忽视这个状态栏了。

结果设计师拿着测试版过来问，你们安卓这个状态栏颜色怎么没改啊。

我：这个做不了。

设计师一脸质疑。

我：Android4.4以上系统版本可以修改状态栏颜色，但是只有小米的MIUI、魅族的Flyme和Android6.0以上系统可以把状态栏文字和图标换成深色，其他的系统状态栏文字都是白色的，换成浅色背景的话就看不到了。

设计师一脸懵逼不知所云。

后来看着这黑色状态栏白色标题栏实在难看,而且自己用的是MIUI，于是还是做了适配。

其实很多国内三方Android系统都有深色状态栏字体模式，但是目前只看到了小米和魅族公开了各自的实现方法，小米的支持MIUI V6以上版本，魅族的支持Flyme4.0以上版本。代码如下：

    /**
     * 设置状态栏图标为深色和魅族特定的文字风格，Flyme4.0以上
     * 可以用来判断是否为Flyme用户
     * @param window 需要设置的窗口
     * @param dark 是否把状态栏字体及图标颜色设置为深色
     * @return  boolean 成功执行返回true
     *
     */
    public static boolean FlymeSetStatusBarLightMode(Window window, boolean dark) {
        boolean result = false;
        if (window != null) {
            try {
                WindowManager.LayoutParams lp = window.getAttributes();
                Field darkFlag = WindowManager.LayoutParams.class
                        .getDeclaredField("MEIZU_FLAG_DARK_STATUS_BAR_ICON");
                Field meizuFlags = WindowManager.LayoutParams.class
                        .getDeclaredField("meizuFlags");
                darkFlag.setAccessible(true);
                meizuFlags.setAccessible(true);
                int bit = darkFlag.getInt(null);
                int value = meizuFlags.getInt(lp);
                if (dark) {
                    value |= bit;
                } else {
                    value &= ~bit;
                }
                meizuFlags.setInt(lp, value);
                window.setAttributes(lp);
                result = true;
            } catch (Exception e) {

            }
        }
        return result;
    }

    /**
     * 设置状态栏字体图标为深色，需要MIUIV6以上
     * @param window 需要设置的窗口
     * @param dark 是否把状态栏字体及图标颜色设置为深色
     * @return  boolean 成功执行返回true
     *
     */
    public static boolean MIUISetStatusBarLightMode(Window window, boolean dark) {
        boolean result = false;
        if (window != null) {
            Class clazz = window.getClass();
            try {
                int darkModeFlag = 0;
                Class layoutParams = Class.forName("android.view.MiuiWindowManager$LayoutParams");
                Field  field = layoutParams.getField("EXTRA_FLAG_STATUS_BAR_DARK_MODE");
                darkModeFlag = field.getInt(layoutParams);
                Method extraFlagField = clazz.getMethod("setExtraFlags", int.class, int.class);
                if(dark){
                    extraFlagField.invoke(window,darkModeFlag,darkModeFlag);//状态栏透明且黑色字体
                }else{
                    extraFlagField.invoke(window, 0, darkModeFlag);//清除黑色字体
                }
                result=true;
            }catch (Exception e){

            }
        }
        return result;
    }

![MIUI深色状态栏字体模式.png](http://upload-images.jianshu.io/upload_images/828721-5e1185afebeac409.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Flyme深色状态栏字体模式.png](http://upload-images.jianshu.io/upload_images/828721-52b873cb24dd7a99.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

官方在Android6.0中提供了亮色状态栏模式，配置只需一行代码：

      if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
                activity.getWindow().getDecorView().setSystemUiVisibility( View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN|View.SYSTEM_UI_FLAG_LIGHT_STATUS_BAR);
       }

![6.0原生深色状态栏字体模式.png](http://upload-images.jianshu.io/upload_images/828721-2c85f6fd8bc1202c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

网上有关状态栏颜色设置的文章很多，下面这段代码是我用的状态栏设置工具类，算是一个总结吧。

官方在4.4版本只提供了状态栏透明的api,大家普遍采用开源库[SystemBarTint](https://github.com/jgilfelt/SystemBarTint)对4.4版本状态栏颜色进行设置，请各位自行下载。

在设置状态栏全透明后，整个activity布局都会上移充满整个屏幕，如果你不想让布局上移的话就需要在根布局设置

       android:fitsSystemWindows="true"

以下为我的状态栏设置工具类：

    /**
 	* Created by 赵晨璞 
 	*/
	public class StatusBarUtil {

    /**
     * 修改状态栏为全透明
     * @param activity
     */
    @TargetApi(19)
    public static void transparencyBar(Activity activity){
        if(Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
            Window window = activity.getWindow();
            window.clearFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS
                    | WindowManager.LayoutParams.FLAG_TRANSLUCENT_NAVIGATION);
            window.getDecorView().setSystemUiVisibility(View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN
                    | View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION
                    | View.SYSTEM_UI_FLAG_LAYOUT_STABLE);
            window.addFlags(WindowManager.LayoutParams.FLAG_DRAWS_SYSTEM_BAR_BACKGROUNDS);
            window.setStatusBarColor(Color.TRANSPARENT);
            window.setNavigationBarColor(Color.TRANSPARENT);
        } else
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
            Window window =activity.getWindow();
            window.setFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS,
                    WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS);
        }
    }

   	/**
     * 修改状态栏颜色，支持4.4以上版本
     * @param activity
     * @param colorId
     */
    public static void setStatusBarColor(Activity activity,int colorId) {

        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
            Window window = activity.getWindow();
	//  	window.addFlags(WindowManager.LayoutParams.FLAG_DRAWS_SYSTEM_BAR_BACKGROUNDS);	
        	window.setStatusBarColor(activity.getResources().getColor(colorId));
        } else if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
			//使用SystemBarTint库使4.4版本状态栏变色，需要先将状态栏设置为透明
            transparencyBar(activity);
            SystemBarTintManager tintManager = new SystemBarTintManager(activity);
            tintManager.setStatusBarTintEnabled(true);
            tintManager.setStatusBarTintResource(colorId);
        }
    }

    /**
     *设置状态栏黑色字体图标，
     * 适配4.4以上版本MIUIV、Flyme和6.0以上版本其他Android
     * @param activity
     * @return 1:MIUUI 2:Flyme 3:android6.0
     */
    public static int StatusBarLightMode(Activity activity){
        int result=0;
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
            if(MIUISetStatusBarLightMode(activity.getWindow(), true)){
                result=1;
            }else if(FlymeSetStatusBarLightMode(activity.getWindow(), true)){
                result=2;
            }else if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
                activity.getWindow().getDecorView().setSystemUiVisibility( View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN|View.SYSTEM_UI_FLAG_LIGHT_STATUS_BAR);
                result=3;
            }
        }
        return result;
    }

    /**
     * 已知系统类型时，设置状态栏黑色字体图标。
     * 适配4.4以上版本MIUIV、Flyme和6.0以上版本其他Android
     * @param activity
     * @param type 1:MIUUI 2:Flyme 3:android6.0
     */
    public static void StatusBarLightMode(Activity activity,int type){
        if(type==1){
           MIUISetStatusBarLightMode(activity.getWindow(), true);
        }else if(type==2){
            FlymeSetStatusBarLightMode(activity.getWindow(), true);
        }else if(type==3){
            activity.getWindow().getDecorView().setSystemUiVisibility( View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN|View.SYSTEM_UI_FLAG_LIGHT_STATUS_BAR);
        }

    }
    
    /**
     * 清除MIUI或flyme或6.0以上版本状态栏黑色字体
     */
    public static void StatusBarDarkMode(Activity activity,int type){
        if(type==1){
            MIUISetStatusBarLightMode(activity.getWindow(), false);
        }else if(type==2){
            FlymeSetStatusBarLightMode(activity.getWindow(), false);
        }else if(type==3){
            activity.getWindow().getDecorView().setSystemUiVisibility(View.SYSTEM_UI_FLAG_VISIBLE);
        }

    }


    /**
     * 设置状态栏图标为深色和魅族特定的文字风格
     * 可以用来判断是否为Flyme用户
     * @param window 需要设置的窗口
     * @param dark 是否把状态栏字体及图标颜色设置为深色
     * @return  boolean 成功执行返回true
     *
     */
    public static boolean FlymeSetStatusBarLightMode(Window window, boolean dark) {
        boolean result = false;
        if (window != null) {
            try {
                WindowManager.LayoutParams lp = window.getAttributes();
                Field darkFlag = WindowManager.LayoutParams.class
                        .getDeclaredField("MEIZU_FLAG_DARK_STATUS_BAR_ICON");
                Field meizuFlags = WindowManager.LayoutParams.class
                        .getDeclaredField("meizuFlags");
                darkFlag.setAccessible(true);
                meizuFlags.setAccessible(true);
                int bit = darkFlag.getInt(null);
                int value = meizuFlags.getInt(lp);
                if (dark) {
                    value |= bit;
                } else {
                    value &= ~bit;
                }
                meizuFlags.setInt(lp, value);
                window.setAttributes(lp);
                result = true;
            } catch (Exception e) {

            }
        }
        return result;
    }

    /**
     * 设置状态栏字体图标为深色，需要MIUIV6以上
     * @param window 需要设置的窗口
     * @param dark 是否把状态栏字体及图标颜色设置为深色
     * @return  boolean 成功执行返回true
     *
     */
    public static boolean MIUISetStatusBarLightMode(Window window, boolean dark) {
        boolean result = false;
        if (window != null) {
            Class clazz = window.getClass();
            try {
                int darkModeFlag = 0;
                Class layoutParams = Class.forName("android.view.MiuiWindowManager$LayoutParams");
                Field  field = layoutParams.getField("EXTRA_FLAG_STATUS_BAR_DARK_MODE");
                darkModeFlag = field.getInt(layoutParams);
                Method extraFlagField = clazz.getMethod("setExtraFlags", int.class, int.class);
                if(dark){
                    extraFlagField.invoke(window,darkModeFlag,darkModeFlag);//状态栏透明且黑色字体
                }else{
                    extraFlagField.invoke(window, 0, darkModeFlag);//清除黑色字体
                }
                result=true;
            }catch (Exception e){

            }
        }
        return result;
    }

	}


适配浅色状态栏深色字体的时候发现底层版本为Android6.0.1的MIUI7.1系统不支持View.SYSTEM_UI_FLAG_LIGHT_STATUS_BAR设置，还是得用MIUI自己的深色字体方法。

三方Android系统对底层改的挺乱的，也是让人头痛。O__O 

***
神马？写了这么多，你就说了句图里的妹子挺好看的。。。

图片是随手从网上找的。。。

谁说程序员不能文艺？欢迎关注微信订阅号《核桃42》。推送的内容关于科技、设计、心理、人文，用好文补脑。
![核桃42](http://upload-images.jianshu.io/upload_images/828721-2b9cde7b3a350c73.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我的简书主页[http://www.jianshu.com/users/990c16f1edc0/latest_articles](http://www.jianshu.com/users/990c16f1edc0/latest_articles)