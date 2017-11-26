title: Android控件架构与自定义控件详解
categories: 
  - 移动开发
  - AndroidHero
tags:
  - Android
  - 自定义控件
date: 2016-10-08 19:18:29
author: mzeht
avatar: /images/favicon.png
---

#View的测量
Android帮助测量View的类--`MeasureSpec`类
MeasureSpec是一个32位的int值，高两位为测量的模式，低30位为测量的大小
使用位运算提高并优化效率

<!-- more -->

##测量的模式
- `EXACTLY` 精确值模式
控件layout_width,layout_height属性指定为具体数值时，或者是match_parent属性时

- `AT_MOST` 最大值模式
控件layout_width,layout_height属性使用warp_content，控件的大小取不超过父级控件的最大值

- `UNSPECIFIED` 
不指定测量模式，可以任意大小，通常自定义View绘制时才用到


##Demo

- View类默认的`onMeasure()`方法只支持EXACTLY模式，如果该View要支持warp_content属性
     就必须要重写onMeasure()方法 指定AT_MOST模式下warp_content时的大小
     
    ```
     @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
    }
    
    ```
    
    进入`onMeasure`方法,看源码
    可以发现最终调用
	
    
    
    ```
    setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
    ```
    
 
    
- setMeasuredDimension(int measuredWidth, int measuredHeight)里面的参数相当于px单位   
     
 ```
public class MyView extends View {
    public MyView(Context context) {
        super(context);
    }

    public MyView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public MyView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

    /**
     *
     * @param widthMeasureSpec
     * @param heightMeasureSpec
     * View类默认的onMeasure()方法只支持EXACTLY模式，如果该View要支持warp_content属性
     * 就必须要重写onMeasure()方法 指定AT_MOSt模式下warp_content时的大小
     */
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        setMeasuredDimension(measure(widthMeasureSpec), measure(heightMeasureSpec));
    }

    private int measure(int measureSpec) {
        int result = 0;
        int specMode = MeasureSpec.getMode(measureSpec);
        int specSize = MeasureSpec.getSize(measureSpec);

        //精确值模式，参数直接使用
        if (specMode == MeasureSpec.EXACTLY) {
            result = specSize;

            //最大值模式，wrap_content时取不超过父级空间的最大值
        } else if (specMode == MeasureSpec.AT_MOST) {
            result = 200;
            result = Math.min(result, specSize);
        }

        /**
         * 200PX
         * setMeasuredDimension(int measuredWidth, int measuredHeight)里面的参数相当于px单位
         */
        return result;
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        canvas.drawColor(Color.GRAY);
        int width = getWidth();
        int height = getHeight();
        Log.d("xys", "width : " + width + " height : " + height);
    }
}
```


# ViewGroup的测量
ViewGroup大小为warp_content时，需要遍历子View，调用其Measure方法，获得所有子View的大小，以便确定自己的大小;
测量完毕后，将子View放到合适的位置，遍历使用子View的Layout方法，指定其显示的具体位置，从而决定其布局的位置；
自定义ViewGroup时，通常会重写onLayout()方法来控制子View显示位置的逻辑；
如果还要支持warp_content属性，必须重写onMeasure()方法，这点与View相同


# ViewGroup的绘制
ViewGroup通常不需要绘制，如果没有指定ViewGroup的背景颜色，ViewGroup的onDarw()方法不会被调用；
ViewGroup会调用dispathchDarw()方法绘制子View，同样遍历子View，调用子View的绘制方法完成绘制；

# 自定义View
- 通常重写onDarw方法来绘制View的显示内容；
- 使用warp_content属性，必须重写onMeasure方法
- 自定义attrs属性，可以设置新的配置属性

## 一些重要的回调方法
- onFinishInflate 从xml加载组件后
- onSizeChanged 组件大小改变后
- onMeasure 回调此方法进行测量
- onLayout 回调此方法确定显示位置
- onTouchEvent 触摸事件时回调

