title: 升级到rxjava2+retrofit2
categories: 
  - 移动开发
  - Android
author: mzeht
avatar: /images/favicon.png
authorDesc: 一个写代码的
date: 2017-06-01 12:22:18
tags:
keywords:
description:
photos:
---

之前项目使用rxjava1和retorfit2，抽空在新项目升级到韧性java2


## 依赖文件
```
    
    compile 'com.squareup.okhttp3:okhttp:3.4.2'
    compile 'com.google.code.gson:gson:2.7'
    compile 'io.reactivex.rxjava2:rxjava:2.0.7'
    compile 'io.reactivex.rxjava2:rxandroid:2.0.1'
    compile 'com.squareup.retrofit2:retrofit:2.3.0'
    compile 'com.squareup.retrofit2:converter-gson:2.2.0'
//    compile 'com.squareup.retrofit2:adapter-rxjava:2.1.0'
//    compile 'com.jakewharton.retrofit:retrofit2-rxjava2-adapter:1.0.0'
    compile 'com.squareup.retrofit2:adapter-rxjava2:2.2.0'
    compile 'com.squareup.retrofit2:retrofit-converters:2.1.0'
    compile 'com.squareup.okhttp3:logging-interceptor:3.4.2'
//    compile 'com.trello:rxlifecycle:1.0'
    compile 'com.trello.rxlifecycle2:rxlifecycle:2.1.0'

//    compile 'com.trello:rxlifecycle-android:1.0'
    compile 'com.trello.rxlifecycle2:rxlifecycle-android:2.1.0'

//    compile 'com.trello:rxlifecycle-components:1.0'
    compile 'com.trello.rxlifecycle2:rxlifecycle-components:2.1.0'
```

1.`rxlifecycle` 也升级到`rxlifecycle2` 一个rxjava生命周期管理库

2.retrofit2 支持rxjava2之前

采用

```
 compile 'com.jakewharton.retrofit:retrofit2-rxjava2-adapter:1.0.0'
```

之后

```
compile 'com.squareup.retrofit2:adapter-rxjava2:2.2.0'
```











