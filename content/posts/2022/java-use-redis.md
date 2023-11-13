---
title: "Java连接远端服务器的Redis"
date: 2022-03-29T11:16:14+08:00
draft: false
# summary: "博客的概述" # 文章简单描述，会展示在主页
categories:
- java

tags:
- redis
- java

---


# 前言

在**远端服务器**安装好`redis`后，在没有做其他的处理时,直接在**本地**用`java`代码连接会出现一些问题，下面就是我出现的问题及解决方法

## 需要的依赖

Java redis 驱动

* 下载驱动包  [`jedis.jar`](https://mvnrepository.com/artifact/redis.clients/jedis)
* 在你的 `classpath` 中添加该驱动包

![picture 13](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122035625.png)
![picture 14](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122036806.png)  

>如果是maven项目的话，可以添加依赖
>
>```xml
><dependency>
>    <groupId>redis.clients</groupId>
>    <artifactId>jedis</artifactId>
>    <version>4.2.0</version>
></dependency>
>```

## 启动redis服务

* **首先肯定要开启服务器的`redis`服务**  
  
在服务器命令行输入以下命令启动`redis`服务

```shell
# redis-server /opt/redis-stable/redis.conf
```

可以看到`redis`启动后的输出日志  
![terminal 5](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122039559.png)

## 编写和运行测试代码

编写测试连接的代码  

```java
        //连接redis服务
        Jedis jedis = new Jedis("192.168.163.131", 6379);
        // 如果redis服务设置了密码,我只是测试，所以没有设置密码
        //jedis.auth("66666");
        //检查服务是否正常运行
        System.out.println("返回值: " + jedis.ping());
```

运行代码，一开始报错信息摘要:  

`java.net.SocketTimeoutException: Connect timed out...`  

详细报错:

```java
redis.clients.jedis.exceptions.JedisConnectionException: Failed to connect to any host resolved for DNS name.

at redis.clients.jedis.DefaultJedisSocketFactory.connectToFirstSuccessfulHost(DefaultJedisSocketFactory.java:63)
at redis.clients.jedis.DefaultJedisSocketFactory.createSocket(DefaultJedisSocketFactory.java:87)
at redis.clients.jedis.Connection.connect(Connection.java:180)
at redis.clients.jedis.Connection.sendCommand(Connection.java:152)
at redis.clients.jedis.Connection.sendCommand(Connection.java:135)
at redis.clients.jedis.Jedis.ping(Jedis.java:355)
at RedisTest.connectionTest(RedisTest.java:18) <24 internal lines>
Suppressed: java.net.SocketTimeoutException: Connect timed out
    at java.base/sun.nio.ch.NioSocketImpl.timedFinishConnect(NioSocketImpl.java:546)
    at java.base/sun.nio.ch.NioSocketImpl.connect(NioSocketImpl.java:597)
    at java.base/java.net.SocksSocketImpl.connect(SocksSocketImpl.java:327)
    at java.base/java.net.Socket.connect(Socket.java:633)
    at redis.clients.jedis.DefaultJedisSocketFactory.connectToFirstSuccessfulHost(DefaultJedisSocketFactory.java:73)
... 30 more


Process finished with exit code -1

```

这个时候我们可以查看服务器的`redis`服务**是否开启**，但是，我是提前开启了的，所以不是这个原因。  

那么我们应该检查服务器的防火墙是否开启  

```shell
# systemctl status firewalld
```

![picture 12](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122040378.png)

可以看到我的防火墙是开启了的，接着输入下面的命令，查看**开放端口情况**

```shell
# firewall-cmd --zone=public --list-ports
```

我的除了默认的ssh服务的22端口，还没有开放任何端口，所以需要开放`redis`服务的端口`6379`  

```shell
# firewall-cmd --zone=public --add-port=6379/tcp --permanent  
```

之后重启防火墙服务  

```shell
# restart firewalld
```

再次运行测试代码  

发现依旧报同样的错误:  

`java.net.SocketTimeoutException: Connect timed out...`  

详细报错:

```java
redis.clients.jedis.exceptions.JedisConnectionException: Failed to connect to any host resolved for DNS name.

at redis.clients.jedis.DefaultJedisSocketFactory.connectToFirstSuccessfulHost(DefaultJedisSocketFactory.java:63)
at redis.clients.jedis.DefaultJedisSocketFactory.createSocket(DefaultJedisSocketFactory.java:87)
at redis.clients.jedis.Connection.connect(Connection.java:180)
at redis.clients.jedis.Connection.sendCommand(Connection.java:152)
at redis.clients.jedis.Connection.sendCommand(Connection.java:135)
at redis.clients.jedis.Jedis.ping(Jedis.java:355)
at RedisTest.connectionTest(RedisTest.java:18) <24 internal lines>
Suppressed: java.net.SocketTimeoutException: Connect timed out
    at java.base/sun.nio.ch.NioSocketImpl.timedFinishConnect(NioSocketImpl.java:546)
    at java.base/sun.nio.ch.NioSocketImpl.connect(NioSocketImpl.java:597)
    at java.base/java.net.SocksSocketImpl.connect(SocksSocketImpl.java:327)
    at java.base/java.net.Socket.connect(Socket.java:633)
    at redis.clients.jedis.DefaultJedisSocketFactory.connectToFirstSuccessfulHost(DefaultJedisSocketFactory.java:73)
... 30 more


Process finished with exit code -1

```

为什么还是出现错误的，经过查阅，发现redis默认只允许在本地连接，远程连接需要进行配置  

我们接着需要到redis的配置文件中进行修改，我的配置文件是在`/opt/redis-stable/redis.conf`  

>我们需要修改`bind配置`，这个`bind配置`指定`redis`接收指定`IP地址`的请求，默认绑定接口是 127.0.0.1

* 注释掉划红线的配置  

`注释掉这个bind配置相当于接收网络中所有ip的请求，在生产环境中最好设置指定的ip而不是注释掉`  


![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122041543.png)  
也就是在前面加上`#`
  
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122042505.png)

