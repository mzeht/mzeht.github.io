title: popupwindow封装使用
categories: 
  - 移动开发
  - Android
author: mzeht
avatar: /images/favicon.png
authorDesc: 一个写代码的
date: 2017-05-28 15:00:46
tags:
keywords:
description:
photos:
---

## 一个封装封装类

```
public class CustomPopWindow implements PopupWindow.OnDismissListener{
    private static final float DEFAULT_ALPHA = 0.7f;
    private Context mContext;
    private int mWidth;
    private int mHeight;
    private boolean mIsFocusable = true;
    private boolean mIsOutside = true;
    private int mResLayoutId = -1;
    private View mContentView;
    private PopupWindow mPopupWindow;
    private int mAnimationStyle = -1;

    private boolean mClippEnable = true;//default is true
    private boolean mIgnoreCheekPress = false;
    private int mInputMode = -1;
    private PopupWindow.OnDismissListener mOnDismissListener;
    private int mSoftInputMode = -1;
    private boolean mTouchable = true;//default is ture
    private View.OnTouchListener mOnTouchListener;

    private Window mWindow;//当前Activity 的窗口
    /**
     * 弹出PopWindow 背景是否变暗，默认不会变暗。
     */
    private boolean mIsBackgroundDark = false;

    private float mBackgroundDrakValue = 0;// 背景变暗的值，0 - 1

    private CustomPopWindow(Context context){
        mContext = context;
    }

    public int getWidth() {
        return mWidth;
    }

    public int getHeight() {
        return mHeight;
    }

    /**
     *
     * @param anchor
     * @param xOff
     * @param yOff
     * @return
     */
    public CustomPopWindow showAsDropDown(View anchor, int xOff, int yOff){
        if(mPopupWindow!=null){
            mPopupWindow.showAsDropDown(anchor,xOff,yOff);
        }
        return this;
    }

    public CustomPopWindow showAsDropDown(View anchor){
        if(mPopupWindow!=null){
            mPopupWindow.showAsDropDown(anchor);
        }
        return this;
    }

    @RequiresApi(api = Build.VERSION_CODES.KITKAT)
    public CustomPopWindow showAsDropDown(View anchor, int xOff, int yOff, int gravity){
        if(mPopupWindow!=null){
            mPopupWindow.showAsDropDown(anchor,xOff,yOff,gravity);
        }
        return this;
    }


    /**
     * 相对于父控件的位置（通过设置Gravity.CENTER，下方Gravity.BOTTOM等 ），可以设置具体位置坐标
     * @param parent 父控件
     * @param gravity
     * @param x the popup's x location offset
     * @param y the popup's y location offset
     * @return
     */
    public CustomPopWindow showAtLocation(View parent, int gravity, int x, int y){
        if(mPopupWindow!=null){
            mPopupWindow.showAtLocation(parent,gravity,x,y);
        }
        return this;
    }

    /**
     * 添加一些属性设置
     * @param popupWindow
     */
    private void apply(PopupWindow popupWindow){
        popupWindow.setClippingEnabled(mClippEnable);
        if(mIgnoreCheekPress){
            popupWindow.setIgnoreCheekPress();
        }
        if(mInputMode!=-1){
            popupWindow.setInputMethodMode(mInputMode);
        }
        if(mSoftInputMode!=-1){
            popupWindow.setSoftInputMode(mSoftInputMode);
        }
        if(mOnDismissListener!=null){
            popupWindow.setOnDismissListener(mOnDismissListener);
        }
        if(mOnTouchListener!=null){
            popupWindow.setTouchInterceptor(mOnTouchListener);
        }
        popupWindow.setTouchable(mTouchable);



    }

    private PopupWindow build(){

        if(mContentView == null){
            mContentView = LayoutInflater.from(mContext).inflate(mResLayoutId,null);
        }

        // 2017.3.17 add
        // 获取当前Activity的window
        Activity activity = (Activity) mContentView.getContext();
        if(activity!=null && mIsBackgroundDark){
            //如果设置的值在0 - 1的范围内，则用设置的值，否则用默认值
            final  float alpha = (mBackgroundDrakValue > 0 && mBackgroundDrakValue < 1) ? mBackgroundDrakValue : DEFAULT_ALPHA;
            mWindow = activity.getWindow();
            WindowManager.LayoutParams params = mWindow.getAttributes();
            params.alpha = alpha;
            mWindow.setAttributes(params);
        }


        if(mWidth != 0 && mHeight!=0 ){
            mPopupWindow = new PopupWindow(mContentView,mWidth,mHeight);
        }else{
            mPopupWindow = new PopupWindow(mContentView, ViewGroup.LayoutParams.WRAP_CONTENT, ViewGroup.LayoutParams.WRAP_CONTENT);
        }
        if(mAnimationStyle!=-1){
            mPopupWindow.setAnimationStyle(mAnimationStyle);
        }

        apply(mPopupWindow);//设置一些属性

        mPopupWindow.setFocusable(mIsFocusable);
        mPopupWindow.setBackgroundDrawable(new ColorDrawable(Color.TRANSPARENT));
        mPopupWindow.setOutsideTouchable(mIsOutside);

        if(mWidth == 0 || mHeight == 0){
            mPopupWindow.getContentView().measure(View.MeasureSpec.UNSPECIFIED, View.MeasureSpec.UNSPECIFIED);
            //如果外面没有设置宽高的情况下，计算宽高并赋值
            mWidth = mPopupWindow.getContentView().getMeasuredWidth();
            mHeight = mPopupWindow.getContentView().getMeasuredHeight();
        }

        // 添加dissmiss 监听
        mPopupWindow.setOnDismissListener(this);


        mPopupWindow.update();

        return mPopupWindow;
    }

    @Override
    public void onDismiss() {
        dissmiss();
    }

    /**
     * 关闭popWindow
     */
    public void dissmiss(){

        if(mOnDismissListener!=null){
            mOnDismissListener.onDismiss();
        }

        //如果设置了背景变暗，那么在dissmiss的时候需要还原
        if(mWindow!=null){
            WindowManager.LayoutParams params = mWindow.getAttributes();
            params.alpha = 1.0f;
            mWindow.setAttributes(params);
        }
        if(mPopupWindow!=null && mPopupWindow.isShowing()){
            mPopupWindow.dismiss();
        }
    }


    public static class PopupWindowBuilder{
        private CustomPopWindow mCustomPopWindow;

        public PopupWindowBuilder(Context context){
            mCustomPopWindow = new CustomPopWindow(context);
        }
        public PopupWindowBuilder size(int width,int height){
            mCustomPopWindow.mWidth = width;
            mCustomPopWindow.mHeight = height;
            return this;
        }


        public PopupWindowBuilder setFocusable(boolean focusable){
            mCustomPopWindow.mIsFocusable = focusable;
            return this;
        }



        public PopupWindowBuilder setView(int resLayoutId){
            mCustomPopWindow.mResLayoutId = resLayoutId;
            mCustomPopWindow.mContentView = null;
            return this;
        }

        public PopupWindowBuilder setView(View view){
            mCustomPopWindow.mContentView = view;
            mCustomPopWindow.mResLayoutId = -1;
            return this;
        }

        public PopupWindowBuilder setOutsideTouchable(boolean outsideTouchable){
            mCustomPopWindow.mIsOutside = outsideTouchable;
            return this;
        }

        /**
         * 设置弹窗动画
         * @param animationStyle
         * @return
         */
        public PopupWindowBuilder setAnimationStyle(int animationStyle){
            mCustomPopWindow.mAnimationStyle = animationStyle;
            return this;
        }


        public PopupWindowBuilder setClippingEnable(boolean enable){
            mCustomPopWindow.mClippEnable =enable;
            return this;
        }


        public PopupWindowBuilder setIgnoreCheekPress(boolean ignoreCheekPress){
            mCustomPopWindow.mIgnoreCheekPress = ignoreCheekPress;
            return this;
        }

        public PopupWindowBuilder setInputMethodMode(int mode){
            mCustomPopWindow.mInputMode = mode;
            return this;
        }

        public PopupWindowBuilder setOnDissmissListener(PopupWindow.OnDismissListener onDissmissListener){
            mCustomPopWindow.mOnDismissListener = onDissmissListener;
            return this;
        }


        public PopupWindowBuilder setSoftInputMode(int softInputMode){
            mCustomPopWindow.mSoftInputMode = softInputMode;
            return this;
        }


        public PopupWindowBuilder setTouchable(boolean touchable){
            mCustomPopWindow.mTouchable = touchable;
            return this;
        }

        public PopupWindowBuilder setTouchIntercepter(View.OnTouchListener touchIntercepter){
            mCustomPopWindow.mOnTouchListener = touchIntercepter;
            return this;
        }

        /**
         * 设置背景变暗是否可用
         * @param isDark
         * @return
         */
        public PopupWindowBuilder enableBackgroundDark(boolean isDark){
            mCustomPopWindow.mIsBackgroundDark = isDark;
            return this;
        }

        /**
         * 设置北京变暗的值
         * @param darkValue
         * @return
         */
        public PopupWindowBuilder setBgDarkAlpha(float darkValue){
            mCustomPopWindow.mBackgroundDrakValue = darkValue;
            return this;
        }


        public CustomPopWindow create(){
            //构建PopWindow
            mCustomPopWindow.build();
            return mCustomPopWindow;
        }

    }

}
```

