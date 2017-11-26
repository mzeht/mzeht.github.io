title: Centos下docker导致的升级内核
categories: 
  - 运维
  - docker
tags:
  - Centos
date: 2017-02-01 13:50:42
author: mzeht
avatar: /images/favicon.png
---

接着上一篇文章

形如下面的错误出现了

```
$ docker run --rm -ti ubuntu:14.04 bin/bash                                       
FATA[0000] Error response from daemon: mkdir /var/lib/docker/overlay/c4a8f5e516d401534f2d994f5546f7e08639ffd675eb3573267f76d79394f172-init/merged/dev/shm: invalid argument
```

Centos下安装docker正常，拉取镜像正常，就在启动容器的时候报错
<!-- more -->

一番谷歌，github

[https://github.com/docker/docker/issues/10294](https://github.com/docker/docker/issues/10294)

# 首先检查docker info

如果有

```
WARNING: bridge-nf-call-iptables is disabled
WARNING: bridge-nf-call-ip6tables is disabled
```

就需要添加内核参数，具体看上一篇


之后看到

```
Fixed...

from docker official site.

To configure Docker to use the overlay storage driver your Docker host must be running version 3.18 of the Linux kernel (preferably newer) with the overlay kernel module loaded. OverlayFS can operate on top of most supported Linux filesystems. However, ext4 is currently recommended for use in production environments.
so, update kernel from 3.10.0 to 3.18.0 + fixed the issue.
```

提示我们要升级Centos内核3.10.0 to 3.18.0 + 

# 升级内核
## 查看机器内核版本

```
uname -r
4.9.6-1.el7.elrepo.x86_64
```
这里我是升级后在写的文章，升级之前是3.10

## 查看Linux各版本内核
[网站地址](https://www.kernel.org/)

![](http://7xqtsx.com1.z0.glb.clouddn.com/17-2-1/55278371-file_1485959943374_7bc7.png)

稳定版 4.9.7

## 导入key

```
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
```
## 安装elrepo的yum源

```
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm
```

## 安装内核
```
 yum --enablerepo=elrepo-kernel install  kernel-ml-devel kernel-ml -y
```
## 查看默认启动顺序

```
awk -F\' '$1=="menuentry " {print $2}' /etc/grub2.cfg
CentOS Linux (4.9.6-1.el7.elrepo.x86_64) 7 (Core)
CentOS Linux (3.10.0-514.2.2.el7.x86_64) 7 (Core)
CentOS Linux (3.10.0-327.22.2.el7.x86_64) 7 (Core)
CentOS Linux (3.10.0-327.el7.x86_64) 7 (Core)
CentOS Linux (0-rescue-7d26c16f128042a684ea474c9e2c240f) 7 (Core)
```
## 调整启动顺序

```
grub2-set-default 0
```
## 重新启动

```
reboot
```

## 验证

```
 uname -r
4.9.6-1.el7.elrepo.x86_64
```
成功







