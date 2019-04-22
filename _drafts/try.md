# try

三次握手和四次挥手详细介绍
TCP有哪些手段保证可靠交付
URL从输入到页面渲染全流程
如何预防中间人攻击
DNS解析会出错吗，为什么
ES6的Set内部实现
如何应对流量劫持
算法：top-K问题，分成top-1,top-2,top-K三小问
跨域
webpack的plugins和loaders的实现原理
vue和react谈谈区别和选型考虑
webpack如何优化编译速度
事件循环机制，node和浏览器的事件循环机制区别
单元测试编写有哪些原则
一个大型项目如何分配前端开发的工作
typescript有什么好处
JWT优缺点
选择器优先级
基本数据类型
nginx负载均衡配置
前端性能优化手段
301 302 307 308 401 403
vue的nextTick实现原理以及应用场景
vue组件间通信
谈谈XSS防御，以及Content-Security-Policy细节
场景题：一个气球从右上角移动到中间，然后抖动，如何实现
场景题：一个关于外边距合并的高度计算
mobx-react如何驱动react组件重渲染
forceUpdate经历了哪些生命周期，子组件呢?
React key场景题：列表使用index做key，删除其中一个后，如何表现？
算法：实现setter(obj, 'a.b.c' ,val)
手写冒泡排序
JWT细节，适用场景
方案题：不同前端技术栈的项目，如何实现一套通用组件方案？
ES6特性
闭包和this一起谈谈
postcss配置
Promise内部实现原理
vuex, mobx, redux各自的特点和区别
react生命周期
各方面谈谈性能优化
serviceworker如何保证离线缓存资源更新
virtual dom有哪些好处
Vue3 proxy解决了哪些问题？
Vue响应式原理
发布订阅模式和观察者模式的异同
图片懒加载实现
css垂直居中
CI/CD流程
谈谈性能优化
react生命周期
key的作用
hooks
vue和react区别，选型考虑
canvas优化绘制性能
webpack性能优化手段
事件循环
如何解决同步调用代码耗时太高的问题
手写Promise实现
vue组件间通信
性能优化
vuex数据流动过程
谈谈css预处理器机制
算法：Promise串行
CI/CD整体流程
SSR对性能优化的提升在哪里
vue组件间通信
react和vue更新机制的区别
Vue3 proxy的优劣
symbol应用
深拷贝
dns解析流程
怼项目
跨域
性能优化
ssr优缺点
贝塞尔曲线
怼项目
Vue3 proxy优缺点
ES6特性
Vue组件间通信
性能优化
ssr性能优化，node中间层细节处理
如何编写loaders和plugins
性能优化
webpack 热更新原理
vue和react组件通信
谈谈eleme框架源码
谈谈项目
个人兴趣爱好


css

1. 三栏布局

float
bfc (set middle area overflow is hidden)
position: absolute
双飞翼布局
flex
table
2. 垂直居中

个人感觉这个问很多，我一般就是答以下几种：

line-height: height 有被问到该值是不是等于高度设置的值，这个没有答好，回来测试发现是跟盒模型相关的，需要是 computed height
absolute + transform 居中为什么要使用 transform（为什么不使用marginLeft / Top），这是一道重绘重排的问题。
flex + align-items: center 我会对 flex 容器以及 flex 项目的每个 css 属性都了解一遍，并且写了一些小 demo。
3. 盒模型

4. BFC

概念
如何触发
怎么应用
5. CSS 预处理器

一般回答 变量 / 嵌套 / 自动前缀 / 条件语句 / 循环语句

我是一直很欣赏张鑫旭大神对 CSS 研究到非常透彻的境界，但是总结下来，对 CSS 一般不会考得很深，业界对 CSS 的专注度其实不够，包括我个人也是没投入很多时间精力在 CSS 上，如果有更深入的理解当然是更好的。

JS

1. 原型

其实之前刚毕业的时候就在啃高级程序设计，三年后还是在考这个点。

闭包 / 作用域 / this 指向
实现 继承
es5 实现 class
es5 实现 new
2. var let const

let/const 也存在变量声明提升，只是没有初始化分配内存。 一个变量有三个操作，声明(提到作用域顶部)，初始化(赋默认值)，赋值(继续赋值)。

