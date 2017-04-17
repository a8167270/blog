---
title: Http协议
date: 2017-04-14 10:43:19
category: 网络
tags: 协议
---

大三学习了《计算机网络》的课程，但是老师只把几层协议简单的介绍了一下，就没有深入的去学习。这么多年，一直在使用Http，在去年的时候才开始使用到Https，回过头来发现关于Http的细节性的东西遗漏了很多。欠过账的迟早要还的！在我对Servlet进行研究时，发现很多底层上的参数搞不清楚到底是协议上的还是Servlet上的。所以，还是从Http的协议开始入手，重新学习和总结一下。

## 1、Http简介

HTTP协议是Hyper Text Transfer Protocol（超文本传输协议）的缩写,是用于从万维网（WWW:World Wide Web ）服务器传输超文本到本地浏览器的传送协议。

HTTP协议工作于客户端-服务端架构为上。浏览器作为HTTP客户端通过URL向HTTP服务端即WEB服务器发送所有请求。Web服务器根据接收到的请求后，向客户端发送响应信息。

![image_1bdb8ch37lma16jm7ng5qludb9.png-55.7kB][1]

HTTP是一个基于TCP/IP通信协议来传递数据（HTML 文件, 图片文件, 查询结果等）。

HTTP默认端口号为80，Https的默认端口为443。

![image_1bdb8fnhh17e61k2cgvm1qrg1rf7m.png-19.7kB][2]

### 特点

1、简单快速：客户向服务器请求服务时，只需传送请求方法和路径。请求方法常用的有GET、HEAD、POST。每种方法规定了客户与服务器联系的类型不同。由于HTTP协议简单，使得HTTP服务器的程序规模小，因而通信速度很快。

2、灵活：HTTP允许传输任意类型的数据对象。正在传输的类型由Content-Type加以标记。

3.无连接：无连接的含义是限制每次连接只处理一个请求。服务器处理完客户的请求，并收到客户的应答后，即断开连接。采用这种方式可以节省传输时间。

4.无状态：HTTP协议是无状态协议。无状态是指协议对于事务处理没有记忆能力。缺少状态意味着如果后续处理需要前面的信息，则它必须重传，这样可能导致每次连接传送的数据量增大。另一方面，在服务器不需要先前信息时它的应答就较快。

5、支持B/S及C/S模式。

### TCP & HTTP & UDP:

TCP/IP是个协议组，可分为四个层次：网络接口层、网络层、传输层和应用层。

|网络层|所包含协议|
|---|---|
|网络层|IP协议、ICMP协议、ARP协议、RARP协议和BOOTP协议|
|传输层|TCP协议、UDP协议|
|应用层|有FTP、HTTP、TELNET、SMTP、DNS等协议|

因此，HTTP本身就是一个协议，是从Web服务器传输超文本到本地浏览器的传送协议。

### socket
socket是为了实现通信过程而建立成来的通信管道，其真实的代表是客户端和服务器端的一个通信进程，双方进程通过socket进行通信，而通信的规则采用指定的协议。

socket只是一种连接模式，不是协议，tcp、udp，简单的说（虽然不准确）是两个最基本的协议,很多其它协议都是基于这两个协议如，http就是基于tcp的，.用socket可以创建tcp连接，也可以创建udp连接，这意味着，用socket可以创建任何协议的连接，因为其它协议都是基于此的。

## 2.Http消息
HTTP使用统一资源标识符（Uniform Resource Identifiers, URI）来传输数据和建立连接。
### URL和URI区别
|URI|URL|
|---|---|
|URI，是uniform resource identifier，统一资源标识符，用来唯一的标识一个资源。|URL是uniform resource locator，统一资源定位器，它是一种具体的URI，即URL可以用来标识一个资源，而且还指明了如何locate这个资源URL是Internet上用来描述信息资源的字符串，主要用在各种WWW客户程序和服务器程序上。|
|Web上可用的每种资源如HTML文档、图像、视频片段、程序等都是一个来URI来定位的|采用URL可以用一种统一的格式来描述各种信息资源，包括文件、服务器的地址和目录等|
|URI一般由三部组成：1、访问资源的命名机制。 2、存放资源的主机名。3、资源自身的名称，由路径表示，着重强调于资源。|URL一般由三部组成：1、协议(或称为服务方式)。2、存有该资源的主机IP地址(有时也包括端口号)。3、主机资源的具体地址。如目录和文件名等|

