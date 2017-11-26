title: ListView使用技巧
categories: 
  - 移动开发
  - AndroidHero
tags:
  - Android
  - ListView
date: 2016-10-10 22:20:33
author: mzeht
avatar: /images/favicon.png
---

本来想跳过的，看了下书，实在是太熟悉了，待办，先占个坑
<!-- more -->
# 常用优化技巧

## 使用ViewHolder

```
public class NotifyAdapter extends BaseAdapter {

    private List<String> mData;
    private LayoutInflater mInflater;

    public NotifyAdapter(Context context, List<String> data) {
        this.mData = data;
        mInflater = LayoutInflater.from(context);
    }

    @Override
    public int getCount() {
        return mData.size();
    }

    @Override
    public Object getItem(int position) {
        return mData.get(position);
    }

    @Override
    public long getItemId(int position) {
        return position;
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        ViewHolder holder = null;
        // 判断是否缓存
        if (convertView == null) {
            holder = new ViewHolder();
            // 通过LayoutInflater实例化布局
            convertView = mInflater.inflate(R.layout.notify_item, null);
            holder.img = (ImageView) convertView.findViewById(R.id.imageView);
            holder.title = (TextView) convertView.findViewById(R.id.textView);
            convertView.setTag(holder);
        } else {
            // 通过tag找到缓存的布局
            holder = (ViewHolder) convertView.getTag();
        }
        // 设置布局中控件要显示的视图
        holder.img.setBackgroundResource(R.mipmap.ic_launcher);
        holder.title.setText(mData.get(position));
        return convertView;
    }

    public final class ViewHolder {
        public ImageView img;
        public TextView title;
    }
}	

```


## UI设置


```
//分割线
android:divider="@android:color/darker_gray"
android:dividerHeight="10dp"
//设置为透明
android:divider="@null"
//取消滚动条
android:scrollbars="none"
取消item点击效果
android:listSelector="@android:color/transparent"
```

## 设置需要显示的Item

-------

