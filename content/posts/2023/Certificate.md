---
title: "本地测试使用自签名证书以开启网站https（例子说明：Nginx、Tomcat）"
date:  2023-05-16T20:41:49+08:00
draft: false
# summary: "博客的概述" # 文章简单描述，会展示在主页
categories:
- 计算机网络

tags:
- 数字证书
- 计算机网路
- web安全
- nginx
- tomcat

---

# 数字证书


## 简介
数字证书是由证书颁发机构(CA)签名并颁发的电子文件,用于建立网络连接的身份认证和加密通信。SSL 证书是数字证书的一种。

## 工作原理

>SSL 证书包含以下信息：
>
> * 针对其颁发证书的域名
> * 证书颁发给哪一个人、组织或设备
> * 证书由哪一证书颁发机构颁发
> * 证书颁发机构的数字签名
> * 关联的子域
> * 证书的颁发日期
> * 证书的到期日期
> * 公钥

1. 客户端向服务器发起连接请求，并请求建立安全连接。如果服务器支持SSL协议，它会向客户端返回一个数字证书。

2. 客户端收到服务器的数字证书，并验证证书的合法性。这个过程包括检查数字证书是否由受信任的证书颁发机构（CA）签发，证书是否已过期，证书的主题名称是否与访问的网站域名匹配等。

3. 如果数字证书被验证为有效，客户端会生成一个随机的对称密钥，并使用服务器的公钥加密这个密钥，然后将其发送给服务器。

4. 服务器使用自己的私钥解密客户端发送的对称密钥，然后开始使用这个密钥加密和解密通信数据。

5. 现在，客户端和服务器之间建立了安全连接，可以开始在加密的通道上发送和接收数据。

## 证书链

在一个证书链中，最顶层的证书被称为根证书（Root Certificate），它是一个自签名的数字证书，表明了该证书颁发机构的身份和可信任性。下面的证书则是通过逐级签名形成的链式结构，每个证书都以上一级证书颁布者的公钥进行签名，以确保其合法性和真实性。

## 获取SSL证书和自签名证书

1. 购买证书：可以从不同的 SSL 证书供应商（如 DigiCert、GlobalSign 等）购买 SSL 证书。这些供应商会提供不同类型的证书，例如域名验证、组织验证或扩展验证证书。

2. 免费证书：可以使用免费的 SSL 证书，例如 Let's Encrypt。这些证书由非营利组织颁发，因此相对更便宜且易于获取。

3. 自签名证书：可以生成自己的 SSL 证书并将其用于您的网站。但是，由于这些证书没有经过信任的第三方认证机构的颁发，因此在某些情况下可能会被浏览器视为不安全。

本地测试，并不需要使用公开信任的证书，可以使用自签名证书来启用HTTPS，下面就来讲一下如何生成自签名证书吧

## 前提条件

安装 OpenSSL 工具

