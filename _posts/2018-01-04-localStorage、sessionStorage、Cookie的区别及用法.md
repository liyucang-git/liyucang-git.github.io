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

# localStorage、sessionStorage、Cookie 的区别及用法

## h5 webstorage

HTML5 提供了在客户端存储数据的新方法 webstorage，包括 localStorage 和 sessionStorage。

### localStorage

localStorage 生命周期是永久，这意味着除非用户显示在浏览器提供的 UI 上清除 localStorage 信息，否则这些信息将永远存在。存放数据大小为一般为 5MB，而且它仅在客户端(即浏览器)中保存，不参与和服务器的通信。

### sessionStorage

sessionStorage 仅在当前会话下有效，关闭页面或浏览器后被清除。存放数据大小为一般为 5MB，而且它仅在客户端(即浏览器)中保存，不参与和服务器的通信。源生接口可以接受，亦可再次封装来对 Object 和 Array 有更好的支持。

localStorage 和 sessionStorage 使用时使用相同的 API:

```
// sessionStorage相应替换即可

localStorage.setItem("key"，"value");//以“key”为名称存储一个值“value”

localStorage.getItem("key");//获取名称为“key”的值

localStorage.removeItem("key");//删除名称为“key”的信息。

localStorage.clear();​//清空localStorage中所有信息

localStorage也支持通过.或者[]来修改储存对象:

localStorage.lastname="LiYucang";
```

## Cookie

生命期为只在设置的 cookie 过期时间之前一直有效，即使窗口或浏览器关闭。 存放数据大小为 4K 左右 。与服务器端通信:每次都会携带在 HTTP 头中，如果使用 cookie 保存过多数据会带来性能问题。但 Cookie 需要程序员自己封装，源生的 Cookie 接口不友好。

```
document.cookie
"UM_distinctid=1614224fe3a42a-0b44590d13c232-5a133411-144000-1614224fe3b7c1; CNZZDATA1261400616=1397455982-1517230919-null%7C1517230919"
```

我们在随意一个网页的控制台里敲上`document.cookie`就可以看到如上格式的输出。打开 chrome 浏览器控制台的 Application 项，我们可以看到当前页面的 cookie。

从控制台中我们发现每个 cookie 具有如下项:name、value、domain、path、expires/max-age、size、http、sequre。

其实，在 Javascript 脚本里，一个 cookie 实际就是一个字符串属性。当你读取 cookie 的值时，就得到一个字符串，里面当前 WEB 页使用的所有 cookies 的名称和值。每个 cookie 除了 name 名称和 value 值这两个属性以外，还有一些其他属性。这些属性主要有:expires 过期时间、 path 路径、 domain 域、以及 secure 安全。

- expires/max-age – 过期时间。指定 cookie 的生命期，如果想让 cookie 的存在期限超过当前浏览器会话时间，就必须使用这个属性。当过了到期日期时，浏览器就可以删除 cookie 文件，没有任何影响。

  - Expires 在 HTTP/1.0 中已经定义，Cache-Control:max-age 在 HTTP/1.1 中才有定义，为了向下兼容，仅使用 max-age 不够；
  - Expires 指定一个绝对的过期时间(GMT 格式)，这么做会导致至少 2 个问题:
    1. 客户端和服务器时间不同步导致 Expires 的配置出现问题
    2. 很容易在配置后忘记具体的过期时间，导致过期来临出现浪涌现象；
  - max-age 指定的是从 cookie 生成后的存活时间，这个时间是个相对值(比如:3600s)，相对的是在 http 请求中，cookie 第一次生成时，服务器记录的 Request_time(请求时间)。

- path – 路径。指定与 cookie 关联的 WEB 页。值可以是一个目录，或者是一个路径。如果http://www.zdnet.com/devhead /index.html 建立了一个 cookie，那么在http://www.zdnet.com/devhead/目录里的所有页面，以及该目录下面任何子目录里的页面都可以 访问这个 cookie。这就是说，在http://www.zdnet.com/devhead/stories/articles 里的任何页面都可以访问http://www.zdnet.com/devhead/index.html建立的cookie。但是，如果http: //www.zdnet.com/zdnn/ 需要访问http://www.zdnet.com/devhead/index.html设置的cookes，该怎么办？这时，我们要把cookies 的 path 属性设置成“/”。在指定路径的时候，凡是来自同一服务器，URL 里有相同路径的所有 WEB 页面都可以共享 cookies。现在看另一个例子: 如果想让 http://www.zdnet.com/devhead/filters/ 和http://www.zdnet.com/devhead/stories/共享cookies，就要把path设成“/devhead”。

