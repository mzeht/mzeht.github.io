title: butterknife 8.0.1 AndroidStudio peizhi
categories:
 - 移动开发
 - Android
tags:
  - AndroidStudio
  - Butterknife
date: 2016-09-08 19:18:29
author: mzeht
avatar: /images/favicon.png
---

#前言
butterknife是一个android上的注解库，升级的很快，在7.0.1版本中只要添加依赖就可以了，但是8.0.1版本中配置稍有不同

<!-- more -->

##在project 下的build.gradle文件下配置classpath

```
buildscript {
    repositories {
        jcenter()
    }
    dependencies {

        classpath 'com.android.tools.build:gradle:2.1.3'
        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

```
##在moudle下的builde.gradle文件中配置plugin

```
apply plugin: 'com.android.application'
apply plugin: 'com.neenbedankt.android-apt'
```


##在moudle下的builde.gradle文件中配置dependencies

```
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar']
    compile 'com.jakewharton:butterknife:8.0.1'
    apt 'com.jakewharton:butterknife-compiler:8.0.1'
}
```


butterknife之所以配置变化这么大，主要是在于apt，取代了之前的注解实现方法







