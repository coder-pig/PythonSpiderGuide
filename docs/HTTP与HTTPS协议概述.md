# HTTP与HTTPS协议概述

&nbsp;

## 简述一次网络请求

一个习以为常的操作：打开浏览器 → 输入网址 → 回车 → 等待网页加载结束 → 浏览网页<br>
问题：整个过程的背后都发生了什么？ 笔者以访问百度为例，输入域名回车后：

### 1、通过URL查找服务器IP

> 将域名解析为对应服务器的IP地址，如www.baidu.com，浏览器并不认识这个域名，拆解下：<br>
> www是`服务器名`，baidu是`公司名或私人名`，com是 `域名根服务器`<br>
> 浏览器先访问 `本地hosts` 文件，检查文件中是否有与域名匹配的ip地址，有直接访问IP对应服务器；<br>
> 没有则向 `上层DNS服务器询问`，如果没有，继续向上层服务器访问，直到DNS根服务器。

可以在cmd终端中使用 `tracert` 命令查看数据包发送到目标服务器经过的路由结点及延迟，示例如下：

![](../media/pics/tracert_jump.png)

Tips：可输入`tracert -? ` 查看此命令的详细用法信息~

### 2、三次握手建立TCP连接

获取服务器IP后，就是跟服务器建立连接连接了，如图：

![](../media/pics/三次握手.png)

> - 客户端发送一个带 SYN 标志的数据包给服务器；
> - 服务器接收后，回传一个带 SYN/ACK 标志的数据包表示信息确认；
> - 客户端回传一个带 ACK 标志的数据包，表示握手结束，连接建立成功；

### 3、发送HTTP请求

客户端与服务器建立连接后就可以发送 HTTP 请求了，浏览器除了会发送<br>
`请求行` 与 `请求头信息` 外，还会发送 `一个空行` 代表请求头消息发送结束；<br>
如果是 POST 提交，还会提交 `请求体`.

### 4、服务器响应请求

Web服务器解析用户请求，进行处理，尔后把处理结果组装成响应报文，返回给客户端。

### 5、浏览器解析HTML

浏览器解析服务器返回的HTML代码，并请求其中用到的css/js/图片等资源。

### 6、页面渲染后呈现给用户

渲染的顺序是从上往下的，下载与渲染同时进行，最后加载完成显示到浏览器上。

## URI、URL与URN


**URI**（Uniform Resource Identifier）**统一资源标志符**，用于标记一个网络资源，URL和URN的父类。<br>
**URL**（Uniform Resource Locator）**统一资源定位符**，用地址标记一个网络资源，强调的是给资源定位。 <br>
**URN**（Uniform Resource Name）**统一资源命名**，用特定命名空间的名字标识资源，强调的是给资源命名。<br>

举个简单例子以示区分：

> 你在某个技术沙龙上认识了一位大牛，他自称Java架构师，加了他还有，怎么备注呢？<br>
> 如果是 **URN → Java/架构师/X某**，只关心给他命名，而不指定如何定位； <br>
> 如果是 **URL → X厂://Java/架构师/X某**，除了命名，还指定了定位具体到哪家公司；<br>

### URI的组成

访问资源的命名机制 + 存放资源的主机名 + 资源本身的名称（路径标识，还是着重于强调资源）

### URL的组成

协议（服务方式）+ 存有该资源的主机IP（有时还包含端口号）+ 主机资源的具体地址（如目录，文件名等）

**Tips**：网络上的URN用得不多，几乎所有的URI都是URL，所以一般网络连接常称为URL。

## HTTP请求报文

**HTTP**，Hyper Text Transfer Protocol（超文本传输协议），用于万维网服务器传输<br>
超文本到本地浏览器的传送协议，基于TCP/IP通信协议来传递数据。<br>

**HTTP** 是无状态的，因此限制每次连接只处理一个请求，服务器在处理完客户端请求，<br>
并接收到客户端的应答后，即断开连接，这种方式的好处就是节省传输时间。

当然，如果想保持连接的话，可以在请求首部字段中添加请求头Connection: keep-alive，<br>
表明使用持久连接，或者通过Cookies这类方式间接的保存用户之前的HTTP通信状态。

客户端与服务器完成三次握手建立TCP连接后，开始发送HTTP请求，请求由下述四个部分组成：

### 1、请求行

由 `请求方法`、`URL` 和 `HTTP协议版本` 三个字段组成，使用空格进行分隔，访问百度请求行内容如下：

```
GET /index.php HTTP/1.1
```

依次讲解下三个字段，先是 HTTP协议版本：

> HTTP/1.1 → HTTP/主版本号.此版本号 → 常用的有 HTTP/1.0 与 HTTP/1.1

然后是URL构成示例：

<img src="../media/pics/URL构成示例.jpg" width="300px">

最后是请求方法，HTTP 1.1中定义了八种请求方法，具体作用如下：

