title: "ConnectManager网络请求的三层封装（模块－转换－底层）"
date: 2016-03-18 14:01:47
author: mzeht
avatar: /images/favicon.png
categories: 
 - 移动开发
tags:
- Android
- 设计模式
- 网络通讯
- 封装
---

基于httpclient，java封装一套Android网络请求框架
<!-- more -->
调用层－数据解析层－底层

##调用层

```
private void getSystemToken() {
        GetSystemTokenParam param = new GetSystemTokenParam();
        param.loginId=Constants.LOGINID;
        String password=mPasswordView.getText().toString();
        String encpwd=PasswordUtil.getEncPwd(Constants.LOGINID, password);
        param.password= encpwd;
//        param.password= password;
        L.i("systemID",mUsernameView.getText().toString());
        L.i("明文密码",mPasswordView.getText().toString());
        L.i("loginID",Constants.LOGINID);
        L.i("password 加密结果为",encpwd);
        L.i("password 解密结果为",PasswordUtil.getDecPwd(Constants.LOGINID,encpwd));
        ConnectManager.getInstance().getSystemToken(param,Tokencallback);
    }


    private final AbstractRequestListener<GetSystemTokenBean> Tokencallback = new AbstractRequestListener<GetSystemTokenBean>() {

        @Override
        public void onComplete(GetSystemTokenBean bean) {
            tokenBean = bean;
            tHandler.sendEmptyMessage(ActivityForResultUtil.REQUEST_DATA_SUCCESS);
        }

        @Override
        public void onError(CustomError customError) {

            Message msg = tHandler.obtainMessage();
            msg.obj = customError;
            msg.what = ActivityForResultUtil.REQUEST_DATA_ERROR;
            tHandler.sendMessage(msg);

        }

        @Override
        public void onFault(CustomError fault) {

            Message msg = tHandler.obtainMessage();
            msg.obj = fault;
            msg.what = ActivityForResultUtil.REQUEST_DATA_FAULT;
            tHandler.sendMessage(msg);
        }

    };
```


##数据转化层
```
 /**
     * 请求systemToken
     *
     * @param params
     * @param callback
     */
    public void getSystemToken(final GetSystemTokenParam params,
                             final AbstractRequestListener<GetSystemTokenBean> callback) {
        exector.execute(new Runnable() {

            @Override
            public void run() {

                try {
                    String method = params.getParam().getString("method");
                    String response = HttpConnectionUtil.openUrl(
                            Constants.SERVER_URL + method, "GET",
                            params.getParam(),false);
                    GetSystemTokenBean bean = GetSystemTokenBean.getSystemTokenBean(response);
                    if (callback != null) {
                        callback.onComplete(bean);
                    }
                } catch (RuntimeException e) {
                    e.printStackTrace();
                    if (callback != null) {
                        callback.onFault(new CustomError(network_error));
                    }
                } catch (JSONException e) {
                    e.printStackTrace();
                    if (callback != null) {
                        callback.onError(new CustomError(json_error));
                    }
                }
            }
        });

    }
```

##网络底层
```
 /**
     * 发送http请求
     *
     * @param url
     * @param method
     *            GET 或 POST
     * @param params
     * @return
     */
    public static String openUrl(String url, String method, Bundle params,boolean Token) {
        if (method.equals("GET")) {
            if (params != null) {
                url = url + "?" + encodeUrl(params);
            }
        }
        String response = "";

        try {
            L.i(method + " URL: " + url);
            HttpClient httpClient = HttpConnectionUtil.getNewHttpClient();
            HttpGet httpGet = new HttpGet(url);
            if(Token==false){

            }else{

                httpGet.setHeader("systemToken", Constants.SYSTEMTOKEN);
            }
            //连接超时
            httpClient.getParams().setParameter(CoreConnectionPNames.CONNECTION_TIMEOUT, 20000);
            // 读取超时
            httpClient.getParams().setParameter(CoreConnectionPNames.SO_TIMEOUT, 20000);

            HttpResponse resp = httpClient.execute(httpGet);

            if (resp.getStatusLine().getStatusCode() != HttpStatus.SC_OK) {
                L.i(LOG_TAG, "httpGet fail, status code = " + resp.getStatusLine().getStatusCode());

            }else{
                response = EntityUtils.toString(resp.getEntity());

            }
        } catch (Exception e) {
            throw new RuntimeException(e.getMessage(), e);
        }

        L.i("LOG_TAG"+response);
        return response;
    }
```

