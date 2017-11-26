title: Gitflow 工作流
categories: 
  - 文档
author: mzeht
avatar: /images/favicon.png
authorDesc: 一个写代码的
date: 2017-05-28 16:14:36
tags:
keywords:
description:
photos: http://nvie.com/img/git-model@2x.png
---
##简介

![](http://nvie.com/img/git-model@2x.png)
[slide]
1. `feature`（多个、玫红）。主要是自己玩了，差不多的时候要合并回develop去。从不与master交互。
2. `develop`（1个、黄色）。主要是和feature以及release交互。
3. `release`（同一时间1个、绿色）。总是基于develop，最后又合并回develop。当然对应的tag跑到master这边去了。生命周期很短，只是为了发布
4. `hotfix`（同一时间1个、红色）。总是基于master，并最后合并到master和develop。生命周期较短，用了修复bug或小粒度修改发布。
5. `master`（1个蓝色）。没有什么东西，仅是一些关联的tag，因从不在master上开发。


[slide]
1. `master`和`develop`都具有象征意义。master分支上的代码总是稳定的（stable build），随时可以发布出去。develop上的代码总是从feature上合并过来的，可以进行Nightly Builds，但不直接在develop上进行开发。当develop上的feature足够多以至于可以进行新版本的发布时，可以创建release分支
2. `release`分支基于develop，进行很简单的修改后就被合并到master，并打上tag，表示可以发布了。紧接着release将被合并到develop；此时develop可能往前跑了一段，出现合并冲突，需要手工解决冲突后再次合并。这步完成后就删除release分支。
3. 当从已发布版本中发现bug要修复时，就应用到`hotfix`分支了。hotfix基于master分支，完成bug修复或紧急修改后，要merge回master，打上一个新的tag，并merge回develop，删除hotfix分支。
4. 由此可见release和hotfix的生命周期都较短，master/develop虽然总是存在但却不常使用。

[slide]
## 分支详解
### 主分支 （Historical Branches）
![](http://upload-images.jianshu.io/upload_images/54009-70ef16000c0832b0.jpg)
1. 主分支是所有开发活动的核心分支。所有的开发活动产生的输出物最终都会反映到主分支的代码中。主分支分为master分支和development分支。

[slide]
#### master 分支
master分支上存放的应该是随时可供在生产环境中部署的代码（Production Ready state）。当开发活动告一段落，产生了一份新的可供部署的代码时，master分支上的代码会被更新。同时，每一次更新，最好添加对应的版本号标签（TAG）。
[slide]
#### develop分支
develop分支是保存当前最新开发成果的分支。通常这个分支上的代码也是可进行每日夜间发布的代码（Nightly build）。因此这个分支有时也可以被称作整合分支（integration branch）。


[slide]
## 辅助分支
![](http://upload-images.jianshu.io/upload_images/54009-7265a72025bddb1a.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1. 用于开发新功能时所使用的feature分支；
2. 用于辅助版本发布的release分支；
3. 用于修正生产代码中的缺陷的hotfix分支。





[slide]
### feature 分支
使用规范：
1. 可以从develop分支发起feature分支
2. 代码必须合并回develop分支
3. feature分支（有时也可以被叫做“topic分支”）通常是在开发一项新的软件功能的时候使用，这个分支上的代码变更最终合并回develop分支或者干脆被抛弃掉（例如实验性且效果不好的代码变更）

[slide]
### release分支
![](http://upload-images.jianshu.io/upload_images/54009-be502bd4658d9d79.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
1. 可以从develop分支派生
2. 必须合并回develop分支和master分支
3. 分支命名惯例：release-*



