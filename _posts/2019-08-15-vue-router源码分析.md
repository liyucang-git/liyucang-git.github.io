---
layout: post
title: vue-router 源码分析
subtitle: 源码探索之旅
date: 2019-08-15
author: Li Yucang
catalog: true
tags:
  - vue-router
  - 源码解析
---

# vue-router 源码分析

## 路由知识

### 路由的概念

路由这个概念最开始是在后端出现的，以前使用模板引擎开发页面的时候经常会看到这样的路径：

```
http://hometown.xxx.edu.cn/bbs/forum.php
```

有时还会有带.asp 或.html 的路径，这就是所谓的 SSR(Server Side Render)，通过服务端渲染，直接返回页面。

其响应过程是这样的

1. 浏览器发出请求

2. 服务器监听到 80 端口（或 443）有请求过来，并解析 url 路径

3. 根据服务器的路由配置，返回相应信息（可以是 html 字串，也可以是 json 数据，图片等）

4. 浏览器根据数据包的 Content-Type 来决定如何解析数据

简单来说路由就是用来跟后端服务器进行交互的一种方式，通过不同的路径，来请求不同的资源，请求不同的页面是路由的其中一种功能。

就像路由器在网络层中扮演的角色一样，肩负着将数据包正确导向目的地址的重任，只不过在这里变成了客户端浏览器的指路人，所谓的前端路由，指的是一种能力，即：

> 不依赖于服务器，根据不同的 URL 渲染不同的页面

### 前端路由与后端路由

在 Ajax 还没有诞生的时候，路由的工作是交给后端来完成的，当进行页面切换的时候，浏览器会发送不同的 URL 请求，服务器接收到浏览器的请求时，通过解析不同的 URL 去拼接需要的 Html 或模板，然后将结果返回到浏览器端进行渲染。

服务器端路由同样是有利亦有弊。它的好处是安全性更高，更严格得控制页面的展现。这在某些场景中是很有用的，譬如下单支付流程，每一步只有在上一步成功执行之后才能抵达。这在服务器端可以为每一步流程添加验证机制，只有验证通过才返回正确的页面。那么前端路由不能实现每一步的验证？

自然不是，姑且相信你的代码可以写的很严谨，保证正常情况下流程不会错，但是另一个不得不面对的事实是：前端是毫无安全性可言的。用户可以肆意修改代码来进入不同的流程，你可能会为此添加不少的处理逻辑。相较之下，当然是后端控制页面的进入权限更为安全和简便。

另一方面，后端路由无疑增加了服务器端的负荷，并且需要 reload 页面，用户体验其实不佳。

### 前端路由的出现

在 90s 年代初，大多数的网页都是通过直接返回 HTML 的，用户的每次更新操作都需要重新刷新页面。及其影响交互体验，随着网络的发展，迫切需要一种方案来改善这种情况。

1996，微软首先提出 iframe 标签，iframe 带来了异步加载和请求元素的概念，随后在 1998 年，微软的 Outloook Web App 团队提出 Ajax 的基本概念（XMLHttpRequest 的前身），并在 IE5 通过 ActiveX 来实现了这项技术。在微软实现这个概念后，其他浏览器比如 Mozilia，Safari，Opera 相继以 XMLHttpRequest 来实现 Ajax。（😭 兼容问题从此出现，话说微软命名真喜欢用 X，MFC 源码一大堆。。）不过在 IE7 发布时，微软选择了妥协，兼容了 XMLHttpRequest 的实现。

有了 Ajax 后，用户交互就不用每次都刷新页面，体验带来了极大的提升。

但真正让这项技术发扬光大的，(｡･∀･)ﾉﾞ还是后来的 Google Map，它的出现向人们展现了 Ajax 的真正魅力，释放了众多开发人员的想象力，让其不仅仅局限于简单的数据和页面交互，为后来异步交互体验方式的繁荣发展带来了根基。

而异步交互体验的更高级版本就是我们熟知的 SPA，SPA 不单单在页面交互上做到了不刷新，而且在页面之间跳转也做到了不刷新，为了做到这一点，就促使了前端路由的诞生。

### 前端路由的实现方式

前端路由其实只要解决两个问题：