windows：[https://slproweb.com/products/Win32OpenSSL.html](https://slproweb.com/products/Win32OpenSSL.html)

linux:[https://www.openssl.org/source/](https://www.openssl.org/source/)

本文使用的版本是 `OpenSSL 3.0`

## 创建根 CA 证书

### 1.生成 RSA 私钥

打开命令行终端，进入要存储证书的目录:

>openssl-genpkey文档：[https://www.openssl.org/docs/man3.0/man1/openssl-genpkey.html](https://www.openssl.org/docs/man3.0/man1/openssl-genpkey.html)

输入以下命令
```bash
openssl genpkey -algorithm RSA -out ca.pem -pkeyopt rsa_keygen_bits:2048
```

>genpkey：这告诉 OpenSSL 使用 "genpkey" 子命令，用于生成新的私钥。
>
>-algorithm RSA：此选项指定要使用的密钥算法。在本例中，使用的是 RSA。
>
>-out ca.pem：这告诉 OpenSSL 将生成的密钥写入名为 "ca.pem" 的文件中。
>
>-pkeyopt rsa_keygen_bits:2048：这设置 RSA 密钥的大小为 2048 位。RSA 密钥的位数越大，安全性就越高，但生成和使用密钥的成本也越高。

这将生成一个 RSA 私钥并将其保存到名为 key.pem 的文件中

### 2.生成根证书签名请求（CSR）

>openssl-req文档:[https://www.openssl.org/docs/man3.0/man1/openssl-req.html](https://www.openssl.org/docs/man3.0/man1/openssl-req.html)

**纯命令行的形式**：
```bash
openssl req -new -key ca.pem -out ca.csr -subj "/OU=Self Sign CA/O=Self Sign/CN=Self Sign" -addext "keyUsage=critical,keyCertSign,cRLSign" -addext "basicConstraints=critical,CA:TRUE"
```
>req：该选项指定我们要使用“req”子命令来生成CSR。
>
>-new：该选项告诉OpenSSL创建一个新的CSR。
>
>-key ca.pem：该选项指定在创建CSR时要使用的私钥文件的路径。
>
>-out ca.csr：该选项指定要为创建的CSR使用的文件名。
>
>-subj "/OU=Self Sign CA/O=Self Sign/CN=Self Sign"：该选项指定CSR的主题。在此例中，主题设置为通用名称“Self Sign”，组织单位“Self Sign CA”和组织“Self Sign”。
>
>-addext "keyUsage=critical,keyCertSign,cRLSign"：该选项向CSR添加密钥用途扩展。密钥用途扩展指定证书中的公钥可以如何使用。
>
>-addext "basicConstraints=critical,CA:TRUE"：该选项向CSR添加基本约束扩展。基本约束扩展指定证书是否可以用作CA（证书颁发机构）。在此例中，基本约束扩展指示证书可以用作CA。

**使用配置文件的方式**:
```bash
openssl req -new -key ca.pem -out ca.csr -config ca_openssl.cnf
```
>-config ca_openssl.cnf：这个选项指定 OpenSSL 使用的配置文件，来指定 CSR 的各个属性。

`ca_openssl.cnf`内容如下:
```
[req]
distinguished_name = req_distinguished_name
req_extensions = ext

[req_distinguished_name]
organizationName = Organization Name (eg, company)
organizationName_default = Self Sign
organizationalUnitName = Organizational Unit Name (eg, section)
organizationalUnitName_default = Self Sign CA
commonName = Common Name (e.g. server FQDN or YOUR name)
commonName_max = 64
commonName_default = Self Sign


[ext]
keyUsage = critical,keyCertSign,cRLSign
basicConstraints = critical,CA:TRUE
```



### 3.生成自签根证书
>openssl-x509文档:[https://www.openssl.org/docs/man3.0/man1/openssl-x509.html](https://www.openssl.org/docs/man3.0/man1/openssl-x509.html)

使用以下命令生成根证书:

如果生成的证书签名请求使用的是纯命令行的形式，则使用下面的命令签署：
```bash
openssl x509 -req -days 365 -in ca.csr -signkey ca.pem -out ca.crt -copy_extensions copy
```
>-signkey(key的别名) ca.pem：这个选项指定用于签署证书的私钥文件的路径。
>
>-copy_extensions copy：这个选项指定将 CSR 中的扩展属性复制到生成的证书中。

如果使用的是配置文件的方式，则使用下面的命令：
```bash
openssl x509 -req -days 365 -in ca.csr -signkey ca.pem -out ca.crt -config ca_openssl.cnf -extensions ext
```

稍后你将使用此证书来为服务器证书签名

## 创建服务器证书

### 1.创建服务器 RSA 私钥

```bash
openssl genpkey -algorithm RSA -out server.pem -pkeyopt rsa_keygen_bits:2048
```

### 2.创建 CSR（证书签名请求）

CSR 是请求证书时向 CA 提供的公钥。 CA 将针对此特定请求颁发证书。

>服务器证书的 CN 必须与颁发者的域不同。 例如，在本例中，颁发者的 CN 是 Self Sign，服务器证书的 CN 是 localhost

>注意: 本例是在本地运行服务的，所以subjectAltName可以配置为`subjectAltName = DNS:localhost,IP:127.0.0.1,IP:127.0.0.2`，如果你的服务不是运行在本地，而是其他域名或者IP的情况下需要根据你的情况修改，比如：
>- 你的服务器IP为192.168.199.151，需要修改成为`subjectAltName = IP:192.168.199.151`
>- 或者你使用的是域名，则需要修改成你自己的域名`subjectAltName = DNS:example.com`

命令1：
```bash
openssl req -new -key server.pem -out server.csr -subj "/C=CN/ST=Beijing/L=Beijing/O=local/OU=local/CN=localhost" -addext "subjectAltName = DNS:localhost,IP:127.0.0.1,IP:127.0.0.2"
```

或者

命令2：
```bash
openssl req -new -key server.pem -out server.csr  -config openssl.cnf
```

openssl.cnf
```
[req]
distinguished_name = req_distinguished_name
req_extensions = v3_req

[req_distinguished_name]
countryName                 = Country Name (2 letter code)
countryName_default         = CN
stateOrProvinceName         = State or Province Name (full name)
stateOrProvinceName_default = Beijing
localityName                = Locality Name (eg, city)
localityName_default        = Beijing
organizationName            = Organization Name (eg, company)
organizationName_default    = local
organizationalUnitName            = Organizational Unit Name (eg, section)
organizationalUnitName_default    = local
commonName                  = Common Name (e.g. server FQDN or YOUR name)
commonName_max              = 64
commonName_default          = localhost

[v3_req]
subjectAltName = @alt_names

[alt_names]
DNS.1 = localhost
IP.1 = 127.0.0.1
IP.2 = 127.0.0.2
```

>注意：[RFC 2818](https://datatracker.ietf.org/doc/html/rfc2818)描述了匹配域名与证书的两种方法：使用subjectAlternativeName扩展中可用的名称，或在缺少SAN扩展的情况下回退到commonName。回退到commonName已经在RFC 2818（2000年发布）中被弃用，但许多TLS客户端仍然支持这种方式,通常还存在错误。
>
>使用subjectAlternativeName 字段,可以明确证书是否表达对IP地址或者域名的绑定,并完全符合Name Constraints定义。但是commonName 是不明确的,正因为如此,基于它的支持已成为Chrome、它使用的库以及TLS生态系统中的安全BUG来源。
>
>废弃commonName 而带来的兼容性风险很低。RFC 2818 已经弃用它近两十年了,最低要求（所有公开信任的证书颁发机构必须遵守）自2012年以来要求存在subjectAltName。自Firefox 48以来，Firefox已经要求所有新颁发的公开信任证书必须包含subjectAltName。

下面命令可以查看证书签名请求内容：
```bash
openssl req -text -in server.csr -noout 
```

### 3.使用 CSR 和私钥生成证书，并使用 CA 的根私钥为该证书签名

使用以下命令以创建证书：

命令1(对应上一步的命令1）：
```bash
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.pem -CAcreateserial -out server.crt -days 365 -copy_extensions copy
```

命令2（对应上一步的命令2）：
```bash
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.pem -CAcreateserial -out server.crt -days 365 -extfile openssl.cnf -extensions v3_req
```

查看生成的证书内容:
```bash
openssl x509 -text -in server.crt -noout
```


## 访问服务器验证

### 安装和卸载根证书
#### 安装
将根证书添加到计算机的受信任根存储中。

`双击ca.crt`文件，在弹出窗口中，点击安装证书
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230516170750.png)

接着下一步

![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230516170919.png)

下一步点击浏览
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230516171041.png)
在弹出窗口中选择`受信任的根证书颁发机构`，确定后下一步
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230516171120.png)
最后点击完成。
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230516171233.png)
#### 卸载
如果需要卸载这个证书，win+r键输入`certlm.msc`，打开管理窗口。

![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230516174219.png)

右侧打开`受信任的根证书颁发机构`>`证书`

![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230516174255.png)

找到`Self Sign`（这个是上面创建的根证书的名字）证书右键删除即可。
### nginx实现https访问
使用`nginx`搭建一个`https`服务器
配置文件：
```
server {
       listen       443 ssl;
       server_name  localhost;

       #ssl证书
       ssl_certificate      D:\\xxx\\test\\server.crt;
       #私钥文件
       ssl_certificate_key  D:\\xxx\\test\\server.pem;

       ssl_session_cache    shared:SSL:1m;
       ssl_session_timeout  5m;

       ssl_ciphers  HIGH:!aNULL:!MD5;
       ssl_prefer_server_ciphers  on;

       location / {
           root   html;
           index  index.html index.htm;
       }
    }
```

启动`nginx`后访问网站`https://localhost`，然后单击浏览器地址框中的锁定图标来验证站点和证书信息。
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230516171605.png)
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230516171636.png)
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230516171710.png)