[自定义控件常用api](http://www.jianshu.com/p/744550c02cf1s)

## 自定义View的三种方法
- 拓展现有控件
- 组合控件
- 重写View实现全新控件

### 拓展现有控件

拓展TextView

> Demo1

```
public class MyTextView extends TextView {

    private Paint mPaint1,mPaint2;
    public MyTextView(Context context) {
        super(context);
        initView();
    }

    public MyTextView(Context context, AttributeSet attrs) {
        super(context, attrs);
        initView();
    }

    public MyTextView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        initView();
    }

    @Override
    protected void onDraw(Canvas canvas) {
        // 绘制外层矩形
        canvas.drawRect(
                0, 0,
                getMeasuredWidth(),
                getMeasuredHeight(),
                mPaint1);
        // 绘制内层矩形
        canvas.drawRect(
                10,
                10,
                getMeasuredWidth() - 10,
                getMeasuredHeight() - 10,
                mPaint2);
        canvas.save();
        // 绘制文字前平移10像素
        canvas.translate(10, 0);
        // 父类完成的方法，即绘制文本
        super.onDraw(canvas);
        canvas.restore();

    }

    private  void initView(){
        mPaint1=new Paint();
        mPaint1.setColor(getResources().getColor(android.R.color.holo_blue_light));
        mPaint1.setStyle(Paint.Style.FILL);
        mPaint2=new Paint();
        mPaint2.setColor(Color.YELLOW);
        mPaint2.setStyle(Paint.Style.FILL);
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
    }
}
```

xml
```
<com.wpmac.androidhero.unit3.view.MyTextView android:layout_width="match_parent"
                                                 android:layout_height="wrap_content"
                                                 android:textSize="20sp"
                                                 android:text="Android hero"/>
```

![](http://7xqtsx.com1.z0.glb.clouddn.com/public/16-11-9/63189918.jpg)

> Demo2

文字添加线性变换效果

```
public class ShineTextView extends TextView {

    private LinearGradient mLinearGradient;
    private Matrix mGradientMatrix;
    private Paint mPaint;
    private int mViewWidth = 0;
    private int mTranslate = 0;

    public ShineTextView(Context context) {
        super(context);
    }

    public ShineTextView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public ShineTextView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        if (mViewWidth == 0) {
            mViewWidth = getMeasuredWidth();
            if (mViewWidth > 0) {
                mPaint = getPaint();
                mLinearGradient = new LinearGradient(
                        0,
                        0,
                        mViewWidth,
                        0,
                        new int[]{
                                Color.BLUE, 0xffffffff,
                                Color.BLUE},
                        null,
                        Shader.TileMode.CLAMP);
                mPaint.setShader(mLinearGradient);
                mGradientMatrix = new Matrix();
            }
        }
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        if (mGradientMatrix != null) {
            mTranslate += mViewWidth / 5;
            if (mTranslate > 2 * mViewWidth) {
                mTranslate = -mViewWidth;
            }
            mGradientMatrix.setTranslate(mTranslate, 0);
            mLinearGradient.setLocalMatrix(mGradientMatrix);
            postInvalidateDelayed(100);
        }
    }
}


```
### 复合控件
- 定义自定义属性

新建或修改attrs.xml

```
<?xml version="1.0" encoding="utf-8"?>
<resources>

    <declare-styleable name="TopBar">

        <attr name="mtitle" format="string"/>
        <attr name="mtitleTextSize" format="dimension"/>
        <attr name="mtitleTextColor" format="color"/>
        <attr name="mleftTextColor" format="color"/>
        <attr name="mleftBackground" format="reference|color"/>
        <attr name="mleftText" format="string"/>
        <attr name="mrightTextColor" format="color"/>
        <attr name="mrightBackground" format="reference|color"/>
        <attr name="mrightText" format="string"/>
    </declare-styleable>


</resources>
```

[关于format类型](http://www.cnblogs.com/jisheng/archive/2013/01/10/2854891.html)

- 引入命名空间


```
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
                xmlns:custom="http://schemas.android.com/apk/res-auto"
                xmlns:tools="http://schemas.android.com/tools"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:padding="5dp"
                tools:context=".MainActivity">
    <com.wpmac.androidhero.unit3.view.TopBar
        android:id="@+id/topBar"
        android:layout_width="wrap_content"
        android:layout_height="40dp"
        custom:mleftBackground="@drawable/blue_button"
        custom:mleftText="Back"
        custom:mleftTextColor="#FFFFFF"
        custom:mrightBackground="@drawable/blue_button"
        custom:mrightText="More"
        custom:mrightTextColor="#FFFFFF"
        custom:mtitle="自定义标题"
        custom:mtitleTextColor="#FFFFFF"
        custom:mtitleTextSize="10sp" />


</RelativeLayout>

```

- 获得自定义的属性值

注意 `ta.recycle()`

```
// 通过这个方法，将你在atts.xml中定义的declare-styleable
        // 的所有属性的值存储到TypedArray中
        TypedArray ta = context.obtainStyledAttributes(attrs,
                R.styleable.TopBar);
        // 从TypedArray中取出对应的值来为要设置的属性赋值
        mLeftTextColor = ta.getColor(
                R.styleable.TopBar_mleftTextColor, 0);
        mLeftBackground = ta.getDrawable(
                R.styleable.TopBar_mleftBackground);
        mLeftText = ta.getString(R.styleable.TopBar_mleftText);
....

// 获取完TypedArray的值后，一般要调用
        // recyle方法来避免重新创建的时候的错误
        ta.recycle();
        
```

- 组合控件

```
// 为组件元素设置相应的布局元素
        mLeftParams = new RelativeLayout.LayoutParams(
                RelativeLayout.LayoutParams.WRAP_CONTENT,
                RelativeLayout.LayoutParams.MATCH_PARENT);
        mLeftParams.addRule(RelativeLayout.ALIGN_PARENT_LEFT, TRUE);
        // 添加到ViewGroup
        addView(mLeftButton, mLeftParams);
```

- 定义回调借口

```
// 接口对象，实现回调机制，在回调方法中
    // 通过映射的接口对象调用接口中的方法
    // 而不用去考虑如何实现，具体的实现由调用者去创建
    public interface topbarClickListener {
        // 左按钮点击事件
        void leftClick();
        // 右按钮点击事件
        void rightClick();
    }
```

- 暴露接口，绑定接口


```
// 映射传入的接口对象
    private topbarClickListener mListener;
    
    // 暴露一个方法给调用者来注册接口回调
    // 通过接口来获得回调者对接口方法的实现
    public void setOnTopbarClickListener(topbarClickListener mListener) {
        this.mListener = mListener;
    }


```

```
// 按钮的点击事件，不需要具体的实现，
        // 只需调用接口的方法，回调的时候，会有具体的实现
        mRightButton.setOnClickListener(new OnClickListener() {

            @Override
            public void onClick(View v) {
                mListener.rightClick();
            }
        });

        mLeftButton.setOnClickListener(new OnClickListener() {

            @Override
            public void onClick(View v) {
                mListener.leftClick();
            }
        });

```

- 实现接口

```
 @BindView(R.id.topBar)
    TopBar mTopBar;
   ......

 mTopBar.setOnTopbarClickListener(new TopBar.topbarClickListener() {
            @Override
            public void leftClick() {
                Toast.makeText(TopBarTestActivity.this,
                        "left", Toast.LENGTH_SHORT)
                        .show();
            }

            @Override
            public void rightClick() {
                Toast.makeText(TopBarTestActivity.this,
                        "right", Toast.LENGTH_SHORT)
                        .show();
            }
        });

```

### 重写View实现控件
- 弧线圈
![](http://7xqtsx.com1.z0.glb.clouddn.com/public/16-11-10/55565089.jpg)

```
public class CircleProgressView extends View {

    private int mMeasureHeigth;
    private int mMeasureWidth;

    private Paint mCirclePaint;
    private float mCircleXY;
    private float mRadius;

    private Paint mArcPaint;
    private RectF mArcRectF;
    private float mSweepAngle;
    private float mSweepValue = 66;

    private Paint mTextPaint;
    private String mShowText;
    private float mShowTextSize;

    public CircleProgressView(Context context, AttributeSet attrs,
                              int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

    public CircleProgressView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public CircleProgressView(Context context) {
        super(context);
    }

    @Override
    protected void onMeasure(int widthMeasureSpec,
                             int heightMeasureSpec) {
        mMeasureWidth = MeasureSpec.getSize(widthMeasureSpec);
        mMeasureHeigth = MeasureSpec.getSize(heightMeasureSpec);
        setMeasuredDimension(mMeasureWidth, mMeasureHeigth);
        initView();
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        // 绘制圆
        canvas.drawCircle(mCircleXY, mCircleXY, mRadius, mCirclePaint);
        // 绘制弧线
        canvas.drawArc(mArcRectF, 270, mSweepAngle, false, mArcPaint);
        // 绘制文字
        canvas.drawText(mShowText, 0, mShowText.length(),
                mCircleXY, mCircleXY + (mShowTextSize / 4), mTextPaint);
    }

    private void initView() {
        float length = 0;
        if (mMeasureHeigth >= mMeasureWidth) {
            length = mMeasureWidth;
        } else {
            length = mMeasureHeigth;
        }

        mCircleXY = length / 2;
        mRadius = (float) (length * 0.5 / 2);
        mCirclePaint = new Paint();
        //设置防锯齿
        mCirclePaint.setAntiAlias(true);
        mCirclePaint.setColor(getResources().getColor(
                android.R.color.holo_blue_bright));

        mArcRectF = new RectF(
                (float) (length * 0.1),
                (float) (length * 0.1),
                (float) (length * 0.9),
                (float) (length * 0.9));
        mSweepAngle = (mSweepValue / 100f) * 360f;
        mArcPaint = new Paint();
        mArcPaint.setAntiAlias(true);
        mArcPaint.setColor(getResources().getColor(
                android.R.color.holo_blue_bright));
        mArcPaint.setStrokeWidth((float) (length * 0.1));
        mArcPaint.setStyle(Style.STROKE);

        mShowText = setShowText();
        mShowTextSize = setShowTextSize();
        mTextPaint = new Paint();
        mTextPaint.setTextSize(mShowTextSize);
        mTextPaint.setTextAlign(Paint.Align.CENTER);
    }

    private float setShowTextSize() {
        this.invalidate();
        return 50;
    }

    private String setShowText() {
        this.invalidate();
        return "Android Skill";
    }

    public void forceInvalidate() {
        this.invalidate();
    }

    public void setSweepValue(float sweepValue) {
        if (sweepValue != 0) {
            mSweepValue = sweepValue;
        } else {
            mSweepValue = 25;
        }
        this.invalidate();
    }
}

```

- 动态条形图

![](http://7xqtsx.com1.z0.glb.clouddn.com/public/16-11-10/97709616.jpg)

```

public class VolumeView extends View {

    private int mWidth;
    private int mRectWidth;
    private int mRectHeight;
    private Paint mPaint;
    private int mRectCount;
    private int offset = 5;
    private double mRandom;
    private LinearGradient mLinearGradient;

    public VolumeView(Context context) {
        super(context);
        initView();
    }

    public VolumeView(Context context, AttributeSet attrs) {
        super(context, attrs);
        initView();
    }

    public VolumeView(Context context, AttributeSet attrs,
                      int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        initView();
    }

    private void initView() {
        mPaint = new Paint();
        mPaint.setColor(Color.BLUE);
        mPaint.setStyle(Paint.Style.FILL);
        mRectCount = 12;
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        Log.d("onSizeChanged",w+"/"+h+"/"+oldw+"/"+oldh+"/");
        mWidth = getWidth();
        mRectHeight = getHeight();
        mRectWidth = (int) (mWidth * 0.6 / mRectCount);
        mLinearGradient = new LinearGradient(
                0,
                0,
                mRectWidth,
                mRectHeight,
                Color.YELLOW,
                Color.BLUE,
                Shader.TileMode.CLAMP);
        mPaint.setShader(mLinearGradient);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        for (int i = 0; i < mRectCount; i++) {
            mRandom = Math.random();
            float currentHeight = (float) (mRectHeight * mRandom);
            canvas.drawRect(
                    (float) (mWidth * 0.4 / 2 + mRectWidth * i + offset),
                    currentHeight,
                    (float) (mWidth * 0.4 / 2 + mRectWidth * (i + 1)),
                    mRectHeight,
                    mPaint);
        }

        //重绘延迟300ms
        postInvalidateDelayed(300);
    }
}

```

# 自定义ViewGroup

重写onMeasure()测量子View；
重写onLayout()确定子View的位置；
重写onTouchEvent()等增加响应事件；

## 实现类似ScrollView的功能


```public class ScrollView extends ViewGroup {
    private int mScreenHeight;
    private Scroller mScroller;
    private int mLastY;
    private int mStart;
    private int mEnd;

    public ScrollView(Context context) {
        super(context);
        initView(context);
    }

    public ScrollView(Context context, AttributeSet attrs) {
        super(context, attrs);
        initView(context);


    }

    public ScrollView(Context context, AttributeSet attrs,
                        int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        initView(context);
    }

    private void initView(Context context) {
        WindowManager wm = (WindowManager) context.getSystemService(
                Context.WINDOW_SERVICE);
        DisplayMetrics dm = new DisplayMetrics();
        wm.getDefaultDisplay().getMetrics(dm);
        mScreenHeight = dm.heightPixels;
        mScroller = new Scroller(context);
    }

    @Override
    protected void onLayout(boolean changed,
                            int l, int t, int r, int b) {
        int childCount = getChildCount();
        // 设置ViewGroup的高度
        MarginLayoutParams mlp = (MarginLayoutParams) getLayoutParams();
        mlp.height = mScreenHeight * childCount;
        setLayoutParams(mlp);
        for (int i = 0; i < childCount; i++) {
            View child = getChildAt(i);
            if (child.getVisibility() != View.GONE) {
                child.layout(l, i * mScreenHeight,
                        r, (i + 1) * mScreenHeight);
            }
        }
    }

    @Override
    protected void onMeasure(int widthMeasureSpec,
                             int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        int count = getChildCount();
        for (int i = 0; i < count; ++i) {
            View childView = getChildAt(i);
            measureChild(childView,
                    widthMeasureSpec, heightMeasureSpec);
        }
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        int y = (int) event.getY();
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                mLastY = y;
                mStart = getScrollY();
                break;
            case MotionEvent.ACTION_MOVE:
                if (!mScroller.isFinished()) {
                    mScroller.abortAnimation();
                }
                int dy = mLastY - y;
                if (getScrollY() < 0) {
                    dy = 0;
                }
                if (getScrollY() > getHeight() - mScreenHeight) {
                    dy = 0;
                }
                scrollBy(0, dy);
                mLastY = y;
                break;
            case MotionEvent.ACTION_UP:
                int dScrollY = checkAlignment();
                if (dScrollY > 0) {
                    if (dScrollY < mScreenHeight / 3) {
                        mScroller.startScroll(
                                0, getScrollY(),
                                0, -dScrollY);
                    } else {
                        mScroller.startScroll(
                                0, getScrollY(),
                                0, mScreenHeight - dScrollY);
                    }
                } else {
                    if (-dScrollY < mScreenHeight / 3) {
                        mScroller.startScroll(
                                0, getScrollY(),
                                0, -dScrollY);
                    } else {
                        mScroller.startScroll(
                                0, getScrollY(),
                                0, -mScreenHeight - dScrollY);
                    }
                }
                break;
        }
        postInvalidate();
        return true;
    }
    private int checkAlignment() {
        int mEnd = getScrollY();
        boolean isUp = ((mEnd - mStart) > 0) ? true : false;
        int lastPrev = mEnd % mScreenHeight;
        int lastNext = mScreenHeight - lastPrev;
        if (isUp) {
            //向上的
            return lastPrev;
        } else {
            return -lastNext;
        }
    }
    @Override
    public void computeScroll() {
        super.computeScroll();
        if (mScroller.computeScrollOffset()) {
            scrollTo(0, mScroller.getCurrY());
            postInvalidate();
        }
    }
}


```

# 事件拦截机制分析
xml

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
    android:id="@+id/activity_view_group"
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context="com.wpmac.androidhero.unit3.ViewGroupActivity">
    <com.wpmac.androidhero.unit3.viewgroup.MyViewGroupA
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@android:color/holo_blue_bright">

        <com.wpmac.androidhero.unit3.viewgroup.MyViewGroupB
            android:layout_width="300dp"
            android:layout_height="300dp"
            android:background="@android:color/holo_green_dark">

            <com.wpmac.androidhero.unit3.viewgroup.MyView
                android:layout_width="150dp"
                android:layout_height="150dp"
                android:background="@android:color/darker_gray" />

        </com.wpmac.androidhero.unit3.viewgroup.MyViewGroupB>

    </com.wpmac.androidhero.unit3.viewgroup.MyViewGroupA>
</RelativeLayout>

```

ViewGroupB-ViewGroupA-View 依次嵌套

```
public class MyViewGroupA extends LinearLayout {

    public MyViewGroupA(Context context) {
        super(context);
    }

    public MyViewGroupA(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public MyViewGroupA(Context context, AttributeSet attrs,
                        int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

    @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        Log.d("xys", "ViewGroupA dispatchTouchEvent" + ev.getAction());
        return super.dispatchTouchEvent(ev);
    }

    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        Log.d("xys", "ViewGroupA onInterceptTouchEvent" + ev.getAction());
        return super.onInterceptTouchEvent(ev);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        Log.d("xys", "ViewGroupA onTouchEvent" + event.getAction());
        return super.onTouchEvent(event);
    }
}

```


- ViewGroup监听dispatchTouchEvent，onTouchEvent，onInterceptTouchEvent方法
- View打印dispatchTouchEvent，onTouchEvent方法
- ViewGroup比View多一个onInterceptTouchEvent方法

## 一般情况，点击View

```

11-10 14:17:38.782 27002-27002/com.wpmac.androidhero D/xys: ViewGroupA dispatchTouchEvent0
11-10 14:17:38.782 27002-27002/com.wpmac.androidhero D/xys: ViewGroupA onInterceptTouchEvent0
11-10 14:17:38.782 27002-27002/com.wpmac.androidhero D/xys: ViewGroupB dispatchTouchEvent0
11-10 14:17:38.782 27002-27002/com.wpmac.androidhero D/xys: ViewGroupB onInterceptTouchEvent0
11-10 14:17:38.782 27002-27002/com.wpmac.androidhero D/xys: View dispatchTouchEvent0
11-10 14:17:38.782 27002-27002/com.wpmac.androidhero D/xys: View onTouchEvent0
11-10 14:17:38.782 27002-27002/com.wpmac.androidhero D/xys: ViewGroupB onTouchEvent0
11-10 14:17:38.783 27002-27002/com.wpmac.androidhero D/xys: ViewGroupA onTouchEvent0
```
![](http://7xqtsx.com1.z0.glb.clouddn.com/public/16-11-10/86008071.jpg)
[所有Demo地址](https://github.com/mzeht/AndroidHero)

-------

ViewGroupA拦截

```
    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        Log.d("xys", "ViewGroupA onInterceptTouchEvent" + ev.getAction());
        return true;
    }
```

日志

```
11-10 16:08:24.717 23761-23761/com.wpmac.androidhero D/xys: ViewGroupA dispatchTouchEvent0
11-10 16:08:24.717 23761-23761/com.wpmac.androidhero D/xys: ViewGroupA onInterceptTouchEvent0
11-10 16:08:24.717 23761-23761/com.wpmac.androidhero D/xys: ViewGroupA onTouchEvent0
```
![](http://7xqtsx.com1.z0.glb.clouddn.com/public/16-11-10/6102134.jpg)
-------
ViewGroupA不拦截，ViewGroupB拦截

```
11-10 16:15:49.484 28880-28880/com.wpmac.androidhero D/xys: ViewGroupA dispatchTouchEvent0![]()
11-10 16:15:49.484 28880-28880/com.wpmac.androidhero D/xys: ViewGroupA onInterceptTouchEvent0
11-10 16:15:49.484 28880-28880/com.wpmac.androidhero D/xys: ViewGroupB dispatchTouchEvent0
11-10 16:15:49.484 28880-28880/com.wpmac.androidhero D/xys: ViewGroupB onInterceptTouchEvent0
11-10 16:15:49.484 28880-28880/com.wpmac.androidhero D/xys: ViewGroupB onTouchEvent0
11-10 16:15:49.484 28880-28880/com.wpmac.androidhero D/xys: ViewGroupA onTouchEvent0
```
![](http://7xqtsx.com1.z0.glb.clouddn.com/public/16-11-10/954580.jpg)

-------
View onTouchEven设置为true

event.getAction() 获得的返回值：

 

//触摸屏幕时刻
case MotionEvent.ACTION_DOWN:  // = 0

break;
//触摸并移动时刻
case MotionEvent.ACTION_MOVE:  // = 2

break;


//终止触摸时刻
case MotionEvent.ACTION_UP:  // = 1
break;



-------
仅仅View onTouchEvent设置return true

```
11-10 17:13:44.984 27187-27187/com.wpmac.androidhero D/xys: ViewGroupA dispatchTouchEvent0
11-10 17:13:44.984 27187-27187/com.wpmac.androidhero D/xys: ViewGroupA onInterceptTouchEvent0
11-10 17:13:44.984 27187-27187/com.wpmac.androidhero D/xys: ViewGroupB dispatchTouchEvent0
11-10 17:13:44.984 27187-27187/com.wpmac.androidhero D/xys: ViewGroupB onInterceptTouchEvent0
11-10 17:13:44.984 27187-27187/com.wpmac.androidhero D/xys: View dispatchTouchEvent0
11-10 17:13:44.985 27187-27187/com.wpmac.androidhero D/xys: View onTouchEvent0
11-10 17:13:45.012 27187-27187/com.wpmac.androidhero D/xys: ViewGroupA dispatchTouchEvent1
11-10 17:13:45.012 27187-27187/com.wpmac.androidhero D/xys: ViewGroupA onInterceptTouchEvent1
11-10 17:13:45.012 27187-27187/com.wpmac.androidhero D/xys: ViewGroupB dispatchTouchEvent1
11-10 17:13:45.012 27187-27187/com.wpmac.androidhero D/xys: ViewGroupB onInterceptTouchEvent1
11-10 17:13:45.012 27187-27187/com.wpmac.androidhero D/xys: View dispatchTouchEvent1
11-10 17:13:45.012 27187-27187/com.wpmac.androidhero D/xys: View onTouchEvent1
```


ViewGroupB onTouchEvent 设置为true

```
11-10 17:33:53.022 5387-5387/com.wpmac.androidhero D/xys: ViewGroupA dispatchTouchEvent0
11-10 17:33:53.022 5387-5387/com.wpmac.androidhero D/xys: ViewGroupA onInterceptTouchEvent0
11-10 17:33:53.022 5387-5387/com.wpmac.androidhero D/xys: ViewGroupB dispatchTouchEvent0
11-10 17:33:53.022 5387-5387/com.wpmac.androidhero D/xys: ViewGroupB onInterceptTouchEvent0
11-10 17:33:53.022 5387-5387/com.wpmac.androidhero D/xys: View dispatchTouchEvent0
11-10 17:33:53.022 5387-5387/com.wpmac.androidhero D/xys: View onTouchEvent0
11-10 17:33:53.022 5387-5387/com.wpmac.androidhero D/xys: ViewGroupB onTouchEvent0
11-10 17:33:53.096 5387-5387/com.wpmac.androidhero D/xys: ViewGroupA dispatchTouchEvent1
11-10 17:33:53.096 5387-5387/com.wpmac.androidhero D/xys: ViewGroupA onInterceptTouchEvent1
11-10 17:33:53.096 5387-5387/com.wpmac.androidhero D/xys: ViewGroupB dispatchTouchEvent1
11-10 17:33:53.096 5387-5387/com.wpmac.androidhero D/xys: ViewGroupB onTouchEvent1

```

![](http://7xqtsx.com1.z0.glb.clouddn.com/public/16-11-10/470111.jpg)


