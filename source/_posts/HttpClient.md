---
title: HttpClient
date: 2017-04-14 10:56:56
category: Java
tags: JavaWeb
toc: true
---
## 1. 简介
HttpClient是Apache Jakarta Common下的子项目，用来提供高效的、最新的、功能丰富的支持HTTP协议的客户端编程工具包，并且它支持HTTP协议最新的版本和建议。HttpClient已经应用在很多的项目中，比如Apache Jakarta上很著名的另外两个开源项目Cactus和HTMLUnit都使用了HttpClient。

## 2. 使用方法
使用HttpClient发送请求、接收响应很简单，一般需要如下几步即可。 
1. 创建HttpClient对象。
2. 创建请求方法的实例，并指定请求URL。如果需要发送GET请求，创建HttpGet对象；如果需要发送POST请求，创建HttpPost对象。
3. 如果需要发送请求参数，可调用HttpGet、HttpPost共同的setParams(HetpParams params)方法来添加请求参数；对于HttpPost对象而言，也可调用setEntity(HttpEntity entity)方法来设置请求参数。
4. 调用HttpClient对象的execute(HttpUriRequest request)发送请求，该方法返回一个HttpResponse。
5. 调用HttpResponse的getAllHeaders()、getHeaders(String name)等方法可获取服务器的响应头；调用HttpResponse的getEntity()方法可获取HttpEntity对象，该对象包装了服务器的响应内容。程序可通过该对象获取服务器的响应内容。
6. 释放连接。无论执行方法是否成功，都必须释放连接

<!-- more -->

代码整体示例：
```java
import org.apache.commons.httpclient.*;
import org.apache.commons.httpclient.methods.*;
import org.apache.commons.httpclient.params.HttpMethodParams;
import java.io.*;

public class HttpClientTutorial {
  
  private static String url = "http://www.apache.org/";
  
  public static void main(String[] args) {
      
    // 创建HttpClient对象
    HttpClient client = new HttpClient();
    
    // 创建请求方法的实例
    GetMethod method = new GetMethod(url);
    
    // 请求参数
    method.getParams().setParameter(HttpMethodParams.RETRY_HANDLER, 
    		new DefaultHttpMethodRetryHandler(3, false));
            
    try {
      // Execute the method.
      int statusCode = client.executeMethod(method);
      
      if (statusCode != HttpStatus.SC_OK) {
        System.err.println("Method failed: " + method.getStatusLine());
      }
      
      // Read the response body.
      byte[] responseBody = method.getResponseBody();
      
      // Deal with the response.
      // Use caution: ensure correct character encoding and is not binary data
      System.out.println(new String(responseBody));
    } catch (HttpException e) {
        
      System.err.println("Fatal protocol violation: " + e.getMessage());
      e.printStackTrace();
    } catch (IOException e) {
        
      System.err.println("Fatal transport error: " + e.getMessage());
      e.printStackTrace();
    } finally {
        
      // Release the connection.
      method.releaseConnection();
    }  
  }
}
```