#具体封装
##AbstractRequestListener
public abstract class AbstractRequestListener<T extends Object>{


	/**
	 * 将response请求解析为针对具体请求的bean
	 *
	 * @param response
	 * 			请求完成后的响应字符串
	 * @return
	 * 			若解析成功，返回解析后的对象，否则返回null
	 */
	@SuppressWarnings("unchecked")
    public T parse(String response) {
	    Class<?> c = this.getGenericType();
        try {
            Constructor<T> constructor = (Constructor<T>) c.getDeclaredConstructor(String.class);
            T result = constructor.newInstance(response);
            return result;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
	}

	/**
	 * 获取T的类型
	 * @param
	 * @return
	 */
	private Class<?> getGenericType() {
        Type genType = getClass().getGenericSuperclass();
        if (!(genType instanceof ParameterizedType)) {
            return Object.class;
        }
        Type[] params = ((ParameterizedType) genType).getActualTypeArguments();
        if (params.length < 1) {
            throw new RuntimeException("Index outof bounds");
        }
        if (!(params[0] instanceof Class)) {
            return Object.class;
        }
        return (Class<?>) params[0];
    }

	 /**
     * 请求完成后以对象形式返回服务器的响应的结果
     *
     * @param bean
     * 			服务器返回的响应字符串解析后得到的对象
     *        
     */
    public abstract void onComplete(T bean);

    /**
     * 服务器返回了错误结果，已经正确的链接上了服务器但有错误如：缺少参数.等
     *
     * @param customError
     */
    public abstract void onError(CustomError customError);

    /**
     * 在请求期间发生了严重问题（如：网络故障、访问的地址不存在等）
     *
     * @param fault
     */
    public abstract void onFault(CustomError fault);


}

##CustomError
```
public class CustomError extends RuntimeException {

	private static final long serialVersionUID = 1L;
    /**
     * 错误码
     */
	private int mErrorCode;

	/** 原始响应URL */
	private String mResponse;

	public CustomError() {
		super();
	}
	public CustomError(String errorMessage) {
		super(errorMessage);
	}

	public CustomError(int errorCode, String errorMessage, String orgResponse) {
		super(errorMessage);
		this.mErrorCode = errorCode;
		this.mResponse = orgResponse;
	}

	public String getOrgResponse() {
		return mResponse;
	}

	public int  getErrorCode() {
		return mErrorCode;
	}
	@Override
	public String toString() {
		return "errorCode:" + this.mErrorCode + "\nerrorMessage:"
				+ this.getMessage() + "\norgResponse:" + this.mResponse;
	}

	/**
	 * 将服务器返回的errorMessage转换成定义的易于理解的字符串
	 *
	 * @param errorCode
	 * 			服务器返回的错误代码
	 * @param errorMessage
	 * 			服务器返回的错误字符串，和错误代码一一对应
	 * @return
	 */
	public static String interpretErrorMessage(int errorCode, String errorMessage) {
		switch (errorCode)
		{
			/**
			 * 错误处理
			 */
		}

		return errorMessage;
	}
}
```

##RequestParam
```
 */
public abstract class RequestParam {


	/**
	 * 设置请求参数
	 * @return
	 */
    public abstract Bundle getParam();

}
```

##ResponseBean
```
public abstract class ResponseBean implements Serializable {

	public String response;
	//

	public ResponseBean()
	{

	}
	public ResponseBean(String response) throws JSONException {
		// TODO Auto-generated constructor stub

		if(response == null || response.equals("") ||response.equals("null"))
			throw new JSONException("数据解析异常");

		this.response = response;

	}

	public void setResponse(String response){
		this.response = response;
	}

	public String getResponse() {
		return response;
	}
	/**
	 * 保存
	 */
	protected  void saveModel()
	{
		   L.i("save_model");
	}

	/**
	 * 更新
	 */
	protected  void updateModel()
	{
		 L.i("update_model");
	}


	/**
	 * 删除
	 */
	protected   void deleteModel()
	{
		 L.i("delete_model");
	}



}

```

#用法
参见Demo，给别人做的毕业设计。

Demo地址<https://github.com/mzeht/ZhiHuiYun>


