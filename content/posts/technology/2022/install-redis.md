---
title: "Redis安装教程"
date: 2022-03-29T11:11:06+08:00
draft: false
# summary: "博客的概述" # 文章简单描述，会展示在主页
# categories:
# - category 1
# - category 2

tags:
- redis
- 环境安装

---


**其他相关文章**  

{{<linkx "java-use-redis.md">}}


---
>本次安装环境:  
>
>* VMware 16.2.2
>* Linux系统版本:`CentOS Linux release 7.9.2009 (Core)`

# 前言

`Redis`是一个开放源码（BSD授权）的内存数据结构存储，用作数据库、缓存和消息中介。


# 安装运行redis

## 下载源文件

在命令行输入以下命令，从`redis`官网提供的下载站点获取最新稳定版本的redis的源文件

```shell
# wget https://download.redis.io/redis-stable.tar.gz
```

## 解压redis源文件

在下载下来的`redis`源文件压缩包目录下，执行以下命令

```shell
# tar -xzvf redis-stable.tar.gz -C /opt
```

> `注意：-C 指定解压到的位置，这里我解压到/opt下`

## 编译redis

切换到解压到的目录下

```shell
# cd /opt/redis-stable
```

然后执行`make`命令

```shell
# pwd
/opt/redis-stable

# make
```

会看到相关信息的输出

![terminal 1](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122003311.png)

构建结束时的信息

![terminal 2](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122004998.png)

如果编译没有问题，会在src目录中会生成几个redis二进制文件

* redis-server： 用来启动`redis`服务的程序

* redis-cli：redis客户端管理工具，是与`redis`交互的命令行程序
  
* redis-benchmark：`redis`基准性能测试程序，可以用来测试`redis`的相关性能

我们现在还不能在其他地方执行redis-server之类的命令，我们需要把这些二进制文件安装到/usr/local/bin中,执行`make install`命令

```shell
# pwd
/opt/redis-stable

# make install
```

## 运行redis

安装后，您可以通过执行`redis-server`命令，开启redis服务

```shell
# redis-server
```

执行这个命令后，如果顺利的话，我们会看到redis的启动日志

![terminal 3](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122005239.png)
我们还可以看到有一个警告  

![terminal 4](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122006159.png)

**警告信息：**
>警告:没有指定配置文件，`使用默认配置`。要指定配置文件，请使用redis-server /path/to/redis.conf

根据提示，我们可以用下面的方式启动redis服务  

```shell
# redis-server /opt/redis-stable/redis.conf
```
![5](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122004440.png)
可以发现没有了刚才的警告

## 测试redis服务是否正常

不要关闭启动redis服务的窗口，重新开启一个命令行窗口，在新的窗口输入

```shell
# redis-cli
```

会看到进入`>`符号的模式

```shell
127.0.0.1:6379>
```

要检查`redis`服务是否正常工作，我们可以发送`ping`命令测试

```shell
127.0.0.1:6379>ping
```

如果服务器运作正常的话会输出

```shell
PONG
```

退出`redis-cli`程序

```shell
127.0.0.1:6379>quit
```

或  

```shell
127.0.0.1:6379>exit
```

## 停止redis服务

因为默认情况下，redis不作为守护程序运行，我们只要终止redis的命令行界面的运行就行

* 我们可以在启动redis-server的窗口，直接按 `Ctrl+C` 来停止redis服务
* 或则直接关闭这个命令行窗口

# 总结

以上就是我在虚拟机上安装redis的过程