- 在页面不刷新的前提下实现 url 变化
- 捕捉到 url 的变化，以便执行页面替换逻辑

在 2014 年之前，大家是通过 hash 来实现路由，url hash 就是类似于：

```
http://www.xxx.com/#/login
```

这种 #。后面 hash 值的变化，并不会导致浏览器向服务器发出请求，浏览器不发出请求，也就不会刷新页面。另外每次 hash 值的变化，还会触发 hashchange 这个事件，通过这个事件我们就可以知道 hash 值发生了哪些变化。然后我们便可以监听 hashchange 来实现更新页面部分内容的操作：

```
function matchAndUpdate () {
  // todo 匹配 hash 做 dom 更新操作
}

window.addEventListener('hashchange', matchAndUpdate)
```

后来，因为 HTML5 标准发布。多了两个 API，pushState 和 replaceState，通过这两个 API 可以改变 url 地址且不会发送请求。同时还有 popstate 事件。

通过这些就能用另一种方式来实现前端路由了，但原理都是跟 hash 实现相同的。用了 HTML5 的实现，单页路由的 url 就不会多出一个#，变得更加美观。但因为没有 # 号，所以当用户刷新页面之类的操作时，浏览器还是会给服务器发送请求。为了避免出现这种情况，所以这个实现需要服务器的支持，需要把所有路由都重定向到根页面：

```
function matchAndUpdate () {
   // todo 匹配路径 做 dom 更新操作
}
window.addEventListener('popstate', matchAndUpdate)
```

## Vue-Router 的实现方式

我们在用 Vue 开发过实际项目的时候都会用到 Vue-Router 这个官方插件来帮我们解决路由的问题。Vue-Router 的能力十分强大，它支持 hash、history、abstract 3 种路由方式，提供了 `<router-link>` 和 `<router-view>` 2 种组件，还提供了简单的路由配置和一系列好用的 API。

大部分同学已经掌握了路由的基本使用，但使用的过程中也难免会遇到一些坑，那么这一章我们就来深挖 Vue-Router 的实现细节，一旦我们掌握了它的实现原理，那么就能在开发中对路由的使用更加游刃有余。

### 路由注册

Vue 从它的设计上就是一个渐进式 JavaScript 框架，它本身的核心是解决视图渲染的问题，其它的能力就通过插件的方式来解决。Vue-Router 就是官方维护的路由插件，在介绍它的注册实现之前，我们先来分析一下 Vue 通用的插件注册原理。

**Vue.use**

Vue 提供了 Vue.use 的全局 API 来注册这些插件，所以我们先来分析一下它的实现原理，定义在 vue/src/core/global-api/use.js 中：

```
export function initUse (Vue: GlobalAPI) {
  Vue.use = function (plugin: Function | Object) {
    const installedPlugins = (this._installedPlugins || (this._installedPlugins = []))
    if (installedPlugins.indexOf(plugin) > -1) {
      return this
    }

    const args = toArray(arguments, 1)
    args.unshift(this)
    if (typeof plugin.install === 'function') {
      plugin.install.apply(plugin, args)
    } else if (typeof plugin === 'function') {
      plugin.apply(null, args)
    }
    installedPlugins.push(plugin)
    return this
  }
}
```

Vue.use 接受一个 plugin 参数，并且维护了一个 \_installedPlugins 数组，它存储所有注册过的 plugin；接着又会判断 plugin 有没有定义 install 方法，如果有的话则调用该方法，并且该方法执行的第一个参数是 Vue；最后把 plugin 存储到 installedPlugins 中。

可以看到 Vue 提供的插件注册机制很简单，每个插件都需要实现一个静态的 install 方法，当我们执行 Vue.use 注册插件的时候，就会执行这个 install 方法，并且在这个 install 方法的第一个参数我们可以拿到 Vue 对象，这样的好处就是作为插件的编写方不需要再额外去 import Vue 了。

**路由安装**

Vue-Router 的入口文件是 src/index.js，其中定义了 VueRouter 类，也实现了 install 的静态方法：VueRouter.install = install，它的定义在 src/install.js 中。

