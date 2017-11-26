title: Spring boot整合retrofit简化三方接口调用
categories: 
  - 后端开发
  - java
tags:
  - Spring boot
  - retrofit
  - okHttp
date: 2017-01-22 15:05:33
author: mzeht
avatar: /images/favicon.png
---

公司改用srpingboot微服务架构后，服务之间的调用十分频繁，而同事的写法都是基于httpclient的简单工具类，写起来比较混乱，手动调用json转换为实体类，后期维护不方便，而且接口地址，参数（手动拼接）不好管理，于是想到了我在android中常常使用的retrofit+okhttp，于是动手移植到springboot项目上

<!-- more -->

# 引入依赖

```
<dependency>
			<groupId>com.squareup.retrofit2</groupId>
			<artifactId>retrofit</artifactId>
			<version>2.1.0</version>
		</dependency>

		<dependency>
			<groupId>com.squareup.retrofit2</groupId>
			<artifactId>converter-gson</artifactId>
			<version>2.1.0</version>
		</dependency>


		<dependency>
			<groupId>com.squareup.okhttp3</groupId>
			<artifactId>logging-interceptor</artifactId>
			<version>3.1.2</version>
		</dependency>
```

从上到下为retrofit基本框架，gson解析器，日志拦截器

## 初步实现方法（不结合spring特性）
提炼返回格式，使用泛型

```
public class BasicResult<T> implements Serializable {

	private static final long serialVersionUID = 517161603122400300L;
	private boolean success = false;
	private String message;
	private T data;

	public BasicResult(boolean success , String message){
		this.success = success ;
		this.message = message ;
	}

	public BasicResult(boolean success , String message, T data){
		this.success = success ;
		this.message = message ;
		this.data = data ;
	}

	/**
	 * @return the success
	 */
	public final boolean isSuccess() {
		return success;
	}

	/**
	 * @param success
	 *            the success to set
	 */
	public final void setSuccess(boolean success) {
		this.success = success;
	}

	/**
	 * @return the message
	 */
	public final String getMessage() {
		return message;
	}

	/**
	 * @param message
	 *            the message to set
	 */
	public final void setMessage(String message) {
		this.message = message;
	}

	public T getData() {
		return data;
	}

	public void setData(T data) {
		this.data = data;
	}


}
```


定义请求体

```
public class Useregister {

    private String companyName;	//公司名称	string
    private String emailAddress;    //邮箱地址	string
    private String password; //登录密码;	string
    private String phoneNumber;	//手机号码	string
    private String verificationCode;	//验证码

    ....
```

定义api接口

```

public interface UserApi {

    @POST("/common/user/v1/register")
    Call<BasicResult> listRepos(@Body Useregister useregister);
}
```


构造api接口

```
public class ApiRetrofit {

    public UserApi userApi;
    public static final String USER_BASE_URL = "http://10.10.168.79:10100/";

    public UserApi getUserApi() {
        return userApi;
    }

    ApiRetrofit() {
        HttpLoggingInterceptor logInterceptor = new HttpLoggingInterceptor(new HttpLoggingInterceptor.Logger() {
            public void log(String s) {
                //打印日志
            }
        });
        logInterceptor.setLevel(HttpLoggingInterceptor.Level.BODY);
        OkHttpClient client = new OkHttpClient.Builder()
                .addInterceptor(logInterceptor)
                .build();
        Retrofit retrofit_user = new Retrofit.Builder()
                .baseUrl(USER_BASE_URL)
                .client(client)
                .addConverterFactory(GsonConverterFactory.create())
                .build();
        userApi = retrofit_user.create(UserApi.class);

    }
}
```

工厂类提供单列api

```
public class ApiFactory {
    protected static final Object monitor = new Object();
    static UserApi userApi = null;

    public static UserApi getUserApi() {
        synchronized (monitor) {
            if (userApi == null) {
                userApi = new ApiRetrofit().getUserApi();
            }
            return userApi;
        }
    }
}
```

较为通用的java实现，没有结合spring注入的特性，

实际调用


```
Useregister useregister=new Useregister();
Call<BaseResult> res=ApiFactory.getUserApi.listRepos(useregister);
Response<BaseResult> response = call.execute();
if (!response.isSuccessful()) {
            throw new Exception(response.message());
} else {
            return response.body();
}
```

