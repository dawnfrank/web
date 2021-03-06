#HTTP
>超文本传输协议（HyperText Transfer Protocol）是一种用于分布式、协作式和超媒体信息系统的应用层协议。HTTP是万维网的数据通信的基础。设计HTTP最初的目的是为了提供一种发布和接收HTML页面的方法。

<div>
![](osi七层模型.gif)
</div>

##HTTP请求方法
`HTTP/1.1`协议中共定义了8种HTTP请求方法，HTTP请求方法也被叫做“请求动作”，不同的方法规定了不同的操作指定的资源方式。服务端也会根据不同的请求方法做不同的响应。

方法|说明
-|-
GET|请求指定的页面信息，并返回实体主体。
HEAD|类似于get请求，只不过返回的响应中没有具体的内容，用于获取报头
POST|向指定资源提交数据进行处理请求（例如提交表单或者上传文件）。数据被包含在请求体中。POST请求可能会导致新的资源的建立和/或已有资源的修改。
PUT|从客户端向服务器传送的数据取代指定的文档的内容。
DELETE|请求服务器删除指定的页面。
CONNECT|HTTP/1.1协议中预留给能够将连接改为管道方式的代理服务器。
OPTIONS|允许客户端查看服务器的性能。
TRACE|回显服务器收到的请求，主要用于测试或诊断。

##HTTP请求流程
在Python中的处理http请求的在`http`模块，`http`模块通过区分服务器和`HttpRequestHandler`而分离了**接受请求**和**处理请求**两个功能。
  * 1.服务器接收来自客户端的请求
  * 2.服务器把请求报文转发给`HttpRequestHandler`来处理
  * 3.`HttpRequestHandler`处理请求报文并发出响应数据
  * 4.服务器关闭整个请求

<div>
![](http.png)
</div>

  * `do_SPAM()`就是根据`HTTP`的请求方法不同而对应的调用`do_GET()`、`do_POST()`等等。

####HTTPServer
启动服务器之后，HTTPServer的主要处理请求在函数`_handle_request_noblock()`中
```
    def _handle_request_noblock(self):
        """Handle one request, without blocking.

        I assume that selector.select() has returned that the socket is
        readable before this function was called, so there should be no risk of
        blocking in get_request().
        """
        try:
            request, client_address = self.get_request()
        except OSError:
            return
        if self.verify_request(request, client_address):
            try:
                self.process_request(request, client_address)
            except Exception:
                self.handle_error(request, client_address)
                self.shutdown_request(request)
            except:
                self.shutdown_request(request)
                raise
        else:
            self.shutdown_request(request)
```


  * `get_request`：获取客户端发过来的数据和客户端地址信息。
  * `verify_request`：检验是否需要处理此请求，默认返回True
  * `process_request`：调用`finish_request`和`shutdown_request`
  * `finish_request`：实例化请求处理类（`RequestHandler`）
  * `shutdown_request`：停止发送请求
  * `close_request`：关闭请求

####HTTPRequestHandler

<div align=center>
![](requirehandler类结构.png)
</div>


HTTP请求处理的基类,该协议识别请求的三个部分：
  * 请求行：`<command> <path> <version>`
    * `command`：请求方法，一般为`GET`或`POST`
    * `path`：请求的URL，URL编码规范 (使用 %xx 来解释 ASCII 字符，其中xx为字符16进制).
    * `version`：HTTP版本，`HTTP/1.0`或者`HTTP/1.1`
  * RFC-822形式的请求头：每行通过CRLF分开
  * 消息主体（数据）
  * 请求头和消息中间由空行隔开


<div align=center>
![](http请求报文.png)
</div>