var 是一开始变量声明提升，然后初始化成 undefined，代码执行到那行的时候赋值。
let 是一开始变量声明提升，然后没有初始化分配内存，代码执行到那行初始化，之后对变量继续操作是赋值。因为没有初始化分配内存，所以会报错，这是暂时性死区。
const 是只有声明和初始化，没有赋值操作，所以不可变。
const 只是保证了指向的内存地址不变，而不是内部数据结构不变。确保不会被其他类型的值所替代。

3. 实现 promise

面 tx 的时候有让我手写实现 promise，因为有看过一点 bluebird 的源码，所以对我来说还好。写了一半就让我停下来，我一般会边写边讲解，一上来先把框架搭好，例如

class Promise {
    constructor(executor) {
        // 设置属性 status value resolveCbs rejectCbs
    }
    then(onResolved, onRejected) {}
    catch (cb) {
        return this.then(null, cb)
    }
}
然后再慢慢填充，这种做法会一上来让人感觉你是有思路的，而且了解 promise 的语法。

4. promise 链式

例如 promises = []，实现必须上一个异步完成后再去跑下一个任务。我是写出两种方案：

// 1.
const template = Promise.resolve();
promises.forEach((p) => {
    template = template.then(p)
})
// 2. 使用 await 
个人感觉对 promise、async/await 都问比较多，包括比较火的问打印顺序那种题还有捕获异常的问题我也遇到过，只要对语法非常熟悉加上稍微了解实现细节都是没问题。

5. 实现 bind

6. 实现事件系统 eventEmitter

跟 promise class 一样，先搭建一个框架：

class EventEmitter {
    constructor() {
        this.events = {}
    }
    emit (eventName, args) {}
    on (eventName, callback) {}
    off (eventName, callback?) {}
}
之前有写过事件系统，但是没有考虑 once 之类的 method，也是面试官说了需求再一点点补充的。

7. 手写 Proxy / Object.defineProperty

8. 事件委托

9. Event Loop

JS 执行是单线程的，它是基于事件循环的。事件循环大致分为以下几个步骤：

（1）所有同步任务都在主线程上执行，形成一个执行栈。

（2）主线程之外，还存在一个"任务队列"。只要异步任务有了运行结果，就在"任务队列"之中放置一个事件。

（3）一旦"执行栈"中的所有同步任务执行完毕，系统就会读取"任务队列"，看看里面有哪些事件。那些对应的异步任务，于是结束等待状态，进入执行栈，开始执行。

（4）主线程不断重复上面的第三步。

event loop 基本每次都会被问到，一般就是说微任务、宏任务，怎么样的运行过程，以上是比较书面一点的回答，我自己也没记得。

10. Webpack

loader plugin 的区别，一开始被问到还有点惊讶，不同作用的功能被问到一起。
tree-shaking 的工作原理
code splitting用的是什么插件
如何提高 webpack 构件速度的
利用 DllPlugin 预编译资源模块
使用 Happypack 加速代码构建
我写过很多小项目，所以配过很多次 webpack，从 V2 到 V4，不过 webpack 一般没有问很多。

11. 前端性能提升

一般我会分为以下几个方面来回答，一般会引申到网络、缓存方面的问题：

server：

使用 cdn
减少不必要的数据返回
使用 gzip
缓存 （etag / expires ...）
content：

