---
title: "最新 Docker 安装教程"
date: 2022-09-23T20:41:44+08:00
draft: false
# summary: "博客的概述" # 文章简单描述，会展示在主页
categories:
- 安装教程

tags:
- 环境安装
- docker

---

# 前言

在CentOS上安装 Docker Engine，需要一个维护版本的 CentOS 7、CentOS 8(stream) 或 CentOS 9(stream)

>本次安装环境:
>
>* VMware 16.2.2
>* Linux系统版本:`CentOS Linux release 7.9.2009 (Core)`
>* 为了方便我是使用`root`用户安装的,如果使用非`root`用户，一些命令需要添加`sudo`

# 安装

## 启用 centos-extras 存储库

>默认情况下它是开启的，但如果禁用过它，则需要重新启用它

**具体做法如下：**

查看centos-extras是否开启

```bash
# yum repolist all
```


![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122251938.png)

 

上图显示`extras`没有开启

命令行输入以下命令

```bash
# vim /etc/yum.repos.d/CentOS-Base.repo
```

内容如下

```bash
# ...省略其他内容
[extras]
name=CentOS-$releasever - Extras
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/extras/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
enabled=0

```

将`enabled`的值设置为`1`

```bash
# ...省略其他内容
[extras]
name=CentOS-$releasever - Extras
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/extras/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
enabled=1

```

按`esc`键,输入`:wq`保存退出

并在命令行输入如下命令 清除缓存 和 建立新的元数据缓存

```bash
# yum clean all
# yum makecache
```

>过程中可能出现`/var/run/yum.pid 已被锁定`的问题，可以先`ctrl+c`退出，等一会再试

再次查看centos-extras是否开启

```bash
# yum repolist all
```

显示结果如下:
  
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122252263.png)

## 卸载旧版本

如果之前安装过旧版本的`docker`，需要先将之前安装的依赖项移除

>旧版本的 `Docker` 被称为 `docker` 或 `docker-engine`

```bash
# yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

`/var/lib/docker/`的内容，包括镜像、容器、卷和网络，都将被保留。Docker 引擎包现在称为 docker-ce

>1. docker-ce是docker公司维护的开源项目，是一个基于moby项目的免费的容器产品
>2. docker-ee是docker公司维护的闭源产品，是docker公司的商业产品

## 安装方式选择

`docker-ce`的安装有多种方式，比如可以通过下载`rpm`包手动安装，还有可以通过`自动化脚本`快速安装。  

推荐通过使用`Docker`的存储库进行安装，这种安装方式也方便我们日后的升级，本文也是以这种安装方式为例。

>`rpm`包地址[https://download.docker.com/linux/centos/](https://download.docker.com/linux/centos/)
>`自动化脚本`地址[https://get.docker.com/](https://get.docker.com/)

## 存储库安装Docker

在新主机上首次安装 `Docker Engine` 之前，需要设置 `Docker` 存储库。之后，我们可以从存储库安装和更新 `Docker`。

### 设置存储库

安装 `yum-utils` 软件包（它提供 `yum-config-manager`程序），再设置存储库

```bash
# yum install -y yum-utils
# yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

### 安装 Docker 引擎

安装最新版的`Docker Engine`, `containerd`和`Docker Compose`，如果想安装特定版本的可以到[官网查看安装过程](https://docs.docker.com/engine/install/centos/#install-docker-engine)

```bash
# yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

官方文档提到：
>如果提示接受 GPG 密钥，请验证指纹是否与`060A 61C5 1B55 8A7F 742B 77AA C52F EB6B 621E 9F35`匹配，如果是，则接受它。

不过我在安装时没有出现这种提示。

命令执行后会安装 `Docker`，但不会启动 `Docker`。

它还创建了一个 `docker` 组，但是默认情况下它不会将任何用户添加到该组中。

```bash
# cat /etc/group
```

  
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122252487.png)

### 启动并验证Docker

```bash
# systemctl start docker

#此命令会下载测试镜像并在容器中运行它。当容器运行时，它会打印一条消息并退出
# docker run hello-world
```

输出如下：
  
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122252550.png)

## 卸载 Docker Engine

1. 卸载 Docker Engine、CLI、Containerd 和 Docker Compose 软件包

```bash
# yum remove docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

2. 主机上的镜像、容器、卷或自定义配置文件不会自动删除。

    删除所有镜像、容器和卷：

```bash
# rm -rf /var/lib/docker
# rm -rf /var/lib/containerd
```

必须手动删除任何已编辑的配置文件

## 配置镜像加速器

### 配置阿里云镜像加速器

[https://cr.console.aliyun.com/cn-heyuan/instances/mirrors](https://cr.console.aliyun.com/cn-heyuan/instances/mirrors)

  
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122252852.png)

按上图的操作文档添加，之后重启`docker`

```bash
# systemctl daemon-reload
# systemctl restart docker
```

检查加速器是否配置成功

```bash
# docker info
```

在输出的信息中如果有下面的内容：

```bash
Registry Mirrors:
  https://[你配置的域名]/
```

说明配置成功

# 总结

至此有关`Docker`相关的安装教程结束

# 参考链接:
[官网安装教程](https://docs.docker.com/engine/install/centos/)
