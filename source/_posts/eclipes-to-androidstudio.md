title: "Android project from eclipes to androidstudio"
date: 2016-02-25 14:55:23
categories:
 - 工具
 - IDE
tags:
 - debug
author: mzeht
avatar: /images/favicon.png
---
##jar包冲突
>Error:Execution failed for task ':app:transformResourcesWithMergeJavaResForDebug'.
com.android.build.api.transform.TransformException: com.android.builder.packaging.DuplicateFileException: Duplicate files copied in APK META-INF/LICENSE.txt
  	File1: /Users/wpmac/AndroidStudioProjects/Android_XinJiang/app/libs/httpmime-4.1.1.jar
  	File2: /Users/wpmac/AndroidStudioProjects/Android_XinJiang/app/libs/commons-codec-1.4.jar
  	
  	
  	
 ```
  	
  	packagingOptions {
        exclude 'META-INF/DEPENDENCIES'
        exclude 'META-INF/NOTICE'
        exclude 'META-INF/LICENSE'
        exclude 'META-INF/LICENSE.txt'
        exclude 'META-INF/NOTICE.txt'
    }
    
 ```
    
   
##图片资源报错 
   > AAPT: /Users/wpmac/AndroidStudioProjects/Android_JieQian/app/src/main/res/drawable-xhdpi/suspected_association_normal.png: libpng warning: iCCP: Not recognizing known sRGB profile that has been edited
AAPT err(Facade for 203993302) : No Delegate set : lost message:/Users/wpmac/AndroidStudioProjects/Android_JieQian/app/src/main/res/drawable-xhdpi/register_info_normal.png: libpng warning: iCCP: Not recognizing known sRGB profile that has been edited
AAPT: /Users/wpmac/AndroidStudioProjects/Android_JieQian/app/src/main/res/drawable-xxhdpi/attention_pressed.png: libpng warning: iCCP: Not recognizing known sRGB profile that has been edited
AAPT: /Users/wpmac/AndroidStudioProjects/Android_JieQian/app/src/main/res/drawable-hdpi/splash_imageview.png: libpng warning: iCCP: Not recognizing known sRGB profile that has been edited
AAPT: /Users/wpmac/AndroidStudioProjects/Android_JieQian/app/src/main/res/drawable-xhdpi/province_select.png: libpng warning: iCCP: Not recognizing known sRGB profile that has been edited

buildtools 23.0.2  warning: iCCP: Not recognizing known sRGB profile that has been edited

21.1.1 no warning   iCCP: Not recognizing known sRGB profile that has been edited


not a png file :check png file 