BaseResult里面传入实体类也是OK的，可以直接拿到解析好的实体对象，如



```
Call<ApiResult<UserBean>> call = apiRetrofit.getCommonApi().info(userParam);
        Response<ApiResult<UserBean>> response = call.execute();
        if (!response.isSuccessful()) {
            throw new Exception(response.message());
        } else {
            return response.body();
        }
```

## 结合springboot，去除工厂类，改为注入

以上实现在非spring框架下也可用，但是既然用了spring，就要好好利用，spring框架下还有工厂类代码，有点说不过去


```

@Component
public class ApiRetrofit {

    private static final Logger log = Logger.getLogger(ApiRetrofit.class);

    private CommonApi commonApi;

    public CommonApi getCommonApi() {
        return commonApi;
    }

    private CorpApi corpApi;

    public CorpApi getCorpApi(){
        return corpApi;
    }

    @Value("${api.common}")
    private  String COMMON_BASE_URL ;//= "http://10.10.168.79:10100/";

    @Value("${api.corp}")
    private String CROP_BASE_URL;

    @PostConstruct
    public void init(){
        HttpLoggingInterceptor logInterceptor = new HttpLoggingInterceptor(new HttpLogger());
        logInterceptor.setLevel(HttpLoggingInterceptor.Level.BODY);

        OkHttpClient client = new OkHttpClient.Builder()
                .addNetworkInterceptor(logInterceptor)
                .build();

        Retrofit retrofit_common = new Retrofit.Builder()
                .baseUrl(COMMON_BASE_URL)
                .client(client)
                .addConverterFactory(GsonConverterFactory.create())
                .build();
        commonApi = retrofit_common.create(CommonApi.class);

        Retrofit retrofit_corp = new Retrofit.Builder()
                .baseUrl(CROP_BASE_URL+"/")
                .client(client)
                .addConverterFactory(GsonConverterFactory.create())
                .build();
        corpApi = retrofit_corp.create(CorpApi.class);

    }


    private class HttpLogger implements HttpLoggingInterceptor.Logger {
//        private StringBuilder mMessage = new StringBuilder();

        @Override
        public void log(String message) {
            log.info(message);
//
//            if (message.startsWith("--> POST")) {
//                mMessage.setLength(0);
//                mMessage.append("\n");
//            }
//            // 以{}或者[]形式的说明是响应结果的json数据，需要进行格式化
//            if ((message.startsWith("{") && message.endsWith("}"))
//                    || (message.startsWith("[") && message.endsWith("]"))) {
//                message = JSONUtil.formatJson(message);
//            }
//            mMessage.append(message.concat("\n"));
//            // 请求或者响应结束，打印整条日志
//            if (message.startsWith("<-- END HTTP")) {
//                log.info(mMessage.toString());
//            }

        }
    }
}
```

@Value注入配置的baseurl的值
@Component 申明注入
@PostConstruct 在构造方法前初始化相关变量

实际调用

```

/**
     * 获取用户信息
     * @param phone
     * @return
     * @throws Exception
     */
    public ApiResult<UserBean> info(String phone) throws Exception {
        if (StringUtils.isEmpty(phone))
            throw new Exception("手机号不能为空");
        UserParam userParam = new UserParam();
        userParam.setPhoneNumber(phone);
        Call<ApiResult<UserBean>> call = apiRetrofit.getCommonApi().info(userParam);
        Response<ApiResult<UserBean>> response = call.execute();
        if (!response.isSuccessful()) {
            throw new Exception(response.message());
        } else {
            return response.body();
        }
    }
```


## 分页请求泛型


```

@JsonInclude(JsonInclude.Include.NON_NULL)
public class ApiPagesResult<T> implements Serializable{


    private static final long serialVersionUID = -8421200997970330114L;
    @JsonProperty("success")
    private boolean success = false;
    @JsonProperty("message")
    private String message;
    @JsonProperty("data")
    private Data<T> data;

    

}
```



```


@JsonInclude(JsonInclude.Include.NON_NULL)
public class Data<R>  implements Serializable{

    @JsonProperty("total")
    private Integer total;

    @JsonProperty("currentPage")
    private Integer currentPage;

    @JsonProperty("pageSize")
    private Integer pageSize;

    @JsonProperty("pages")
    private Integer pages;

    @JsonProperty("dataList")
    private List<R> dataList;


```







1