主要的方法有：
  * `handle()`：这个方法是请求处理类真正处理请求具体工作的方法，例如解析到来的请求，处理数据，并发回响应等。在BaseHTTPRequestHandler中它是一个入口文件，将调用其他的方法完成请求处理。
  * `handle_one_request()`：用于处理请求。
  * `parse_request()`：解析请求
  * `send_error()`：发回错误响应
  * `send_response()`：创建响应首行和响应首部等内容。
  * `send_header()`：设置响应首部内容。
  * `end_headers()`：调用此方法可以在首部后增加一个空行，表示首部内容结束
  * `do_SPAM()`：这个方法中的`SPAM`指代GET、POST、HEAD等请求方法，需要在请求处理类中构建具体的请求处理方法，例如`do_GET`处理GET请求，`do_POST`处理POST请求。`do_SPAM()` 方法可以调用`send_response()`、`send_header()`、`end_headers()`等方法创建响应首行和响应首部等内容。


##附录1：HTTP响应头

<div align=center>
![](http响应报文.png)
</div>


##附录2： HTTP版本历史
在HTTP的发展过程中，出现了很多HTTP版本，其中的大部分协议都是向下兼容的。在进行HTTP请求时，客户端在请求时会告诉服务器它采用的协议版本号，而服务器则会在使用相同或者更早的协议版本进行响应。

####HTTP/0.9
这是HTTP最早大规模使用的版，现已过时。在这个版本中 只有GET一种请求方法，在HTTP通讯也没有指定版本号，也不支持请求头信息。该版本不支持POST等方法，因此客户端向服务器传递信息的能力非常有限。HTTP/0.9的请求只有如下一行：

`GET www.itbilu.com`

####HTTP/1.0
这个版本是第一个在HTTP通讯中指定版本号的协议版本，`HTTP/1.0`至今仍被广泛采用，特别是在代理服务器中。

`HTTP/1.0`支持：`GET`、`POST`、`HEAD`三种HTTP请求方法。

####HTTP/1.1
HTTP/1.1是当前正在使用的版本。该版本默认采用持久连接，并能很好地配合代理服务器工作。还支持以管道方式同时发送多个请求，以便降低线路负载，提高传输速度。

`HTTP/1.1`新增了：`OPTIONS`、`PUT`、`DELETE`、`TRACE`、`CONNECT`五种HTTP请求方法。



##附录3：HTTP请求字段

