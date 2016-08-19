title: 这是一篇Android RecyclerView使用指南
date: 2016-06-18
tags: Android
---
RecyclerView从2014年发布到现在已经很长时间了，使用已经相当普遍。

本文主要介绍了RecyclerView的基础使用、自动加载更多数据、item的拖拽和划动删除。详细效果和使用请看我的demo;
<!--more-->

[本文RecyclerViewDemo](https://github.com/zhaochenpu/RecyclerViewDemo) 
![RecyclerView三种自带布局](http://upload-images.jianshu.io/upload_images/828721-d05ff584167469fa.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

本文图片接口来自干货集中营http://gank.io/api


##RecyclerView基础用法##

RecyclerView是support.v7包中的控件，可以说是ListView和GridView的增强升级版。

官方对RecyclerView的描述是（不翻译不是因为我英语差啊，真的）：
> A flexible view for providing a limited window into a large data set.

RecyclerView的用法和ListView类似。我用网格布局为例，先上代码再细说。

首先在布局中要创建这个控件：

        <?xml version="1.0" encoding="utf-8"?>
     <android.support.design.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/grid_coordinatorLayout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true"
    android:background="#ffffff"
    tools:context="com.zcp.recyclerviewddmo.GridActivity">
    <!--工具栏-->
    <android.support.design.widget.AppBarLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:theme="@style/AppTheme.AppBarOverlay">

        <android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            android:background="?attr/colorPrimary"
            />

    </android.support.design.widget.AppBarLayout>
    <!--下拉刷新控件-->
    <android.support.v4.widget.SwipeRefreshLayout
        android:id="@+id/grid_swipe_refresh"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior"
        >

        <!--本文主角-->
        <android.support.v7.widget.RecyclerView
            android:id="@+id/grid_recycler"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            />
    </android.support.v4.widget.SwipeRefreshLayout>
    </android.support.design.widget.CoordinatorLayout>

Activity的代码如下：

    public class GridActivity extends AppCompatActivity {

    private static RecyclerView recyclerview;
    private CoordinatorLayout coordinatorLayout;
    private GridAdapter mAdapter;
    private List<Meizi> meizis;
    private GridLayoutManager mLayoutManager;
    private int lastVisibleItem ;
    private int page=1;
    private ItemTouchHelper itemTouchHelper;
    private SwipeRefreshLayout swipeRefreshLayout;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_grid);
        Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);

        initView();//初始化布局
        setListener();//设置监听事件

        //执行加载数据
        new GetData().execute("http://gank.io/api/data/福利/10/1");

    }

    private void initView(){
        coordinatorLayout=(CoordinatorLayout)findViewById(R.id.grid_coordinatorLayout);

        recyclerview=(RecyclerView)findViewById(R.id.grid_recycler);
        mLayoutManager=new GridLayoutManager(GridActivity.this,3,GridLayoutManager.VERTICAL,false);//设置为一个3列的纵向网格布局
        recyclerview.setLayoutManager(mLayoutManager);

        swipeRefreshLayout=(SwipeRefreshLayout) findViewById(R.id.grid_swipe_refresh) ;
        //调整SwipeRefreshLayout的位置
        swipeRefreshLayout.setProgressViewOffset(false, 0,  (int) TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP, 24, getResources().getDisplayMetrics()));
    }

    private void setListener(){
        //swipeRefreshLayout刷新监听
        swipeRefreshLayout.setOnRefreshListener(new SwipeRefreshLayout.OnRefreshListener() {
            @Override
            public void onRefresh() {
                page=1;
                new GetData().execute("http://gank.io/api/data/福利/10/1");
            }
        });
        }


    private class GetData extends AsyncTask<String, Integer, String> {

        @Override
        protected void onPreExecute() {
            super.onPreExecute();
            //设置swipeRefreshLayout为刷新状态
            swipeRefreshLayout.setRefreshing(true);
        }

        @Override
        protected String doInBackground(String... params) {

            return MyOkhttp.get(params[0]);
        }

        protected void onPostExecute(String result) {
            super.onPostExecute(result);
            if(!TextUtils.isEmpty(result)){

                JSONObject jsonObject;
                Gson gson=new Gson();
                String jsonData=null;

                try {
                    jsonObject = new JSONObject(result);
                    jsonData = jsonObject.getString("results");
                } catch (JSONException e) {
                    e.printStackTrace();
                }
                if(meizis==null||meizis.size()==0){
                    meizis= gson.fromJson(jsonData, new TypeToken<List<Meizi>>() {}.getType());

                    Meizi pages=new Meizi();
                    pages.setPage(page);
                    meizis.add(pages);//在数据链表中加入一个用于显示页数的item
                }else{
                    List<Meizi>  more= gson.fromJson(jsonData, new TypeToken<List<Meizi>>() {}.getType());
                    meizis.addAll(more);

                    Meizi pages=new Meizi();
                    pages.setPage(page);
                    meizis.add(pages);//在数据链表中加入一个用于显示页数的item
                }

                if(mAdapter==null){
                    recyclerview.setAdapter(mAdapter = new GridAdapter(GridActivity.this,meizis));//recyclerview设置适配器

                    //实现适配器自定义的点击监听
                    mAdapter.setOnItemClickListener(new GridAdapter.OnRecyclerViewItemClickListener() {
                        @Override
                        public void onItemClick(View view) {
                            int position=recyclerview.getChildAdapterPosition(view);
                            SnackbarUtil.ShortSnackbar(coordinatorLayout,"点击第"+position+"个",SnackbarUtil.Info).show();
                           //彩色Snackbar工具类，请看我之前的文章《没时间解释了，快使用Snackbar!——Android Snackbar花式使用指南》http://www.jianshu.com/p/cd1e80e64311
                        }
                        @Override
                        public void onItemLongClick(View view) {
                           
                        }
                    });       
                }else{
                    //让适配器刷新数据
                    mAdapter.notifyDataSetChanged();
                }
            }
            //停止swipeRefreshLayout加载动画
            swipeRefreshLayout.setRefreshing(false);
        }
    }
    }