例如：在浏览器地址栏键入URL，按下回车之后会经历以下流程：

1、浏览器向 DNS 服务器请求解析该 URL 中的域名所对应的 IP 地址;

2、解析出 IP 地址后，根据该 IP 地址和默认端口 80，和服务器建立TCP连接;

3、浏览器发出读取文件(URL 中域名后面部分对应的文件)的HTTP 请求，该请求报文作为 TCP 三次握手的第三个报文的数据发送给服务器;

4、服务器对浏览器请求作出响应，并把对应的 html 文本发送给浏览器;

5、释放 TCP连接;

6、浏览器将该 html 文本并显示内容; 　　

### 客户端请求消息
客户端发送一个HTTP请求到服务器的请求消息包括以下格式：请求行（request line）、请求头部（header）、空行和请求数据四个部分组成，下图给出了请求报文的一般格式。
![image_1bdbfq85p1hlknb01gfb1olnlv313.png-15.1kB][3]

#### Get请求
```
GET /562f25980001b1b106000338.jpg HTTP/1.1
Host    img.mukewang.com
User-Agent    Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/51.0.2704.106 Safari/537.36
Accept    image/webp,image/*,*/*;q=0.8
Referer    http://www.imooc.com/
Accept-Encoding    gzip, deflate, sdch
Accept-Language    zh-CN,zh;q=0.8
```
第一部分：请求行，用来说明请求类型,要访问的资源以及所使用的HTTP版本.

GET说明请求类型为GET,[/562f25980001b1b106000338.jpg]为要访问的资源，该行的最后一部分说明使用的是HTTP1.1版本。

第二部分：请求头部，紧接着请求行（即第一行）之后的部分，用来说明服务器要使用的附加信息

从第二行起为请求头部，HOST将指出请求的目的地.User-Agent,服务器端和客户端脚本都能访问它,它是浏览器类型检测逻辑的重要基础.该信息由你的浏览器来定义,并且在每个请求中自动发送等等

第三部分：空行，请求头部后面的空行是必须的

即使第四部分的请求数据为空，也必须有空行。

第四部分：请求数据也叫主体，可以添加任意的其他数据。

这个例子的请求数据为空。

#### post请求
```
POST / HTTP1.1
Host:www.wrox.com
User-Agent:Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; .NET CLR 2.0.50727; .NET CLR 3.0.04506.648; .NET CLR 3.5.21022)
Content-Type:application/x-www-form-urlencoded
Content-Length:40
Connection: Keep-Alive

name=Professional%20Ajax&publisher=Wiley
```
第一部分：请求行，第一行明了是post请求，以及http1.1版本。
第二部分：请求头部，第二行至第六行。
第三部分：空行，第七行的空行。
第四部分：请求数据，第八行。

### 响应消息Response

HTTP响应也由四个部分组成，分别是：状态行、消息报头、空行和响应正文。
![image_1bdbg60ceq2slf71po2416m781g.png-110.7kB][4]

## 3、请求方法和状态码
### 请求方法
根据HTTP标准，HTTP请求可以使用多种请求方法。
HTTP1.0定义了三种请求方法： GET, POST 和 HEAD方法。
HTTP1.1新增了五种请求方法：OPTIONS, PUT, DELETE, TRACE 和 CONNECT 方法。

|序号|方法|描述|
|---|---|---|
|1|   GET	|请求指定的页面信息，并返回实体主体。|
|2|	HEAD	类|似于get请求，只不过返回的响应中没有具体的内容，用于获取报头|
|3|	POST	向|指定资源提交数据进行处理请求（例如提交表单或者上传文件）。数据被包含在请求体中。POST请求可能会导致新的资源的建立和/或已有资源的修改。|
|4|	PUT	从客|户端向服务器传送的数据取代指定的文档的内容。|
|5|	DELETE|	请求服务器删除指定的页面。|
|6|CONNECT|	HTTP/1.1协议中预留给能够将连接改为管道方式的代理服务器。|
|7|	OPTION|S	允许客户端查看服务器的性能。|
|8|	TRACE	|回显服务器收到的请求，主要用于测试或诊断。|