协议头|说明|示例
-|-
Accept|	可接受的响应内容类型（Content-Types）| 	Accept: text/plain
Accept-Charset| 可接受的字符集 | Accept-Charset: utf-8
Accept-Encoding|接收的编码方法，通常指是否支持压缩，支持什么压缩方法（gzip，deflate）|Accept-Encoding: zh-CN,zh;q=0.8
Accept-Language|可接受的响应内容语言列表。|	Accept-Language: en-US
Accept-Datetime|可接受的按照时间来表示的响应内容版本|Accept-Datetime: Sat, 26 Dec 2015 17:30:00 GMT
Authorization|表示HTTP协议中需要认证资源的认证信息|	Authorization: Basic OSdjJGRpbjpvcGVuIANlc2SdDE==
Cache-Control|用来指定当前的请求/回复中的，是否使用缓存机制。|Cache-Control: no-cache
Connection|	客户端（浏览器）想要优先使用的连接类型|Connection: keep-alive<br>Connection: Upgrade
Cookie|由之前服务器通过Set-Cookie（见下文）设置的一个HTTP协议Cookie|Cookie: $Version=1; Skin=new;
Content-Length|以8进制表示的请求体的长度|Content-Length: 348
Content-MD5|请求体的内容的二进制 MD5 散列值（数字签名），以 Base64 编码的结果|Content-MD5: oD8dH2sgSW50ZWdyaIEd9D==
Content-Type|	请求体的MIME类型 （用于POST和PUT请求中）|Content-Type: application/x-www-form-urlencoded
Date|发送该消息的日期和时间|Date: Dec, 26 Dec 2015 17:30:00 GMT
Expect|表示客户端要求服务器做出特定的行为|Expect: 100-continue
From|发起此请求的用户的邮件地址|From: user@itbilu.com
Host|表示服务器的域名以及服务器所监听的端口号。如果所请求的端口是对应的服务的标准端口（80），则端口号可以省略。|Host: www.itbilu.com:80<br>Host: www.itbilu.com
If-Match|仅当客户端提供的实体与服务器上对应的实体相匹配时，才进行对应的操作。主要用于像 PUT 这样的方法中，仅当从用户上次更新某个资源后，该资源未被修改的情况下，才更新该资源。|If-Match: "9jd00cdj34pss9ejqiw39d82f20d0ikd"
If-Modified-Since|允许在对应的资源未被修改的情况下返回304未修改|	If-Modified-Since: Dec, 26 Dec 2015 17:30:00 GMT
If-None-Match|允许在对应的内容未被修改的情况下返回304未修改（ 304 Not Modified ），参考 超文本传输协议 的实体标记|If-None-Match: "9jd00cdj34pss9ejqiw39d82f20d0ikd"
If-Range|如果该实体未被修改过，则向返回所缺少的那一个或多个部分。否则，返回整个新的实体|	If-Range: "9jd00cdj34pss9ejqiw39d82f20d0ikd"
If-Unmodified-Since|仅当该实体自某个特定时间以来未被修改的情况下，才发送回应。|If-Unmodified-Since: Dec, 26 Dec 2015 17:30:00 GMT
Max-Forwards|限制该消息可被代理及网关转发的次数。|Max-Forwards: 10
Origin|发起一个针对跨域资源共享的请求|	Origin: http://www.itbilu.com
Pragma|与具体的实现相关，这些字段可能在请求/回应链中的任何时候产生。|Pragma: no-cache
Proxy-Authorization|用于向代理进行认证的认证信息。|	Proxy-Authorization: Basic IOoDZRgDOi0vcGVuIHNlNidJi2==
Range|表示请求某个实体的一部分，字节偏移以0开始。|	Range: bytes=500-999
Referer|表示浏览器所访问的前一个页面，可以认为是之前访问页面的链接将浏览器带到了当前页面。Referer其实是Referrer这个单词，但RFC制作标准时给拼错了，后来也就将错就错使用Referer了。|Referer: http://itbilu.com/nodejs
TE|	浏览器预期接受的传输时的编码方式：可使用回应协议头Transfer-Encoding中的值（还可以使用"trailers"表示数据传输时的分块方式）用来表示浏览器希望在最后一个大小为0的块之后还接收到一些额外的字段。|	TE: trailers,deflate
User-Agent|	浏览器的身份标识字符串|User-Agent: Mozilla/……
Upgrade|要求服务器升级到一个高版本协议。|	Upgrade: HTTP/2.0, SHTTP/1.3, IRC/6.9, RTA/x11
Via|	告诉服务器，这个请求是由哪些代理发出的。|	Via: 1.0 fred, 1.1 itbilu.com.com (Apache/1.1)
Warning|一个一般性的警告，表示在实体内容体中可能存在错误。|	Warning: 199 Miscellaneous warning

##附录4：HTTP响应字段

