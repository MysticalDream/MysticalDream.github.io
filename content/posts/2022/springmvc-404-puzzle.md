---
title: "【noHandlerFound(DispatcherServlet.java:1278)和No mapping for GET】SpringMVC 404和中文乱码问题和解决方案记录"
date: 2022-04-03T19:50:13+08:00
draft: false
# summary: "博客的概述" # 文章简单描述，会展示在主页
categories:
- java

tags:
- spring
- java

---

# 前言

刚开始学习`springmvc`，却被`404`和`乱码`搞得晕头转向,为此还没学多少`springmvc`的其他知识，就开始调试它的源码查找问题了,还花了一天的时间，这里先记录一下。**如有错误，可以指出。**

# demo项目概述

## 项目结构


![picture 2](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122049184.png)  

## tomcat服务器配置


![picture 3](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122049496.png)  

## 相关java代码

```java
package com.wcj.springmvc.controller;

import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.mvc.Controller;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * @description: UserController
 * @date: *****
 * @author: ******
 */
public class UserController implements Controller {

    @Override
    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
        return new ModelAndView("/WEB-INF/templates/index.html");
    }
}

```

## 相关配置文件

- **springmvc-servlet.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    
    <bean name="/" class="com.wcj.springmvc.controller.UserController"/>

</beans>

```

- **web.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <filter>
        <filter-name>characterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <!--设置编码格式-->
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
        <!--设置编码格式的生效范围-->
        <init-param>
            <param-name>forceEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>

    <filter-mapping>
        <filter-name>characterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <servlet>
        <servlet-name>dispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springmvc-servlet.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>dispatcherServlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>


    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:spring-context.xml</param-value>
    </context-param>


</web-app>
```

# 出现的问题

## 404问题

按上述配置后访问会出现404问题

 
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122049440.png) 
控制台输出

```java
[WARN ] org.springframework.web.servlet.DispatcherServlet.noHandlerFound(DispatcherServlet.java:1278) - No mapping for GET /my_springmvc/WEB-INF/templates/index.html
```

由于刚刚学习，对于springmvc的处理机制还没熟悉，
不确定是否按下面的方式返回`ModelAndView`对象是否正确

```java
  return new ModelAndView("/WEB-INF/templates/index.html");
```

于是，我尝试了将`index.html`改为`index.jsp`，在`/WEB-INF/templates`目录下新建一个index.jsp，发现访问正常，这个可以说明这样配置是可以访问的

>其实是因为我们访问`/`这个路径会被配置为` <url-pattern>/</url-pattern>`的`dispatcherServlet`处理，`dispatcherServlet`会调用我们配置了路径为`/`的处理方法，即`UserController`。`springmvc`获取`ModelAndView`,将请求转发到能处理`/WEB-INF/templates/index.jsp`的`servlet`上，这个请求实际上会转发给`tomcat`处理`jsp`的`servlet`，所以可以正确响应
 
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122049556.png) 

## 404问题尝试解决方案

>如果对tomcat的路径匹配感兴趣可以看一下我之前写的  
{{<linkx "tomcat-url-pattern.md">}}

### 配置默认映射(失败)

---

我在网上查了一下,发现有些是说配置如下映射可以解决html访问的问题  

```xml

 <servlet-mapping>
        <servlet-name>default</servlet-name>
        <url-pattern>*.html</url-pattern>
</servlet-mapping>

```

然而这个配置并没有生效，这个配置看似可以匹配`/WEB-INF/templates/index.html`这个路径。但是，我们一开始访问的路径是`/`，而且这个并**不会**匹配到配置为` <url-pattern>/</url-pattern>`的`dispatcherServlet`,而是匹配到`tomcat`默认的`DefaultServlet`,这个和tomcat的路径匹配规则有关，因为我们配置了上面的`*.html`拓展名映射，这个就会导致`/`匹配到了`DefaultServlet`，它实际会访问`/index.html`，但是我并没有在根目录下放置`index.html`这个文件，所以会出现`404`的问题。  

看来这个配置只是解决访问根目录下的`index.html`的问题。但是，我要访问的是在`/WEB-INF/templates/index.html`这个路径下的  

### 配置jsp映射(成功)

---

接着，我又看到一个配置  

