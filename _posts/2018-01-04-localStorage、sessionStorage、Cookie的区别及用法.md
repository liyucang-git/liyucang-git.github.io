---
layout: post
title: localStorage、sessionStorage、Cookie的区别及用法
subtitle: 比较几种浏览器缓存的异同
date: 2018-01-04
author: Li Yucang
catalog: true
tags:
    - 缓存
    - 浏览器
---

# localStorage、sessionStorage、Cookie的区别及用法

## h5 webstorage

HTML5 提供了在客户端存储数据的新方法webstorage，包括localStorage和sessionStorage。

### localStorage

localStorage生命周期是永久，这意味着除非用户显示在浏览器提供的UI上清除localStorage信息，否则这些信息将永远存在。存放数据大小为一般为5MB，而且它仅在客户端(即浏览器)中保存，不参与和服务器的通信。

### sessionStorage

sessionStorage仅在当前会话下有效，关闭页面或浏览器后被清除。存放数据大小为一般为5MB，而且它仅在客户端(即浏览器)中保存，不参与和服务器的通信。源生接口可以接受，亦可再次封装来对Object和Array有更好的支持。

localStorage和sessionStorage使用时使用相同的API:

````
// sessionStorage相应替换即可

localStorage.setItem("key"，"value");//以“key”为名称存储一个值“value”

localStorage.getItem("key");//获取名称为“key”的值

localStorage.removeItem("key");//删除名称为“key”的信息。

localStorage.clear();​//清空localStorage中所有信息

localStorage也支持通过.或者[]来修改储存对象:

localStorage.lastname="LiYucang";
````

## Cookie

生命期为只在设置的cookie过期时间之前一直有效，即使窗口或浏览器关闭。 存放数据大小为4K左右 。与服务器端通信:每次都会携带在HTTP头中，如果使用cookie保存过多数据会带来性能问题。但Cookie需要程序员自己封装，源生的Cookie接口不友好。

````
document.cookie
"UM_distinctid=1614224fe3a42a-0b44590d13c232-5a133411-144000-1614224fe3b7c1; CNZZDATA1261400616=1397455982-1517230919-null%7C1517230919"
````

我们在随意一个网页的控制台里敲上`document.cookie`就可以看到如上格式的输出。打开chrome浏览器控制台的Application项，我们可以看到当前页面的cookie。

从控制台中我们发现每个cookie具有如下项:name、value、domain、path、expires/max-age、size、http、sequre。

其实，在Javascript脚本里，一个cookie 实际就是一个字符串属性。当你读取cookie的值时，就得到一个字符串，里面当前WEB页使用的所有cookies的名称和值。每个cookie除了name名称和value值这两个属性以外，还有一些其他属性。这些属性主要有:expires过期时间、 path路径、 domain域、以及 secure安全。 

* expires/max-age – 过期时间。指定cookie的生命期，如果想让cookie的存在期限超过当前浏览器会话时间，就必须使用这个属性。当过了到期日期时，浏览器就可以删除cookie文件，没有任何影响。 
  * Expires在HTTP/1.0中已经定义，Cache-Control:max-age在HTTP/1.1中才有定义，为了向下兼容，仅使用max-age不够；
  *  Expires指定一个绝对的过期时间(GMT格式)，这么做会导致至少2个问题:
      1. 客户端和服务器时间不同步导致Expires的配置出现问题 
      2. 很容易在配置后忘记具体的过期时间，导致过期来临出现浪涌现象；
  * max-age 指定的是从cookie生成后的存活时间，这个时间是个相对值(比如:3600s)，相对的是在http请求中，cookie第一次生成时，服务器记录的Request_time(请求时间)。

* path – 路径。指定与cookie关联的WEB页。值可以是一个目录，或者是一个路径。如果http://www.zdnet.com/devhead /index.html 建立了一个cookie，那么在http://www.zdnet.com/devhead/目录里的所有页面，以及该目录下面任何子目录里的页面都可以 访问这个cookie。这就是说，在http://www.zdnet.com/devhead/stories/articles 里的任何页面都可以访问http://www.zdnet.com/devhead/index.html建立的cookie。但是，如果http: //www.zdnet.com/zdnn/ 需要访问http://www.zdnet.com/devhead/index.html设置的cookes，该怎么办？这时，我们要把cookies 的path属性设置成“/”。在指定路径的时候，凡是来自同一服务器，URL里有相同路径的所有WEB页面都可以共享cookies。现在看另一个例子: 如果想让 http://www.zdnet.com/devhead/filters/ 和http://www.zdnet.com/devhead/stories/共享cookies，就要把path设成“/devhead”。 