接下来我们看RecyclerView的适配器：

    public  class GridAdapter extends RecyclerView.Adapter<RecyclerView.ViewHolder> implements View.OnClickListener, View.OnLongClickListener {

    private Context mContext;
    private List<Meizi> datas;//数据
    
    //自定义监听事件
    public static interface OnRecyclerViewItemClickListener {
        void onItemClick(View view);
        void onItemLongClick(View view);
    }
    private OnRecyclerViewItemClickListener mOnItemClickListener = null;
    public void setOnItemClickListener(OnRecyclerViewItemClickListener listener) {
        mOnItemClickListener = listener;
    }

    //适配器初始化
    public GridAdapter(Context context,List<Meizi> datas) {
        mContext=context;
        this.datas=datas;
    }

    @Override
    public int getItemViewType(int position) {
        //判断item类别，是图还是显示页数（图片有URL）
        if (!TextUtils.isEmpty(datas.get(position).getUrl())) {
            return 0;
        } else {
            return 1;
        }
    }

    @Override
    public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType)
    {
        //根据item类别加载不同ViewHolder
        if(viewType==0){
            View view = LayoutInflater.from(mContext
                    ).inflate(R.layout.grid_meizi_item, parent,
                    false);//这个布局就是一个imageview用来显示图片
            MyViewHolder holder = new MyViewHolder(view);

           //给布局设置点击和长点击监听
            view.setOnClickListener(this);
            view.setOnLongClickListener(this);

            return holder;
        }else{
            MyViewHolder2 holder2=new MyViewHolder2(LayoutInflater.from(
                    mContext).inflate(R.layout.page_item, parent,
                    false));//这个布局就是一个textview用来显示页数
            return holder2;
        }

    }

    @Override
    public void onBindViewHolder(RecyclerView.ViewHolder holder, int position) {
     //将数据与item视图进行绑定，如果是MyViewHolder就加载网络图片，如果是MyViewHolder2就显示页数
        if(holder instanceof MyViewHolder){
            Picasso.with(mContext).load(datas.get(position).getUrl()).into(((MyViewHolder) holder).iv);//加载网络图片
        }else if(holder instanceof MyViewHolder2){
            ((MyViewHolder2) holder).tv.setText(datas.get(position).getPage()+"页");
        }

    }

    @Override
    public int getItemCount()
    {
        return datas.size();//获取数据的个数
    }

    //点击事件回调
    @Override
    public void onClick(View v) {
        if (mOnItemClickListener != null) {
            mOnItemClickListener.onItemClick(v);
        }
    }
    @Override
    public boolean onLongClick(View v) {
        if (mOnItemClickListener!= null) {
          mOnItemClickListener.onItemLongClick(v);
       }
        return false;
    }
    
    //自定义ViewHolder，用于加载图片
    class MyViewHolder extends RecyclerView.ViewHolder
    {
        private ImageButton iv;

        public MyViewHolder(View view)
        {
            super(view);
            iv = (ImageButton) view.findViewById(R.id.iv);
        }
    }
    //自定义ViewHolder，用于显示页数
    class MyViewHolder2 extends RecyclerView.ViewHolder
    {
        private TextView tv;

        public MyViewHolder2(View view)
        {
            super(view);
            tv = (TextView) view.findViewById(R.id.tv);
        }
    }

        //添加一个item
        public void addItem(Meizi meizi, int position) {
            meizis.add(position, meizi);
            notifyItemInserted(position);
            recyclerview.scrollToPosition(position);//recyclerview滚动到新加item处
        }

        //删除一个item
        public void removeItem(final int position) {
            meizis.remove(position);
            notifyItemRemoved(position);
        }
    }