### 状态码
当浏览者访问一个网页时，浏览者的浏览器会向网页所在服务器发出请求。当浏览器接收并显示网页前，此网页所在的服务器会返回一个包含HTTP状态码的信息头（server header）用以响应浏览器的请求。
HTTP状态码的英文为HTTP Status Code。

|分类	|分类描述|
|---|---|
|1**|	信息，服务器收到请求，需要请求者继续执行操作|
|2**|	成功，操作被成功接收并处理|
|3**|	重定向，需要进一步的操作以完成请求|
|4**|	客户端错误，请求包含语法错误或无法完成请求|
|5**|	服务器错误，服务器在处理请求的过程中发生了错误|

常见的状态码
```
200 OK                        //客户端请求成功
400 Bad Request               //客户端请求有语法错误，不能被服务器所理解
401 Unauthorized              //请求未经授权，这个状态代码必须和WWW-Authenticate报头域一起使用 
403 Forbidden                 //服务器收到请求，但是拒绝提供服务
404 Not Found                 //请求资源不存在，eg：输入了错误的URL
500 Internal Server Error     //服务器发生不可预期的错误
503 Server Unavailable        //服务器当前不能处理客户端的请求，一段时间后可能恢复正常
```
### HTTP content-type
Content-Type，内容类型，一般是指网页中存在的Content-Type，用于定义网络文件的类型和网页的编码，决定浏览器将以什么形式、什么编码读取这个文件，这就是经常看到一些Asp网页点击的结果却是下载到的一个文件或一张图片的原因。

### HTTP 响应头信息
HTTP请求头提供了关于请求，响应或者其他的发送实体的信息。

