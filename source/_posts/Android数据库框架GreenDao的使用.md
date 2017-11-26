title: Android数据库框架GreenDao的使用
categories: 
  - 移动开发
  - Android
tags:
  - 数据库
  - greendao
date: 2017-01-02 01:01:33
author: mzeht
avatar: /images/favicon.png
---

最近的一个Android项目需要用到数据库的功能，之前的一个项目采用的是xutis，感觉用起来不是很方便，公司的项目我来之前同事用的基本实现
为了开发效率，决定采用greendao，看了不少介绍greendao的文章, 发现3.0前后用法变化较大，版本介绍各有不同，到了官方网站确定最新版本，开始研究

[greendao github地址](https://github.com/greenrobot/greenDAO)
[greendao 官方网站地址](http://greenrobot.org/greendao/)

<!-- more -->

# 基本配置
在项目的project的build.gradle文件中配置

```
dependencies {
        classpath 'com.android.tools.build:gradle:2.2.3'
        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'
        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
        classpath 'org.greenrobot:greendao-gradle-plugin:3.2.1'
    }
```

在项目需要用到的module的build.gradle文件中配置

```
apply plugin: 'org.greenrobot.greendao'
android {
		//greendao 数据库版本
		schemaVersion 2
		//greendao 插件生成代码位置
        targetGenDir 'src/main/java'
}
dependencies {
 compile 'org.greenrobot:greendao:3.2.0'
 }
```

# 流程概要
1. 编写数据库实体类
2. Make project编译工程，greendao插件生成代码
3. 编写代码，封装使用


# 使用示例
## 新建实体类

```
@Entity
public class Order {

    @Id
    private Long id;
    @Property
    private String userId;
    @Property
    private String OrderNumber;
    @Property
    private String phoneNumber;
    @Property
    private String PaymentType;
    @Property
    private boolean isChecked;
    @Property
    private String moudleName;
    
    //...............
}
```

## markeproject 
Androidstudio （顶部工具栏上那个绿色小锤子，想xcode学习？），
生成代码
![](http://7xqtsx.com1.z0.glb.clouddn.com/17-1-2/55678525-file_1483291295356_2436.png)
可以看见没有被git跟踪的三个棕色文件就是生成的文件
## 封装使用
新建一个DbManager.java


```
public class DBManager {

    private final static String dbName = "test_db";
    private static DBManager mInstance;
    private DaoMaster.DevOpenHelper openHelper;
    private Context context;
    private DaoSession daoSession;
    private DaoMaster daoMaster;


    public static void initialize(Context context) {
        if (mInstance == null) {
            synchronized (DBManager.class) {
                if (mInstance == null) {
                    mInstance = new DBManager(context);
                }
            }
        }


    }

    public DBManager(Context context) {
        this.context = context;
        openHelper = new DaoMaster.DevOpenHelper(context, dbName, null);
        daoMaster=new DaoMaster(openHelper.getWritableDb());
        daoSession = daoMaster.newSession();

        //打印日志
        QueryBuilder.LOG_SQL=true;
        QueryBuilder.LOG_VALUES=true;
    }

    /**
     * 获取单例引用
     *
     * @param
     * @return
     */
    public static DBManager getInstance() {


        return mInstance;
    }

    /**
     * 获取session对象
     * @return
     */
    public DaoSession getDaoSession() {
        return daoSession;
    }
}
```


在Application中进行DbManager的初始化

```
public class BaimiApplication extends Application {


    private static Context mContext;

    @Override
    public void onCreate() {
        super.onCreate();
        mContext = this;
        BasePreference.initialize(mContext);
        DBManager.initialize(mContext);
    }

    /**
     * @return
     * 全局的上下文
     */
    public static Context getmContext() {
        return mContext;
    }

}
```

DbManager里面还可以进行进一步的封装，比如一些常用操作
## 调用

### 插入数据
 这部分我还没来的及封装
 
```
private void saveDb() {

        OrderDao orderdao = DBManager.getInstance().getDaoSession().getOrderDao();
        Order order=new Order();
        order.setPaymentType(paymentType);
        order.setOrderNumber(orderEdit.getText().toString());
        order.setPhoneNumber(phoneEdit.getText().toString());
        order.setChecked(false);
        order.setUserId(BasePreference.getInstance().getLoginName());
        order.setMoudleName("快递接件");
        orderdao.insert(order);
    }
```

### 查询数据
新建了一个DbHelper进行封装，主要是同种查询条件多处调用，写的累。。
链式调研，很舒服

```
public class DbHelper {

    /**
     * @param moduleName
     * 获取当前用户订单列表
     * @return
     */
    public  static  List<Order> getUserOrderByMouleName(String moduleName){
        OrderDao orderdao = DBManager.getInstance().getDaoSession().getOrderDao();
        List<Order> orders = orderdao.queryBuilder()
                .where(OrderDao.Properties.MoudleName.eq(moduleName))
                .where(OrderDao.Properties.UserId.eq(BasePreference.getInstance().getLoginName()))
                .list();

        return orders;
    }
}
```

### 删除数据

```
OrderDao mOrderdao = DBManager.getInstance().getDaoSession().getOrderDao();
 for (Order order : mOrderList) {
            if (order.isChecked() == true) {
                mOrderdao.delete(order);
            }
        }
```

### 更新数据

也是基于实体类生成的Dao文件,利用AS智能提示查看一下方法和参数信息，就可以使用了

## 查看日志

```
01-02 01:43:45.549 31150-31150/com.wpmac.momi D/greenDAO: Built SQL for query: SELECT T."_id",T."USER_ID",T."ORDER_NUMBER",T."PHONE_NUMBER",T."PAYMENT_TYPE",T."IS_CHECKED",T."MOUDLE_NAME" FROM "ORDER" T  WHERE T."MOUDLE_NAME"=? AND T."USER_ID"=?
01-02 01:43:45.549 31150-31150/com.wpmac.momi D/greenDAO: Values for query: [快递接件, 007]
01-02 01:43:51.983 31150-31150/com.wpmac.momi D/greenDAO: Built SQL for query: SELECT T."_id",T."USER_ID",T."ORDER_NUMBER",T."PHONE_NUMBER",T."PAYMENT_TYPE",T."IS_CHECKED",T."MOUDLE_NAME" FROM "ORDER" T  WHERE T."MOUDLE_NAME"=? AND T."USER_ID"=?
01-02 01:43:51.983 31150-31150/com.wpmac.momi D/greenDAO: Values for query: [快递接件, 007]
01-02 01:43:51.987 31150-31150/com.wpmac.momi D/greenDAO: Built SQL for query: SELECT T."_id",T."USER_ID",T."ORDER_NUMBER",T."PHONE_NUMBER",T."PAYMENT_TYPE",T."IS_CHECKED",T."MOUDLE_NAME" FROM "ORDER" T  WHERE T."MOUDLE_NAME"=? AND T."USER_ID"=?
01-02 01:43:51.987 31150-31150/com.wpmac.momi D/greenDAO: Values for query: [快递接件, 007]
01-02 01:43:52.813 31150-31150/com.wpmac.momi D/greenDAO: Built SQL for query: SELECT T."_id",T."USER_ID",T."ORDER_NUMBER",T."PHONE_NUMBER",T."PAYMENT_TYPE",T."IS_CHECKED",T."MOUDLE_NAME" FROM "ORDER" T  WHERE T."MOUDLE_NAME"=? AND T."USER_ID"=?
01-02 01:43:52.813 31150-31150/com.wpmac.momi D/greenDAO: Values for query: [快递接件, 007]
01-02 01:43:52.815 31150-31150/com.wpmac.momi D/greenDAO: Built SQL for query: SELECT T."_id",T."USER_ID",T."ORDER_NUMBER",T."PHONE_NUMBER",T."PAYMENT_TYPE",T."IS_CHECKED",T."MOUDLE_NAME" FROM "ORDER" T  WHERE T."MOUDLE_NAME"=? AND T."USER_ID"=?
01-02 01:43:52.815 31150-31150/com.wpmac.momi D/greenDAO: Values for query: [快递接件, 007]
```


# 后记
总的来说，greendao的使用和后端的hibernate ORM框架使用方式类似
以后会不会出现和Mybatis类似的androidORM框架呢，希望有生之年能看见
联想到Mybatis使用时的日志配置，可以实时打印出语句和参数信息
寻找了一下greendao打印日志的方法 在DbManager.java里面已经添加了



