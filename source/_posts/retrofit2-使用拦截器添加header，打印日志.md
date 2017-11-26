title: retrofit2 使用拦截器添加header，打印日志
categories: 技术
author: mzeht
avatar: /images/favicon.png
authorDesc: 一个写代码的
date: 2017-06-01 12:47:47
tags:
keywords:
description:
photos:
---

开发中可能遇到有的接口需要header传递一些信息，几个还好，可以用@header注解手动添加

如果每个接口都需要，一个个写header就很傻了，可以使用retrofit2的拦截器做到

## 常用配置


```
OkHttpClient.Builder builder = new OkHttpClient.Builder();


        builder.connectTimeout(5, TimeUnit.SECONDS)
                .writeTimeout(5, TimeUnit.SECONDS)
                .readTimeout(5,TimeUnit.SECONDS)
                .addNetworkInterceptor(new StethoInterceptor())
        .addInterceptor(new Interceptor() {
            @Override
            public Response intercept(Chain chain) throws IOException {
                L.d("Interceptor");

                // Request customization: add request headers
                String Nonce = String.valueOf(System.currentTimeMillis());
                String CheckSum = EncryptUtils.encryptSHA1ToString("DA3B979ABB5A4FC0B65382BC1A344A84"+Nonce);
                Request.Builder requestBuilder = chain.request().newBuilder()
                        .addHeader("AppKey", "10000001")
                        .addHeader("Nonce", Nonce)
                        .addHeader("CheckSum", CheckSum)
                        .addHeader("x-session-token", BasePreference.getInstance().getToken())
                        .addHeader("AppSecret", "XXXXXXXXXXXXXXXXXXXXXXXXXXXXX");

                Request request = requestBuilder.build();
                return chain.proceed(request);
            }
        });
        if (BuildConfig.DEBUG) {
            // Log信息拦截器
            HttpLoggingInterceptor loggingInterceptor = new HttpLoggingInterceptor();
            loggingInterceptor.setLevel(HttpLoggingInterceptor.Level.BODY);
            //设置 Debug Log 模式
            builder.addInterceptor(loggingInterceptor);
        }

        OkHttpClient client = builder.build();

        Retrofit retrofit_baimi = new Retrofit.Builder()
                .baseUrl(BASE_URL+"/")
                .client(client)
                .addConverterFactory(CustomGsonConverterFactory.create())
                .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
                .build();
        mSmartApi = retrofit_baimi.create(SmartApi.class);
```


OkHttpClient 采用build模式 链式设置


```
addInterceptor(new Interceptor() {
            @Override
            public Response intercept(Chain chain) throws IOException {
                L.d("Interceptor");

                // Request customization: add request headers
                String Nonce = String.valueOf(System.currentTimeMillis());
                String CheckSum = EncryptUtils.encryptSHA1ToString("DA3B979ABB5A4FC0B65382BC1A344A84"+Nonce);
                Request.Builder requestBuilder = chain.request().newBuilder()
                        .addHeader("AppKey", "10000001")
                        .addHeader("Nonce", Nonce)
                        .addHeader("CheckSum", CheckSum)
                        .addHeader("x-session-token", BasePreference.getInstance().getToken())
                        .addHeader("AppSecret", "XXXXXXXXXXXXXXXXXXXXXXXXXXXXX");

                Request request = requestBuilder.build();
                return chain.proceed(request);
            }
        });
```

拦截请求 加入我们的header


```
if (BuildConfig.DEBUG) {
            // Log信息拦截器
            HttpLoggingInterceptor loggingInterceptor = new HttpLoggingInterceptor();
            loggingInterceptor.setLevel(HttpLoggingInterceptor.Level.BODY);
            //设置 Debug Log 模式
            builder.addInterceptor(loggingInterceptor);
        }
```

如果是开发配置下 拦截body级别日志 打印信息方便调试

`注意` 日志拦截器建议最后添加 才能拦截到添加header之后的日志