|应答头|说明|
|---|---|
|Allow|	服务器支持哪些请求方法（如GET、POST等）。|
|Content-Encoding|	文档的编码（Encode）方法。只有在解码之后才可以得到Content-Type头指定的内容类型。利用gzip压缩文档能够显著地减少HTML文档的下载时间。Java的GZIPOutputStream可以很方便地进行gzip压缩，但只有Unix上的Netscape和Windows上的IE 4、IE 5才支持它。因此，Servlet应该通过查看Accept-Encoding头（即request.getHeader("Accept-Encoding")）检查浏览器是否支持gzip，为支持gzip的浏览器返回经gzip压缩的HTML页面，为其他浏览器返回普通页面。|
|Content-Length	|表示内容长度。只有当浏览器使用持久HTTP连接时才需要这个数据。如果你想要利用持久连接的优势，可以把输出文档写入 ByteArrayOutputStream，完成后查看其大小，然后把该值放入Content-Length头，最后通过byteArrayStream.writeTo(response.getOutputStream()发送内容。|
|Content-Type|	表示后面的文档属于什么MIME类型。Servlet默认为text/plain，但通常需要显式地指定为text/html。由于经常要设置Content-Type，因此HttpServletResponse提供了一个专用的方法setContentType。|
|Date|	当前的GMT时间。你可以用setDateHeader来设置这个头以避免转换时间格式的麻烦。|
|Expires|	应该在什么时候认为文档已经过期，从而不再缓存它？|
|Last-Modified|	文档的最后改动时间。客户可以通过If-Modified-Since请求头提供一个日期，该请求将被视为一个条件GET，只有改动时间迟于指定时间的文档才会返回，否则返回一个304（Not Modified）状态。Last-Modified也可用setDateHeader方法来设置。|
|Location|	表示客户应当到哪里去提取文档。Location通常不是直接设置的，而是通过HttpServletResponse的sendRedirect方法，该方法同时设置状态代码为302。|
|Refresh	|表示浏览器应该在多少时间之后刷新文档，以秒计。除了刷新当前文档之外，你还可以通过setHeader("Refresh", "5; URL=http://host/path")让浏览器读取指定的页面。 |
|Server|	服务器名字。Servlet一般不设置这个值，而是由Web服务器自己设置。|
|Set-Cookie|	设置和页面关联的Cookie。Servlet不应使用response.setHeader("Set-Cookie", ...)，而是应使用HttpServletResponse提供的专用方法addCookie。参见下文有关Cookie设置的讨论。|
|WWW-Authenticate	|客户应该在Authorization头中提供什么类型的授权信息？在包含401（Unauthorized）状态行的应答中这个头是必需的。例如，response.setHeader("WWW-Authenticate", "BASIC realm=＼"executives＼"")。 |

## 4、补充
### get和post区别

* GET在浏览器回退时是无害的，而POST会再次提交请求。

* GET产生的URL地址可以被Bookmark，而POST不可以。

* GET请求会被浏览器主动cache，而POST不会，除非手动设置。

* GET请求只能进行url编码，而POST支持多种编码方式。

* GET请求参数会被完整保留在浏览器历史记录里，而POST中的参数不会被保留。

* GET请求在URL中传送的参数是有长度限制的，而POST么有。

* 对参数的数据类型，GET只接受ASCII字符，而POST没有限制。

* GET比POST更不安全，因为参数直接暴露在URL上，所以不能用来传递敏感信息。

* GET参数通过URL传递，POST放在Request body中。


**GET和POST还有一个重大区别：**

GET产生一个TCP数据包；POST产生两个TCP数据包。

对于GET方式的请求，浏览器会把http header和data一并发送出去，服务器响应200（返回数据）；
而对于POST，浏览器先发送header，服务器响应100 continue，浏览器再发送data，服务器响应200 ok（返回数据）。

因为POST需要两步，时间上消耗的要多一点，看起来GET比POST更有效。因此Yahoo团队有推荐用GET替换POST来优化网站性能。但这是一个坑！跳入需谨慎。为什么？
1. GET与POST都有自己的语义，不能随便混用。
2. 据研究，在网络环境好的情况下，发一次包的时间和发两次包的时间差别基本可以无视。而在网络环境差的情况下，两次包的TCP在验证数据包完整性上，有非常大的优点。
3. 并不是所有浏览器都会在POST中发送两次包，Firefox就只发送一次。

### Https
HTTPS（全称：Hypertext Transfer Protocol over Secure Socket Layer），是以安全为目标的HTTP通道，简单讲是HTTP的安全版。即HTTP下加入SSL层，HTTPS的安全基础是SSL，因此加密的详细内容请看SSL。

不使用SSL/TLS的HTTP通信，就是不加密的通信。所有信息明文传播，带来了三大风险。
>（1） 窃听风险（eavesdropping）：第三方可以获知通信内容。
（2） 篡改风险（tampering）：第三方可以修改通信内容。
（3） 冒充风险（pretending）：第三方可以冒充他人身份参与通信。

SSL/TLS协议是为了解决这三大风险而设计的，希望达到：
>（1） 所有信息都是加密传播，第三方无法窃听。
（2） 具有校验机制，一旦被篡改，通信双方会立刻发现。
（3） 配备身份证书，防止身份被冒充。

![image_1bdbnhqg6p5rtpr1q6f13nnmrk2a.png-337.3kB][5]




**参考文章**
[1、HTTP 协议入门](http://www.ruanyifeng.com/blog/2016/08/http.html)
[2、HTTP 教程](http://www.runoob.com/http/http-intro.html)
[3、看完还不懂HTTPS我直播吃翔](http://www.shellsec.com/news/38129.html)


  [1]: http://static.zybuluo.com/a8167270/or284kgq1fb4jnl0r5nc05be/image_1bdb8ch37lma16jm7ng5qludb9.png
  [2]: http://static.zybuluo.com/a8167270/81kxa9epi097u1lmf1i392d9/image_1bdb8fnhh17e61k2cgvm1qrg1rf7m.png
  [3]: http://static.zybuluo.com/a8167270/vjdci2jg0wv18fqq0g67kvcv/image_1bdbfq85p1hlknb01gfb1olnlv313.png
  [4]: http://static.zybuluo.com/a8167270/r69ad6bar07rb5gkbrf13m75/image_1bdbg60ceq2slf71po2416m781g.png
  [5]: http://static.zybuluo.com/a8167270/iy10phozcvv9r36zjk9pvyne/image_1bdbnhqg6p5rtpr1q6f13nnmrk2a.png