---
layout: post
title: 彻底搞懂HTTP和HTTPS协议
subtitle: 探讨http和https的发展史
date: 2019-01-02
author: Li Yucang
catalog: true
tags:
    - web安全
    - http
    - https
---

# 彻底搞懂HTTP和HTTPS协议

## HTTP和HTTPS发展历史

http协议(超文本传输协议)，是一个基于请求与响应，无状态的，应用层的协议，常基于TCP/IP协议传输数据，互联网上应用最为广泛的一种网络协议,所有的WWW文件都必须遵守这个标准。设计HTTP的初衷是为了提供一种发布和接收HTML页面的方法。

设计HTTP最初的目的是为了将超文本标记语言(HTML)文档从Web服务器传送到客户端的浏览器。也是说对于前端来说，我们所写的HTML页面将要放在我们的 web 服务器上，用户端通过浏览器访问url地址来获取网页的显示内容；

随着web2.0的到来，我们的网站从单纯的html网页模式向内容更丰富、更加注重交互并以用户为中心的应用创新，出现了如blog，sns等产品，同时我们的 HTML 页面有了 CSS，Javascript，随着Ajax的出现和大量使用，更是提升了我们对于网站交互的体验，以上这些其实都是基于 HTTP 协议的；

同样到了移动互联网时代，我们页面可以跑在手机端浏览器里面，但是和 PC 相比，手机端的网络情况更加复杂，这使得我们开始了不得不对 HTTP 进行深入理解并不断优化过程中。

http协议发展历史：

版本|产生时间|内容|发展现状
-|-|-|-
HTTP/0.9|1991年|不涉及数据包传输，规定客户端和服务器之间通信格式，只能GET请求|没有作为正式的标准
HTTP/1.0|1996年|传输内容格式不限制，增加PUT、PATCH、HEAD、 OPTIONS、DELETE命令|正式作为标准
HTTP/1.1|1997年|持久连接(长连接)、节约带宽、HOST域、管道机制、分块传输编码|2015年前使用最广泛
HTTP/2|2015年|多路复用、服务器推送、头信息压缩、二进制协议等|逐渐覆盖市场

### HTTP 0.9

HTTP 是基于 TCP/IP 协议的应用层协议，最早版本是1991年发布的0.9版。该版本极其简单：

1. 只接受 GET 一种请求方法，且不支持请求头。
2. 协议规定，服务器只能回应HTML格式的字符串，不能回应别的格式。
3. 由于该版本不支持 POST 方法，所以客户端无法向服务器传递太多信息。

````
客户端请求格式
GET /index.html

服务器响应格式
<html>
  <body>Hello World</body>
</html>
````

### HTTP1.0

HTTP1.0最早在网页中使用是在1996年，那个时候只是使用一些较为简单的网页上和网络请求上。这是第一个在通讯中指定版本号的HTTP 协议版本，至今仍被广泛采用，特别是在代理服务器中。

1. 首先，任何格式的内容都可以发送。这使得互联网不仅可以传输文字，还能传输图像、视频、二进制文件。这为互联网的大发展奠定了基础。

2. 其次，除了GET命令，还引入了POST命令和HEAD命令，丰富了浏览器与服务器的互动手段。

3. 再次，HTTP请求和回应的格式也变了。除了数据部分，每次通信都必须包括头信息（HTTP header），用来描述一些元数据。

4. 其他的新增功能还包括状态码（status code）、多字符集支持、多部分发送（multi-part type）、权限（authorization）、缓存（cache）、内容编码（content encoding）等。