数据类：

     public class Meizi {

    private String url;//图片地址
    private int page;//页数

    public String getUrl() {
        return url;
    }

    public void setUrl(String url) {
        this.url = url;
    }

    public int getPage() {
        return page;
    }

    public void setPage(int page) {
        this.page = page;
    }  
    }

RecyclerView的使用和ListView很像吧。

RecyclerView需要通过setLayoutManager()方法设置布局管理器，RecyclerView有三个默认布局管理器LinearLayoutManager、GridLayoutManager、StaggeredGridLayoutManager，都支持横向和纵向排列以及反向滑动。如果想把RecyclerView改为横向滑动，也可以通过调用
     
    mLayoutManager.setOrientation(GridLayoutManager.HORIZONTAL);


RecyclerView不像ListView一样提供item的点击监听，所以我们只能自己实现。RecyclerView的item点击事件监听可以向我本文中为item的view设置监听，也可以在recyclerView.addOnItemTouchListener里去判断手势来实现。

通过调用如下方法可以设置item加载或移除时的动画

    recyclerView.setItemAnimator(new DefaultItemAnimator());

更多的动画效果可以看别人写的[RecyclerViewItemAnimators ](https://github.com/gabrielemariotti/RecyclerViewItemAnimators)这个项目。

##自动加载更多##

要想实现滑动到列表某处时自动加载下一页（比如剩最后两个item时），可以通过对RecyclerView设置滑动监听，获取当前显示的最后一个item在适配器中的位置，如果该item的位置小于或等于适配器item总个数减2，就加载下一页数据。
GridLayoutManager和LinearLayoutManager时方法如下：

      //recyclerview滚动监听
        recyclerview.addOnScrollListener(new RecyclerView.OnScrollListener() {
            @Override
            public void onScrollStateChanged(RecyclerView recyclerView, int newState) {
                super.onScrollStateChanged(recyclerView, newState);
                //0：当前屏幕停止滚动；1时：屏幕在滚动 且 用户仍在触碰或手指还在屏幕上；2时：随用户的操作，屏幕上产生的惯性滑动；
                // 滑动状态停止并且剩余少于两个item时，自动加载下一页
                if (newState == RecyclerView.SCROLL_STATE_IDLE
                        && lastVisibleItem +2>=mLayoutManager.getItemCount()) {
                    new GetData().execute("http://gank.io/api/data/福利/10/"+(++page));
                }
            }

            @Override
            public void onScrolled(RecyclerView recyclerView, int dx, int dy) {
                super.onScrolled(recyclerView, dx, dy);
                //获取加载的最后一个可见视图在适配器的位置。
                lastVisibleItem = mLayoutManager.findLastVisibleItemPosition();
            }
        });

StaggeredGridLayoutManager因为item的位置是交错的，findLastVisibleItemPosition()方法返回的是一个数组，所以我们得先判断下这个数组的最大值，上面的onScrolled方法写成这样

        int[] positions= mLayoutManager.findLastVisibleItemPositions(null);
        lastVisibleItem =Math.max(positions[0],positions[1]);//根据StaggeredGridLayoutManager设置的列数来定
 ![自动加载下一页.gif](http://upload-images.jianshu.io/upload_images/828721-6af52fcfd0223f13.gif?imageMogr2/auto-orient/strip)               
[XRecyclerView](https://github.com/jianghejie/XRecyclerView)是别人写的一个库，实现了RecyclerView的下拉头和底部加载布局，需要的可以看下。

##Item拖拽和滑动删除##
ItemTouchHelper是一个处理RecyclerView的滑动删除和拖拽的辅助类，RecyclerView 的item拖拽移动和滑动删除就靠它来实现。

ItemTouchHelper的监听如下

        itemTouchHelper=new ItemTouchHelper(new ItemTouchHelper.Callback() {

            //用于设置拖拽和滑动的方向
            @Override
            public int getMovementFlags(RecyclerView recyclerView, RecyclerView.ViewHolder viewHolder) {
                int dragFlags=0,swipeFlags=0;
                if(recyclerView.getLayoutManager() instanceof StaggeredGridLayoutManager||recyclerView.getLayoutManager() instanceof GridLayoutManager){
                   //网格式布局有4个方向
                   dragFlags=ItemTouchHelper.UP|ItemTouchHelper.DOWN|ItemTouchHelper.LEFT|ItemTouchHelper.RIGHT;
                }else if(recyclerView.getLayoutManager() instanceof LinearLayoutManager){
                   //线性式布局有2个方向
                   dragFlags=ItemTouchHelper.UP|ItemTouchHelper.DOWN;
                   
                   swipeFlags = ItemTouchHelper.START|ItemTouchHelper.END; //设置侧滑方向为从两个方向都可以
                }
                return makeMovementFlags(dragFlags,swipeFlags);//swipeFlags 为0的话item不滑动
            }

            //长摁item拖拽时会回调这个方法
            @Override
            public boolean onMove(RecyclerView recyclerView, RecyclerView.ViewHolder viewHolder, RecyclerView.ViewHolder target) {
                int from=viewHolder.getAdapterPosition();
                int to=target.getAdapterPosition();
                Collections.swap(meizis,from,to);//交换数据链表中数据的位置
                mAdapter.notifyItemMoved(from,to);//更新适配器中item的位置
                return true;
            }

            @Override
            public void onSwiped(RecyclerView.ViewHolder viewHolder, int direction) {
            //这里处理滑动删除
            }

            @Override
            public boolean isLongPressDragEnabled() {
                return false;//返回true则为所有item都设置可以拖拽
            }
        });

itemTouchHelper需要与recyclerView绑定才有效果，在recyclerView初始化的时候调用

         itemTouchHelper.attachToRecyclerView(recyclerview);

因为我想要网格布局中的图片item可拖拽，而页数item不可拖拽，所以我isLongPressDragEnabled()方法返回的false，而在item的长点击事件监听中做具体处理。

           mAdapter.setOnItemClickListener(new GridAdapter.OnRecyclerViewItemClickListener() {
           @Override
           public void onItemClick(View view) {
           }

            @Override
            public void onItemLongClick(View view) {
                  itemTouchHelper.startDrag(recyclerview.getChildViewHolder(view));//设置拖拽item
               }
           });

如果你想为item设置拖拽和滑动时的响应动画效果，可以利用ItemTouchHelper的下面三个方法。用线性布局示例：

    //当item拖拽开始时调用
    @Override
    public void onSelectedChanged(RecyclerView.ViewHolder viewHolder, int actionState) {
    super.onSelectedChanged(viewHolder, actionState);
     if(actionState==ItemTouchHelper.ACTION_STATE_DRAG){
           viewHolder.itemView.setBackgroundColor(Color.LTGRAY);//拖拽时设置背景色为灰色
        }
    }

    //当item拖拽完成时调用
    @Override
    public void clearView(RecyclerView recyclerView, RecyclerView.ViewHolder viewHolder) {
    super.clearView(recyclerView, viewHolder);
     viewHolder.itemView.setBackgroundColor(Color.WHITE);//拖拽停止时设置背景色为白色
    }

     //当item视图变化时调用
     @Override
     public void onChildDraw(Canvas c, RecyclerView recyclerView, RecyclerView.ViewHolder viewHolder, float dX, float dY, int actionState, boolean isCurrentlyActive) {
     super.onChildDraw(c, recyclerView, viewHolder, dX, dY, actionState, isCurrentlyActive);
      //根据item滑动偏移的值修改item透明度。screenwidth是我提前获得的屏幕宽度
      viewHolder.itemView.setAlpha(1-Math.abs(dX)/screenwidth);
     }
![拖拽和滑动删除.gif](http://upload-images.jianshu.io/upload_images/828721-ec32cbb9616fff01.gif?imageMogr2/auto-orient/strip)


***

就是这些。　［］～（￣▽￣）～＊

[RecyclerViewDemo](https://github.com/zhaochenpu/RecyclerViewDemo) 

本文送给说“三百六十行，行行出码农”，然后来跟我抢饭碗的两位兄弟。

我知道你们都喜欢看妹子 →_→...

谁说程序员不能文艺？欢迎关注微信订阅号《核桃42》。推送的内容关于科技、设计、心理、人文，用好文补脑。
![核桃42](http://upload-images.jianshu.io/upload_images/828721-2b9cde7b3a350c73.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我的简书主页[http://www.jianshu.com/users/990c16f1edc0/latest_articles](http://www.jianshu.com/users/990c16f1edc0/latest_articles)