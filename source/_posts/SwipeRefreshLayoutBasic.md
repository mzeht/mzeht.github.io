title: "SwipeRefreshLayoutBasic"
layout: post
categories: 
 - 移动开发
 - Android
date: 2016-01-30 00:20:12
author: mzeht
avatar: /images/favicon.png
tags: 
- Android 
- Matertal Design
---
# SwipeRefreshLayout配合Recyclerview实现下拉刷新列表
![image](https://github.com/mzeht/SwipeRefreshListDemo/blob/master/app/src/main/res/acess/sample.gif )

![image](https://github.com/mzeht/SwipeRefreshListDemo/blob/master/app/src/main/res/acess/click.gif )

SwipeRefreshLayout是google官方的刷新控件
Recyclerview则是未来取代listview，girdview等的新控件

.IDE android studio 1.5(2.o preview之前apk打包有bug，暂时不用了)

.Demo地址 <https://github.com/mzeht/SwipeRefreshListDemo> 

<!-- more -->
##.MainActivity
activity_main.xml


	<?xml version="1.0" encoding="utf-8"?>
	<android.support.design.widget.CoordinatorLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true"
    tools:context="com.example.mzeht.swiperefreshlistdemo.MainActivity">

    <android.support.design.widget.AppBarLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:theme="@style/AppTheme.AppBarOverlay">

        <android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            android:background="?attr/colorPrimary"
            app:popupTheme="@style/AppTheme.PopupOverlay"/>

    </android.support.design.widget.AppBarLayout>

    <include layout="@layout/content_main"/>

    <android.support.design.widget.FloatingActionButton
        android:id="@+id/fab"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="bottom|end"
        android:layout_margin="@dimen/fab_margin"
        android:src="@android:drawable/ic_dialog_email"/>

	</android.support.design.widget.CoordinatorLayout>
##.在其中的content_main.xml
SwipeRefreshLayout 嵌套一个子元素 （如有多个，可用FrameLayout封装成一个）

		<?xml version="1.0" encoding="utf-8"?>
	<RelativeLayout
   	 	xmlns:android="http://schemas.android.com/apk/res/android"
   	 xmlns:app="http://schemas.android.com/apk/res-	auto"
   	 xmlns:tools="http://schemas.android.com/tools"
   	 android:layout_width="match_parent"
    	android:layout_height="match_parent"
    	app:layout_behavior="@string/	appbar_scrolling_view_behavior"
    	tools:context="com.example.mzeht.swiperefreshlistdemo.MainActivity"
    	tools:showIn="@layout/activity_main">

    <android.support.v4.widget.SwipeRefreshLayout
        android:id="@+id/swiperefresh"
        android:layout_width="match_parent"
        android:layout_height="match_parent">
        <android.support.v7.widget.RecyclerView
            android:id="@+id/recycler"
            android:layout_width="match_parent"
            android:layout_height="match_parent">
        </android.support.v7.widget.RecyclerView>
    </android.support.v4.widget.SwipeRefreshLayout>
	</RelativeLayout>
	
	
##.RecyclerView的Item
*同经典的listviewItem，只是adapter略有区别

	<?xml version="1.0" encoding="utf-8"?>
	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              xmlns:tools="http://schemas.android.com/tools"
              android:orientation="horizontal"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
    android:id="@+id/ll">

    <ImageView
        android:id="@+id/image"
        android:src="@mipmap/ic_launcher"
        android:padding="5dp"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:focusable="false"
        />
    <TextView
        android:id="@+id/tv"
        android:layout_width="wrap_content"
        android:textSize="18sp"
        android:gravity="center"
        android:layout_height="match_parent"
        android:focusable="false"
        tools:text="ss"/>

	</LinearLayout>






	
##.RecyclerView的分割线
*RecyclerView 默认没有分割线，需要自定义一个，希望以后有默认可选，这里定义一个


	public class MyItemDecoration extends RecyclerView.ItemDecoration {
    private static final int[] ATTRS = {android.R.attr.listDivider};
    private Drawable mDivider;
    public MyItemDecoration(Context context)
    {
        TypedArray array = context.obtainStyledAttributes(ATTRS);
        // 获取分隔条
        mDivider = array.getDrawable(0);
        array.recycle();
    }
    @Override
    public void onDrawOver(Canvas c, RecyclerView parent, RecyclerView.State state)
    {
        super.onDrawOver(c, parent, state);
        int count = parent.getChildCount();
        int left = parent.getPaddingLeft();
        int right = parent.getWidth()-parent.getPaddingRight();
        for(int i = 0; i < count; i++)
        {
            View v = parent.getChildAt(i);
            RecyclerView.LayoutParams params = (RecyclerView.LayoutParams) v.getLayoutParams();
            int top = v.getBottom() + params.bottomMargin;
            int bottom = top + mDivider.getIntrinsicHeight();
            mDivider.setBounds(left,top,right,bottom);
            mDivider.draw(c);
        }
    }

	}
	
*MainActivity
	

		mRecyclerView= (RecyclerView) findViewById(R.id.recycler);
        mRefreshlayout= (SwipeRefreshLayout) findViewById(R.id.swiperefresh);
        mLinearLayoutManager = new LinearLayoutManager(this);
        mAdapter = new MainRecyclerAdapter(DataSource.generateData(20),MainActivity.this);
        mRecyclerView.setAdapter(mAdapter);

        //每个item高度一致，可设置为true，提高性能
        mRecyclerView.setHasFixedSize(true);
        mRecyclerView.setLayoutManager(mLinearLayoutManager);
        //分隔线
        mRecyclerView.addItemDecoration(new MyItemDecoration(this));
        mRefreshlayout.setOnRefreshListener(new SwipeRefreshLayout.OnRefreshListener() {
            @Override
            public void onRefresh() {

                new UpdateTask().execute();

            }
        });
        
   *开启异步任务模拟耗时操作
   
   	private class UpdateTask extends AsyncTask<Void,Void,List<String>>
    {
        @Override
        protected List<String> doInBackground(Void... params)
        {
            try
            {
                Thread.sleep(2000);
            } catch (InterruptedException e)
            {
                e.printStackTrace();
            }
            List<String> strings = new ArrayList<>();
            strings.add("新数据1");
            strings.add("新数据2");
            strings.add("新数据3");
            strings.add("新数据4");
            return strings;
        }
        @Override
        protected void onPostExecute(List<String> strings)
        {
            mAdapter.addItems(strings);
            //通知刷新完毕
            mRefreshlayout.setRefreshing(false);
            //滚动到列首部--->这是一个很方便的api，可以滑动到指定位置
            mRecyclerView.scrollToPosition(0);
        }
    }
    
    
    
##RecylerView的Item点击事件监听，内部各个子控件的监听
    *在设计复杂listview 的时候，往往有Item点击事件和各个子控件的事件需要监听
    *但是RecylerView没有提供OnItemOnclick的方法，需要我们自己去写
    首先定义一个外部接口
    
	```java
   
    public interface OnListClickListener {

        public void OnItemClick(View view,String data);


        public void OnItemTextClick(View view,String data);



        public void OnItemIconClick(View view ,String data);
	}
	
	
*适配Adapter

public class MainRecyclerAdapter extends RecyclerView.Adapter<MainRecyclerAdapter.ViewHolder> {

    private List<String> datas = null;
    private OnListClickListener mListener;

    public MainRecyclerAdapter(List<String> datas,OnListClickListener l)
    {
        this.datas = datas;
        this.mListener=l;
    }
    @Override
    public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType)
    {
        final View itemView = LayoutInflater.from(parent.getContext()).inflate(R.layout.recycler_item, parent, false);
	//        itemView.setOnClickListener(new View.OnClickListener()
	//        {
	//            @Override
	//            public void onClick(View v)
	//            {
	//                if(mListener != null)
	//                {
	//                    mListener.OnItemClick(v, (String) itemView.getTag());
	//                }
	//            }
	//        });
        ViewHolder viewHolder= new ViewHolder(itemView);




        return viewHolder;
    }
    @Override
    public void onBindViewHolder(final ViewHolder holder, final int position)
    {
        String s = datas.get(position);
        holder.bindData(s);
        holder.itemView.setTag(s);
        holder.mContent.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                mListener.OnItemTextClick(holder.mContent, (String) holder.itemView.getTag());
            }
        });

        holder.mImageView.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                mListener.OnItemIconClick(holder.mImageView, (String) holder.itemView.getTag());
            }
        });

        holder.itemView.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                mListener.OnItemClick(v, (String) holder.itemView.getTag());
            }
        });


    }
    @Override
    public int getItemCount()
    {
        return datas.size();
    }
    /**
     * 批量增加
     * */
    public void addItems(List<String> items)
    {
        if (items == null)
            return;
        this.datas.addAll(0, items);
        this.notifyItemRangeInserted(0, items.size());
    }

    static class ViewHolder extends RecyclerView.ViewHolder
    {
        private TextView mContent;
        private ImageView mImageView;
        public ViewHolder(View itemView)
        {
            super(itemView);
            mContent = (TextView) itemView.findViewById(R.id.tv);
            mImageView= (ImageView) itemView.findViewById(R.id.image);
        }
        public void bindData(String s)
        {
            if (s != null)
                mContent.setText(s);
        }

    }
	}
	
*通过对Item的根试图的监听，实现Item点击监听，不用设置最外层布局id，可以看出，Recyclerview强制对listview进行复用，对子控件的个性化定制在bind操作中进行，接口事件传递到Activity层，可以通过TAG传递数据，解耦合，如：



 	@Override
    	public void OnItemClick(View view, String data) {
        Toast.makeText(MainActivity.this, "来自Item的点击事件:" + data, Toast.LENGTH_SHORT).show();

    }

    @Override
    public void OnItemTextClick(View view, String data) {
        Toast.makeText(MainActivity.this, "来自text的点击事件:" + data, Toast.LENGTH_SHORT).show();

    }

    @Override
    public void OnItemIconClick(View view, String data) {
        Toast.makeText(MainActivity.this, "来自icon的点击事件:" + data, Toast.LENGTH_SHORT).show();

    }



   



	