````
客户端请求格式
GET /index.html HTTP/1.0
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_5)
Accept: */*

服务器响应格式
HTTP/1.0 200 OK 
Content-Type: text/plain
Content-Length: 137582
Expires: Thu, 05 Dec 1997 16:00:00 GMT
Last-Modified: Wed, 5 August 1996 15:55:28 GMT
Server: Apache 0.84

<html>
  <body>Hello World</body>
</html>
````

回应的格式是"头信息 + 一个空行（\r\n） + 数据"。其中，第一行是"协议版本 + 状态码（status code） + 状态描述"。

#### Content-Type 字段

关于字符的编码，1.0版规定，头信息必须是 ASCII 码，后面的数据可以是任何格式。因此，服务器回应的时候，必须告诉客户端，数据是什么格式，这就是Content-Type字段的作用。

下面是一些常见的Content-Type字段的值：

````
text/plain
text/html
text/css
image/jpeg
image/png
image/svg+xml
audio/mp4
video/mp4
application/javascript
application/pdf
application/zip
application/atom+xml
````

这些数据类型总称为MIME type，每个值包括一级类型和二级类型，之间用斜杠分隔。

除了预定义的类型，厂商也可以自定义类型。

`application/vnd.debian.binary-package`

上面的类型表明，发送的是Debian系统的二进制数据包。

MIME type还可以在尾部使用分号，添加参数。

`Content-Type: text/html; charset=utf-8`

上面的类型表明，发送的是网页，而且编码是UTF-8。

客户端请求的时候，可以使用Accept字段声明自己可以接受哪些数据格式。

`Accept: */*`

上面代码中，客户端声明自己可以接受任何格式的数据。

MIME type不仅用在HTTP协议，还可以用在其他地方，比如HTML网页。

````
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />等同于 <meta charset="utf-8" />
````

#### Content-Encoding 字段

由于发送的数据可以是任何格式，因此可以把数据压缩后再发送。Content-Encoding字段说明数据的压缩方法。

````
Content-Encoding: gzip
Content-Encoding: compress
Content-Encoding: deflate
````

客户端在请求时，用Accept-Encoding字段说明自己可以接受哪些压缩方法。

`Accept-Encoding: gzip, deflate`

#### 局限性

HTTP/1.0 版的主要缺点是，每个TCP连接只能发送一个请求。发送数据完毕，连接就关闭，如果还要请求其他资源，就必须再新建一个连接。

TCP连接的新建成本很高，因为需要客户端和服务器三次握手，并且开始时发送速率较慢（slow start）。所以，HTTP 1.0版本的性能比较差。随着网页加载的外部资源越来越多，这个问题就愈发突出了。

为了解决这个问题，有些浏览器在请求时，用了一个非标准的Connection字段。

`Connection: keep-alive`

这个字段要求服务器不要关闭TCP连接，以便其他请求复用。服务器同样回应这个字段。

`Connection: keep-alive`

一个可以复用的TCP连接就建立了，直到客户端或服务器主动关闭连接。但是，这不是标准字段，不同实现的行为可能不一致，因此不是根本的解决办法。

### HTTP1.1

1997年1月，HTTP/1.1 版本发布，只比 1.0 版本晚了半年。它进一步完善了 HTTP 协议，一直用到了20年后的今天，直到现在还是最流行的版本。 持久连接被默认采用，并能很好地配合代理服务器工作。还支持以管道方式同时发送多个请求，以便降低线路负载，提高传输速度。

#### 持久连接

1.1 版的最大变化，就是引入了持久连接（persistent connection），即TCP连接默认不关闭，可以被多个请求复用，不用声明`Connection: keep-alive`

客户端和服务器发现对方一段时间没有活动，就可以主动关闭连接。不过，规范的做法是，客户端在最后一个请求时，发送Connection: close，明确要求服务器关闭TCP连接。目前，对于同一个域名，大多数浏览器允许同时建立6个持久连接。

#### 管道机制

1.1 版还引入了管道机制（pipelining），即在同一个TCP连接里面，客户端可以同时发送多个请求。这样就进一步改进了HTTP协议的效率。

举例来说，客户端需要请求两个资源。以前的做法是，在同一个TCP连接里面，先发送A请求，然后等待服务器做出回应，收到后再发出B请求。管道机制则是允许浏览器同时发出A请求和B请求，但是服务器还是按照顺序，先回应A请求，完成后再回应B请求。

#### Content-Length 字段

一个TCP连接现在可以传送多个回应，势必就要有一种机制，区分数据包是属于哪一个回应的。这就是Content-length字段的作用，用于声明本次回应的数据长度。

`Content-Length: 3495`

上面代码告诉浏览器，本次回应的长度是3495个字节，后面的字节就属于下一个回应了。

在1.0版中，Content-Length字段不是必需的，因为浏览器发现服务器关闭了TCP连接，就表明收到的数据包已经全了。

#### 分块传输编码

使用Content-Length字段的前提条件是，服务器发送回应之前，必须知道回应的数据长度。

对于一些很耗时的动态操作来说，这意味着，服务器要等到所有操作完成，才能发送数据，显然这样的效率不高。更好的处理方法是，产生一块数据，就发送一块，采用"流模式"（stream）取代"缓存模式"（buffer）。
因此，1.1版规定可以不使用Content-Length字段，而使用"分块传输编码"（chunked transfer encoding）。只要请求或回应的头信息有`Transfer-Encoding`字段，就表明回应将由数量未定的数据块组成。

`Transfer-Encoding: chunked`

每个非空的数据块之前，会有一个16进制的数值，表示这个块的长度。最后是一个大小为0的块，就表示本次回应的数据发送完了。下面是一个例子。

````
HTTP/1.1 200 OK
Content-Type: text/plain
Transfer-Encoding: chunked

25
This is the data in the first chunk

1C
and this is the second one

3
con

8
sequence

0
````

#### 缓存处理

在HTTP1.0中主要使用header里的If-Modified-Since,Expires来做为缓存判断的标准，HTTP1.1则引入了更多的缓存控制策略例如Entity tag，If-Unmodified-Since, If-Match, If-None-Match等更多可供选择的缓存头来控制缓存策略。

#### Range 和 Content-Range 字段

HTTP1.1则在请求头引入了Range和 Content-Range 头域，它允许只请求资源的某个部分，即返回码是206（Partial Content），这样就方便了开发者自由的选择以便于充分利用带宽和连接，把一个大的文件切割成小的文件块传输，产生了我们所谓的断点续传功能；

Range用于请求头中，指定第一个字节的位置和最后一个字节的位置，一般格式：

`Range:(unit=first byte pos)-[last byte pos]`

`Content-Range`用于响应头，指定整个实体中的一部分的插入位置，他也指示了整个实体的长度。在服务器向客户返回一个部分响应，它必须描述响应覆盖的范围和整个实体长度。一般格式：

`Content-Range: bytes (unit first byte pos) - [last byte pos]/[entity legth]`

#### 错误通知的管理

在HTTP1.1中新增了24个错误状态响应码，如409（Conflict）表示请求的资源与资源的当前状态发生冲突；410（Gone）表示服务器上的某个资源被永久性的删除。

#### 增加了多种请求方法

1.1版还新增了许多动词方法：PUT、PATCH、HEAD、 OPTIONS、DELETE

方法|说明|支持的http协议版本
-|-|-
GET|获取资源|1.0、1.1
POST|传输实体主体|1.0、1.1
PUT|传输文件|1.0、1.1
HEAD|获取报文首部|1.0、1.1
DElETE|删除文件|1.0、1.1
OPTIONS|询问支持的方法|1.1
TRACE|追踪路径|1.1
CONNECT|要求用隧道协议连接代理|1.1
LINK|建立和资源的连接|1.0
UNLINK|断开连接关系|1.0

#### Host头处理

客户端请求的头信息新增了Host字段，用来指定服务器的域名。

`Host: www.example.com`

在HTTP1.0中认为每台服务器都绑定一个唯一的IP地址，因此，请求消息中的URL并没有传递主机名（hostname）。但随着虚拟主机技术的发展，在一台物理服务器上可以存在多个虚拟主机（Multi-homed Web Servers），并且它们共享一个IP地址。HTTP1.1的请求消息和响应消息都应支持Host头域，且请求消息中如果没有Host头域会报告一个错误（400 Bad Request）。

作者：似水牛年
链接：https://www.jianshu.com/p/1eb384ea0aef
来源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。

#### 局限性

虽然1.1版允许复用TCP连接，但是同一个TCP连接里面，所有的数据通信是按次序进行的。服务器只有处理完一个回应，才会进行下一个回应。要是前面的回应特别慢，后面就会有许多请求排队等着。这称为"队头堵塞"（Head-of-line blocking）。

为了避免这个问题，只有两种方法：一是减少请求数，二是同时多开持久连接。这导致了很多的网页优化技巧，比如合并脚本和样式表、将图片嵌入CSS代码、域名分片（domain sharding）等等。如果HTTP协议设计得更好一些，这些额外的工作是可以避免的。

### SPDY 协议

2009年，谷歌公开了自行研发的 SPDY 协议，主要解决 HTTP/1.1 效率不高的问题。

SPDY（读作“SPeeDY”）是Google开发的基于TCP的应用层协议，用以最小化网络延迟，提升网络速度，优化用户的网络使用体验。SPDY并不是一种用于替代HTTP的协议，而是对HTTP协议的增强。新协议的功能包括数据流的多路复用、请求优先级以及HTTP报头压缩。

互联网工程任务组（IETF）对谷歌提出的SPDY协议进行了标准化，于2015年5推出了类似于SPDY协议的 HTTP 2.0 协议标准（简称HTTP/2）。谷歌因此宣布放弃对SPDY协议的支持，转而支持HTTP/2。此外，著名的开源HTTP服务器软件 Nginx 也于2015年9月移除了对SPDY的支持，转而支持HTTP/2。因此，建议新的网站不要部署SPDY，转为部署HTTP/2。

所以，从历史上来看，SPDY协议只是 HTTP/2 的基础，其主要特性都在 HTTP/2 之中得到继承。鉴于此，我们不必要过多了解这个协议的具体内容了。

### HTTP/2

2015年，HTTP/2 发布。它不叫 HTTP/2.0，是因为标准委员会不打算再发布子版本了，下一个新版本将是 HTTP/3。

#### 二进制协议

HTTP/1.1 版的头信息肯定是文本（ASCII编码），数据体可以是文本，也可以是二进制。HTTP/2 则是一个彻底的二进制协议，头信息和数据体都是二进制，并且统称为"帧"（frame）：头信息帧和数据帧。

二进制协议的一个好处是，可以定义额外的帧。HTTP/2 定义了近十种帧，为将来的高级应用打好了基础。如果使用文本实现这种功能，解析数据将会变得非常麻烦，二进制解析则方便得多。

既然又要保证HTTP的各种动词，方法，首部都不受影响，那就需要在应用层(HTTP2.0)和传输层(TCP or UDP)之间增加一个二进制分帧层。

![](/img/localBlog/11224747-a446336fd842f0d8.jpg)

在二进制分帧层上， HTTP/2 会将所有传输的信息分割为更小的消息和帧,并对它们采用二进制格式的编码 ，其中HTTP1.x的首部信息会被封装到Headers帧，而我们的request body则封装到Data帧里面。

#### 头信息压缩

HTTP 协议不带有状态，每次请求都必须附上所有信息。所以，请求的很多字段都是重复的，比如Cookie和User Agent，一模一样的内容，每次请求都必须附带，这会浪费很多带宽，也影响速度。

HTTP/2 对这一点做了优化，引入了头信息压缩机制（header compression）。一方面，头信息使用gzip或compress压缩后再发送；另一方面，客户端和服务器同时维护一张“首部表”来跟踪和存储之前发送的键-值对，对于相同的数据，不再通过每次请求和响应发送；通信期间几乎不会改变的通用键-值对(User Agent、Accept等等) 只需发送一次。

事实上,如果请求中不包含首部(例如对同一资源的轮询请求),那么 首部开销就是零字节。此时所有首部都自动使用之前请求发送的首部。

如果首部发生变化了，那么只需要发送变化了数据在Headers帧里面，新增或修改的首部帧会被追加到“首部表”。首部表在 HTTP 2.0 的连接存续期内始终存在,由客户端和服务器共同渐进地更新 。

#### 多路复用

上面我们说过了HTTP1.1协议的"队头堵塞"问题，在HTTP1.1的协议中，我们传输的request和response都是基本于文本的，这样就会引发一个问题：所有的数据必须按顺序传输，比如需要传输：hello world，只能从h到d一个一个的传输，不能并行传输，因为接收端并不知道这些字符的顺序，所以并行传输在HTTP1.1是不能实现的。