响应头|说明|示例
-|-
Access-Control-Allow-Origin|指定哪些网站可以跨域源资源共享|Access-Control-Allow-Origin: *
Accept-Patch|指定服务器所支持的文档补丁格式|	Accept-Patch: text/example;charset=utf-8
Accept-Ranges|	服务器所支持的内容范围|	Accept-Ranges: bytes
Age|	响应对象在代理缓存中存在的时间，以秒为单位|	Age: 12
Allow|对于特定资源的有效动作|Allow: GET, HEAD
Cache-Control|通知从服务器到客户端内的所有缓存机制，表示它们是否可以缓存这个对象及缓存有效时间。其单位为秒|Cache-Control: max-age=3600
Connection|针对该连接所预期的选项|	Connection: close
Content-Disposition|对已知MIME类型资源的描述，浏览器可以根据这个响应头决定是对返回资源的动作，如：将其下载或是打开。|Content-Disposition: attachment; filename="fname.ext"
Content-Encoding|响应资源所使用的编码类型。|Content-Encoding: gzip
Content-Language|响就内容所使用的语言|Content-Language: zh-cn
Content-Length|响应消息体的长度，用8进制字节表示|Content-Length: 348
Content-Location|所返回的数据的一个候选位置|Content-Location: /index.htm
Content-Range|如果是响应部分消息，表示属于完整消息的哪个部分|	Content-Range: bytes 21010-47021/47022
Content-Type|当前内容的MIME类型|Content-Type: text/html; charset=utf-8
Date|	此条消息被发送时的日期和时间|Date: Tue, 15 Nov 1994 08:12:31 GMT
ETag|对于某个资源的某个特定版本的一个标识符，通常是一个 消息散列|	ETag: "737060cd8c284d8af7ad3082f209582d"
Expires|指定一个日期/时间，超过该时间则认为此回应已经过期|	Expires: Thu, 01 Dec 1994 16:00:00 GMT
Last-Modified|所请求的对象的最后修改日期(按照 RFC 7231 中定义的“超文本传输协议日期”格式来表示)|Last-Modified: Dec, 26 Dec 2015 17:30:00 GMT
Link|用来表示与另一个资源之间的类型关系，此类型关系是在RFC 5988中定义|Link: ; rel="alternate"
Location|用于在进行重定向，或在创建了某个新资源时使用。|Location: http://www.itbilu.com/nodejs
P3P|	P3P策略相关设置|P3P: CP="This is not a P3P policy!
Pragma|	与具体的实现相关，这些响应头可能在请求/回应链中的不同时候产生不同的效果|	Pragma: no-cache
Proxy-Authenticate|	要求在访问代理时提供身份认证信息。|Proxy-Authenticate: Basic
Public-Key-Pins|	用于防止中间攻击，声明网站认证中传输层安全协议的证书散列值|Public-Key-Pins: max-age=2592000; pin-sha256="……";
Refresh|用于重定向，或者当一个新的资源被创建时。默认会在5秒后刷新重定向。	|	Refresh: 5; url=http://itbilu.com
Retry-After|	如果某个实体临时不可用，那么此协议头用于告知客户端稍后重试。其值可以是一个特定的时间段(以秒为单位)或一个超文本传输协议日期。|示例1:Retry-After: 120<br>示例2: Retry-After: Dec, 26 Dec 2015 17:30:00 GMT
Server|服务器的名称|	Server: nginx/1.6.3
Set-Cookie|设置HTTP cookie|Set-Cookie: UserID=itbilu; Max-Age=3600; Version=1
Status|通用网关接口的响应头字段，用来说明当前HTTP连接的响应状态。|	Status: 200 OK
Trailer|Trailer用户说明传输中分块编码的编码信息|Trailer: Max-Forwards
Transfer-Encoding|用表示实体传输给用户的编码形式。包括：chunked、compress、 deflate、gzip、identity。|Transfer-Encoding: chunked
Upgrade|	要求客户端升级到另一个高版本协议。|Upgrade: HTTP/2.0, SHTTP/1.3, IRC/6.9, RTA/x11
Vary|	告知下游的代理服务器，应当如何对以后的请求协议头进行匹配，以决定是否可使用已缓存的响应内容而不是重新从原服务器请求新的内容。|	Vary: *
Via|告知代理服务器的客户端，当前响应是通过什么途径发送的。|	Via: 1.0 fred, 1.1 itbilu.com (nginx/1.6.3)
Warning|	一般性警告，告知在实体内容体中可能存在错误。|	Warning: 199 Miscellaneous warning
WWW-Authenticate|表示在请求获取这个实体时应当使用的认证模式。|	WWW-Authenticate: Basic