### tomcat 实现https访问
要在tomcat使用https，需要将服务端证书和私钥文件转换为 `Java Keystore`格式。

将证书导出成浏览器支持的.p12格式 ：
```bash
openssl pkcs12 -export -in server.crt -inkey server.pem -out server.p12
```

接着需要输入一个密码即可

```bash
keytool -importkeystore -srckeystore server.p12 -srcstoretype pkcs12 -destkeystore server.jks -deststoretype jks
```
>keytool时jdk自带的密钥和证书管理工具

输入一个`jks密钥库`密码和输入生成`p12文件`时输入的密码即可。

在Tomcat安装目录的server.xml文件中定义如下配置:

**Tomcat 7：**
```
 <Connector
           protocol="org.apache.coyote.http11.Http11NioProtocol"
           port="8443" maxThreads="200"
           scheme="https" secure="true" SSLEnabled="true"
           keystoreFile="${catalina.home}/conf/server.jks" keystorePass="123456"
           clientAuth="false" sslProtocol="TLS"/>
```
>keystoreFile指定jks文件位置，keystorePass指定`jks密钥库`的密码

**Tomcat 8：**

```
<Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"
               maxThreads="150" SSLEnabled="true">
        <SSLHostConfig>
            <Certificate certificateKeystoreFile="${catalina.home}/conf/server.jks" certificateKeystorePassword="123456" type="RSA" />
        </SSLHostConfig>
    </Connector>
```

或

使用`http/2`
```
<Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"
               maxThreads="150" SSLEnabled="true">
      <UpgradeProtocol className="org.apache.coyote.http2.Http2Protocol" />
        <SSLHostConfig>
            <Certificate certificateKeystoreFile="${catalina.home}/conf/server.jks"
               certificateKeystorePassword="123456"
                         type="RSA" />
        </SSLHostConfig>
    </Connector>
```

记住要将生成的 `Java Keystore 文件(server.jks)`复制到 Tomcat 的 conf 目录下。

进入Tomcat的bin目录，执行startup来启动tomcat。

访问`https://localhost:8443`查看效果：
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230516180858.png)