```
import View from './components/view'
import Link from './components/link'

export let _Vue

export function install (Vue) {
  if (install.installed && _Vue === Vue) return
  install.installed = true

  _Vue = Vue

  const isDef = v => v !== undefined

  const registerInstance = (vm, callVal) => {
    let i = vm.$options._parentVnode
    if (isDef(i) && isDef(i = i.data) && isDef(i = i.registerRouteInstance)) {
      i(vm, callVal)
    }
  }

  Vue.mixin({
    beforeCreate () {
      if (isDef(this.$options.router)) {
        this._routerRoot = this
        this._router = this.$options.router
        this._router.init(this)
        Vue.util.defineReactive(this, '_route', this._router.history.current)
      } else {
        this._routerRoot = (this.$parent && this.$parent._routerRoot) || this
      }
      registerInstance(this, this)
    },
    destroyed () {
      registerInstance(this)
    }
  })

  Object.defineProperty(Vue.prototype, '$router', {
    get () { return this._routerRoot._router }
  })

  Object.defineProperty(Vue.prototype, '$route', {
    get () { return this._routerRoot._route }
  })

  Vue.component('RouterView', View)
  Vue.component('RouterLink', Link)

  const strats = Vue.config.optionMergeStrategies
  // use the same hook merging strategy for route hooks
  strats.beforeRouteEnter = strats.beforeRouteLeave = strats.beforeRouteUpdate = strats.created
}
```

当用户执行 Vue.use(VueRouter) 的时候，实际上就是在执行 install 函数，为了确保 install 逻辑只执行一次，用了 install.installed 变量做已安装的标志位。另外用一个全局的 `_Vue` 来接收参数 Vue，因为作为 Vue 的插件对 Vue 对象是有依赖的，但又不能去单独去 import Vue，因为那样会增加包体积，所以就通过这种方式拿到 Vue 对象。

Vue-Router 安装最重要的一步就是利用 Vue.mixin 去把 beforeCreate 和 destroyed 钩子函数注入到每一个组件中。Vue.mixin 的定义，在 vue/src/core/global-api/mixin.js 中：

```
export function initMixin (Vue: GlobalAPI) {
  Vue.mixin = function (mixin: Object) {
    this.options = mergeOptions(this.options, mixin)
    return this
  }
}
```

它的实现实际上非常简单，就是把要混入的对象通过 mergeOptions 合并到 Vue 的 options 中，由于每个组件的构造函数都会在 extend 阶段合并 Vue.options 到自身的 options 中，所以也就相当于每个组件都定义了 mixin 定义的选项。

回到 Vue-Router 的 install 方法，先看混入的 beforeCreate 钩子函数，对于根 Vue 实例而言，执行该钩子函数时定义了 `this._routerRoot` 表示它自身；`this._router` 表示 VueRouter 的实例 router，它是在 new Vue 的时候传入的；另外执行了 `this._router.init()` 方法初始化 router，这个逻辑之后介绍，然后用 defineReactive 方法把 `this._route` 变成响应式对象，这个作用我们之后会介绍。而对于子组件而言，由于组件是树状结构，在遍历组件树的过程中，它们在执行该钩子函数的时候 `this._routerRoot` 始终指向的离它最近的传入了 router 对象作为配置而实例化的父实例。

对于 beforeCreate 和 destroyed 钩子函数，它们都会执行 registerInstance 方法，这个方法的作用我们也是之后会介绍。

接着给 Vue 原型上定义了 `$router` 和 `$route` 2 个属性的 get 方法，这就是为什么我们可以在组件实例上可以访问 `this.$router` 以及 `this.$route`，它们的作用之后介绍。

接着又通过 Vue.component 方法定义了全局的 `<router-link>` 和 `<router-view>` 2 个组件，这也是为什么我们在写模板的时候可以使用这两个标签，它们的作用也是之后介绍。

最后定义了路由中的钩子函数的合并策略，和普通的钩子函数一样。

**总结**

那么到此为止，我们分析了 Vue-Router 的安装过程，Vue 编写插件的时候通常要提供静态的 install 方法，我们通过 Vue.use(plugin) 时候，就是在执行 install 方法。Vue-Router 的 install 方法会给每一个组件注入 beforeCreate 和 destoryed 钩子函数，在 beforeCreate 做一些私有属性定义和路由初始化工作，下一节我们就来分析一下 VueRouter 对象的实现和它的初始化工作。
