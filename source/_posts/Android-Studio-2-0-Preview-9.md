title: "Android Studio 2.0 Preview 9"
date: 2016-02-02 18:40:59
categories:
 - 工具
 - IDE
tags: AndroidStudio
author: mzeht
avatar: /images/favicon.png
---
一不注意android studio preview 2.0 又要升级了，升级到preview2.0 9版本

下载地址<http://tools.android.com/download/studio/builds/2-0-preview-9>
这里有最新的版本更新消息
之前的preview 2.0 的确有不可忽视的bug

<!-- more -->


>We've just pushed Android Studio 2.0 Preview 9 to the canary channel -- along with 2.0.0-alpha9 of the Gradle plugin to jcenter (and as part of the bundled offline repository within the IDE).

>In this release, we've completely turned off in-memory dexing by default. We've spent the last couple of previews trying to fine-tune it, but there are lingering issues which continues to affect users. This should hopefully make the builds work a lot better for many of you. (If things were already working well, you can continue with in-memory dexing by turning it on with android.dexOptions.dexInProcess=true.)

 **We've also continued to fix various Instant Run scenarios; in particular, using APK splits on API 23 seems to trigger some platform bugs, so for now we've switched over to using multidex for coldswap for both Lollipop and Marshmallow.**


没错，果然是bug，就是打包出问题,个人经历为生成的apk fragment资源管理混乱，主题资源编译出错，同样的打包方式，viewpage切换一个出错一个正常，过渡动画一个默认，一个自定义，导致我用回了1.5稳定版，不知道这个版本彻底解决没，从preview 9 开始 默认使用gradle2.10版本

gradle的android构建插件为
```
classpath 'com.android.tools.build:gradle:2.0.0-alpha6'
```


