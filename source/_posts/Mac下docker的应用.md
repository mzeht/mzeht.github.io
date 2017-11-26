title: mac下docker的应用
categories: 
  - 运维
  - java
tags:
  - docker
date: 2017-01-28 09:17:42
author: mzeht
avatar: /images/favicon.png
---

# mac安装docker

[下载地址](https://download.docker.com/mac/stable/Docker.dmg)
一路next

最新版
![](http://7xqtsx.com1.z0.glb.clouddn.com/17-1-28/31352377-file_1485609643545_57d2.png)

安装好以上安装包后，打开kitematic,提示下载此软件，
下载好后拖入application

 kitematic是docker收购的gui客户端
 <!-- more -->
 ktematic启动方式既可以是监控已运行的docker，也可以是另启动一个虚拟机，如图
 ![](http://7xqtsx.com1.z0.glb.clouddn.com/17-2-1/67359041-file_1485919722478_98b.png)
 此处建议前者（启动速度快，ktematic默认就是前者启动方式，后者还要virtualbox）
 
 ![](http://7xqtsx.com1.z0.glb.clouddn.com/17-2-1/41048694-file_1485919676611_9d85.png)

上图左侧就是docker客户端启动后，存在的容器及相关信息，可以运行，重启，查看dockerhub上的文档，有容器内日志，web项目右侧还有预览画面，非常方便，但是相关命令还是要熟悉，gui为辅助，大部分服务器上的操作还是要靠手敲命令

## 配置加速器
[阿里云加速器获取地址](https://cr.console.aliyun.com/#/accelerator)
注册一个阿里云账号，访问以上地址，查看加速器，就可以看到分配的加速器地址

[DaoCloud加速器](https://www.daocloud.io/mirror#accelerator-doc)
注册并获得加速器地址

到客户端的Daemon选项卡配置获取到的地址
![](http://7xqtsx.com1.z0.glb.clouddn.com/17-2-1/58855725-file_1485919966984_1093e.png)
preferens-Damon-Basic
Registry mirrors

## 命令测试
```
➜  ~ docker --version
Docker version 1.13.0, build 49bf474
➜  ~ docker--compose --version
zsh: command not found: docker--compose
➜  ~ docker-compose --version
docker-compose version 1.10.0, build 4bd6f1a
➜  ~ docker-machine --version
docker-machine version 0.9.0, build 15fd4c7
➜  ~
```

# 获取镜像
docker search 

```
➜  ~ docker search rap
NAME                            DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
neo4j                           Neo4j is a highly scalable, robust native ...   282       [OK]
r-base                          R is a system for statistical computation ...   147       [OK]
arangodb                        ArangoDB - a distributed database with a f...   78        [OK]
orientdb                        OrientDB a Multi-Model Open Source NoSQL D...   46        [OK]
ubuntu-debootstrap              debootstrap --variant=minbase --components...   27        [OK]
rapidftr/rapidftr               UNICEF RapidFTR (Family Tracing and Reunif...   2                    [OK]
rapi/psgi                       Image for running PSGI apps, comes with Ra...   2                    [OK]
masonliu/rap                    Docker Image for RAP https://github.com/th...   1
huangfushun/rap                 ali rap web ui docker file                      1                    [OK]
lihuansse/rap                   RAP 接口管理工具                                      1
rapidoid                        Rapidoid is a high-performance HTTP server...   1         [OK]
ukgovdatascience/rap-docker     Environment for reproducible analytical pi...   0                    [OK]
rapidreg/rapidreg                                                               0                    [OK]
raphiz/raphael.li                                                               0                    [OK]
rapi/rapidapp                   Standard RapidApp base image, from officia...   0                    [OK]
team19awshack/rapidprowebhook   RapidProWebhook                                 0                    [OK]
tp81/rapidstorm                 RapidSTORM docker image                         0                    [OK]
qdsang/rap                      docker rap alpine                               0                    [OK]
raphsfeir/raphaelsfeir          Portfolio and blog for me                       0                    [OK]
rapidpro/rapidpro-base          Base Python 2.7 + geolibs Docker image for...   0                    [OK]
tonimichel/rapidflask           A boilerplate flask application for easy s...   0                    [OK]
raphaelgroup/magi                                                               0                    [OK]
andreweskeclarke/rapidftr                                                       0                    [OK]
raphiz/mathe.raphael.li                                                         0                    [OK]
divio/rapidpro                  RapidPro in docker.                             0                    [OK]
```

GUI

![](http://7xqtsx.com1.z0.glb.clouddn.com/17-2-1/45031596-file_1485920300648_448b.png)

我有时gui客户端网络错误，修改偏好设置-网络-dns-8.8.8.8后正常

## 拉取镜像

 docker pull 【镜像名称】
 
##  列出镜像
docker images

```
➜  ~ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
mzeht/webv2         latest              8e684c1fb7d3        30 hours ago        182 MB
nginx               v2                  8e684c1fb7d3        30 hours ago        182 MB
nginx               latest              cc1b61406712        7 days ago          182 MB
ubuntu              14.04               b969ab9f929b        11 days ago         188 MB
hello-world         latest              48b5124b2768        2 weeks ago         1.84 kB
centos              latest              67591570dd29        6 weeks ago         192 MB
lihuansse/rap       latest              3ded84cdf4f5        3 months ago        1.14 GB
```

## 删除本地镜像

如果要删除本地的镜像，可以使用 `docker rmi` 命令，其格式为：

```bash
docker rmi [选项] <镜像1> [<镜像2> ...]
```

*注意 `docker rm` 命令是删除容器，不要混淆。*

### 用 ID、镜像名、摘要删除镜像

其中，`<镜像>` 可以是 `镜像短 ID`、`镜像长 ID`、`镜像名` 或者 `镜像摘要`。

比如我们有这么一些镜像：

```bash
$ docker images
REPOSITORY                  TAG                 IMAGE ID            CREATED             SIZE
centos                      latest              0584b3d2cf6d        3 weeks ago         196.5 MB
redis                       alpine              501ad78535f0        3 weeks ago         21.03 MB
docker                      latest              cf693ec9b5c7        3 weeks ago         105.1 MB
nginx                       latest              e43d811ce2f4        5 weeks ago         181.5 MB
```

我们可以用镜像的完整 ID，也称为 `长 ID`，来删除镜像。使用脚本的时候可能会用长 ID，但是人工输入就太累了，所以更多的时候是用 `短 ID` 来删除镜像。`docker images` 默认列出的就已经是短 ID 了，一般取前3个字符以上，只要足够区分于别的镜像就可以了。

比如这里，如果我们要删除 `redis:alpine` 镜像，可以执行：

```bash
$ docker rmi 501
Untagged: redis:alpine
Untagged: redis@sha256:f1ed3708f538b537eb9c2a7dd50dc90a706f7debd7e1196c9264edeea521a86d
Deleted: sha256:501ad78535f015d88872e13fa87a828425117e3d28075d0c117932b05bf189b7
Deleted: sha256:96167737e29ca8e9d74982ef2a0dda76ed7b430da55e321c071f0dbff8c2899b
Deleted: sha256:32770d1dcf835f192cafd6b9263b7b597a1778a403a109e2cc2ee866f74adf23
Deleted: sha256:127227698ad74a5846ff5153475e03439d96d4b1c7f2a449c7a826ef74a2d2fa
Deleted: sha256:1333ecc582459bac54e1437335c0816bc17634e131ea0cc48daa27d32c75eab3
Deleted: sha256:4fc455b921edf9c4aea207c51ab39b10b06540c8b4825ba57b3feed1668fa7c7
```

我们也可以用`镜像名`，也就是 `<仓库名>:<标签>`，来删除镜像。

```bash
$ docker rmi centos
Untagged: centos:latest
Untagged: centos@sha256:b2f9d1c0ff5f87a4743104d099a3d561002ac500db1b9bfa02a783a46e0d366c
Deleted: sha256:0584b3d2cf6d235ee310cf14b54667d889887b838d3f3d3033acd70fc3c48b8a
Deleted: sha256:97ca462ad9eeae25941546209454496e1d66749d53dfa2ee32bf1faabd239d38
```

当然，更精确的是使用 `镜像摘要` 删除镜像。

```bash
$ docker images --digests
REPOSITORY                  TAG                 DIGEST                                                                    IMAGE ID            CREATED             SIZE
node                        slim                sha256:b4f0e0bdeb578043c1ea6862f0d40cc4afe32a4a582f3be235a3b164422be228   6e0c4c8e3913        3 weeks ago         214 MB

$ docker rmi node@sha256:b4f0e0bdeb578043c1ea6862f0d40cc4afe32a4a582f3be235a3b164422be228
Untagged: node@sha256:b4f0e0bdeb578043c1ea6862f0d40cc4afe32a4a582f3be235a3b164422be228
```

### Untagged 和 Deleted

如果观察上面这几个命令的运行输出信息的话，你会注意到删除行为分为两类，一类是 `Untagged`，另一类是 `Deleted`。我们之前介绍过，镜像的唯一标识是其 ID 和摘要，而一个镜像可以有多个标签。

因此当我们使用上面命令删除镜像的时候，实际上是在要求删除某个标签的镜像。所以首先需要做的是将满足我们要求的所有镜像标签都取消，这就是我们看到的 `Untagged` 的信息。因为一个镜像可以对应多个标签，因此当我们删除了所指定的标签后，可能还有别的标签指向了这个镜像，如果是这种情况，那么 `Delete` 行为就不会发生。所以并非所有的 `docker rmi` 都会产生删除镜像的行为，有可能仅仅是取消了某个标签而已。

当该镜像所有的标签都被取消了，该镜像很可能会失去了存在的意义，因此会触发删除行为。镜像是多层存储结构，因此在删除的时候也是从上层向基础层方向依次进行判断删除。镜像的多层结构让镜像复用变动非常容易，因此很有可能某个其它镜像正依赖于当前镜像的某一层。这种情况，依旧不会触发删除该层的行为。直到没有任何层依赖当前层时，才会真实的删除当前层。这就是为什么，有时候会奇怪，为什么明明没有别的标签指向这个镜像，但是它还是存在的原因，也是为什么有时候会发现所删除的层数和自己 `docker pull` 看到的层数不一样的源。

除了镜像依赖以外，还需要注意的是容器对镜像的依赖。如果有用这个镜像启动的容器存在（即使容器没有运行），那么同样不可以删除这个镜像。之前讲过，容器是以镜像为基础，再加一层容器存储层，组成这样的多层存储结构去运行的。因此该镜像如果被这个容器所依赖的，那么删除必然会导致故障。如果这些容器是不需要的，应该先将它们删除，然后再来删除镜像。

### 用 docker images 命令来配合

像其它可以承接多个实体的命令一样，可以使用 `docker images -q` 来配合使用 `docker rmi`，这样可以成批的删除希望删除的镜像。比如之前我们介绍过的，删除虚悬镜像的指令是：

```bash
$ docker rmi $(docker images -q -f dangling=true)
```

我们在“镜像列表”章节介绍过很多过滤镜像列表的方式都可以拿过来使用。

比如，我们需要删除所有仓库名为 `redis` 的镜像：

```bash
$ docker rmi $(docker images -q redis)
```

或者删除所有在 `mongo:3.2` 之前的镜像：

```bash
$ docker rmi $(docker images -q -f before=mongo:3.2)
```

充分利用你的想象力和 Linux 命令行的强大，可以完成很多非常赞的功能。


## 容器

##启动容器
启动容器有两种方式，一种是基于镜像新建一个容器并启动，另外一个是将在终止状态（stopped）的容器重新启动。

因为 Docker 的容器实在太轻量级了，很多时候用户都是随时删除和新创建容器。

###新建并启动
所需要的命令主要为 `docker run`。

例如，下面的命令输出一个 “Hello World”，之后终止容器。
```
$ sudo docker run ubuntu:14.04 /bin/echo 'Hello world'
Hello world
```
这跟在本地直接执行 `/bin/echo 'hello world'` 几乎感觉不出任何区别。

下面的命令则启动一个 bash 终端，允许用户进行交互。
```
$ sudo docker run -t -i ubuntu:14.04 /bin/bash
root@af8bae53bdd3:/#
```
其中，`-t` 选项让Docker分配一个伪终端（pseudo-tty）并绑定到容器的标准输入上， `-i` 则让容器的标准输入保持打开。

在交互模式下，用户可以通过所创建的终端来输入命令，例如

```
root@af8bae53bdd3:/# pwd
/
root@af8bae53bdd3:/# ls
bin boot dev etc home lib lib64 media mnt opt proc root run sbin srv sys tmp usr var
```

当利用 `docker run` 来创建容器时，Docker 在后台运行的标准操作包括：

* 检查本地是否存在指定的镜像，不存在就从公有仓库下载
* 利用镜像创建并启动一个容器
* 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层
* 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
* 从地址池配置一个 ip 地址给容器
* 执行用户指定的应用程序
* 执行完毕后容器被终止

###启动已终止容器
可以利用 `docker start` 命令，直接将一个已经终止的容器启动运行。

容器的核心为所执行的应用程序，所需要的资源都是应用程序运行所必需的。除此之外，并没有其它的资源。可以在伪终端中利用 `ps` 或 `top` 来查看进程信息。

```
root@ba267838cc1b:/# ps
  PID TTY          TIME CMD
    1 ?        00:00:00 bash
   11 ?        00:00:00 ps
```
可见，容器中仅运行了指定的 bash 应用。这种特点使得 Docker 对资源的利用率极高，是货真价实的轻量级虚拟化。


##后台(background)运行

更多的时候，需要让 Docker在后台运行而不是直接把执行命令的结果输出在当前宿主机下。此时，可以通过添加 `-d` 参数来实现。

下面举两个例子来说明一下。

如果不使用 `-d` 参数运行容器。

```
$ sudo docker run ubuntu:14.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"
hello world
hello world
hello world
hello world
```
容器会把输出的结果(STDOUT)打印到宿主机上面

如果使用了 `-d` 参数运行容器。

```
$ sudo docker run -d ubuntu:14.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"
77b2dc01fe0f3f1265df143181e7b9af5e05279a884f4776ee75350ea9d8017a
```
此时容器会在后台运行并不会把输出的结果(STDOUT)打印到宿主机上面(输出结果可以用docker logs 查看)。

**注：** 容器是否会长久运行，是和docker run指定的命令有关，和 `-d` 参数无关。

使用 `-d` 参数启动后会返回一个唯一的 id，也可以通过 `docker ps` 命令来查看容器信息。

```
$ sudo docker ps
CONTAINER ID  IMAGE         COMMAND               CREATED        STATUS       PORTS NAMES
77b2dc01fe0f  ubuntu:14.04  /bin/sh -c 'while tr  2 minutes ago  Up 1 minute        agitated_wright
```
要获取容器的输出信息，可以通过 `docker logs` 命令。

```
$ sudo docker logs [container ID or NAMES]
hello world
hello world
hello world
. . .
```














