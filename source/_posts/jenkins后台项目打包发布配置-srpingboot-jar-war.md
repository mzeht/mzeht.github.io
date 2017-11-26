title: 'jenkins项目打包发布配置(srpingboot-jar,war,apk)'
categories:
  - 后端开发
  - java
author: mzeht
avatar: /images/favicon.png
authorDesc: 一个写代码的
date: 2017-05-28 16:03:37
tags:
keywords:
description:
photos:
---
## 两种不同打包
jar类 springboot项目
1. springboot框架
2. 生成jar包
3. 内置tomcat
4. 运行环境 java

war类 springmvc项目
1. springmvc
2. 生成war包
3. 需要外部tomcat


## jar类配置
1. 配置代码分支来源
2. 打包
3. 传输
4. 启动


### 配置代码
### 打包
![](http://7xqtsx.com1.z0.glb.clouddn.com/17-5-28/33852371.jpg)
### 传输
![](http://7xqtsx.com1.z0.glb.clouddn.com/17-5-28/21535521.jpg)
### 邮件

### 脚本

```
#!/bin/sh
#defined
ID=`ps -ef|grep "official-web-0.0.1-SNAPSHOT" | grep "java"| awk '{print $2}'`
echo $ID
kill -9 $ID
echo "kill $ID finish"
nohup java -jar official-web-0.0.1-SNAPSHOT.jar &
....
```

[slide]
## war类配置
1. 配置代码分支来源
2. 打包
3. 传输
4. 启动

[slide]
### 配置代码

### 打包

### 传输

### 脚本

```
# !bin/sh
# jenkins传输war包位置
jenkins_dir="/root/jenkinsproject"
# war_name="appservice.war"
#项目名称
project_name="appservice"
#tomcat路径
tomcat_dir="/usr/tomcat/apache-tomcat-7.0.42"
# 当前脚本名称,查找进称号需要排除
shell_name="appservice-deploy.sh"
pid=`ps -ef | grep $tomcat_dir | grep -v $shell_name | grep -v grep | awk '{print $2}'`
echo "old pid:".$pid
if test "$pid"
then
  echo "kill old pid: $pid ing"
  kill -9 $pid
  # 睡眠5s
  sleep 20s
else
        echo "not find old pid"
fi
#删除旧的war包和生成文件夹
rm -rf $tomcat_dir/webapps/$project_name*
# 拷贝新的war包
cp $jenkins_dir/$project_name.war  $tomcat_dir/webapps
#启动tomcat
sh $tomcat_dir/bin/startup.sh
sleep 10s
# 检查启动
newpid=`ps -ef | grep $tomcat_dir | grep -v $shell_name | grep -v grep | awk '{print $2}'`
if test "$newpid"
then
        echo "new pid:".$newpid
#    刷新nginx
    /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf -s reload
        exit
else
        echo "not find new pid"
        exit 1
fi
....
```

[slide]

## apk类配置

1. 配置代码分支来源
2. 打包
3. 传输

[slide]
### 配置代码

### 打包
![](http://7xqtsx.com1.z0.glb.clouddn.com/17-5-28/67349864.jpg)

### 传输




### 脚本

```


    applicationVariants.all { variant ->
        if (variant.buildType.name.equals('release')) {
            variant.outputs.each { output ->
                def outputFile = output.outputFile
                if (outputFile != null && outputFile.name.endsWith('.apk')) {
                    def fileName = "baimimobile_v${defaultConfig.versionName}_${buildTime()}.apk"
                    output.outputFile = new File(outputFile.parent, fileName)
                }
            }
        }
    }
```
### 演示

```
BUILD SUCCESSFUL

Total time: 1 mins 27.55 secs
Build step 'Invoke Gradle script' changed build result to SUCCESS
Root directory to find files to upload : /Users/wpmac/.jenkins/workspace/baimi-mobile-android
found remote files : [app/build/outputs/apk/baimimobile_v1.1.6_2017-05-18_1126228.apk]
Deploying archive app/build/outputs/apk/baimimobile_v1.1.6_2017-05-18_1126228.apk to fir.im (description="ecf02a33f60e3ad65c2bc0cae240c7f1", groups="")
You can use ENV var $FIR_DOWNLOAD_URL in other plugins.
Deployment to fir.im Done.
 Your app short url: http://fir.im/baimimobile
Finished: SUCCESS
```