第几项置顶显示
ListView.setSelection(n);
//特殊用法,显示出数据的最后一项
 mListView.setSelection(mData.size() - 1);
 [参考](http://blog.csdn.net/szyangzhen/article/details/47972509)
 
 
-------
平滑移动

```
//        mListView.smoothScrollBy(1000,1000);
//        mListView.smoothScrollByOffset(15);
//        mListView.smoothScrollToPosition(15);
```

## 动态修改数据

```

public class NotifyTest extends Activity {

    private List<String> mData;
    private ListView mListView;
    private NotifyAdapter mAdapter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.notify);
        mData = new ArrayList<String>();
        for (int i = 0; i < 20; i++) {
            mData.add("" + i);
        }
        mListView = (ListView) findViewById(R.id.listView);
        mAdapter = new NotifyAdapter(this, mData);
        mListView.setAdapter(mAdapter);
        for (int i = 0; i < mListView.getChildCount(); i++) {
            View view = mListView.getChildAt(i);
        }
    }

    public void btnAdd(View view) {
        mData.add("new");
        mAdapter.notifyDataSetChanged();
        mListView.setSelection(mData.size() - 1);
//        mListView.smoothScrollBy(1000,1000);
//        mListView.smoothScrollByOffset(15);
//        mListView.smoothScrollToPosition(15);
    }
}
```
**传入的List对象要一致**

## 遍历所有Item

```
for (int i = 0; i < mListView.getChildCount(); i++) {
            View view = mListView.getChildAt(i);
        }

```

## 处理空ListView

```
listView.setEmptyView(findViewById(R.id.empty_view))
```


## ListView滑动监听

### OnTouchListener
有很多事件类型

```
mListView.setOnTouchListener(new View.OnTouchListener() {
            @Override
            public boolean onTouch(View v, MotionEvent event) {

                switch (event.getAction()) {
                    case MotionEvent.ACTION_DOWN:
                        //触摸时操作

                        break;

                    case MotionEvent.ACTION_MOVE:
                        //移动时操作
                        break;

                    case MotionEvent.ACTION_UP:
                        //移动时操作
                        break;
                }
                return false;
            }
        });

```


### OnScrollListener


```
 mListView.setOnScrollListener(new AbsListView.OnScrollListener() {
            @Override
            public void onScrollStateChanged(AbsListView view, int scrollState) {

                switch (scrollState){
                    case SCROLL_STATE_IDLE:
                        //滑动停止时
                        Log.d("test","SCROLL_STATE_IDLE");
                        break;
                    case SCROLL_STATE_TOUCH_SCROLL:
                        // 正在滚动
                        Log.d("test","SCROLL_STATE_TOUCH_SCROLL");
                        break;
                    case SCROLL_STATE_FLING:
                        //手指抛动时，即手指用力滑动
                        Log.d("test","SCROLL_STATE_FLING");
                        break;
                }

            }

            /**
             *
             * @param view
             * @param firstVisibleItem 当前能看见的第一个item的id（从0开始），包括没有完整显示的item
             * @param visibleItemCount 当前能看见的item总数
             * @param totalItemCount 整个listview的item总数
             */
            @Override
            public void onScroll(AbsListView view, int firstVisibleItem, int visibleItemCount, int totalItemCount) {
                    //滚动时一直调用
                Log.d("test","onScroll");

                /**
                 * 用法1 当前可视的第一个item的id加上当前可视的item数量等于listview总数的时候，可以判断滚动到了最后一行
                 *
                 */
                if (firstVisibleItem+visibleItemCount==totalItemCount&&totalItemCount>0){
                    //滚动到了最后一行
                }

                /**
                 * 判断滚动的方向
                 */
                Log.d("判断前",firstVisibleItem+"/"+lastVisableItemPostion);
                if (firstVisibleItem>lastVisableItemPostion){
                    //上滑
                    Log.d("test", "上滑");
                }else if (firstVisibleItem<lastVisableItemPostion){
                    //下滑
                    Log.d("test", "下滑");
                }

                lastVisableItemPostion=firstVisibleItem;
                Log.d("判断后",firstVisibleItem+"/"+lastVisableItemPostion);
            }
        });
```
 
 
###  其他方法


```
  /**
         * 获得可见的item的位置
         */

        mListView.getFirstVisiblePosition();
        mListView.getLastVisiblePosition();
```

## ListView常用拓展
-------------

### 让ListView具有弹性


```
public class FlexibleListView extends ListView {

    private static int mMaxOverDistance = 50;
    private Context mContext;

    public FlexibleListView(Context context, AttributeSet attrs,
                            int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        this.mContext = context;
        initView();
    }

    public FlexibleListView(Context context, AttributeSet attrs) {
        super(context, attrs);
        this.mContext = context;
        initView();
    }

    public FlexibleListView(Context context) {
        super(context);
        this.mContext = context;
        initView();
    }

    private void initView() {
        DisplayMetrics metrics = mContext.getResources().getDisplayMetrics();
        float density = metrics.density;
        mMaxOverDistance = (int) (density * mMaxOverDistance);
    }
	
	
	 /**
     * 滑动到边缘时调用的方法
     * @param deltaX
     * @param deltaY
     * @param scrollX
     * @param scrollY
     * @param scrollRangeX
     * @param scrollRangeY
     * @param maxOverScrollX
     * @param maxOverScrollY
     * Number of pixels to overscroll by in either direction
     *          along the Y axis.
     * @param isTouchEvent
     * @return
     */
    @Override
    protected boolean overScrollBy(int deltaX, int deltaY,
                                   int scrollX, int scrollY,
                                   int scrollRangeX, int scrollRangeY,
                                   int maxOverScrollX, int maxOverScrollY,
                                   boolean isTouchEvent) {
        return super.overScrollBy(deltaX, deltaY,
                scrollX, scrollY,
                scrollRangeX, scrollRangeY,
                maxOverScrollX, mMaxOverDistance,
                isTouchEvent);
    }
}
```

### 自动显示，隐藏的ListView
初始化操作

```
/获取系统认为的最低滑动距离
        mTouchSlop = ViewConfiguration.get(this).getScaledTouchSlop();
        mToolbar = (Toolbar) findViewById(R.id.toolbar);
        mListView = (ListView) findViewById(R.id.listview);
        for (int i = 0; i < mStr.length; i++) {
            mStr[i] = "Item " + i;
        }

        //Listview添加头部View
        View header = new View(this);
        header.setLayoutParams(new AbsListView.LayoutParams(
                AbsListView.LayoutParams.MATCH_PARENT,
                (int) getResources().getDimension(
                        R.dimen.abc_action_bar_default_height_material)));
        mListView.addHeaderView(header);
        mListView.setAdapter(new ArrayAdapter<String>(
                ScrollHideListView.this,
                android.R.layout.simple_expandable_list_item_1,
                mStr));
        mListView.setOnTouchListener(myTouchListener);
```

设置监听器


```
View.OnTouchListener myTouchListener = new View.OnTouchListener() {
        @Override
        public boolean onTouch(View v, MotionEvent event) {
            switch (event.getAction()) {
                case MotionEvent.ACTION_DOWN:
                    mFirstY = event.getY();
                    break;
                case MotionEvent.ACTION_MOVE:
                    mCurrentY = event.getY();
                    if (mCurrentY - mFirstY > mTouchSlop) {
                        direction = 0;// down
                    } else if (mFirstY - mCurrentY > mTouchSlop) {
                        direction = 1;// up
                    }
                    if (direction == 1) {
                        if (mShow) {
                            toolbarAnim(1);//hide
                            mShow = !mShow;
                        }
                    } else if (direction == 0) {
                        if (!mShow) {
                            toolbarAnim(0);//show
                            mShow = !mShow;
                        }
                    }
                    break;
                case MotionEvent.ACTION_UP:
                    break;
            }
            return false;
        }
    };
```

动画控制


```

private ObjectAnimator mAnimator;

private void toolbarAnim(int flag) {
        if (mAnimator != null && mAnimator.isRunning()) {
            mAnimator.cancel();
        }
        if (flag == 0) {
            mAnimator = ObjectAnimator.ofFloat(mToolbar,
                    "translationY", mToolbar.getTranslationY(), 0);
        } else {
            mAnimator = ObjectAnimator.ofFloat(mToolbar,
                    "translationY", mToolbar.getTranslationY(),
                    -mToolbar.getHeight());
        }
        mAnimator.start();
    }
```

### 动态改变Listview布局

这里以点击某一项listview为例子


```
public class FocusListViewAdapter extends BaseAdapter {

    private List<String> mData;
    private Context mContext;
    private int mCurrentItem = 0;

    public FocusListViewAdapter(Context context, List<String> data) {
        this.mContext = context;
        this.mData = data;
    }

    public int getCount() {
        return mData.size();
    }

    public Object getItem(int position) {
        return mData.get(position);
    }

    public long getItemId(int position) {
        return position;
    }

    /**
     *
     * @param position
     * @param convertView
     * @param parent
     * @return
     * 绘制View的方法中对记录的position进行区别处理
     * 初次绘制时调用
     */
    public View getView(int position, View convertView, ViewGroup parent) {
        LinearLayout layout = new LinearLayout(mContext);
        layout.setOrientation(LinearLayout.VERTICAL);
        if (mCurrentItem == position) {
            layout.addView(addFocusView(position));
        } else {
            layout.addView(addNormalView(position));
        }
        return layout;
    }


    //暴露动态改变记录position的方法
    public void setCurrentItem(int currentItem) {
        this.mCurrentItem = currentItem;
    }


    //加载布局1
    private View addFocusView(int i) {
        ImageView iv = new ImageView(mContext);
        iv.setImageResource(R.mipmap.ic_launcher);
        return iv;
    }

    //加载布局2
    private View addNormalView(int i) {
        LinearLayout layout = new LinearLayout(mContext);
        layout.setOrientation(LinearLayout.HORIZONTAL);
        ImageView iv = new ImageView(mContext);
        iv.setImageResource(R.drawable.in_icon);
        layout.addView(iv, new LinearLayout.LayoutParams(
                LinearLayout.LayoutParams.WRAP_CONTENT,
                LinearLayout.LayoutParams.WRAP_CONTENT));
        TextView tv = new TextView(mContext);
        tv.setText(mData.get(i));
        layout.addView(tv, new LinearLayout.LayoutParams(
                LinearLayout.LayoutParams.WRAP_CONTENT,
                LinearLayout.LayoutParams.WRAP_CONTENT));
        layout.setGravity(Gravity.CENTER);
        return layout;
    }
}
```

control


```
public class FocusListViewTest extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.focus);
        ListView listView = (ListView) findViewById(R.id.focus_listView);
        List<String> data = new ArrayList<String>();
        data.add("I am item 1");
        data.add("I am item 2");
        data.add("I am item 3");
        data.add("I am item 4");
        data.add("I am item 5");
        final FocusListViewAdapter adapter = new FocusListViewAdapter(this, data);
        listView.setAdapter(adapter);
        listView.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> parent, View view,
                                    int position, long id) {
                adapter.setCurrentItem(position);
                /**
                 * listView 初次绘制某一项数据时调用geiview方法
                 * 之后需要notifyDataSetChanged才能重绘View
                 */
                adapter.notifyDataSetChanged();
            }
        });
    }
}
```