|请求方法|作用|
|:-:|:-|
|GET|请求指定页面，并返回页面内容|
|POST|一般用于提交表单或上传数据，数据被包含在请求体中|
|PUT|客户端向服务器发送数据取代指定文档内容|
|DELETE|请求服务器删除指定页面|
|HEAD|与GET类似，只是返回的响应无具体内容，一般用于获取报头|
|OPTIONS|允许客户端查看服务器的性能|
|TRACE|回显服务器收到的请求，一般用于测试或诊断|
|CONNECT|HTTP/1.1协议中预留，可将服务器作为跳板，代替客户端访问其他网页|

GET和POST方式用得较多，简单说下区别：

> GET：请求参数都包含在URL中，数据可在URL中看到，无请求体，故只能携带少量数据(最多1024字节);<br>
> POST：数据通过表单形式传输，包含在请求体中，故可携带大量信息。

### 2、请求头

用于说明服务器要使用的附加信息，常用的头信息如下：

|Header|解释|示例|
|:-|:-|:-|
|User-Agent|简称UA，供服务器识别客户使用的操作系统及版本、浏览器及版本等信息，写爬虫时常加上此信息以伪装成浏览器。|User-Agent: Mozilla/5.0 (Linux; X11)|
|Referer|标识请求是从哪个页面过来的，即来路，服务器常用此信息来做来源统计和防盗链处理|Referer:http://blog.csdn.net/coder_pig|
|Cookie/Cookies|网站为辨别用户仅限会话跟踪而存储在用户本地的数据，主要功能是维持当前访问会话。|Cookie: $Version=1; Skin=new;|
|Host|	指定请求的服务器的域名和端口号|	Host: www.zcmhi.com|
|Content-Type|	请求的与实体对应的MIME信息|	Content-Type:application/x-www-form-urlencoded|
|Accept|请求报头域，指定客户端可接受的内容类型|Accept: text/plain, text/html|
|Accept-Charset	|指定客户端可接受的字符编码集|	Accept-Charset: iso-8859-5|
|Accept-Encoding	|指定客户端可支持的web服务器返回内容压缩编码类型	|Accept-Encoding: compress, gzip|
|Accept-Language|	指定客户端可接受的语言|	Accept-Language: en,zh|
|Accept-Ranges|	可以请求网页实体的一个或者多个子范围字段|	Accept-Ranges: bytes|
|Authorization|	HTTP授权的授权证书|	Authorization:BasicQWx2FtZQ==|
|Cache-Control|	指定请求和响应遵循的缓存机制	|Cache-Control: no-cache|
|Connection|	表示是否需要持久连接，HTTP 1.1默认进行持久连接|	Connection: close|
|Content-Length|	请求的内容长度|	Content-Length: 348|
|Date|	请求发送的日期和时间|	Date: Tue, 15 Nov 2010 08:12:31 GMT|
|Expect|	请求的特定的服务器行为|	Expect: 100-continue|
|From|	发出请求的用户的Email	|From: user@email.com|
|If-Match	|只有请求内容与实体相匹配才有效|	If-Match: "737060cd8c284d8af7ad3082f209582d"|
|If-Modified-Since	|如果请求的部分在指定时间之后被修改则请求成功，未被修改则返回304代码|	If-Modified-Since: Sat, 29 Oct 2010 19:43:31 GMT|
|If-None-Match	|如果内容未改变返回304代码，参数为服务器先前发送的Etag，与服务器回应的Etag比较判断是否改变|	If-None-Match: "737060cd8c284d8af7ad3082f209582d"|
|If-Range	|如果实体未改变，服务器发送客户端丢失的部分，否则发送整个实体。参数也为Etag	|If-Range: "737060cd8c284d8af7ad3082f209582d"|
|If-Unmodified-Since	|只在实体在指定时间之后未被修改才请求成功|	If-Unmodified-Since: Sat, 29 Oct 2010 19:43:31 GMT|
|Max-Forwards	|限制信息通过代理和网关传送的时间	|Max-Forwards: 10|
|Pragma	|用来包含实现特定的指令|	Pragma: no-cache|
|Proxy-Authorization	|连接到代理的授权证书|	Proxy-Authorization: Basic QWxhc2FtZQ==|
|Range|	只请求实体的一部分，指定范围|	Range: bytes=500-999|
|TE|	客户端愿意接受的传输编码，并通知服务器接受接受尾加头信息	|TE: trailers,deflate;q=0.5|
|Upgrade	|向服务器指定某种传输协议以便服务器进行转换（如果支持）	|Upgrade:HTTP/2.0,SHTTP/1.3,IRC/6.9, RTA/x11|
|Via	|通知中间网关或代理服务器地址，通信协议	|Via: 1.0 fred, 1.1 nowhere.com (Apache/1.1)|
|Warning|	关于消息实体的警告信息	|Warn: 199 Miscellaneous warning|

### 3、空行

必不可少！请起头的最后会有一个空行，标识请求头头部结束，接下来为请求数据。

### 4、请求正文

一般承载POST请求中的表单数据，请求头 `Content-Type` 需使用正确类型才能正常提交，对应关系如下表：

|Content-Type|提交数据类型|
|:-|:-|
|application/x-www-form-urlencoded|表单数据|
|multipart/form-data|文件|
|application/json|JSON|
|text/xml|XML|

## HTTP响应报文

## HTTPS
