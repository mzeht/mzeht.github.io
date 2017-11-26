title: Android Studio Gradle 进阶设置
categories: 
 - 移动开发
 - Android
tags:
  - AndroidStudio
  - Gradle
date: 2016-09-07 01:10:55
author: mzeht
avatar: /images/favicon.png
---

##配置独立的签名信息
一般来说，我们可以直接把签名的信息明文写在gradle配置文件中，但是在多人协作或者开源代码的时候不是很方便，前者可能存放签名文件的位置需要修改，后者则希望隐藏签名信息，在看不少开源项目的时候，发现很多是这样做的

<!-- more -->

signingConfigs位置， `android的下一级`

```
signingConfigs {
        release {
            storeFile file(RELEASE_STORE_FILE)
            storePassword RELEASE_STORE_PASSWORD
            keyAlias RELEASE_KEY_ALIAS
            keyPassword RELEASE_KEY_PASSWORD
        }

```

在对应位置引用常量，在properties中再进行具体赋值
在gradle.properties中

```
RELEASE_KEY_PASSWORD=xxxx
RELEASE_KEY_ALIAS=xxx
RELEASE_STORE_PASSWORD=xxx
RELEASE_STORE_FILE=../.keystore/xxx.jks
```

这样不会直接暴露签名信息，也方便配置

##多渠道打包

在product flavor下定义不同类型, 把AndroiManifest中的channel替换

productFlavors位置： `android的下一级`

```
android {
      productFlavors {
        fir {
            manifestPlaceholders = [UMENG_CHANNEL_VALUE: "fir"]
        }
        GooglePlay {
            manifestPlaceholders = [UMENG_CHANNEL_VALUE: "GooglePlay"]
        }
        Umeng {
            manifestPlaceholders = [UMENG_CHANNEL_VALUE: "Umeng"]
        }
       
    }
}
```

如果flavor标识是数字开头，要加以处理，添加""

替换AndroiManifest中的字段

```
        <meta-data
            android:name="UMENG_CHANNEL"
            android:value="${UMENG_CHANNEL_VALUE}"/>
```

##定制生成apk文件名称

```
def buildTime() {
    return new Date().format("yyyy-MM-dd HH:mm:ss")
}
def hostName() {
    return System.getProperty("user.name") + "@" + InetAddress.localHost.hostName
}


android {

   //...
   
    productFlavors {
        fir {
            manifestPlaceholders = [UMENG_CHANNEL_VALUE: "fir"]
        }
        GooglePlay {
            manifestPlaceholders = [UMENG_CHANNEL_VALUE: "GooglePlay"]
        }
        Umeng {
            manifestPlaceholders = [UMENG_CHANNEL_VALUE: "Umeng"]
        }

    }

   buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            signingConfig signingConfigs.app1
        }

    }

    applicationVariants.all { variant ->
        variant.outputs.each { output ->
            def outputFile = output.outputFile
            if (outputFile != null && outputFile.name.endsWith('.apk')) {

                def fileName = "mzeht_v${defaultConfig.versionName}_${hostName()}_${buildTime()}_${variant.productFlavors[0].name}.apk"
                output.outputFile = new File(outputFile.parent, fileName)
            }
        }
    }

    }
```

同样注意层级关系,在android同级下定义一个时间函数和一个host函数并引用，生成的apk包如下：

