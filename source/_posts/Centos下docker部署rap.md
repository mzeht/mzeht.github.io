title: Centos下docker部署rap
categories: 
  - 运维
  - docker
tags:
  - Centos
date: 2017-02-01 13:43:42
author: mzeht
avatar: /images/favicon.png
---
[rap官网](http://rap.taobao.org/org/index.do)
[rapgithub地址](https://github.com/thx/RAP/wiki/home_cn)

RAP是一个GUI的WEB接口管理工具。在RAP中，您可定义接口的URL、请求&响应细节格式等等。通过分析这些数据，RAP提供MOCK服务、测试服务等自动化工具。RAP同时提供大量企业级功能，帮助企业和团队高效的工作。

在前后端分离的开发模式下，我们通常需要定义一份接口文档来规范接口的具体信息。如一个请求的地址、有几个参数、参数名称及类型含义等等。RAP 首先方便团队录入、查看和管理这些接口文档，并通过分析结构化的文档数据，重复利用并生成自测数据、提供自测控制台等等... 大幅度提升开发效率

<!-- more -->
1.`强大的GUI工具` 给力的用户体验，你将会爱上使用RAP来管理您的API文档。
2.`完善的MOCK服务` 文档定义好的瞬间，所有接口已经准备就绪。有了MockJS，无论您的业务模型有多复杂，它都能很好的满足。

# 安装docker

由于我是在阿里云上的服务器
## 采用阿里云的脚本进行安装

```
curl -sSL http://acs-public-mirror.oss-cn-hangzhou.aliyuncs.com/docker-engine/internet | sh -
```

## 添加内核参数

默认配置下，在 CentOS 使用 Docker 可能会碰到下面的这些警告信息：

```
WARNING: bridge-nf-call-iptables is disabled
WARNING: bridge-nf-call-ip6tables is disabled
```
很不巧我就遇到了

添加内核配置参数以启用这些功能。

```
$ sudo tee -a /etc/sysctl.conf <<-EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```

然后重新加载 sysctl.conf


```
$ sudo sysctl -p
```

# 配置加速器
 参考阿里云论坛的一篇文章[here](https://yq.aliyun.com/articles/29941)

CentOS的配置方式略微复杂，需要先将默认的配置文件复制出来
/lib/systemd/system/docker.service -> /etc/systemd/system/docker.service
然后再将加速器地址添加到配置文件的启动命令
重启Docker就可以了。


```
sudo cp -n /lib/systemd/system/docker.service /etc/systemd/system/docker.service
sudo sed -i "s|ExecStart=/usr/bin/docker daemon|ExecStart=/usr/bin/docker daemon --registry-mirror=<your accelerate address>|g" /etc/systemd/system/docker.service
sudo systemctl daemon-reload
sudo service docker restart
```
your accelerate address 处填入阿里云账号分配的加速器地址

# 拉取镜像

```
docker search rap
NAME                            DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
neo4j                           Neo4j is a highly scalable, robust native ...   282       [OK]
r-base                          R is a system for statistical computation ...   147       [OK]
arangodb                        ArangoDB - a distributed database with a f...   78        [OK]
orientdb                        OrientDB a Multi-Model Open Source NoSQL D...   46        [OK]
ubuntu-debootstrap              debootstrap --variant=minbase --components...   27        [OK]
rapidftr/rapidftr               UNICEF RapidFTR (Family Tracing and Reunif...   2                    [OK]
rapi/psgi                       Image for running PSGI apps, comes with Ra...   2                    [OK]
huangfushun/rap                 ali rap web ui docker file                      1                    [OK]
lihuansse/rap                   RAP 接口管理工具                                      1
masonliu/rap                    Docker Image for RAP https://github.com/th...   1
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
andreweskeclarke/rapidftr                                                       0                    [OK]
raphiz/mathe.raphael.li                                                         0                    [OK]
raphaelgroup/magi                                                               0                    [OK]
divio/rapidpro                  RapidPro in docker.                             0                    [OK]
```

这里选择lihuansse/rap


```
docker pull lihuansse/rap
```
创建并启动

```
docker run --name rap -p 8080:8080 -d lihuansse/rap:latest
```

然而就在即将完成这最后一步的时候

形如下面的错误出现了

```
$ docker run --rm -ti ubuntu:14.04 bin/bash                                       
FATA[0000] Error response from daemon: mkdir /var/lib/docker/overlay/c4a8f5e516d401534f2d994f5546f7e08639ffd675eb3573267f76d79394f172-init/merged/dev/shm: invalid argument
```

请看下回分解


