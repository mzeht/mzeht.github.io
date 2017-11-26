title: android 模块化统一管理依赖库版本号
categories: 技术
author: mzeht
avatar: /images/favicon.png
authorDesc: 一个写代码的
date: 2017-06-01 12:39:20
tags:
keywords:
description:
photos:
---

在android项目开发中 不断提炼出公用模块，方便以后的开发引入，很是方便，但是编译过程中不可避免遇到版本冲突的情况，
最常见的是sdk版本 v4 v7库版本，这种情况下 可以利用gradle配置统一管理

在项目根目录下新建 `common_config.gradle`

## 配置统一的版本号


```
/**
 * 所有Module 的共享变量，这里修改了，全部地方都能修改
 */
project.ext {
    //1.关于SDK version
    compileSdkVersion = 25
    buildToolsVersion = '25.0.3'

    minSdkVersion = 16
    targetSdkVersion = 25

    //2.关于Debug和Realease 的不同配置

    //3.指定Java version
    sourceCompatibility = JavaVersion.VERSION_1_8
    targetCompatibility = JavaVersion.VERSION_1_8

    //4.support Lib Version ,最好全部使用同一个版本号
    supportLibraryVersion = '25.3.1'

    //5.



}
```

## 引入
在project中引入上述文件


```// Top-level build file where you can add configuration options common to all sub-projects/modules.

buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.3.2'
//        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'
        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
        classpath 'org.greenrobot:greendao-gradle-plugin:3.2.1'
        classpath 'com.jakewharton:butterknife-gradle-plugin:8.4.0'
        //lambda
        classpath 'me.tatarka:gradle-retrolambda:3.6.0'
        //lombok
        classpath 'me.tatarka.retrolambda.projectlombok:lombok.ast:0.2.3.a2'

    }
    configurations.classpath.exclude group: 'com.android.tools.external.lombok'
}

/**
 * 工程中所有的module 都将能使用common_config 中的配置
 */
subprojects {
    apply from: "${project.rootDir}/common_config.gradle"
    dependencies {
//        testCompile 'junit:junit:4.12'
    }
}

allprojects {
    repositories {
        jcenter()
        maven { url "https://jitpack.io" }
       
        mavenCentral()
    }

}

task clean(type: Delete) {
    delete rootProject.buildDir
}

```

## 使用


```
 compileSdkVersion project.ext.compileSdkVersion
    buildToolsVersion project.ext.buildToolsVersion
    
    
    minSdkVersion project.ext.minSdkVersion
        targetSdkVersion project.ext.targetSdkVersion
        
        
        compile "com.android.support:appcompat-v7:$project.ext.supportLibraryVersion"
    compile "com.android.support:design:$project.ext.supportLibraryVersion"
```


