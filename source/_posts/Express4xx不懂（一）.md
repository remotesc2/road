---
title: Express4xx不懂（一）
date: 2016-06-19 10:48:07
tags: 
- node.js
- express4
categories: 
- 学习
- back-end
- node.js
- Express4
---

从0开始用Express
===

盲点
---

### 到底http中的post请求分为几种？  

[HTTP/1.1协议](http://www.ietf.org/rfc/rfc2616.txt)（明显我就从来没看过）规定HTTP请求有 OPTIONS、GET、HEAD、POST、PUT、DELETE、TRACE、CONNECT 这几种，但是平常说实话也就用到 **get** 和 **post**。  

**get** 还好理解，无非是参数跟在url后头传递，通过 `?` 来分割 `url` 和 `params`，不过这里会不会也有[坑](http://www.baidu.com/s?wd=get请求方式)。可是 **post** 就没那么简单，今天在用Express来获取 **post** 参数时，才发现 **post** 提交数据是有几种方式的，记录下（全部来源网络东瞅西瞄）。
   
HTTP 协议是以 ASCII 码传输，建立在 TCP/IP 协议之上的应用层规范。规范把 HTTP 请求分为三个部分：状态行、请求头、消息主体。  
请求格式如下：  

``` xml
<request-line>
<headers>
<blank line>
[<request-body>]

<!--
说明:
第一行必须是一个请求行(request-line),用来说明请求类型,要访问的资源以及所使用的HTTP版本
紧接着是一个首部(header)小节,用来说明服务器要使用的附加信息
之后是一个空行
再后面可以添加任意的其他数据[称之为主体(body)]
-->
```

<!--more-->

举例说明：  

+ get请求  

``` xml
GET / HTTP/1.1
Host: www.google.cn
Accept: */*
Accept-Language: zh-cn
Accept-Encoding: gzip, deflate
User-Agent: Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; .NET CLR 2.0.50727; .NET CLR 3.0.04506.648; .NET CLR 3.5.21022)Connection: Keep-Alive

<!--
说明:
第1行：分为三部分，以空格为分隔符。第一部分说明了该请求是一个GET请求；第二部分是一个斜杠(/)，说明请求的相对路径， Host 部分第2行提供。第三部分说明使用的是HTTP1.1版本(另一个可选荐是1.0).
第2行：是请求的第一个首部， Host 将指出请求的目的地。User-Agent,服务器端和客户端脚本都能访问它，它是浏览器类型检测逻辑的重要基础。该信息由你的浏览器来定义,并且在每个请求中自动发送.Connection,通常将浏览器操作设置为Keep-Alive
第3行：空行，即使不存在请求主体，这个空行也是必需的。
-->
```

+ post请求

``` xml
POST / HTTP1.1
Host:www.google.cn
User-Agent:Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; .NET CLR 2.0.50727; .NET CLR 3.0.04506.648; .NET CLR 3.5.21022)
Content-Type:application/x-www-form-urlencoded
Content-Length:40
Connection: Keep-Alive

name=Professional%20Ajax&publisher=Wiley

<!--
说明：
第1行：请求行开始处的GET改为POST,以表示不同的请求类型。
第2行：Content-Type说明了请求主体的内容是如何编码的.浏览器始终以application/x-www-form-urlencoded的格式编码来传送数据，这是针对简单URL编码的MIME类型。
Content-Length说明了请求主体的字节数（即“name=Professional%20Ajax&publisher=Wiley”的长度）
第3行：空白行
第4行：最后请求主体.名称-值对的形式。
-->
```

+ 再多记录一个，HTTP响应格式

``` xml
<!--格式-->
<status-line>
<headers>
<blank line>
[<response-body>]

<!--例子-->
HTTP/1.1 200 OK
Date: Fri, 22 May 2009 06:07:21 GMT
Content-Type: text/html; charset=UTF-8

<html>
	<head></head>
	<body>
		<!--body goes here-->
	</body>
</html>

<!-—
说明：
HTTP状态码200,找到资源,并且一切正常。
Date：生成响应的日期和时间。
Content-Type:指定了MIME类型的HTML(text/html),编码类型是UTF-8
HTML源文体。
-->
```

我这里的疑惑就是在于 **Content-Type** 的差别是的什么?   

**Content-Type** 有四类：  

1. www-form-urlencoded，常见的form提交  
2. form-data，文件提交  
3. application/json，提交json格式的数据  
4. text/xml，提交xml格式的数据  

> jquery默认的 content-Type配置的是 application/x-www-form-urlencoded

Express 依靠 `bodyParser` 来解析请求包，默认支持的格式有：application/json，application/x-www-form-urlencoded，multipart/form-data，对 xml 没有支持。  

#### www-form-urlencoded

http 默认的 post 请求是这种方式，例如你写一个 `<form>....<input type="submite" /></form>` form表单，里面的submite按钮默认就是这种 `www-form-urlencoded` 方式提交的。提交的数据按照 key1=val1&key2=val2 的方式进行编码，key 和 val 都进行了 URL 转码  

对应 Express 里的接收 `req.body`

``` javascript
router.post('/add', function(req, res, next) {
    console.log(req.body);
    res.send({success: true});
});
```

#### form-data

和 **www-form-urlencoded** 不同之处在于 form 表单里需要有 enctype 标识。  
比如上传文件，必须在form标签里做这样的标识 `enctype="multipart/form-data"`

请求格式如下：

``` xml
POST http://www.example.com HTTP/1.1
Content-Type:multipart/form-data; boundary=----WebKitFormBoundaryrGKCBY7qhFd3TrwA

------WebKitFormBoundaryrGKCBY7qhFd3TrwA
Content-Disposition: form-data; name="text"

title
------WebKitFormBoundaryrGKCBY7qhFd3TrwA
Content-Disposition: form-data; name="file"; filename="chrome.png"
Content-Type: image/png

PNG ... content of chrome.png ...
------WebKitFormBoundaryrGKCBY7qhFd3TrwA--
```

首先，生成一个 boundary 用于分割不同的字段，为了避免与正文内容重复，boundary 很长很复杂。然后 Content-Type 里指明了数据是以 multipart/form-data 来编码，本次请求的 boundary 是什么内容。消息主体里按照字段个数又分为多个结构类似的部分，每部分都是以 --boundary 开始，紧接着是内容描述信息，然后是回车，最后是字段具体内容（文本或二进制）。如果传输的是文件，还要包含文件名和文件类  

nodejs在处理这类表单提交时，需要另外一个中间件 **[connect-multiparty](https://github.com/andrewrk/connect-multiparty)**，他是专门处理此类 post 数据相关的依赖包。   

Express 接收方式如下：

``` javascript
var multipart = require('connect-multiparty');
var multipartMiddleware = multipart();

app.post('/uploadFile', multipartMiddleware, function (req, res) {
	console.log(req.body);
	res.send(req.body);
});
```

> 现阶段标准中原生 \<form\> 表单也只支持这两种方式（通过 \<form\> 元素的 `enctype` 属性指定，默认为 `application/x-www-form-urlencoded`  

#### application/json

bodyParser 支持此类参数解析。  
**注意**：在提交之前需要指定 http 请求头为 `content-type=application/json`。  
请求结构如下 ： 

``` xml
POST /conferenceServer/add? HTTP/1.1
Host: localhost:3000
Content-Type: application/json
Cache-Control: no-cache
Postman-Token: baf71016-20e4-7e65-f672-33c146bb9497

{
    "aaa": 1111,
    "bbb": 222
}
```

Express 接收方式如下：

``` javascript
app.post('/json', function (req, res) {
	console.log(req.body);
	res.send(req.body);
});
```

#### text/xml
不常见，Express 暂时只能自己用代码处理，把请求体参数按照字符串读取出来，然后使用 xml2json 包把字符串解析成json对象。  

**实现思路：**
  
1. 使用 body-parse 得到字符串。然后再转化  
2. xml格式请求需要指定 http 请求头 content-type=text/xml  
3. 利用req上定义的事件 data 来获取http请求流, end 事件结束请求流的处理.  
4. 利用 xml2json 把上面得到的请求参数流(我们直接转化为字符串)转化为 json 对象.

请求格式如下：

``` xml
POST /conferenceServer/add? HTTP/1.1
Host: localhost:3000
Content-Type: text/xml
Cache-Control: no-cache
Postman-Token: 9a31bd8f-b7dd-a31c-f09d-7e55c8b8b1d1

<aaa>
    <b>111</b>
</aaa>
```

Express 接收方式如下：

``` javascript
var express = require('express');
var bodyParser = require('body-parser');
var xml2json = require('xml2json');

var app = express();
var server = require('http').createServer(app);

app.use(bodyParser.urlencoded({
    extended: true
}));

app.post('/xml', function(req, res) {
    req.rawBody = '';
    var json = {};
    req.setEncoding('utf8');

    req.on('data', function(chunk) {
        req.rawBody += chunk;
    });
    req.on('end', function() {
        json = xml2json.toJson(req.rawBody);
        res.send(JSON.stringify(json));
    });

});
```

**注意：**

1. 这里使用了 `bodyParser.urlencoded({ extended: true });`，而这个已经不被推荐



参考：  
---

1. [http://www.cnblogs.com/shaoge/archive/2009/08/14/1546019.html](http://www.cnblogs.com/shaoge/archive/2009/08/14/1546019.html)  
2. [http://yijiebuyi.com/blog/90c1381bfe0efb94cf9df932147552be.html](http://yijiebuyi.com/blog/90c1381bfe0efb94cf9df932147552be.html)  
3. [http://www.tuicool.com/articles/beEJ32a](http://www.tuicool.com/articles/beEJ32a)  
4. [https://imququ.com/post/four-ways-to-post-data-in-http.html#comments](https://imququ.com/post/four-ways-to-post-data-in-http.html#comments)