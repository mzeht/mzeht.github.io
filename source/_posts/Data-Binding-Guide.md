title: Data Binding Guide
categories:
 - 移动开发
 - Android
tags:
  - Data Binding
date: 2016-09-04 14:37:50
author: mzeht
avatar: /images/favicon.png
---

![](http://7xqtsx.com1.z0.glb.clouddn.com/16-9-4/6040336.jpg)

##前言
2016的Google IO 大会上，Android 团队发布了一个数据绑定框架（Data Binding Library）。以后可以直接在 layout 布局 xml 文件中绑定数据了，无需再 findViewById 然后手工设置数据了。其语法和使用方式和 JSP 中的 EL 表达式非常类似。


看到google出品的[Data Binding Library](https://developer.android.com/topic/libraries/data-binding/index.html)本来是想边翻译边写写demo，但是csdn上已经有人翻译了，没必要重复劳动，直接拿来链接：
<!-- more -->



# [Data Binding Guide——google官方文档翻译（上）](http://blog.csdn.net/u014486880/article/details/50508133)

#[ Data Binding Guide——google官方文档翻译（下）](http://blog.csdn.net/u014486880/article/details/50531110)

##防坑指南

引入DataBindingLibrary从最初发布开始，已经有了变化，起初是

在项目的 gradle 配置文件中添加依赖项：

```
dependencies { classpath "com.android.tools.build:gradle:1.2.3" classpath "com.android.databinding:dataBinder:1.0-rc0" }
```

上面的依赖项目前在 jcenter 服务器中，所以确保 repositories 中包含 jcenter。
allprojects { repositories { jcenter() }}


在每个需要使用 Data Binding 功能的模块的 gradle 配置文件中启用该插件（添加在 android 插件之后），

```
apply plugin: ‘com.android.application'apply plugin: 'com.android.databinding'
```

但是现在的google文档中 只要

* To get started with Data Binding, download the library from the Support repository in the Android SDK manager.

* To configure your app to use data binding, add the dataBinding element to your build.gradle file in the app module.

* Use the following code snippet to configure data binding:

```
android {
    ....
    dataBinding {
        enabled = true
    }
}
```


[demo github 地址](https://github.com/jingle1267/DataDindingSample)