---

* 当然，我们也可以指定连接地址  

  
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122042004.png)

---
`注意：`  
>我们如果是**指定连接地址**，一切顺利的话，测试代码就可以正确运行了  

**但是**，如果我们只是**注释掉`bind`配置**发现运行测试代码还是会报错，真是一波三折  

报错摘要:
`redis.clients.jedis.exceptions.JedisDataException: DENIED Redis is running in protected mode because protected mode is enabled, no bind address was specified, no authentication password is requested to clients. In this mode connections are only accepted ...`  

详细信息:  

```java

redis.clients.jedis.exceptions.JedisDataException: DENIED Redis is running in protected mode because protected mode is enabled, no bind address was specified, no authentication password is requested to clients. In this mode connections are only accepted from the loopback interface. If you want to connect from external computers to Redis you may adopt one of the following solutions: 1) Just disable protected mode sending the command 'CONFIG SET protected-mode no' from the loopback interface by connecting to Redis from the same host the server is running, however MAKE SURE Redis is not publicly accessible from internet if you do so. Use CONFIG REWRITE to make this change permanent. 2) Alternatively you can just disable the protected mode by editing the Redis configuration file, and setting the protected mode option to 'no', and then restarting the server. 3) If you started the server manually just for testing, restart it with the '--protected-mode no' option. 4) Setup a bind address or an authentication password. NOTE: You only need to do one of the above things in order for the server to start accepting connections from the outside.

    at redis.clients.jedis.Protocol.processError(Protocol.java:96)
    at redis.clients.jedis.Protocol.process(Protocol.java:137)
    at redis.clients.jedis.Protocol.read(Protocol.java:192)
    at redis.clients.jedis.Connection.readProtocolWithCheckingBroken(Connection.java:316)
    at redis.clients.jedis.Connection.getStatusCodeReply(Connection.java:243)
    at redis.clients.jedis.Jedis.ping(Jedis.java:356)
    at RedisTest.connectionTest(RedisTest.java:18)

```

翻译一下报错的提示信息:
>因为**启用了保护模式**，没有指定绑定地址，没有向客户端请求身份验证密码。在此模式下，**仅接受来自环回接口的连接**。如果您想从外部计算机连接到 `redis`  
您可以采用以下解决方案之一：  
>
>1. 通过从服务器的同一主机连接到 Redis，只需禁用保护模式，从环回接口发送命令 `'CONFIG SET protected-mode no'`，但是如果这样做，请确保 `Redis` 不能从 `Internet` 公开访问。使用 `CONFIG REWRITE` 使此更改永久生效  
>2. 或者，您可以通过编辑 `Redis` 配置文件并将保护模式选项设置为`'no'`来禁用保护模式，然后重新启动服务器  
>3. 如果您手动启动服务器只是为了测试，请使用`"--protected-mode no"`选项重新启动它  
>4. 设置绑定地址或认证密码  
>注意：  
>您只需执行上述操作之一，服务器即可开始接受来自外部的连接

根据提示  
我们可以先关闭原先开启的`redis`服务,重新开启`redis`服务，由于我们只是测试所以可以添加`"--protected-mode no"`选项关闭保护模式，启动`redis`服务

```shell
# redis-server ./redis.conf --protected-mode no
```

如果顺利的话，我们运行测试代码将看到  

成功输出:  
  
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122042336.png)

当然我们也可以按如下方式修改配置文件`/opt/redis-stable/redis.conf`  

修改划红线的地方的配置，原先默认是`yes`，改成`no`  
  
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122042759.png)
修改之后  
  
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122042345.png)

同样运行测试代码  
成功输出:
  
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122043473.png)

# 总结

以上就是我使用java连接远程redis服务的过程