* Domain – 域。指定关联的WEB服务器或域。一个页面可以为本域和任何父域设置cookie，只要是父域不是公共后缀(public suffix)即可。值是域名，比如zdnet.com。这是对path路径属性的一个延伸。如果我们想让 catalog.mycompany.com 能够访问shoppingcart.mycompany.com设置的cookies，该怎么办? 我们可以把domain属性设置成“mycompany.com”，并把path属性设置成“/”。不能把cookies域属性设置成与设置它的服务器的 所在域不同的值。 

* Secure – 安全。指定cookie的值通过网络如何在用户和WEB服务器之间传递。这个属性的值或者是“secure”，或者为空。缺省情况下，该属性为空，也就是使用不安全的HTTP连接传递数据。如果一个 cookie 标记为secure，那么，它与WEB服务器之间就通过HTTPS或者其它安全协议传递数据。不过，设置了secure属性不代表其他人不能看到你机器本 地保存的cookie。换句话说，把cookie设置为secure，只保证cookie与WEB服务器之间的数据传输过程加密，而保存在本地的 cookie文件并不加密。如果想让本地cookie也加密，得自己加密数据。 

我们可以将设置和获取cookie的方法进行简单的封装:

````
function setCookie(name，value) {  
    var days = 30;  
    var expires = new Date();  
    expires.setTime(expires.getTime() + days*24*60*60*1000);  
    document.cookie = name + "="+ escape(value) + ";expires=" + expires.toGMTString();  
}
function getCookie(name) {  
    var start = document.cookie.indexOf(name + "=");   
    var len = start + name.length + 1;   
    if (start == -1)   
        return null;   
    var end = document.cookie.indexOf(';'， len);   
    if (end == -1)   
        end = document.cookie.length;   
    return unescape(document.cookie.substring(len， end));   
}  
````

## 浏览器的同源策略

这里给大家简单介绍一下同源策略的概念。首先，什么页面具有相同源呢:

如果两个页面的协议，端口(如果有指定)和域名都相同，则两个页面具有相同的源。

我们也可以用js将页面的源进行更改:

页面可能会因某些限制而改变他的源。脚本可以将 document.domain 的值设置为其当前域或其当前域的超级域。如果将其设置为其当前域的超级域，则较短的域将用于后续源检查。假设 http://store.company.com/dir/other.html 文档中的一个脚本执行以下语句:

  `document.domain = "company.com";`


这条语句执行之后，页面将会成功地通过对 http://company.com/dir/page.html 的同源检测(假设http://company.com/dir/page.html 将其 document.domain 设置为“company.com”，以表明它希望允许这样做 - 更多有关信息，请参阅 document.domain )。然而，company.com 不能设置 document.domain 为 othercompany.com，因为它不是 company.com 的超级域。

浏览器单独保存端口号。任何的赋值操作，包括 document.domain = document.domain 都会导致端口号被重写为 null 。因此 company.com:8080 不能仅通过设置 document.domain = "company.com" 来与company.com 通信。必须在他们双方中都进行赋值，以确保端口号都为 null 。

注意:使用 document.domain 来允许子域安全访问其父域时，您需要在父域和子域中设置 document.domain 为相同的值。这是必要的，即使这样做只是将父域设置回其原始值。不这样做可能会导致权限错误。

这里简单介绍一下，后续会对cors做一个具体、全面的解析。

## 几种缓存的比较

首先是共同点:**都是保存在浏览器端，且同源的。**

差异:

1. 携带
    * cookie数据始终在同源的http请求中携带(即使不需要)，即cookie在浏览器和服务器间来回传递。
    * sessionStorage和localStorage不会自动把数据发给服务器，仅在本地保存。
2. cookie数据还有路径(path)的概念，可以限制cookie只属于某个路径下。
3. 存储大小限制
    * cookie数据不能超过4k，同时因为每次http请求都会携带cookie，所以cookie只适合保存很小的数据，如会话标识。
    * sessionStorage和localStorage 虽然也有存储大小的限制，但比cookie大得多，可以达到5M或更大。
4. 数据有效期
    * sessionStorage:仅在当前浏览器窗口关闭前有效，自然也就不可能持久保持；
    * localStorage:始终有效，窗口或浏览器关闭也一直保存，因此用作持久数据；
    * cookie只在设置的cookie过期时间之前一直有效，即使窗口或浏览器关闭。
5. 作用域
    * sessionStorage不在不同的浏览器窗口中共享，即使是同一个页面(同一个浏览器开两个相同页面，sessionStorage不共享)；
    * localStorage 在所有同源窗口中都是共享的；
    * cookie也是在所有同源窗口中都是共享的。
6. Web Storage 支持事件通知机制，可以将数据更新的通知发送给监听者。Web Storage 的 api 接口使用更方便。