## 实际使用

```
public class ReadMineActivity extends BaseActivity {

    CustomPopWindow mCustomPopWindow;


    Dialog dateDialog, timeDialog;

    final popupwindow p = new popupwindow();


    class popupwindow {

        @BindView(R.id.is_switch)
        SwitchCompat is_switch;//是否重复
        @BindView(R.id.set_dataandtime)
        LinearLayout setDateAndTimeLL;//选择日期时间菜单

        @BindView(R.id.set_dataandtime_detail)
        LinearLayout setDateAndTimeDetailLL;//选择日期时间

        @BindView(R.id.set_time)
        LinearLayout setTimeLL;

        @BindView(R.id.set_time_detail)
        LinearLayout setTimeDetailLL;

        @BindView(R.id.data_text)
        TextView dateText;//日期

        @BindView(R.id.time1)
        TextView time1;//time

        @BindView(R.id.time2)
        TextView time2;//time

        @BindView(R.id.day1)
        CheckBox day1;
        @BindView(R.id.day2)
        CheckBox day2;
        @BindView(R.id.day3)
        CheckBox day3;
        @BindView(R.id.day4)
        CheckBox day4;
        @BindView(R.id.day5)
        CheckBox day5;
        @BindView(R.id.day6)
        CheckBox day6;
        @BindView(R.id.day7)
        CheckBox day7;

        @BindView(R.id.add_readmine_submit)
        TextView submit;

        @BindView(R.id.readmine_message_edit)
        EditText message;


    }


    @BindView(R.id.bind_newreadmie_ll)
    LinearLayout bindNewReadMineLL;

    @BindView(R.id.recyclerView)
    RecyclerView mRecyclerView;

    MultiTypeAdapter mAdapter;
    Items items = new Items();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_read_mine);
        ButterKnife.bind(this);
        initView();
        setListener();
        initData();
    }

    private void initData() {
        for (int i = 0; i <25 ; i++) {
            ReadmineItem item = new ReadmineItem();
            items.add(item);
        }
        mAdapter.notifyDataSetChanged();
    }

    private void setListener() {
        bindNewReadMineLL.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                mCustomPopWindow.showAsDropDown(bindNewReadMineLL, 0, 20);
            }
        });
    }

    private void initView() {
        //=======================================mCustomPopWindow============================================//
        View contentView = LayoutInflater.from(getActivity()).inflate(R.layout.popwindow_add_readmine, null);
        //处理popWindow 显示内容
        handleLogic(contentView);
        //创建并显示popWindow
        mCustomPopWindow = new CustomPopWindow.PopupWindowBuilder(getActivity())
                .setView(contentView)
                .size(ViewGroup.LayoutParams.MATCH_PARENT, ViewGroup.LayoutParams.WRAP_CONTENT)
                .enableBackgroundDark(false) //弹出popWindow时，背景是否变暗
                .setBgDarkAlpha(0.7f) // 控制亮度
                .setOnDissmissListener(new PopupWindow.OnDismissListener() {
                    @Override
                    public void onDismiss() {
                        Log.e("TAG", "onDismiss");
                    }
                })
                .create();


        //=======================================recyclerView============================================//
        mRecyclerView.setLayoutManager(new WrapContentLinearLayoutManager(getActivity()));
        mAdapter = new MultiTypeAdapter();
        mAdapter.register(ReadmineItem.class, new ReadmineItemViewBinder());
        mAdapter.setItems(items);
        mRecyclerView.setAdapter(mAdapter);




    }

    private void handleLogic(View contentView) {

        ButterKnife.bind(p, contentView);

        p.dateText.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                showDateDialog(DateUtil.getDateForString(p.dateText.getText().toString()));
            }
        });
        p.time1.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                showTimeDialog();
            }
        });
        p.time2.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                showTimeDialog();
            }
        });

        p.setTimeLL.setVisibility(View.GONE);
        p.setTimeDetailLL.setVisibility(View.GONE);
        p.is_switch.setOnCheckedChangeListener(new CompoundButton.OnCheckedChangeListener() {
            @Override
            public void onCheckedChanged(CompoundButton buttonView, boolean isChecked) {
                if (isChecked) {
                    p.setDateAndTimeLL.setVisibility(View.GONE);
                    p.setDateAndTimeDetailLL.setVisibility(View.GONE);
                    p.setTimeLL.setVisibility(View.VISIBLE);
                    p.setTimeDetailLL.setVisibility(View.VISIBLE);
                } else {
                    p.setDateAndTimeLL.setVisibility(View.VISIBLE);
                    p.setDateAndTimeDetailLL.setVisibility(View.VISIBLE);
                    p.setTimeLL.setVisibility(View.GONE);
                    p.setTimeDetailLL.setVisibility(View.GONE);

                }
            }
        });
        p.submit.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                T.showShort(getActivity(),p.dateText.getText().toString()+p.time1.getText().toString()+p.day1.isChecked()+p.message.getText().toString());
            }
        });



    }

    private void showTimeDialog() {
        if (timeDialog == null) {

            TimePickerDialog.Builder builder = new TimePickerDialog.Builder(this);
            timeDialog = builder.setOnTimeSelectedListener(new TimePickerDialog.OnTimeSelectedListener() {
                @Override
                public void onTimeSelected(int[] times) {

                    p.time1.setText(times[0] + ":" + times[1]);
                    p.time2.setText(times[0] + ":" + times[1]);

                }
            }).create();
        }

        timeDialog.show();
    }

    private void showDateDialog(List<Integer> date) {

        //==============================DatePickerDialog===================================//


        if (dateDialog==null){
            DatePickerDialog.Builder Datebuilder = new DatePickerDialog.Builder(this);
            dateDialog=Datebuilder.setOnDateSelectedListener(new DatePickerDialog.OnDateSelectedListener() {
                @Override
                public void onDateSelected(int[] dates) {

                    p.dateText.setText(dates[0] + "-" + (dates[1] > 9 ? dates[1] : ("0" + dates[1])) + "-"
                            + (dates[2] > 9 ? dates[2] : ("0" + dates[2])));

                }

                @Override
                public void onCancel() {

                }
            }).setSelectYear(date.get(0) - 1)
                    .setSelectMonth(date.get(1) - 1)
                    .setSelectDay(date.get(2) - 1)
                    .setMaxYear(DateUtil.getYear())
                    .setMaxMonth(DateUtil.getDateForString(DateUtil.getToday()).get(1))
                    .setMaxDay(DateUtil.getDateForString(DateUtil.getToday()).get(2)).create();

        }
        dateDialog.show();
    }
}

```

## 效果图
![](http://7xqtsx.com1.z0.glb.clouddn.com/17-5-28/39434013.jpg)