- Domain – 域。指定关联的 WEB 服务器或域。一个页面可以为本域和任何父域设置 cookie，只要是父域不是公共后缀(public suffix)即可。值是域名，比如 zdnet.com。这是对 path 路径属性的一个延伸。如果我们想让 catalog.mycompany.com 能够访问 shoppingcart.mycompany.com 设置的 cookies，该怎么办? 我们可以把 domain 属性设置成“mycompany.com”，并把 path 属性设置成“/”。不能把 cookies 域属性设置成与设置它的服务器的 所在域不同的值。

- Secure – 安全。指定 cookie 的值通过网络如何在用户和 WEB 服务器之间传递。这个属性的值或者是“secure”，或者为空。缺省情况下，该属性为空，也就是使用不安全的 HTTP 连接传递数据。如果一个 cookie 标记为 secure，那么，它与 WEB 服务器之间就通过 HTTPS 或者其它安全协议传递数据。不过，设置了 secure 属性不代表其他人不能看到你机器本 地保存的 cookie。换句话说，把 cookie 设置为 secure，只保证 cookie 与 WEB 服务器之间的数据传输过程加密，而保存在本地的 cookie 文件并不加密。如果想让本地 cookie 也加密，得自己加密数据。

我们可以将设置和获取 cookie 的方法进行简单的封装:

```
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
```

## 浏览器的同源策略

这里给大家简单介绍一下同源策略的概念。首先，什么页面具有相同源呢:

如果两个页面的协议，端口(如果有指定)和域名都相同，则两个页面具有相同的源。

我们也可以用 js 将页面的源进行更改:

页面可能会因某些限制而改变他的源。脚本可以将 document.domain 的值设置为其当前域或其当前域的超级域。如果将其设置为其当前域的超级域，则较短的域将用于后续源检查。假设 http://store.company.com/dir/other.html 文档中的一个脚本执行以下语句:

`document.domain = "company.com";`

这条语句执行之后，页面将会成功地通过对 http://company.com/dir/page.html 的同源检测(假设http://company.com/dir/page.html 将其 document.domain 设置为“company.com”，以表明它希望允许这样做 - 更多有关信息，请参阅 document.domain )。然而，company.com 不能设置 document.domain 为 othercompany.com，因为它不是 company.com 的超级域。

浏览器单独保存端口号。任何的赋值操作，包括 document.domain = document.domain 都会导致端口号被重写为 null 。因此 company.com:8080 不能仅通过设置 document.domain = "company.com" 来与 company.com 通信。必须在他们双方中都进行赋值，以确保端口号都为 null 。

注意:使用 document.domain 来允许子域安全访问其父域时，您需要在父域和子域中设置 document.domain 为相同的值。这是必要的，即使这样做只是将父域设置回其原始值。不这样做可能会导致权限错误。

这里简单介绍一下，后续会对 cors 做一个具体、全面的解析。

## 几种缓存的比较

首先是共同点:**都是保存在浏览器端，且同源的。**

差异:

1. 携带
   - cookie 数据始终在同源的 http 请求中携带(即使不需要)，即 cookie 在浏览器和服务器间来回传递。
   - sessionStorage 和 localStorage 不会自动把数据发给服务器，仅在本地保存。
2. cookie 数据还有路径(path)的概念，可以限制 cookie 只属于某个路径下。
3. 存储大小限制
   - cookie 数据不能超过 4k，同时因为每次 http 请求都会携带 cookie，所以 cookie 只适合保存很小的数据，如会话标识。
   - sessionStorage 和 localStorage 虽然也有存储大小的限制，但比 cookie 大得多，可以达到 5M 或更大。
4. 数据有效期
   - sessionStorage:仅在当前浏览器窗口关闭前有效，自然也就不可能持久保持；
   - localStorage:始终有效，窗口或浏览器关闭也一直保存，因此用作持久数据；
   - cookie 只在设置的 cookie 过期时间之前一直有效，即使窗口或浏览器关闭。
5. 作用域
   - sessionStorage 不在不同的浏览器窗口中共享，即使是同一个页面(同一个浏览器开两个相同页面，sessionStorage 不共享)；
   - localStorage 在所有同源窗口中都是共享的；
   - cookie 也是在所有同源窗口中都是共享的。
6. Web Storage 支持事件通知机制，可以将数据更新的通知发送给监听者。Web Storage 的 api 接口使用更方便。
