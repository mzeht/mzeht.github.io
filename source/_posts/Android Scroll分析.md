title: Android Scroll分析
categories: 
  - 移动开发
  - AndroidHero
tags:
  - Android
  - Scroll
date: 2016-10-12 22:20:33
author: mzeht
avatar: /images/favicon.png
---

# 滑动效果的产生
滑动的本质是移动View，通过不断改变View的坐标实现，与动画的原理类似，不过既然是滑动，就需要监听各种点击，触摸事件

<!-- more -->
## Android坐标系

Android坐标系以屏幕左上角为原点，如图
![](http://7xqtsx.com1.z0.glb.clouddn.com/public/16-11-22/68901668.jpg)

## 视图坐标系
子视图以父视图左上角为坐标原点，如图
![](http://7xqtsx.com1.z0.glb.clouddn.com/public/16-11-23/13206722.jpg)
触控事件中，通过getX(),getY()获得的坐标就是视图中的坐标系

## 触控事件


```
 //单点触摸按下动作
    public static final int ACTION_DOWN             = 0;
    
    //单点触摸离开动作
    public static final int ACTION_UP               = 1;
    
   //触摸点移动动作
    public static final int ACTION_MOVE             = 2;
    
   //触摸动作取消
    public static final int ACTION_CANCEL           = 3;
    
   //触摸动作超出边界
    public static final int ACTION_OUTSIDE          = 4;

    //多点触摸按下动作
    public static final int ACTION_POINTER_DOWN     = 5;
    
    //多点离开动作
    public static final int ACTION_POINTER_UP       = 6;

    public static final int ACTION_HOVER_MOVE       = 7;

    
    public static final int ACTION_SCROLL           = 8;

   
    public static final int ACTION_HOVER_ENTER      = 9;

   
    public static final int ACTION_HOVER_EXIT       = 10;

 
    public static final int ACTION_BUTTON_PRESS   = 11;

   
    public static final int ACTION_BUTTON_RELEASE  = 12;
```


一般在View的onTouchEvent方法中对event.getAction()进行筛选控制，（不涉及多点触控的情况下）


```

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        int x = (int) event.getX();
        int y = (int) event.getY();
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                lastX = (int) event.getX();
                lastY = (int) event.getY();
                break;
            case MotionEvent.ACTION_MOVE:
                int offsetX = x - lastX;
                int offsetY = y - lastY;
                ((View) getParent()).scrollBy(-offsetX, -offsetY);
                break;
            case MotionEvent.ACTION_UP:
                // 手指离开时，执行滑动过程
                View viewGroup = ((View) getParent());
                mScroller.startScroll(
                        viewGroup.getScrollX(),
                        viewGroup.getScrollY(),
                        -viewGroup.getScrollX(),
                        -viewGroup.getScrollY());
                invalidate();
                break;
        }
        return true;
    }
```

系统提供了很多获取坐标值，相对距离的坐标值的方法，如图
![](http://7xqtsx.com1.z0.glb.clouddn.com/public/16-11-23/82371833.jpg)

大概分为两类：

-------
View提供的获取坐标的方法
getTop(): 获取到View自身顶边到其父布局顶部的距离
getLeft() : 获取到View自身左边到其父布局左边的距离
getRight():
getBottom();


-------
MotionEvent提供的方法
getX():	获取点击事件距离控件左边的距离，即视图坐标
getY():获取点击事件距离控件顶部的距离，即视图坐标
getRawX():获取点击事件距离整个屏幕左边的距离，即绝对坐标
getRawY():....

# 实现滑动的七种方法

## layout方法