![](http://7xqtsx.com1.z0.glb.clouddn.com/16-9-7/53721167.jpg)


##配置依赖版本号
在Maven中我们可以方便的定义spring等核心依赖的版本号，便于统一管理，在Gradle中我们同样可以做到

在看google的架构综合项目时[todo-mvp](https://github.com/googlesamples/android-architecture/tree/todo-mvp/),看见了一种写法

在根目录下的build.gradle文件中

```
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.1.2'

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

allprojects {
    repositories {
        jcenter()
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}

// Define versions in a single place
ext {
    // Sdk and tools
    minSdkVersion = 10
    targetSdkVersion = 22
    compileSdkVersion = 23
    buildToolsVersion = '23.0.2'

    // App dependencies
    supportLibraryVersion = '23.4.0'
    guavaVersion = '18.0'
    junitVersion = '4.12'
    mockitoVersion = '1.10.19'
    powerMockito = '1.6.2'
    hamcrestVersion = '1.3'
    runnerVersion = '0.4.1'
    rulesVersion = '0.4.1'
    espressoVersion = '2.2.1'
}
```

然后在app的build.gradle中

```
apply plugin: 'com.android.application'

android {
    compileSdkVersion rootProject.ext.compileSdkVersion
    buildToolsVersion rootProject.ext.buildToolsVersion

    defaultConfig {
        applicationId "com.example.android.architecture.blueprints.todomvp"
        minSdkVersion rootProject.ext.minSdkVersion
        targetSdkVersion rootProject.ext.targetSdkVersion
        versionCode 1
        versionName "1.0"

        testInstrumentationRunner 'android.support.test.runner.AndroidJUnitRunner'
    }

    buildTypes {
        debug {
            minifyEnabled true
            // Uses new built-in shrinker http://tools.android.com/tech-docs/new-build-system/built-in-shrinker
            useProguard false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            testProguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguardTest-rules.pro'
        }

        release {
            minifyEnabled true
            useProguard true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            testProguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguardTest-rules.pro'
        }
    }

    // If you need to add more flavors, consider using flavor dimensions.
    productFlavors {
        mock {
            applicationIdSuffix = ".mock"
        }
        prod {

        }
    }

    // Remove mockRelease as it's not needed.
    android.variantFilter { variant ->
        if(variant.buildType.name.equals('release')
                && variant.getFlavors().get(0).name.equals('mock')) {
            variant.setIgnore(true);
        }
    }

    // Always show the result of every unit test, even if it passes.
    testOptions.unitTests.all {
        testLogging {
            events 'passed', 'skipped', 'failed', 'standardOut', 'standardError'
        }
    }
}

/*
 Dependency versions are defined in the top level build.gradle file. This helps keeping track of
 all versions in a single place. This improves readability and helps managing project complexity.
 */
dependencies {
    // App's dependencies, including test
    compile "com.android.support:appcompat-v7:$rootProject.supportLibraryVersion"
    compile "com.android.support:cardview-v7:$rootProject.supportLibraryVersion"
    compile "com.android.support:design:$rootProject.supportLibraryVersion"
    compile "com.android.support:recyclerview-v7:$rootProject.supportLibraryVersion"
    compile "com.android.support:support-v4:$rootProject.supportLibraryVersion"
    compile "com.android.support.test.espresso:espresso-idling-resource:$rootProject.espressoVersion"
    compile "com.google.guava:guava:$rootProject.guavaVersion"

    // Dependencies for local unit tests
    testCompile "junit:junit:$rootProject.ext.junitVersion"
    testCompile "org.mockito:mockito-all:$rootProject.ext.mockitoVersion"
    testCompile "org.hamcrest:hamcrest-all:$rootProject.ext.hamcrestVersion"

    // Android Testing Support Library's runner and rules
    androidTestCompile "com.android.support.test:runner:$rootProject.ext.runnerVersion"
    androidTestCompile "com.android.support.test:rules:$rootProject.ext.runnerVersion"

    // Dependencies for Android unit tests
    androidTestCompile "junit:junit:$rootProject.ext.junitVersion"
    androidTestCompile "org.mockito:mockito-core:$rootProject.ext.mockitoVersion"
    androidTestCompile 'com.google.dexmaker:dexmaker:1.2'
    androidTestCompile 'com.google.dexmaker:dexmaker-mockito:1.2'

    // Espresso UI Testing
    androidTestCompile "com.android.support.test.espresso:espresso-core:$rootProject.espressoVersion"
    androidTestCompile ("com.android.support.test.espresso:espresso-contrib:$rootProject.espressoVersion")
    androidTestCompile "com.android.support.test.espresso:espresso-intents:$rootProject.espressoVersion"

    // Resolve conflicts between main and test APK:
    androidTestCompile "com.android.support:support-annotations:$rootProject.supportLibraryVersion"
    androidTestCompile "com.android.support:support-v4:$rootProject.supportLibraryVersion"
    androidTestCompile "com.android.support:recyclerview-v7:$rootProject.supportLibraryVersion"

```

##减少编译错误和忽略lint检查

```
packagingOptions {
        exclude 'META-INF/DEPENDENCIES.txt'
        exclude 'META-INF/LICENSE.txt'
        exclude 'META-INF/NOTICE.txt'
        exclude 'META-INF/NOTICE'
        exclude 'META-INF/LICENSE'
        exclude 'META-INF/DEPENDENCIES'
        exclude 'META-INF/notice.txt'
        exclude 'META-INF/license.txt'
        exclude 'META-INF/dependencies.txt'
        exclude 'META-INF/LGPL2.1'
    }

 

    lintOptions {
        abortOnError false
    }
```


##结语
gralde设置结合AndroidStudio提供的图形界面设置，理解会更快更透彻

File-Project Structure-Modules-app

![](http://7xqtsx.com1.z0.glb.clouddn.com/16-9-7/37008097.jpg)





