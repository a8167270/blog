---
title: Servlet
date: 2017-04-13 15:33:26
category: Java
tags: JavaWeb
toc: true
---

在某种程度上来讲，可以将servlet看作是含有HTML的Java程序；将JSP看作是含有Java代码的HTML页面。JSP文档可以理解成是编写servlet的另一种形式，JSP页面会被翻译成servelt，而servlet会被编译。在整个请求期间运行的就是servlet。

# Servlet简介

## Servlet的生命周期
服务器只对每一个servlet创建单一实例，每个用户请求会创建新的线程，将用户请求交付给相应的doGet和doPost进行处理。

Servlet 生命周期：Servlet 加载--->实例化--->服务--->销毁。


1. init（）：在Servlet的生命周期中，仅执行一次init()方法。它是在服务器装入Servlet时执行的，负责初始化Servlet对象。可以配置服务器，以在启动服务器或客户机首次访问Servlet时装入Servlet。无论有多少客户机访问Servlet，都不会重复执行init（）。

2. service（）：它是Servlet的核心，负责响应客户的请求。每当一个客户请求一个HttpServlet对象，该对象的Service()方法就要调用，而且传递给这个方法一个“请求”（ServletRequest）对象和一个“响应”（ServletResponse）对象作为参数。在HttpServlet中已存在Service()方法。默认的服务功能是调用与HTTP请求的方法相应的do功能。

3. destroy（）： 仅执行一次，在服务器端停止且卸载Servlet时执行该方法。当Servlet对象退出生命周期时，负责释放占用的资源。一个Servlet在运行service()方法时可能会产生其他的线程，因此需要确认在调用destroy()方法时，这些线程已经终止或完成。

Servlet架构图如下：

![Servlet生命周期](Servlet与JSP/ServletLifeCycle.jpg)

## Servlet的工作流程

1. Web Client 向Servlet容器（Tomcat）发出Http请求
2. Servlet容器接收Web Client的请求
3. Servlet容器创建一个HttpRequest对象，将Web Client请求的信息封装到这个对象中。
4. Servlet容器创建一个HttpResponse对象
5. Servlet容器调用HttpServlet对象的service方法，把HttpRequest对象与HttpResponse对象作为参数传给 HttpServlet 对象。
6. HttpServlet调用HttpRequest对象的有关方法，获取Http请求信息。
7. HttpServlet调用HttpResponse对象的有关方法，生成响应数据。
8. Servlet容器把HttpServlet的响应结果传给Web Client。

## Servlet的创建

1. Servlet容器启动时：读取web.xml配置文件中的信息，构造指定的Servlet对象，创建ServletConfig对象，同时将ServletConfig对象作为参数来调用Servlet对象的init方法。

2. 在Servlet容器启动后：客户首次向Servlet发出请求，Servlet容器会判断内存中是否存在指定的Servlet对象，如果没有则创建它，然后根据客户的请求创建HttpRequest、HttpResponse对象，从而调用Servlet对象的service方法。

3. Servlet Servlet容器在启动时自动创建Servlet，这是由在web.xml文件中为Servlet设置的<load-on-startup>属性决定的。从中我们也能看到同一个类型的Servlet对象在Servlet容器中以单例的形式存在。


# Servlet的配置
## 映射配置
### web.xml配置
```xml
<servlet>
	<servlet-name>HelloWorld</servlet-name>
	<servlet-class>jp.co.xiehl.servlet.ch7.HelloWorldServlet</servlet-class>
</servlet>
//servlet映射配置
<servlet-mapping>
	<servlet-name>HelloWorld</servlet-name>
	<url-pattern>/hello</url-pattern>
</servlet-mapping>

访问时：localhost:8080/工程名/hello即可。

```

### servlet3.0 注解
```java
@WebServlet(name=”Hello”, urlPatterns={“/hello.view”}, loadOnStartup=1)
public class HelloServlet extends HttpServlet {}
```
上面的例子表示为，name为Hello的servlet，url为hello.view的。

## 过滤器
Servlet过滤器可以动态的拦截请求和响应，可以实现以下目的：

* 在客户端请求访问后端资源之前，拦截请求
* 在服务端的响应发送客户端之前，处理响应

### 过滤器接口
过滤器是实现`javax.servlet.Filter`接口的类。接口包含以下三个方法：
![Filter interface](Servlet与JSP/filter.png)

过滤器示例：
```java
//导入必需的 java 库
import javax.servlet.*;
import java.util.*;

//实现 Filter 类
public class LogFilter implements Filter  {
	public void  init(FilterConfig config) throws ServletException {
		// 获取初始化参数
		String site = config.getInitParameter("Site"); 
		// 输出初始化参数
		System.out.println("网站名称: " + site); 
	}
	
	public void  doFilter(ServletRequest request, ServletResponse response, FilterChain chain){
		// 输出站点名称
		System.out.println("站点网址：http://www.runoob.com");
		// 把请求传回过滤链
		chain.doFilter(request,response);
	}
	
	public void destroy( ){
		/* 在 Filter 实例被 Web 容器从服务移除之前调用 */
	}
}
```

### 过滤器实现
```xml
<filter>
	<filter-name>LoginFilter</filter-name>
	<filter-class>com.runoob.test.LogFilter</filter-class>
	<init-param>
		<param-name>Site</param-name>
		<param-value>菜鸟教程</param-value>
	</init-param>
</filter>
```

## 2. 状态代码

Web服务器对请求的响应，一般有一个状态行、一些响应报头、一个空行和相应的文档构成；Http响应的状态行由HTTP版本、一个状态代码和一段相关的消息组成。但是消息直接与状态代码相关，而http的版本是由服务器来决定的，故而，servlet需要做的只是设置状态代码。系统自动设置的代码为200。如果需要设置状态代码，则可以使用response.setStatus,response.sendRedirect或response.sendError方法。

### 设置状态代码：setStatus
> 在向客户程序发送任何文档内容之前设置状态代码


setStatus方法以一个整数（状态代码，int类型）为参数，但为了避免出错，尽量不要使用数字，而要使用HttpServletResponse中定义的常量。每个常量的名字都来自于每个常量所对应的标准HTTP1.1消息，全部大写并添加SC（Status Code）前缀，状态代码404对应的消息为Not Found，与之对应的常量是SC_NOT_FOUND。

Http1.1中可用的特定的状态代码，如下

|代码区间|描述|
|---|---|
|100-199|都是信息性的，标示客户应该采取的其他动作|
|200-299|标示请求成功|
|300-399|用于已移走的文件，常常包括Location报头，指出新的地址|
|400-499|表明由客户引发的错误|
|500-599|表示由服务器引发的错误|

## Http相应报头

指定报头，最常用的方式是使用HttpServletResponse的setHeader方法，这个方法接收两个字符串：报头的名称和报头的值。和设置状态代码一样，必须在返回实际的文档之前指定相关报头。

```java
 setHeader（String headerName,String headerValue） 
```
Http允许相同的报头名多次出现，例如，多个Accept和Set-Cookie报头分别指定所支持的不同MIME类型和不同cookie。