减少 http 请求 (css sprites / inline image)
不同资源放在不同域下 (http1.1)
延迟加载 / 延迟执行(立即下载，延迟执行[before DOMContentLoaded]defer) / 预加载(preload)
async，该布尔属性指示浏览器是否在允许的情况下异步执行该脚本。该属性对于内联脚本无作用 (即没有 src 属性的脚本）。
defer，这个布尔属性被设定用来通知浏览器该脚本将在文档完成解析后，触发DOMContentLoaded事件前执行。
精简 HTML 结构
压缩资源
css:

in head
较少的层级（之前被问到过是否有统计过层级多与少对性能的实质影响，实际上我是没有做过此类研究，所以知道结论而不懂过程还是欠缺的）
js:

before
减少 dom 访问（在 body 内放置的 JS 代码是否可以访问到 body 标签）
webpack:

tree shaking 去除没有使用的代码
提取公共包，有被问到
拆分模块，按需加载
优化图片，使用 base64 代替小图
file name with hash (etag)
12. Vue

因为我是写 Vue 居多，所以简历上只写了 Vue 而没有 react 等其他框架，一般都是被问到 Vue。

父子组件通信
生命周期
数据响应(数据劫持) 数据响应的实现由两部分构成: 观察者( watcher ) 和 依赖收集器( Dep )，其核心是 defineProperty这个方法，它重写属性的 get 与 set 方法，从而完成监听数据的改变。一般会要求从解析到收集依赖到通知一套工作原理比较熟悉。我是大概理解，跟着读了一些源码，但是还是没有讲很清楚，遗憾没有表现好。
虚拟 dom、dom diff
nextTick
JS 肯定是重点考察的部分，对象、继承方面只要读透高程再写几个 demo 掌握细节，es6 我是一直翻阮一峰写的 ECMAScript 6 入门，Vue 的话首先把官网的内容掌握了，然后再去读一些博客或者直接上源码也是可以的。

HTTP

1. 跨域

基本都被问同源策略以及引申到跨域来，一般我会说 CORS 以及 jsonp，CORS 会从简单请求跟非简单请求区分开，再讲 options 请求的意义。

2. HTTP 报文

请求行 + 头部信息 + 空白行 + body 有被问到说空白行的意义，我一直以为就是纯粹来标识 headers 的结束，但是面试官说不止这个功能，我后面看了HTTP 权威指南 也没有找到，Stack Overflow 也没找到。。。希望有人知道可以跟我说一下。不过有可能是我听错了题目，毕竟是电话面试。

3. cookie session

一般会问两者的差别，以及引申到 sessionStorage, localStorage, cookie 区别

4. 从输入 URL 到页面加载全过程

一般我会答链接的大部分步骤，按照理解来，这里面我被问到的点有：

缓存，分为强缓存、协议缓存，一般会问到 304 的表现，以及再引申到 301 302 的区别，我会再说 307 的区别。
三次握手
HTTPS 的工作原理
CDN 的工作原理，以及刷新缓存的原理。
浏览器渲染的步骤
重绘重排的概念，以及最佳实践。一直都知道应该用 transform 代替 margin，但是一被问原理，就不太清楚，查了资料是对 translate3d 的元素进行 GPU 加速。
会因为 JS 是单线程而问到阻塞的问题，引申到 async defer 等属性。
status code 有哪些，我们是严格按照 restful 的规范来设计接口，所以这个问题我一直觉得很简单，但是被问到不少次。我记得趣头条的笔试就有，我会把用过的按照 2xx(200, 201, 204, 206) 3xx(301, 302, 304, 307) 4xx(400, 401, 403, 405) 5xx(500, 502, 504) 来分类，我偶尔写写 rails，所以对对应的名词都比较熟悉 贴一篇 list of rails status codes
DNS 解析过程
5. xss，csrf

xss 注入攻击
转义
csrf (cross site request forgery)
Get 请求无副作用
cookie httponly
cors （origin not *）
我是通过看 这篇文章 对安全有更多了解的，推荐一下。

6. sse( server sent event)

因为写过一个 sse 相关的插件，所以被问到过，是如何使用以及 EventSource 的 API。

感觉前端对网络、安全方面要求不是很高，没被问过 HTTP2 或者长连接更多内容，之前看脉脉上一个后端被问从浏览器里访问一个地址，从网络的 tcp/ip 协议、聊到操作系统 io、内存管理、进程管理和文件管理，再聊到负载均衡、限流算法、分布式事务，相比之下前端真的简单很多，不过知识储备多肯定是有用的。

算法


层次遍历一棵二叉树 (这是唯一一道剑指 offer 上的题目了，最简单的 😂 )
字符串中找出最长最多重复的子串





https://m.imooc.com/article/274197?mc_marking=e006a03025dbb036dda59ada344c96c2&mc_channel=weibo
https://m.imooc.com/article/80250?mc_marking=a0c56328db5ab64ff63fa7875e8953c7&mc_channel=weibo
https://segmentfault.com/a/1190000018155877?utm_medium=hao.caibaojian.com&utm_source=hao.caibaojian.com&share_user=1030000000178452#articleHeader72


https://juejin.im/post/5c62ea95e51d457ffe60c084