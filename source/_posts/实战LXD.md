---
title: 实战LXD
date: 2016-09-18 21:46:23
tags: 容器
---
LXD下一代容器……
<!--more-->
## 介绍
### 什么是LXD？
LXD 相当于是一个容器的 "hypervisor" ，使你对LXC有一个新的用户体验.它由３个组件构成：
- 一个系统级的守护进程(lxd)
- 一个命令行客户端(lxc)
- 一个openStack Nova插件(nova-compute-lxd)

这个守护进程对外暴露了一组REST API。

LXD一些大的特性：
- 安全(unprivileged containers, resource restrictions and much more)
- 可扩展 (from containers on your laptop to thousand of compute nodes)
- 直接 (simple, clear API and crisp command line experience)
- 基于镜像 (no more distribution templates, only good, trusted images)
- 动态迁移

### 与LXC的关系
LXD不是LXC的重写，事实上它构建在LXC之上，提供一个新的、更好的用户体验.在LXD底层，它通过liblxc使用LXC去创建和管理容器.

## 安装
Ubuntu 16.o4 LTS 的用户可以直接通过Ubuntu自带仓库安装:
```
apt-get install lxd
```
Ubuntu 14.04 LTS的用户可以通过官方 Ubuntu PPA安装:
```
add-apt-repository ppa:ubuntu-lxc/lxd-stable
apt-get update
apt-get dist-upgrade
apt-get install lxd
```
###  存储方式
推荐使用ZFS，因为它能支持LXD为提供最新最稳定的容器服务所需的所有功能，为每个容器提供包括：磁盘配额，即时快照/回滚，经过优化的迁移(发和收)，以及快速从镜像中创建容器。而且技术上，它比btrfs要更成熟。

Ubuntu 16.04直接安装：
```
sudo apt install zfsutils-linux
```
Ubuntu 14.04可以通过zfsonlinux PPA安装：
```
sudo apt-add-repository ppa:zfs-native/stable
sudo apt update
sudo apt install ubuntu-zfs
```
## 运行第一个容器
LXD是基于镜像的，然而默认存储里是没有镜像的,通过以下命令可以看出:
```
lxc image list
```

LXD知道３个默认的镜像服务器：
1. ubuntu: (for Ubuntu stable images)
1. ubuntu-daily: (for Ubuntu daily images)
1. images: (for a bunch of other distributions)

这样可以列出稳定的ubuntu镜像来：
```
lxc image list ubuntu: | less
```

下面运行第一个容器给它命名为`first`,使用的是 Ubuntu 14.04 的镜像:
```
lxc launch ubuntu:14.04 first
```

列出所有的容器：
```
lxc list
```

查看运行状态的详细信息和容器的配置:
```
lxc info first
lxc config show first
```
## 限制容器资源
默认情况，容器是没有资源限制的,它继承自父环境,可以这样来确认:
```
free -m
lxc exec first -- free -m
```

下面给容器一个内存的限制:
```
lxc config set first limits.memory 64MB
```

再次刚才对容器做的限制已经生效确认:
```
lxc exec first -- free -m
```
## 快照
LXD支持快照和从快照中恢复.

在打快照之前,先对容器做一些改变:
```
lxc exec first -- apt-get update
lxc exec first -- apt-get dist-upgrade -y
lxc exec first -- apt-get autoremove --purge -y
```

现在容器是最新的，也是干净的.就对它做一个快照，命名为`clean`:
```
lxc snapshot first clean
```

毁坏｀first｀这个容器:
```
lxc exec first -- rm -Rf /etc /usr
```

确认容器已经被毁坏:
```
lxc exec first -- bash
```

从快照中恢复容器的状态:
```
lxc restore first clean
```

确认容器已经正常:
```
lxc exec first -- bash
```
## 创建容器镜像
正如上面说道的，LXD是基于镜像的，也就是说，所有的容器可以通过copy一个镜像运行的容器的方式来创建

可以通过一个已存在的容器或者容器快照来创建一个新的镜像

上传通过之前的`clean`快照作为一个新的镜像，并给它取个别名为`clean-ubuntu`:
```
lxc publish first/clean --alias clean-ubuntu
```

删除`first`容器:
```
lxc stop first
lxc delete first
```

通过自己的镜像运行一个新容器:
```
lxc launch clean-ubuntu second
```
## 访问容器里的文件
从容器中拉取一个文件:
```
lxc file pull second/etc/hosts .
```

编辑一个文件并推到容器里:
```
echo "1.2.3.4 my-example" >> hosts
lxc file push hosts second/etc/hosts
```

也可以通过这种机制查看容器的日志文件:
```
lxc file pull second/var/log/syslog - | less
```

当一个容器不再需要时可以停止并删除它:
```
lxc delete --force second
```
## 实用远程镜像服务器
lxc 客户端工具支持多个`remotes`,那些`remotes`可以使只读的镜像服务器或只是LXD主机.

LXC上游运行一个像[这样的服务](https://images.linuxcontainers.org),它提供一系列不同linux分发版本的自动生成的镜像.      

它默认添加在LXD 中，当时你不需要时可以移除或改变它.

列出所有可用的镜像:
```
lxc image list images: | less
```

运行一个Centos 7容器:
```
lxc launch images:centos/7 third
```

确认它确实是Centos 7:
```
lxc exec third -- cat /etc/redhat-release
```

删除它:
```
lxc delete -f third
```

列出配置的所有的`remotes`:
```
lxc remote list
```
## 与远程LXD服务器交互
For this step, you'll need a second demo session, so open a new one here

Copy/paste the "lxc remote add" command from the top of the page of that new session into the shell of your old session.
Then confirm the server fingerprint and enter the password of the remote server.

Note that it may take a few seconds for the new LXD daemon to listen to the network, just retry the command until it answers.

At this point you can list the remote containers with:

lxc list tryit:
And its images with:

lxc image list tryit:
Now, let's start a new container on the remote LXD using the local image we created earlier.

lxc launch clean-ubuntu tryit:fourth
You now have a container called "fourth" running on the remote host "tryit". You can spawn a shell inside it with (then exit):

    lxc exec tryit:fourth bash
    Now let's copy that container into a new one called "fifth":

    lxc copy tryit:fourth tryit:fifth
    And just for fun, move it back to our local lxd while renaming it to "sixth":

    lxc move tryit:fifth sixth
    And confirm it's all still working (then exit):

        lxc start sixth
        lxc exec sixth -- bash
        Then clean everything up:

        lxc delete -f sixth
        lxc delete -f tryit:fourth
        lxc image delete clean-ubuntu
## 小结
We hope this gave you a good introduction to LXD, its capabilities and how easy it is to use.

You're welcome to use the demo service as long as you want to try LXD and play with the latest features.

Enjoy!