```xml
    <servlet-mapping>
        <servlet-name>jsp</servlet-name>
        <url-pattern>*.html</url-pattern>
    </servlet-mapping>
```

![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122050653.png)

将`html`拓展名的请求交给处理`jsp`的`servlet`,显然，这个是可以的，这个就和处理`jsp`的情况一样了，不过这样把`html`的请求交给处理`jsp`的，感觉怪怪的  

### springmvc配置`<mvc:default-servlet-handler/>`标签(成功)  

---
所以再接着，我看到一个说在`springmvc`的配置文件中加入`<mvc:default-servlet-handler/>`，这个配置后也是可以正确访问到的  

>这个配置会在`springmvc`的上下文中定义一个`DefaultServletHttpRequestHandler`,这个会在没有其他更具体的映射（即到控制器）可以匹配时会执行，这个程序会转发请求到默认Servlet

---

#### 注意

> 默认情况下，springmvc会默认读取`DispatcherServlet.properties`文件列举的`HandlerMapping`  
 
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122050932.png) 
  
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122050484.png)
  
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122050669.png)
但是配置了`<mvc:default-servlet-handler/>`会导致默认配置失效，但是这个配置也会自动注册一些`HandlerMapping`；除了会注册`DefaultServletHttpRequestHandler` 还将注册一个`SimpleUrlHandlerMapping`用于映射资源请求，以及一个`HttpRequestHandlerAdapter` ,此外还会注册如下组件  
  
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122050059.png)
所以如果只配置`<mvc:default-servlet-handler/>`，则只会有`SimpleUrlHandlerMapping`和`BeanNameUrlHandlerMapping`这两个`HandlerMapping`，没有`RequestMappingHandlerMapping`,会导致使用`@RequestMapping`之类注解的配置失效，无法映射。
>
> - 解决这个问题可以在`springmvc`配置文件中配置`<mvc:annotation-driven/>`，这个会自动注入`RequestMappingHandlerMapping`和`BeanNameUrlHandlerMapping`  
>     
>![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122051755.png)
当然我们也可以手动注册
>
>```xml
>
><bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping">
>        <property name="order" value="0"/>
></bean>
><bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter"/>
>
>```

## 乱码问题

在处理完404问题后，还有一个初学者经常见到的问题，那就是中文乱码了

- `index.html`文件内容  

  
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122051830.png)
- 浏览器结果  

  
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122051256.png)

- 响应头

  
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122051023.png)

我使用的是idea编辑器，我设置了文件编码都是`utf-8`  


![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122051404.png)

### 解决过程

在web项目中出现了乱码，一般可以通过过滤器来解决，springmvc中提供了一个处理编码得过滤器`CharacterEncodingFilter`
,在web.xml中配置如下过滤器即可

```xml
<filter>
        <filter-name>characterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <!--设置编码格式-->
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
        <!--设置编码格式的生效范围-->
        <init-param>
            <param-name>forceEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>

    <filter-mapping>
        <filter-name>characterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```

然而这个配置并没有解决`index.html`中文乱码得问题，而且，我一开始就配置了这个过滤器的。但是，这个配置确实是设置了编码的了，不过为什么没作用呢？

在java中我知道的可以影响编码的属性有`encoding`和`file.encoding`，于是我想到打印一下`file.encoding`属性的值是否是utf-8

 
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122052318.png) 

输出如下

  
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122052545.png)

发现他的值是`GBK`，于是我在`vm options`中加入
`-Dfile.encoding=utf-8`，重启服务器

  
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122052417.png)

最后发现，乱码问题解决

>`file.encoding`在这里影响的是什么呢?，既然`response`已经设置编码为`utf-8`，那么说明这个响应到浏览器的编码是没有问题的，还有一个就是读取本地的`index.html`文件时所使用的编码，如果以`GBK`读取`index.html`文件，再以`UTF-8`响应文件给浏览器，那么就会出现问题了，可见这个`file.encoding`在这里影响的是读取本地文件的编码

  
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122053353.png)

>浏览器可能会缓存，导致页面没有变化，可以禁用缓存，多刷新几次
  
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122053955.png)

# 总结

遇到问题，我们除了利用好搜索引擎，合理debug查找问题也是一种锻炼，这样可以更好理解代码的流程，提高自己的调式和排错能力
