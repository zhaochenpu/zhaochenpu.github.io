title: Android修改6.0系统文本选择菜单
date: 2017-02-22
tags: Android
---

因为我身患流行性绝症（拖延症），不知不觉中博客已经有半年没更新了 ( ⊙ o ⊙ )。

正好最近在写文本选择菜单的功能，就整理出了一篇文章，算是我与该死的拖延症展开的殊死搏斗吧。

本文主要写了在Android6.0+版本上，修改EditText、TextView的文本选择菜单内容和为其他APP提供自定义文本操作这两个功能。<!--more-->

![](http://upload-images.jianshu.io/upload_images/828721-8047b1d0575a1c14.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##1.修改EditText和TextView的文本选择菜单内容
Android官方控件中，EditText中的文字默认长按呼出文本选择菜单，而TextView需要设置android:textIsSelectable="true"。

我们修改文本选择菜单内容，只需要为TextView或者EditText设置setCustomSelectionActionModeCallback()方法，并且在方法里实现ActionMode.Callback()或ActionMode.Callback2()接口。

ActionMode.Callback()和ActionMode.Callback2()接口的主要内容相同，只是Callback2中多了一个onGetContentRect()方法，重写可以改变弹出菜单的位置。另外，Callback2需要判断sdk23及以上版本，Callback()不用，但是在6.0以下系统中实际是无效的。

我们先在res下的menu文件夹里新建一个菜单文件，我把它命名为selection_action_menu.xml，内容如下

        <?xml version="1.0" encoding="utf-8"?>
        <menu
     xmlns:android="http://schemas.android.com/apk/res/android">
        <item
            android:id="@+id/Informal22"
            android:title="自定义22" />
        <item
            android:id="@+id/Informal33"
            android:title="自定义33" />
     </menu>

然后我们完成文本选择菜单的修改

        ActionMode.Callback2 textSelectionActionModeCallback;
        if (android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.M) {
            textSelectionActionModeCallback = new ActionMode.Callback2() {
                @Override
                public boolean onCreateActionMode(ActionMode actionMode, Menu menu) {
                    MenuInflater menuInflater = actionMode.getMenuInflater();
                    menuInflater.inflate(R.menu.selection_action_menu,menu);
                    return true;//返回false则不会显示弹窗
                }

                @Override
                public boolean onPrepareActionMode(ActionMode actionMode, Menu menu) {
                    return false;
                }

                @Override
                public boolean onActionItemClicked(ActionMode actionMode, MenuItem menuItem) {
                    //根据item的ID处理点击事件
                    switch (menuItem.getItemId()){
                        case R.id.Informal22:
                            Toast.makeText(MainActivity.this, "点击的是22", Toast.LENGTH_SHORT).show();
                            actionMode.finish();//收起操作菜单
                            break;
                        case R.id.Informal33:
                            Toast.makeText(MainActivity.this, "点击的是33", Toast.LENGTH_SHORT).show();
                            actionMode.finish();
                            break;
                    }
                    return false;//返回true则系统的"复制"、"搜索"之类的item将无效，只有自定义item有响应
                }

                @Override
                public void onDestroyActionMode(ActionMode actionMode) {

                }

                @Override
                public void onGetContentRect(ActionMode mode, View view, Rect outRect) {
                    //可选  用于改变弹出菜单的位置
                    super.onGetContentRect(mode, view, outRect);
                }
            };
        }
![增加自定义item](http://upload-images.jianshu.io/upload_images/828721-37581382c6ff1e44.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![增加自定义item](http://upload-images.jianshu.io/upload_images/828721-b87101eb09e878d8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


上面这样写的话会有系统的“复制”、“全选”、“搜索”等item，以及其他APP提供的操作item，如果你（或者你们产品经理）傲娇的想要屏蔽所有非本APP自定义item，可以改成这样

      @Override
      public boolean onCreateActionMode(ActionMode actionMode, Menu menu) {
            return true;//返回false则不会显示弹窗
      }

      @Override
      public boolean onPrepareActionMode(ActionMode actionMode, Menu menu) {
             MenuInflater menuInflater = actionMode.getMenuInflater();
             menu.clear();
             menuInflater.inflate(R.menu.selection_action_menu,menu);
             return true;
      }


![只有自定义item](http://upload-images.jianshu.io/upload_images/828721-b74b3380261516ee.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


然后我们为TextView和EditText设置文本选择操作回调

      textView.setCustomSelectionActionModeCallback(textSelectionActionModeCallback);
      editText.setCustomSelectionActionModeCallback(textSelectionActionModeCallback);

EditText在还有一个插入操作菜单可以设置

      edittext.setCustomInsertionActionModeCallback(textSelectionActionModeCallback);

![EditText插入操作菜单](http://upload-images.jianshu.io/upload_images/828721-4cf44481bda0c3fb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

MIUI上小米重写了EditText和TextView中的文本选择菜单，上面这些代码修改无效。好吧，算你狠。 = =、



##2.为其他APP提供文本选择操作
一般剪切板、翻译、搜索、记事本类APP会提供这种功能，为用户提供便捷操作，同时也刷下自己的存在感。

我先建了一个继承自Activity的CustomTextProcessingActivity，简单写了下布局，我这因为拖延症的常见并发症——懒癌发作，代码就不贴出来了。

然后按照基本法到清单文件里去注册一下

    <activity android:name=".CustomTextProcessingActivity">
        <intent-filter>
            <action android:name="android.intent.action.PROCESS_TEXT"/>
            <category android:name="android.intent.category.DEFAULT"/>
            <data android:mimeType="text/plain"/>
        </intent-filter>
    </activity>

这里可以为activity自定义主题和标签，标签值就是文本选择操作菜单中出现的item名。默认就是APP名称"MY APPLICATION"。如果你想让这个操作只出现在本APP中，可以设置android:exported=”false”。

回到CustomTextProcessingActivity中，获取用户选择的文本只需要这样

     CharSequence text = getIntent().getCharSequenceExtra(Intent.EXTRA_PROCESS_TEXT);

我们还可以判断文本是否可编辑

     boolean isReadonly = getIntent().getBooleanExtra(Intent.EXTRA_PROCESS_TEXT_READONLY, false);

我们点击“MY APPLICATION”的时候，其实系统是通过是 startActivityForResult() 来启动的CustomTextProcessingActivity，所以我们可以返回一个文本。当选中的文本可编辑时（如EditText中的文字），系统会替换文字；不可编辑时，系统会通过toast的方式展示出来。

       //返回一个文本
       Intent intent = new Intent();
       intent.putExtra(Intent.EXTRA_PROCESS_TEXT, "这是替换的文字");
       setResult(RESULT_OK, intent);


![](http://upload-images.jianshu.io/upload_images/828721-71a9a5209693e008.gif?imageMogr2/auto-orient/strip)

这个界面写成弹窗的样式感觉更好，看大家的具体设计了。


在我一加的氢OS上，EditText在setCustomSelectionActionModeCallback()之后就只有第一部分中自定义的item以及系统的“复制”等基础item，本APP和其他的三方APP提供的操作item都无法显示。华为上是不显示第二部分中APP提供的操作item,其他的正常显示。而一加和华为上的TextView都是正常的。

至于小米，EditText和TextView不会显示三方APP提供的操作item。但是WebView中选择文字弹出来的还是原生文本选择操作菜单，有三方APP的操作item，~~估计是小米程序猿漏掉了~~。

***
这篇文章周末就开始写了，现在的时间是周三。嗯...还行，拖得时间不算太长...

欢迎拖延症病友交流病情。。。。
