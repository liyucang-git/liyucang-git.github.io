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

1996，微软首先提出 iframe 标签，iframe 带来了异步加载和请求元素的概念，随后在 1998 年，微软的 Outloook Web App 团队提出 Ajax 的基本概念（XMLHttpRequest 的前身），并在 IE5 通过 ActiveX 来实现了这项技术。在微软实现这个概念后，其他浏览器比如 Mozilia，Safari，Opera 相继以 XMLHttpRequest 来实现 Ajax。（兼容问题从此出现，话说微软命名真喜欢用 X，MFC 源码一大堆。。）不过在 IE7 发布时，微软选择了妥协，兼容了 XMLHttpRequest 的实现。

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

#### Vue.use

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

#### 路由安装

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

#### 总结

那么到此为止，我们分析了 Vue-Router 的安装过程，Vue 编写插件的时候通常要提供静态的 install 方法，我们通过 Vue.use(plugin) 时候，就是在执行 install 方法。Vue-Router 的 install 方法会给每一个组件注入 beforeCreate 和 destoryed 钩子函数，在 beforeCreate 做一些私有属性定义和路由初始化工作，下一节我们就来分析一下 VueRouter 对象的实现和它的初始化工作。

### VueRouter 对象

VueRouter 的实现是一个类，我们先对它做一个简单地分析，它的定义在 src/index.js 中：

```
export default class VueRouter {
  static install: () => void;
  static version: string;

  app: any;
  apps: Array<any>;
  ready: boolean;
  readyCbs: Array<Function>;
  options: RouterOptions;
  mode: string;
  history: HashHistory | HTML5History | AbstractHistory;
  matcher: Matcher;
  fallback: boolean;
  beforeHooks: Array<?NavigationGuard>;
  resolveHooks: Array<?NavigationGuard>;
  afterHooks: Array<?AfterNavigationHook>;

  constructor (options: RouterOptions = {}) {
    this.app = null
    this.apps = []
    this.options = options
    this.beforeHooks = []
    this.resolveHooks = []
    this.afterHooks = []
    this.matcher = createMatcher(options.routes || [], this)

    let mode = options.mode || 'hash'
    this.fallback = mode === 'history' && !supportsPushState && options.fallback !== false
    if (this.fallback) {
      mode = 'hash'
    }
    if (!inBrowser) {
      mode = 'abstract'
    }
    this.mode = mode

    switch (mode) {
      case 'history':
        this.history = new HTML5History(this, options.base)
        break
      case 'hash':
        this.history = new HashHistory(this, options.base, this.fallback)
        break
      case 'abstract':
        this.history = new AbstractHistory(this, options.base)
        break
      default:
        if (process.env.NODE_ENV !== 'production') {
          assert(false, `invalid mode: ${mode}`)
        }
    }
  }

  match (
    raw: RawLocation,
    current?: Route,
    redirectedFrom?: Location
  ): Route {
    return this.matcher.match(raw, current, redirectedFrom)
  }

  get currentRoute (): ?Route {
    return this.history && this.history.current
  }

  init (app: any /* Vue component instance */) {
    process.env.NODE_ENV !== 'production' && assert(
      install.installed,
      `not installed. Make sure to call \`Vue.use(VueRouter)\` ` +
      `before creating root instance.`
    )

    this.apps.push(app)

    // set up app destroyed handler
    // https://github.com/vuejs/vue-router/issues/2639
    app.$once('hook:destroyed', () => {
      // clean out app from this.apps array once destroyed
      const index = this.apps.indexOf(app)
      if (index > -1) this.apps.splice(index, 1)
      // ensure we still have a main app or null if no apps
      // we do not release the router so it can be reused
      if (this.app === app) this.app = this.apps[0] || null
    })

    // main app previously initialized
    // return as we don't need to set up new history listener
    if (this.app) {
      return
    }

    this.app = app

    const history = this.history

    if (history instanceof HTML5History) {
      history.transitionTo(history.getCurrentLocation())
    } else if (history instanceof HashHistory) {
      const setupHashListener = () => {
        history.setupListeners()
      }
      history.transitionTo(
        history.getCurrentLocation(),
        setupHashListener,
        setupHashListener
      )
    }

    history.listen(route => {
      this.apps.forEach((app) => {
        app._route = route
      })
    })
  }

  beforeEach (fn: Function): Function {
    return registerHook(this.beforeHooks, fn)
  }

  beforeResolve (fn: Function): Function {
    return registerHook(this.resolveHooks, fn)
  }

  afterEach (fn: Function): Function {
    return registerHook(this.afterHooks, fn)
  }

  onReady (cb: Function, errorCb?: Function) {
    this.history.onReady(cb, errorCb)
  }

  onError (errorCb: Function) {
    this.history.onError(errorCb)
  }

  push (location: RawLocation, onComplete?: Function, onAbort?: Function) {
    // $flow-disable-line
    if (!onComplete && !onAbort && typeof Promise !== 'undefined') {
      return new Promise((resolve, reject) => {
        this.history.push(location, resolve, reject)
      })
    } else {
      this.history.push(location, onComplete, onAbort)
    }
  }

  replace (location: RawLocation, onComplete?: Function, onAbort?: Function) {
    // $flow-disable-line
    if (!onComplete && !onAbort && typeof Promise !== 'undefined') {
      return new Promise((resolve, reject) => {
        this.history.replace(location, resolve, reject)
      })
    } else {
      this.history.replace(location, onComplete, onAbort)
    }
  }

  go (n: number) {
    this.history.go(n)
  }

  back () {
    this.go(-1)
  }

  forward () {
    this.go(1)
  }

  getMatchedComponents (to?: RawLocation | Route): Array<any> {
    const route: any = to
      ? to.matched
        ? to
        : this.resolve(to).route
      : this.currentRoute
    if (!route) {
      return []
    }
    return [].concat.apply([], route.matched.map(m => {
      return Object.keys(m.components).map(key => {
        return m.components[key]
      })
    }))
  }

  resolve (
    to: RawLocation,
    current?: Route,
    append?: boolean
  ): {
    location: Location,
    route: Route,
    href: string,
    // for backwards compat
    normalizedTo: Location,
    resolved: Route
  } {
    current = current || this.history.current
    const location = normalizeLocation(
      to,
      current,
      append,
      this
    )
    const route = this.match(location, current)
    const fullPath = route.redirectedFrom || route.fullPath
    const base = this.history.base
    const href = createHref(base, fullPath, this.mode)
    return {
      location,
      route,
      href,
      // for backwards compat
      normalizedTo: location,
      resolved: route
    }
  }

  addRoutes (routes: Array<RouteConfig>) {
    this.matcher.addRoutes(routes)
    if (this.history.current !== START) {
      this.history.transitionTo(this.history.getCurrentLocation())
    }
  }
}
```

VueRouter 定义了一些属性和方法，我们先从它的构造函数看，当我们执行 new VueRouter 的时候做了哪些事情。

```
constructor (options: RouterOptions = {}) {
    this.app = null
    this.apps = []
    this.options = options
    this.beforeHooks = []
    this.resolveHooks = []
    this.afterHooks = []
    this.matcher = createMatcher(options.routes || [], this)

    let mode = options.mode || 'hash'
    this.fallback = mode === 'history' && !supportsPushState && options.fallback !== false
    if (this.fallback) {
      mode = 'hash'
    }
    if (!inBrowser) {
      mode = 'abstract'
    }
    this.mode = mode

    switch (mode) {
      case 'history':
        this.history = new HTML5History(this, options.base)
        break
      case 'hash':
        this.history = new HashHistory(this, options.base, this.fallback)
        break
      case 'abstract':
        this.history = new AbstractHistory(this, options.base)
        break
      default:
        if (process.env.NODE_ENV !== 'production') {
          assert(false, `invalid mode: ${mode}`)
        }
    }
  }
```

构造函数定义了一些属性，其中 this.app 表示根 Vue 实例，this.apps 保存持有 `$options.router` 属性的 Vue 实例，this.options 保存传入的路由配置，this.beforeHooks、 this.resolveHooks、this.afterHooks 表示一些钩子函数，我们之后会介绍，this.matcher 表示路由匹配器，我们之后会介绍，this.fallback 表示在浏览器不支持 history.pushState 的情况下，根据传入的 fallback 配置参数，决定是否回退到 hash 模式，this.mode 表示路由创建的模式，this.history 表示路由历史的具体的实现实例，它是根据 this.mode 的不同实现不同，它有 History 基类，然后不同的 history 实现都是继承 History。

实例化 VueRouter 后会返回它的实例 router，我们在 new Vue 的时候会把 router 作为配置的属性传入，回顾一下上一节我们讲 beforeCreate 混入的时候有这么一段代码：

```
    beforeCreate () {
      if (isDef(this.$options.router)) {
        this._routerRoot = this
        this._router = this.$options.router
        this._router.init(this)
        Vue.util.defineReactive(this, '_route', this._router.history.current)
      } else {
        this._routerRoot = (this.$parent && this.$parent._routerRoot) || this
      }
```

所以组件在执行 beforeCreate 钩子函数的时候，如果传入了 router 实例，都会执行 router.init 方法：

```
init (app: any /* Vue component instance */) {
    process.env.NODE_ENV !== 'production' && assert(
      install.installed,
      `not installed. Make sure to call \`Vue.use(VueRouter)\` ` +
      `before creating root instance.`
    )

    this.apps.push(app)

    // set up app destroyed handler
    // https://github.com/vuejs/vue-router/issues/2639
    app.$once('hook:destroyed', () => {
      // clean out app from this.apps array once destroyed
      const index = this.apps.indexOf(app)
      if (index > -1) this.apps.splice(index, 1)
      // ensure we still have a main app or null if no apps
      // we do not release the router so it can be reused
      if (this.app === app) this.app = this.apps[0] || null
    })

    // main app previously initialized
    // return as we don't need to set up new history listener
    if (this.app) {
      return
    }

    this.app = app

    const history = this.history

    if (history instanceof HTML5History) {
      history.transitionTo(history.getCurrentLocation())
    } else if (history instanceof HashHistory) {
      const setupHashListener = () => {
        history.setupListeners()
      }
      history.transitionTo(
        history.getCurrentLocation(),
        setupHashListener,
        setupHashListener
      )
    }

    history.listen(route => {
      this.apps.forEach((app) => {
        app._route = route
      })
    })
  }
```

init 的逻辑很简单，它传入的参数是 Vue 实例，然后存储到 this.apps 中；只有根 Vue 实例会保存到 this.app 中，并且会拿到当前的 this.history，根据它的不同类型来执行不同逻辑，由于我们平时使用 hash 路由多一些，所以我们先看这部分逻辑，先定义了 setupHashListener 函数，接着执行了 history.transitionTo 方法，它是定义在 History 基类中，代码在 src/history/base.js：

```
transitionTo (location: RawLocation, onComplete?: Function, onAbort?: Function) {
  const route = this.router.match(location, this.current)
  // ...
}
```

我们先不着急去看 transitionTo 的具体实现，先看第一行代码，它调用了 this.router.match 函数：

```
match (
  raw: RawLocation,
  current?: Route,
  redirectedFrom?: Location
): Route {
  return this.matcher.match(raw, current, redirectedFrom)
}
```

**总结**

通过这一节的分析，我们大致对 VueRouter 类有了大致了解，知道了它的一些属性和方法，同时了解到在组件的初始化阶段，执行到 beforeCreate 钩子函数的时候会执行 router.init 方法，然后又会执行 history.transitionTo 方法做路由过渡，进而引出了 matcher 的概念，接下来我们先研究一下 matcher 的相关实现。

### matcher

matcher 相关的实现都在 src/create-matcher.js 中，我们先来看一下 matcher 的数据结构：

```
export type Matcher = {
  match: (raw: RawLocation, current?: Route, redirectedFrom?: Location) => Route;
  addRoutes: (routes: Array<RouteConfig>) => void;
};
```

Matcher 返回了 2 个方法，match 和 addRoutes，在上一节我们接触到了 match 方法，顾名思义它是做匹配，那么匹配的是什么，在介绍之前，我们先了解路由中重要的 2 个概念，Loaction 和 Route，它们的数据结构定义在 flow/declarations.js 中。

**Location**

```
declare type Location = {
  _normalized?: boolean;
  name?: string;
  path?: string;
  hash?: string;
  query?: Dictionary<string>;
  params?: Dictionary<string>;
  append?: boolean;
  replace?: boolean;
}
```

Vue-Router 中定义的 Location 数据结构和浏览器提供的 window.location 部分结构有点类似，它们都是对 url 的结构化描述。举个例子：`/abc?foo=bar&baz=qux#hello`，它的 path 是 `/abc`，query 是 `{foo:'bar',baz:'qux'}`。Location 的其他属性我们之后会介绍。

**Route**

```
declare type Route = {
  path: string;
  name: ?string;
  hash: string;
  query: Dictionary<string>;
  params: Dictionary<string>;
  fullPath: string;
  matched: Array<RouteRecord>;
  redirectedFrom?: string;
  meta?: any;
}
```

Route 表示的是路由中的一条线路，它除了描述了类似 Loctaion 的 path、query、hash 这些概念，还有 matched 表示匹配到的所有的 RouteRecord。Route 的其他属性我们之后会介绍。

#### createMatcher

在了解了 Location 和 Route 后，我们来看一下 matcher 的创建过程：

```

export function createMatcher (
  routes: Array<RouteConfig>,
  router: VueRouter
): Matcher {
  const { pathList, pathMap, nameMap } = createRouteMap(routes)

  function addRoutes (routes) {
    createRouteMap(routes, pathList, pathMap, nameMap)
  }

  function match (
    raw: RawLocation,
    currentRoute?: Route,
    redirectedFrom?: Location
  ): Route {
    const location = normalizeLocation(raw, currentRoute, false, router)
    const { name } = location

    if (name) {
      const record = nameMap[name]
      if (process.env.NODE_ENV !== 'production') {
        warn(record, `Route with name '${name}' does not exist`)
      }
      if (!record) return _createRoute(null, location)
      const paramNames = record.regex.keys
        .filter(key => !key.optional)
        .map(key => key.name)

      if (typeof location.params !== 'object') {
        location.params = {}
      }

      if (currentRoute && typeof currentRoute.params === 'object') {
        for (const key in currentRoute.params) {
          if (!(key in location.params) && paramNames.indexOf(key) > -1) {
            location.params[key] = currentRoute.params[key]
          }
        }
      }

      location.path = fillParams(record.path, location.params, `named route "${name}"`)
      return _createRoute(record, location, redirectedFrom)
    } else if (location.path) {
      location.params = {}
      for (let i = 0; i < pathList.length; i++) {
        const path = pathList[i]
        const record = pathMap[path]
        if (matchRoute(record.regex, location.path, location.params)) {
          return _createRoute(record, location, redirectedFrom)
        }
      }
    }
    // no match
    return _createRoute(null, location)
  }

  function redirect (
    record: RouteRecord,
    location: Location
  ): Route {
    const originalRedirect = record.redirect
    let redirect = typeof originalRedirect === 'function'
      ? originalRedirect(createRoute(record, location, null, router))
      : originalRedirect

    if (typeof redirect === 'string') {
      redirect = { path: redirect }
    }

    if (!redirect || typeof redirect !== 'object') {
      if (process.env.NODE_ENV !== 'production') {
        warn(
          false, `invalid redirect option: ${JSON.stringify(redirect)}`
        )
      }
      return _createRoute(null, location)
    }

    const re: Object = redirect
    const { name, path } = re
    let { query, hash, params } = location
    query = re.hasOwnProperty('query') ? re.query : query
    hash = re.hasOwnProperty('hash') ? re.hash : hash
    params = re.hasOwnProperty('params') ? re.params : params

    if (name) {
      // resolved named direct
      const targetRecord = nameMap[name]
      if (process.env.NODE_ENV !== 'production') {
        assert(targetRecord, `redirect failed: named route "${name}" not found.`)
      }
      return match({
        _normalized: true,
        name,
        query,
        hash,
        params
      }, undefined, location)
    } else if (path) {
      // 1. resolve relative redirect
      const rawPath = resolveRecordPath(path, record)
      // 2. resolve params
      const resolvedPath = fillParams(rawPath, params, `redirect route with path "${rawPath}"`)
      // 3. rematch with existing query and hash
      return match({
        _normalized: true,
        path: resolvedPath,
        query,
        hash
      }, undefined, location)
    } else {
      if (process.env.NODE_ENV !== 'production') {
        warn(false, `invalid redirect option: ${JSON.stringify(redirect)}`)
      }
      return _createRoute(null, location)
    }
  }

  function alias (
    record: RouteRecord,
    location: Location,
    matchAs: string
  ): Route {
    const aliasedPath = fillParams(matchAs, location.params, `aliased route with path "${matchAs}"`)
    const aliasedMatch = match({
      _normalized: true,
      path: aliasedPath
    })
    if (aliasedMatch) {
      const matched = aliasedMatch.matched
      const aliasedRecord = matched[matched.length - 1]
      location.params = aliasedMatch.params
      return _createRoute(aliasedRecord, location)
    }
    return _createRoute(null, location)
  }

  function _createRoute (
    record: ?RouteRecord,
    location: Location,
    redirectedFrom?: Location
  ): Route {
    if (record && record.redirect) {
      return redirect(record, redirectedFrom || location)
    }
    if (record && record.matchAs) {
      return alias(record, location, record.matchAs)
    }
    return createRoute(record, location, redirectedFrom, router)
  }

  return {
    match,
    addRoutes
  }
}
```

createMatcher 接收 2 个参数，一个是 router，它是我们 new VueRouter 返回的实例，一个是 routes，它是用户定义的路由配置，来看一下例子中的配置：

```
const Foo = { template: '<div>foo</div>' }
const Bar = { template: '<div>bar</div>' }

const routes = [
  { path: '/foo', component: Foo },
  { path: '/bar', component: Bar }
]
```

createMathcer 首先执行的逻辑是 `const { pathList, pathMap, nameMap } = createRouteMap(routes)` 创建一个路由映射表，createRouteMap 的定义在 `src/create-route-map` 中：

```
export function createRouteMap (
  routes: Array<RouteConfig>,
  oldPathList?: Array<string>,
  oldPathMap?: Dictionary<RouteRecord>,
  oldNameMap?: Dictionary<RouteRecord>
): {
  pathList: Array<string>,
  pathMap: Dictionary<RouteRecord>,
  nameMap: Dictionary<RouteRecord>
} {
  // the path list is used to control path matching priority
  const pathList: Array<string> = oldPathList || []
  // $flow-disable-line
  const pathMap: Dictionary<RouteRecord> = oldPathMap || Object.create(null)
  // $flow-disable-line
  const nameMap: Dictionary<RouteRecord> = oldNameMap || Object.create(null)

  routes.forEach(route => {
    addRouteRecord(pathList, pathMap, nameMap, route)
  })

  // ensure wildcard routes are always at the end
  for (let i = 0, l = pathList.length; i < l; i++) {
    if (pathList[i] === '*') {
      pathList.push(pathList.splice(i, 1)[0])
      l--
      i--
    }
  }

  return {
    pathList,
    pathMap,
    nameMap
  }
}
```

createRouteMap 函数的目标是把用户的路由配置转换成一张路由映射表，它包含 3 个部分，pathList 存储所有的 path，pathMap 表示一个 path 到 RouteRecord 的映射关系，而 nameMap 表示 name 到 RouteRecord 的映射关系。那么 RouteRecord 到底是什么，先来看一下它的数据结构：

```
declare type RouteRecord = {
  path: string;
  regex: RouteRegExp;
  components: Dictionary<any>;
  instances: Dictionary<any>;
  name: ?string;
  parent: ?RouteRecord;
  redirect: ?RedirectOption;
  matchAs: ?string;
  beforeEnter: ?NavigationGuard;
  meta: any;
  props: boolean | Object | Function | Dictionary<boolean | Object | Function>;
}
```

它的创建是通过遍历 routes 为每一个 route 执行 addRouteRecord 方法生成一条记录，来看一下它的定义：

```
function addRouteRecord (
  pathList: Array<string>,
  pathMap: Dictionary<RouteRecord>,
  nameMap: Dictionary<RouteRecord>,
  route: RouteConfig,
  parent?: RouteRecord,
  matchAs?: string
) {
  const { path, name } = route
  if (process.env.NODE_ENV !== 'production') {
    assert(path != null, `"path" is required in a route configuration.`)
    assert(
      typeof route.component !== 'string',
      `route config "component" for path: ${String(
        path || name
      )} cannot be a ` + `string id. Use an actual component instead.`
    )
  }

  const pathToRegexpOptions: PathToRegexpOptions =
    route.pathToRegexpOptions || {}
  const normalizedPath = normalizePath(path, parent, pathToRegexpOptions.strict)

  if (typeof route.caseSensitive === 'boolean') {
    pathToRegexpOptions.sensitive = route.caseSensitive
  }

  const record: RouteRecord = {
    path: normalizedPath,
    regex: compileRouteRegex(normalizedPath, pathToRegexpOptions),
    components: route.components || { default: route.component },
    instances: {},
    name,
    parent,
    matchAs,
    redirect: route.redirect,
    beforeEnter: route.beforeEnter,
    meta: route.meta || {},
    props:
      route.props == null
        ? {}
        : route.components
          ? route.props
          : { default: route.props }
  }

  if (route.children) {
    // Warn if route is named, does not redirect and has a default child route.
    // If users navigate to this route by name, the default child will
    // not be rendered (GH Issue #629)
    if (process.env.NODE_ENV !== 'production') {
      if (
        route.name &&
        !route.redirect &&
        route.children.some(child => /^\/?$/.test(child.path))
      ) {
        warn(
          false,
          `Named Route '${route.name}' has a default child route. ` +
            `When navigating to this named route (:to="{name: '${
              route.name
            }'"), ` +
            `the default child route will not be rendered. Remove the name from ` +
            `this route and use the name of the default child route for named ` +
            `links instead.`
        )
      }
    }
    route.children.forEach(child => {
      const childMatchAs = matchAs
        ? cleanPath(`${matchAs}/${child.path}`)
        : undefined
      addRouteRecord(pathList, pathMap, nameMap, child, record, childMatchAs)
    })
  }

  if (!pathMap[record.path]) {
    pathList.push(record.path)
    pathMap[record.path] = record
  }

  if (route.alias !== undefined) {
    const aliases = Array.isArray(route.alias) ? route.alias : [route.alias]
    for (let i = 0; i < aliases.length; ++i) {
      const alias = aliases[i]
      if (process.env.NODE_ENV !== 'production' && alias === path) {
        warn(
          false,
          `Found an alias with the same value as the path: "${path}". You have to remove that alias. It will be ignored in development.`
        )
        // skip in dev to make it work
        continue
      }

      const aliasRoute = {
        path: alias,
        children: route.children
      }
      addRouteRecord(
        pathList,
        pathMap,
        nameMap,
        aliasRoute,
        parent,
        record.path || '/' // matchAs
      )
    }
  }

  if (name) {
    if (!nameMap[name]) {
      nameMap[name] = record
    } else if (process.env.NODE_ENV !== 'production' && !matchAs) {
      warn(
        false,
        `Duplicate named routes definition: ` +
          `{ name: "${name}", path: "${record.path}" }`
      )
    }
  }
}
```

我们只看几个关键逻辑，首先创建 RouteRecord 的代码如下：

```
  const record: RouteRecord = {
    path: normalizedPath,
    regex: compileRouteRegex(normalizedPath, pathToRegexpOptions),
    components: route.components || { default: route.component },
    instances: {},
    name,
    parent,
    matchAs,
    redirect: route.redirect,
    beforeEnter: route.beforeEnter,
    meta: route.meta || {},
    props:
      route.props == null
        ? {}
        : route.components
          ? route.props
          : { default: route.props }
  }
```

这里要注意几个点，path 是规范化后的路径，它会根据 parent 的 path 做计算；regex 是一个正则表达式的扩展，它利用了 path-to-regexp 这个工具库，把 path 解析成一个正则表达式的扩展，举个例子：

```
var keys = []
var re = pathToRegexp('/foo/:bar', keys)
// re = /^\/foo\/([^\/]+?)\/?$/i
// keys = [{ name: 'bar', prefix: '/', delimiter: '/', optional: false, repeat: false, pattern: '[^\\/]+?' }]
```

components 是一个对象，通常我们在配置中写的 component 实际上这里会被转换成 {components: route.component}；instances 表示组件的实例，也是一个对象类型；parent 表示父的 RouteRecord，因为我们配置的时候有时候会配置子路由，所以整个 RouteRecord 也就是一个树型结构。

```
if (route.children) {
  // ...
  route.children.forEach(child => {
  const childMatchAs = matchAs
    ? cleanPath(`${matchAs}/${child.path}`)
    : undefined
  addRouteRecord(pathList, pathMap, nameMap, child, record, childMatchAs)
})
}
```

如果配置了 children，那么递归执行 addRouteRecord 方法，并把当前的 record 作为 parent 传入，通过这样的深度遍历，我们就可以拿到一个 route 下的完整记录。

```
if (!pathMap[record.path]) {
  pathList.push(record.path)
  pathMap[record.path] = record
}
```

为 pathList 和 pathMap 各添加一条记录。

```
if (name) {
  if (!nameMap[name]) {
    nameMap[name] = record
  }
  // ...
}
```

如果我们在路由配置中配置了 name，则给 nameMap 添加一条记录。

由于 pathList、pathMap、nameMap 都是引用类型，所以在遍历整个 routes 过程中去执行 addRouteRecord 方法，会不断给他们添加数据。那么经过整个 createRouteMap 方法的执行，我们得到的就是 pathList、pathMap 和 nameMap。其中 pathList 是为了记录路由配置中的所有 path，而 pathMap 和 nameMap 都是为了通过 path 和 name 能快速查到对应的 RouteRecord。

再回到 createMatcher 函数，接下来就定义了一系列方法，最后返回了一个对象。

```
return {
  match,
  addRoutes
}
```

也就是说，matcher 是一个对象，它对外暴露了 match 和 addRoutes 方法。

#### addRoutes

addRoutes 方法的作用是动态添加路由配置，因为在实际开发中有些场景是不能提前把路由写死的，需要根据一些条件动态添加路由，所以 Vue-Router 也提供了这一接口：

```
function addRoutes (routes) {
  createRouteMap(routes, pathList, pathMap, nameMap)
}
```

addRoutes 的方法十分简单，再次调用 createRouteMap 即可，传入新的 routes 配置，由于 pathList、pathMap、nameMap 都是引用类型，执行 addRoutes 后会修改它们的值。

#### match

```
  function match (
    raw: RawLocation,
    currentRoute?: Route,
    redirectedFrom?: Location
  ): Route {
    const location = normalizeLocation(raw, currentRoute, false, router)
    const { name } = location

    if (name) {
      const record = nameMap[name]
      if (process.env.NODE_ENV !== 'production') {
        warn(record, `Route with name '${name}' does not exist`)
      }
      if (!record) return _createRoute(null, location)
      const paramNames = record.regex.keys
        .filter(key => !key.optional)
        .map(key => key.name)

      if (typeof location.params !== 'object') {
        location.params = {}
      }

      if (currentRoute && typeof currentRoute.params === 'object') {
        for (const key in currentRoute.params) {
          if (!(key in location.params) && paramNames.indexOf(key) > -1) {
            location.params[key] = currentRoute.params[key]
          }
        }
      }

      location.path = fillParams(record.path, location.params, `named route "${name}"`)
      return _createRoute(record, location, redirectedFrom)
    } else if (location.path) {
      location.params = {}
      for (let i = 0; i < pathList.length; i++) {
        const path = pathList[i]
        const record = pathMap[path]
        if (matchRoute(record.regex, location.path, location.params)) {
          return _createRoute(record, location, redirectedFrom)
        }
      }
    }
    // no match
    return _createRoute(null, location)
  }
```

match 方法接收 3 个参数，其中 raw 是 RawLocation 类型，它可以是一个 url 字符串，也可以是一个 Location 对象；currentRoute 是 Route 类型，它表示当前的路径；redirectedFrom 和重定向相关，这里先忽略。match 方法返回的是一个路径，它的作用是根据传入的 raw 和当前的路径 currentRoute 计算出一个新的路径并返回。

首先执行了 normalizeLocation，它的定义在 `src/util/location.js` 中：

```
export function normalizeLocation (
  raw: RawLocation,
  current: ?Route,
  append: ?boolean,
  router: ?VueRouter
): Location {
  let next: Location = typeof raw === 'string' ? { path: raw } : raw
  // named target
  if (next._normalized) {
    return next
  } else if (next.name) {
    return extend({}, raw)
  }

  // relative params
  if (!next.path && next.params && current) {
    next = extend({}, next)
    next._normalized = true
    const params: any = extend(extend({}, current.params), next.params)
    if (current.name) {
      next.name = current.name
      next.params = params
    } else if (current.matched.length) {
      const rawPath = current.matched[current.matched.length - 1].path
      next.path = fillParams(rawPath, params, `path ${current.path}`)
    } else if (process.env.NODE_ENV !== 'production') {
      warn(false, `relative params navigation requires a current route.`)
    }
    return next
  }

  const parsedPath = parsePath(next.path || '')
  const basePath = (current && current.path) || '/'
  const path = parsedPath.path
    ? resolvePath(parsedPath.path, basePath, append || next.append)
    : basePath

  const query = resolveQuery(
    parsedPath.query,
    next.query,
    router && router.options.parseQuery
  )

  let hash = next.hash || parsedPath.hash
  if (hash && hash.charAt(0) !== '#') {
    hash = `#${hash}`
  }

  return {
    _normalized: true,
    path,
    query,
    hash
  }
}
```

normalizeLocation 方法的作用是根据 raw，current 计算出新的 location，它主要处理了 raw 的两种情况，一种是有 params 且没有 path，一种是有 path 的，对于第一种情况，如果 current 有 name，则计算出的 location 也有 name。

计算出新的 location 后，对 location 的 name 和 path 的两种情况做了处理。

**name**

有 name 的情况下就根据 nameMap 匹配到 record，它就是一个 RouterRecord 对象，如果 record 不存在，则匹配失败，返回一个空路径；然后拿到 record 对应的 paramNames，再对比 currentRoute 中的 params，把交集部分的 params 添加到 location 中，然后在通过 fillParams 方法根据 record.path 和 location.path 计算出 location.path，最后调用 `_createRoute(record, location, redirectedFrom)` 去生成一条新路径，该方法我们之后会介绍。

**path**

通过 name 我们可以很快的找到 record，但是通过 path 并不能，因为我们计算后的 location.path 是一个真实路径，而 record 中的 path 可能会有 param，因此需要对所有的 pathList 做顺序遍历， 然后通过 matchRoute 方法根据 record.regex、location.path、location.params 匹配，如果匹配到则也通过 `_createRoute(record, location, redirectedFrom)`去生成一条新路径。因为是顺序遍历，所以我们书写路由配置要注意路径的顺序，因为写在前面的会优先尝试匹配。

最后我们来看一下 `_createRoute` 的实现：

```
  function _createRoute (
    record: ?RouteRecord,
    location: Location,
    redirectedFrom?: Location
  ): Route {
    if (record && record.redirect) {
      return redirect(record, redirectedFrom || location)
    }
    if (record && record.matchAs) {
      return alias(record, location, record.matchAs)
    }
    return createRoute(record, location, redirectedFrom, router)
  }
```

我们先不考虑 record.redirect 和 record.matchAs 的情况，最终会调用 createRoute 方法，它的定义在 `src/uitl/route.js` 中：

```
export function createRoute (
  record: ?RouteRecord,
  location: Location,
  redirectedFrom?: ?Location,
  router?: VueRouter
): Route {
  const stringifyQuery = router && router.options.stringifyQuery

  let query: any = location.query || {}
  try {
    query = clone(query)
  } catch (e) {}

  const route: Route = {
    name: location.name || (record && record.name),
    meta: (record && record.meta) || {},
    path: location.path || '/',
    hash: location.hash || '',
    query,
    params: location.params || {},
    fullPath: getFullPath(location, stringifyQuery),
    matched: record ? formatMatch(record) : []
  }
  if (redirectedFrom) {
    route.redirectedFrom = getFullPath(redirectedFrom, stringifyQuery)
  }
  return Object.freeze(route)
}
```

createRoute 可以根据 record 和 location 创建出来，最终返回的是一条 Route 路径，我们之前也介绍过它的数据结构。在 Vue-Router 中，所有的 Route 最终都会通过 createRoute 函数创建，并且它最后是不可以被外部修改的。Route 对象中有一个非常重要属性是 matched，它通过 formatMatch(record) 计算而来：

```
function formatMatch (record: ?RouteRecord): Array<RouteRecord> {
  const res = []
  while (record) {
    res.unshift(record)
    record = record.parent
  }
  return res
}
```

可以看它是通过 record 循环向上找 parent，直到找到最外层，并把所有的 record 都 push 到一个数组中，最终返回的就是 record 的数组，它记录了一条线路上的所有 record。matched 属性非常有用，它为之后渲染组件提供了依据。

#### 总结

那么到此，matcher 相关的主流程的分析就结束了，我们了解了 Location、Route、RouteRecord 等概念。并通过 matcher 的 match 方法，我们会找到匹配的路径 Route，这个对 Route 的切换，组件的渲染都有非常重要的指导意义。下一节我们会回到 transitionTo 方法，看一看路径的切换都做了哪些事情。

### 路径切换

history.transitionTo 是 Vue-Router 中非常重要的方法，当我们切换路由线路的时候，就会执行到该方法，前一节我们分析了 matcher 的相关实现，知道它是如何找到匹配的新线路，那么匹配到新线路后又做了哪些事情，接下来我们来完整分析一下 transitionTo 的实现，它的定义在 `src/history/base.js` 中：

```
  transitionTo (
    location: RawLocation,
    onComplete?: Function,
    onAbort?: Function
  ) {
    const route = this.router.match(location, this.current)
    this.confirmTransition(
      route,
      () => {
        this.updateRoute(route)
        onComplete && onComplete(route)
        this.ensureURL()

        // fire ready cbs once
        if (!this.ready) {
          this.ready = true
          this.readyCbs.forEach(cb => {
            cb(route)
          })
        }
      },
      err => {
        if (onAbort) {
          onAbort(err)
        }
        if (err && !this.ready) {
          this.ready = true
          this.readyErrorCbs.forEach(cb => {
            cb(err)
          })
        }
      }
    )
  }
```

transitionTo 首先根据目标 location 和当前路径 this.current 执行 this.router.match 方法去匹配到目标的路径。这里 this.current 是 history 维护的当前路径，它的初始值是在 history 的构造函数中初始化的：

```
this.current = START
```

START 的定义在 src/util/route.js 中：

```
export const START = createRoute(null, {
  path: '/'
})
```

这样就创建了一个初始的 Route，而 transitionTo 实际上也就是在切换 this.current，稍后我们会看到。

拿到新的路径后，那么接下来就会执行 confirmTransition 方法去做真正的切换，由于这个过程可能有一些异步的操作（如异步组件），所以整个 confirmTransition API 设计成带有成功回调函数和失败回调函数，先来看一下它的定义：

```
  confirmTransition (route: Route, onComplete: Function, onAbort?: Function) {
    const current = this.current
    const abort = err => {
      // after merging https://github.com/vuejs/vue-router/pull/2771 we
      // When the user navigates through history through back/forward buttons
      // we do not want to throw the error. We only throw it if directly calling
      // push/replace. That's why it's not included in isError
      if (!isExtendedError(NavigationDuplicated, err) && isError(err)) {
        if (this.errorCbs.length) {
          this.errorCbs.forEach(cb => {
            cb(err)
          })
        } else {
          warn(false, 'uncaught error during route navigation:')
          console.error(err)
        }
      }
      onAbort && onAbort(err)
    }
    if (
      isSameRoute(route, current) &&
      // in the case the route map has been dynamically appended to
      route.matched.length === current.matched.length
    ) {
      this.ensureURL()
      return abort(new NavigationDuplicated(route))
    }

    const { updated, deactivated, activated } = resolveQueue(
      this.current.matched,
      route.matched
    )

    const queue: Array<?NavigationGuard> = [].concat(
      // in-component leave guards
      extractLeaveGuards(deactivated),
      // global before hooks
      this.router.beforeHooks,
      // in-component update hooks
      extractUpdateHooks(updated),
      // in-config enter guards
      activated.map(m => m.beforeEnter),
      // async components
      resolveAsyncComponents(activated)
    )

    this.pending = route
    const iterator = (hook: NavigationGuard, next) => {
      if (this.pending !== route) {
        return abort()
      }
      try {
        hook(route, current, (to: any) => {
          if (to === false || isError(to)) {
            // next(false) -> abort navigation, ensure current URL
            this.ensureURL(true)
            abort(to)
          } else if (
            typeof to === 'string' ||
            (typeof to === 'object' &&
              (typeof to.path === 'string' || typeof to.name === 'string'))
          ) {
            // next('/') or next({ path: '/' }) -> redirect
            abort()
            if (typeof to === 'object' && to.replace) {
              this.replace(to)
            } else {
              this.push(to)
            }
          } else {
            // confirm transition and pass on the value
            next(to)
          }
        })
      } catch (e) {
        abort(e)
      }
    }

    runQueue(queue, iterator, () => {
      const postEnterCbs = []
      const isValid = () => this.current === route
      // wait until async components are resolved before
      // extracting in-component enter guards
      const enterGuards = extractEnterGuards(activated, postEnterCbs, isValid)
      const queue = enterGuards.concat(this.router.resolveHooks)
      runQueue(queue, iterator, () => {
        if (this.pending !== route) {
          return abort()
        }
        this.pending = null
        onComplete(route)
        if (this.router.app) {
          this.router.app.$nextTick(() => {
            postEnterCbs.forEach(cb => {
              cb()
            })
          })
        }
      })
    })
  }
```

首先定义了 abort 函数，然后判断如果满足计算后的 route 和 current 是相同路径的话，则直接调用 this.ensureUrl 和 abort，ensureUrl 这个函数我们之后会介绍。

接着又根据 current.matched 和 route.matched 执行了 resolveQueue 方法解析出 3 个队列：

```
function resolveQueue (
  current: Array<RouteRecord>,
  next: Array<RouteRecord>
): {
  updated: Array<RouteRecord>,
  activated: Array<RouteRecord>,
  deactivated: Array<RouteRecord>
} {
  let i
  const max = Math.max(current.length, next.length)
  for (i = 0; i < max; i++) {
    if (current[i] !== next[i]) {
      break
    }
  }
  return {
    updated: next.slice(0, i),
    activated: next.slice(i),
    deactivated: current.slice(i)
  }
}
```

因为 route.matched 是一个 RouteRecord 的数组，由于路径是由 current 变向 route，那么就遍历对比 2 边的 RouteRecord，找到一个不一样的位置 i，那么 next 中从 0 到 i 的 RouteRecord 是两边都一样，则为 updated 的部分；从 i 到最后的 RouteRecord 是 next 独有的，为 activated 的部分；而 current 中从 i 到最后的 RouteRecord 则没有了，为 deactivated 的部分。

拿到 updated、activated、deactivated 3 个 ReouteRecord 数组后，接下来就是路径变换后的一个重要部分，执行一系列的钩子函数。

#### 导航守卫

官方的说法叫导航守卫，实际上就是发生在路由路径切换的时候，执行的一系列钩子函数。

我们先从整体上看一下这些钩子函数执行的逻辑，首先构造一个队列 queue，它实际上是一个数组；然后再定义一个迭代器函数 iterator；最后再执行 runQueue 方法来执行这个队列。我们先来看一下 runQueue 的定义，在 src/util/async.js 中：

```
export function runQueue (queue: Array<?NavigationGuard>, fn: Function, cb: Function) {
  const step = index => {
    if (index >= queue.length) {
      cb()
    } else {
      if (queue[index]) {
        fn(queue[index], () => {
          step(index + 1)
        })
      } else {
        step(index + 1)
      }
    }
  }
  step(0)
}
```

这是一个非常经典的异步函数队列化执行的模式， queue 是一个 NavigationGuard 类型的数组，我们定义了 step 函数，每次根据 index 从 queue 中取一个 guard，然后执行 fn 函数，并且把 guard 作为参数传入，第二个参数是一个函数，当这个函数执行的时候再递归执行 step 函数，前进到下一个，注意这里的 fn 就是我们刚才的 iterator 函数，那么我们再回到 iterator 函数的定义：

```
    const iterator = (hook: NavigationGuard, next) => {
      if (this.pending !== route) {
        return abort()
      }
      try {
        hook(route, current, (to: any) => {
          if (to === false || isError(to)) {
            // next(false) -> abort navigation, ensure current URL
            this.ensureURL(true)
            abort(to)
          } else if (
            typeof to === 'string' ||
            (typeof to === 'object' &&
              (typeof to.path === 'string' || typeof to.name === 'string'))
          ) {
            // next('/') or next({ path: '/' }) -> redirect
            abort()
            if (typeof to === 'object' && to.replace) {
              this.replace(to)
            } else {
              this.push(to)
            }
          } else {
            // confirm transition and pass on the value
            next(to)
          }
        })
      } catch (e) {
        abort(e)
      }
    }
```

iterator 函数逻辑很简单，它就是去执行每一个 导航守卫 hook，并传入 route、current 和匿名函数，这些参数对应文档中的 to、from、next，当执行了匿名函数，会根据一些条件执行 abort 或 next，只有执行 next 的时候，才会前进到下一个导航守卫钩子函数中，这也就是为什么官方文档会说只有执行 next 方法来 resolve 这个钩子函数。

那么最后我们来看 queue 是怎么构造的：

```
    const queue: Array<?NavigationGuard> = [].concat(
      // in-component leave guards
      extractLeaveGuards(deactivated),
      // global before hooks
      this.router.beforeHooks,
      // in-component update hooks
      extractUpdateHooks(updated),
      // in-config enter guards
      activated.map(m => m.beforeEnter),
      // async components
      resolveAsyncComponents(activated)
    )
```

按照顺序如下：

1. 在失活的组件里调用离开守卫。

2. 调用全局的 beforeEach 守卫。

3. 在重用的组件里调用 beforeRouteUpdate 守卫

4. 在激活的路由配置里调用 beforeEnter。

5. 解析异步路由组件。

接下来我们来分别介绍这 5 步的实现。

第一步是通过执行 extractLeaveGuards(deactivated)，先来看一下 extractLeaveGuards 的定义：

```
function extractLeaveGuards (deactivated: Array<RouteRecord>): Array<?Function> {
  return extractGuards(deactivated, 'beforeRouteLeave', bindGuard, true)
}
```

它内部调用了 extractGuards 的通用方法，可以从 RouteRecord 数组中提取各个阶段的守卫：

```
function extractGuards (
  records: Array<RouteRecord>,
  name: string,
  bind: Function,
  reverse?: boolean
): Array<?Function> {
  const guards = flatMapComponents(records, (def, instance, match, key) => {
    const guard = extractGuard(def, name)
    if (guard) {
      return Array.isArray(guard)
        ? guard.map(guard => bind(guard, instance, match, key))
        : bind(guard, instance, match, key)
    }
  })
  return flatten(reverse ? guards.reverse() : guards)
}
```

这里用到了 flatMapComponents 方法去从 records 中获取所有的导航，它的定义在 src/util/resolve-components.js 中：

```
export function flatMapComponents (
  matched: Array<RouteRecord>,
  fn: Function
): Array<?Function> {
  return flatten(matched.map(m => {
    return Object.keys(m.components).map(key => fn(
      m.components[key],
      m.instances[key],
      m, key
    ))
  }))
}

export function flatten (arr: Array<any>): Array<any> {
  return Array.prototype.concat.apply([], arr)
}
```

flatMapComponents 的作用就是返回一个数组，数组的元素是从 matched 里获取到所有组件的 key，然后返回 fn 函数执行的结果，flatten 作用是把二维数组拍平成一维数组。

那么对于 extractGuards 中 flatMapComponents 的调用，执行每个 fn 的时候，通过 extractGuard(def, name) 获取到组件中对应 name 的导航守卫：

```
function extractGuard (
  def: Object | Function,
  key: string
): NavigationGuard | Array<NavigationGuard> {
  if (typeof def !== 'function') {
    // extend now so that global mixins are applied.
    def = _Vue.extend(def)
  }
  return def.options[key]
}
```

获取到 guard 后，还会调用 bind 方法把组件的实例 instance 作为函数执行的上下文绑定到 guard 上，bind 方法的对应的是 bindGuard：

```
function bindGuard (guard: NavigationGuard, instance: ?_Vue): ?NavigationGuard {
  if (instance) {
    return function boundRouteGuard () {
      return guard.apply(instance, arguments)
    }
  }
}
```

那么对于 extractLeaveGuards(deactivated) 而言，获取到的就是所有失活组件中定义的 beforeRouteLeave 钩子函数。

第二步是 this.router.beforeHooks，在我们的 VueRouter 类中定义了 beforeEach 方法，在 src/index.js 中：

```
beforeEach (fn: Function): Function {
  return registerHook(this.beforeHooks, fn)
}

function registerHook (list: Array<any>, fn: Function): Function {
  list.push(fn)
  return () => {
    const i = list.indexOf(fn)
    if (i > -1) list.splice(i, 1)
  }
}
```

当用户使用 router.beforeEach 注册了一个全局守卫，就会往 router.beforeHooks 添加一个钩子函数，这样 this.router.beforeHooks 获取的就是用户注册的全局 beforeEach 守卫。

第三步执行了 extractUpdateHooks(updated)，来看一下 extractUpdateHooks 的定义：

```
function extractUpdateHooks (updated: Array<RouteRecord>): Array<?Function> {
  return extractGuards(updated, 'beforeRouteUpdate', bindGuard)
}
```

和 extractLeaveGuards(deactivated) 类似，extractUpdateHooks(updated) 获取到的就是所有重用的组件中定义的 beforeRouteUpdate 钩子函数。

第四步是执行 activated.map(m => m.beforeEnter)，获取的是在激活的路由配置中定义的 beforeEnter 函数。

第五步是执行 resolveAsyncComponents(activated) 解析异步组件，先来看一下 resolveAsyncComponents 的定义，在 src/util/resolve-components.js 中：

```
export function resolveAsyncComponents (matched: Array<RouteRecord>): Function {
  return (to, from, next) => {
    let hasAsync = false
    let pending = 0
    let error = null

    flatMapComponents(matched, (def, _, match, key) => {
      // if it's a function and doesn't have cid attached,
      // assume it's an async component resolve function.
      // we are not using Vue's default async resolving mechanism because
      // we want to halt the navigation until the incoming component has been
      // resolved.
      if (typeof def === 'function' && def.cid === undefined) {
        hasAsync = true
        pending++

        const resolve = once(resolvedDef => {
          if (isESModule(resolvedDef)) {
            resolvedDef = resolvedDef.default
          }
          // save resolved on async factory in case it's used elsewhere
          def.resolved = typeof resolvedDef === 'function'
            ? resolvedDef
            : _Vue.extend(resolvedDef)
          match.components[key] = resolvedDef
          pending--
          if (pending <= 0) {
            next()
          }
        })

        const reject = once(reason => {
          const msg = `Failed to resolve async component ${key}: ${reason}`
          process.env.NODE_ENV !== 'production' && warn(false, msg)
          if (!error) {
            error = isError(reason)
              ? reason
              : new Error(msg)
            next(error)
          }
        })

        let res
        try {
          res = def(resolve, reject)
        } catch (e) {
          reject(e)
        }
        if (res) {
          if (typeof res.then === 'function') {
            res.then(resolve, reject)
          } else {
            // new syntax in Vue 2.3
            const comp = res.component
            if (comp && typeof comp.then === 'function') {
              comp.then(resolve, reject)
            }
          }
        }
      }
    })

    if (!hasAsync) next()
  }
}
```

resolveAsyncComponents 返回的是一个导航守卫函数，有标准的 to、from、next 参数。它的内部实现很简单，利用了 flatMapComponents 方法从 matched 中获取到每个组件的定义，判断如果是异步组件，则执行异步组件加载逻辑，这块和我们之前分析 Vue 加载异步组件很类似，加载成功后会执行 `match.components[key] = resolvedDef` 把解析好的异步组件放到对应的 components 上，并且执行 next 函数。

这样在 resolveAsyncComponents(activated) 解析完所有激活的异步组件后，我们就可以拿到这一次所有激活的组件。这样我们在做完这 5 步后又做了一些事情：

```
    runQueue(queue, iterator, () => {
      const postEnterCbs = []
      const isValid = () => this.current === route
      // wait until async components are resolved before
      // extracting in-component enter guards
      const enterGuards = extractEnterGuards(activated, postEnterCbs, isValid)
      const queue = enterGuards.concat(this.router.resolveHooks)
      runQueue(queue, iterator, () => {
        if (this.pending !== route) {
          return abort()
        }
        this.pending = null
        onComplete(route)
        if (this.router.app) {
          this.router.app.$nextTick(() => {
            postEnterCbs.forEach(cb => {
              cb()
            })
          })
        }
      })
    })
```

6. 在被激活的组件里调用 beforeRouteEnter。

7. 调用全局的 beforeResolve 守卫。

8. 调用全局的 afterEach 钩子。

对于第六步有这些相关的逻辑：

```
const postEnterCbs = []
const isValid = () => this.current === route
const enterGuards = extractEnterGuards(activated, postEnterCbs, isValid)

function extractEnterGuards (
  activated: Array<RouteRecord>,
  cbs: Array<Function>,
  isValid: () => boolean
): Array<?Function> {
  return extractGuards(activated, 'beforeRouteEnter', (guard, _, match, key) => {
    return bindEnterGuard(guard, match, key, cbs, isValid)
  })
}

function bindEnterGuard (
  guard: NavigationGuard,
  match: RouteRecord,
  key: string,
  cbs: Array<Function>,
  isValid: () => boolean
): NavigationGuard {
  return function routeEnterGuard (to, from, next) {
    return guard(to, from, cb => {
      next(cb)
      if (typeof cb === 'function') {
        cbs.push(() => {
          poll(cb, match.instances, key, isValid)
        })
      }
    })
  }
}

function poll (
  cb: any,
  instances: Object,
  key: string,
  isValid: () => boolean
) {
  if (instances[key]) {
    cb(instances[key])
  } else if (isValid()) {
    setTimeout(() => {
      poll(cb, instances, key, isValid)
    }, 16)
  }
}
```

extractEnterGuards 函数的实现也是利用了 extractGuards 方法提取组件中的 beforeRouteEnter 导航钩子函数，和之前不同的是 bind 方法的不同。文档中特意强调了 beforeRouteEnter 钩子函数中是拿不到组件实例的，因为当守卫执行前，组件实例还没被创建，但是我们可以通过传一个回调给 next 来访问组件实例。在导航被确认的时候执行回调，并且把组件实例作为回调方法的参数：

```
beforeRouteEnter (to, from, next) {
  next(vm => {
    // 通过 `vm` 访问组件实例
  })
}
```

来看一下这是怎么实现的。

在 bindEnterGuard 函数中，返回的是 routeEnterGuard 函数，所以在执行 iterator 中的 hook 函数的时候，就相当于执行 routeEnterGuard 函数，那么就会执行我们定义的导航守卫 guard 函数，并且当这个回调函数执行的时候，首先执行 next 函数 rersolve 当前导航钩子，然后把回调函数的参数，它也是一个回调函数用 cbs 收集起来，其实就是收集到外面定义的 postEnterCbs 中，然后在最后会执行：

```
if (this.router.app) {
  this.router.app.$nextTick(() => {
    postEnterCbs.forEach(cb => { cb() })
  })
}
```

在根路由组件重新渲染后，遍历 postEnterCbs 执行回调，每一个回调执行的时候，其实是执行 poll(cb, match.instances, key, isValid) 方法，因为考虑到一些了路由组件被套 transition 組件在一些缓动模式下不一定能拿到实例，所以用一个轮询方法不断去判断，直到能获取到组件实例，再去调用 cb，并把组件实例作为参数传入，这就是我们在回调函数中能拿到组件实例的原因。

第七步是获取 this.router.resolveHooks，这个和 this.router.beforeHooks 的获取类似，在我们的 VueRouter 类中定义了 beforeResolve 方法：

```
beforeResolve (fn: Function): Function {
  return registerHook(this.resolveHooks, fn)
}
```

当用户使用 router.beforeResolve 注册了一个全局守卫，就会往 router.resolveHooks 添加一个钩子函数，这样 this.router.resolveHooks 获取的就是用户注册的全局 beforeResolve 守卫。

第八步是在最后执行了 onComplete(route) 后，会执行 this.updateRoute(route) 方法：

```
updateRoute (route: Route) {
  const prev = this.current
  this.current = route
  this.cb && this.cb(route)
  this.router.afterHooks.forEach(hook => {
    hook && hook(route, prev)
  })
}
```

同样在我们的 VueRouter 类中定义了 afterEach 方法：

```
afterEach (fn: Function): Function {
  return registerHook(this.afterHooks, fn)
}
```

当用户使用 router.afterEach 注册了一个全局守卫，就会往 router.afterHooks 添加一个钩子函数，这样 this.router.afterHooks 获取的就是用户注册的全局 afterHooks 守卫。

那么至此我们把所有导航守卫的执行分析完毕了，我们知道路由切换除了执行这些钩子函数，从表象上有 2 个地方会发生变化，一个是 url 发生变化，一个是组件发生变化。接下来我们分别介绍这两块的实现原理。

#### url

当我们点击 router-link 的时候，实际上最终会执行 router.push，如下：

```
push (location: RawLocation, onComplete?: Function, onAbort?: Function) {
  this.history.push(location, onComplete, onAbort)
}
```

this.history.push 函数，这个函数是子类实现的，不同模式下该函数的实现略有不同，我们来看一下平时使用比较多的 hash 模式该函数的实现，在 src/history/hash.js 中：

```
push (location: RawLocation, onComplete?: Function, onAbort?: Function) {
  const { current: fromRoute } = this
  this.transitionTo(location, route => {
    pushHash(route.fullPath)
    handleScroll(this.router, route, fromRoute, false)
    onComplete && onComplete(route)
  }, onAbort)
}
```

push 函数会先执行 this.transitionTo 做路径切换，在切换完成的回调函数中，执行 pushHash 函数：

```
function pushHash (path) {
  if (supportsPushState) {
    pushState(getUrl(path))
  } else {
    window.location.hash = path
  }
}
```

supportsPushState 的定义在 src/util/push-state.js 中：

```
export const supportsPushState = inBrowser && (function () {
  const ua = window.navigator.userAgent

  if (
    (ua.indexOf('Android 2.') !== -1 || ua.indexOf('Android 4.0') !== -1) &&
    ua.indexOf('Mobile Safari') !== -1 &&
    ua.indexOf('Chrome') === -1 &&
    ua.indexOf('Windows Phone') === -1
  ) {
    return false
  }

  return window.history && 'pushState' in window.history
})()
```

如果支持的话，则获取当前完整的 url，执行 pushState 方法：

```
export function pushState (url?: string, replace?: boolean) {
  saveScrollPosition()
  const history = window.history
  try {
    if (replace) {
      history.replaceState({ key: _key }, '', url)
    } else {
      _key = genKey()
      history.pushState({ key: _key }, '', url)
    }
  } catch (e) {
    window.location[replace ? 'replace' : 'assign'](url)
  }
}
```

pushState 会调用浏览器原生的 history 的 pushState 接口或者 replaceState 接口，更新浏览器的 url 地址，并把当前 url 压入历史栈中。

然后在 history 的初始化中，会设置一个监听器，监听历史栈的变化：

```
setupListeners () {
  const router = this.router
  const expectScroll = router.options.scrollBehavior
  const supportsScroll = supportsPushState && expectScroll

  if (supportsScroll) {
    setupScroll()
  }

  window.addEventListener(supportsPushState ? 'popstate' : 'hashchange', () => {
    const current = this.current
    if (!ensureSlash()) {
      return
    }
    this.transitionTo(getHash(), route => {
      if (supportsScroll) {
        handleScroll(this.router, route, current, true)
      }
      if (!supportsPushState) {
        replaceHash(route.fullPath)
      }
    })
  })
}
```

当点击浏览器返回按钮的时候，如果已经有 url 被压入历史栈，则会触发 popstate 事件，然后拿到当前要跳转的 hash，执行 transtionTo 方法做一次路径转换。

同学们在使用 Vue-Router 开发项目的时候，打开调试页面 `http://localhost:8080` 后会自动把 url 修改为 `http://localhost:8080/#/`，这是怎么做到呢？原来在实例化 HashHistory 的时候，构造函数会执行 ensureSlash() 方法：

```
function ensureSlash (): boolean {
  const path = getHash()
  if (path.charAt(0) === '/') {
    return true
  }
  replaceHash('/' + path)
  return false
}

export function getHash (): string {
  // We can't use window.location.hash here because it's not
  // consistent across browsers - Firefox will pre-decode it!
  const href = window.location.href
  const index = href.indexOf('#')
  return index === -1 ? '' : href.slice(index + 1)
}

function getUrl (path) {
  const href = window.location.href
  const i = href.indexOf('#')
  const base = i >= 0 ? href.slice(0, i) : href
  return `${base}#${path}`
}

function replaceHash (path) {
  if (supportsPushState) {
    replaceState(getUrl(path))
  } else {
    window.location.replace(getUrl(path))
  }
}

export function replaceState (url?: string) {
  pushState(url, true)
}
```

这个时候 path 为空，所以执行 replaceHash('/' + path)，然后内部会执行一次 getUrl，计算出来的新的 url 为 `http://localhost:8080/#/`，最终会执行 pushState(url, true)，这就是 url 会改变的原因。

#### 组件

路由最终的渲染离不开组件，Vue-Router 内置了 `<router-view>` 组件，它的定义在 src/components/view.js 中。

```
export default {
  name: 'RouterView',
  functional: true,
  props: {
    name: {
      type: String,
      default: 'default'
    }
  },
  render (_, { props, children, parent, data }) {
    // used by devtools to display a router-view badge
    data.routerView = true

    // directly use parent context's createElement() function
    // so that components rendered by router-view can resolve named slots
    const h = parent.$createElement
    const name = props.name
    const route = parent.$route
    const cache = parent._routerViewCache || (parent._routerViewCache = {})

    // determine current view depth, also check to see if the tree
    // has been toggled inactive but kept-alive.
    let depth = 0
    let inactive = false
    while (parent && parent._routerRoot !== parent) {
      const vnodeData = parent.$vnode && parent.$vnode.data
      if (vnodeData) {
        if (vnodeData.routerView) {
          depth++
        }
        if (vnodeData.keepAlive && parent._inactive) {
          inactive = true
        }
      }
      parent = parent.$parent
    }
    data.routerViewDepth = depth

    // render previous view if the tree is inactive and kept-alive
    if (inactive) {
      return h(cache[name], data, children)
    }

    const matched = route.matched[depth]
    // render empty node if no matched route
    if (!matched) {
      cache[name] = null
      return h()
    }

    const component = cache[name] = matched.components[name]

    // attach instance registration hook
    // this will be called in the instance's injected lifecycle hooks
    data.registerRouteInstance = (vm, val) => {
      // val could be undefined for unregistration
      const current = matched.instances[name]
      if (
        (val && current !== vm) ||
        (!val && current === vm)
      ) {
        matched.instances[name] = val
      }
    }

    // also register instance in prepatch hook
    // in case the same component instance is reused across different routes
    ;(data.hook || (data.hook = {})).prepatch = (_, vnode) => {
      matched.instances[name] = vnode.componentInstance
    }

    // register instance in init hook
    // in case kept-alive component be actived when routes changed
    data.hook.init = (vnode) => {
      if (vnode.data.keepAlive &&
        vnode.componentInstance &&
        vnode.componentInstance !== matched.instances[name]
      ) {
        matched.instances[name] = vnode.componentInstance
      }
    }

    // resolve props
    let propsToPass = data.props = resolveProps(route, matched.props && matched.props[name])
    if (propsToPass) {
      // clone to prevent mutation
      propsToPass = data.props = extend({}, propsToPass)
      // pass non-declared props as attrs
      const attrs = data.attrs = data.attrs || {}
      for (const key in propsToPass) {
        if (!component.props || !(key in component.props)) {
          attrs[key] = propsToPass[key]
          delete propsToPass[key]
        }
      }
    }

    return h(component, data, children)
  }
}
```

`<router-view>` 是一个 functional 组件，它的渲染也是依赖 render 函数，那么 `<router-view>` 具体应该渲染什么组件呢，首先获取当前的路径：

```
const route = parent.$route
```

我们之前分析过，在 src/install.js 中，我们给 Vue 的原型上定义了 `$route`：

```
Object.defineProperty(Vue.prototype, '$route', {
  get () { return this._routerRoot._route }
})
```

然后在 VueRouter 的实例执行 router.init 方法的时候，会执行如下逻辑，定义在 src/index.js 中：

```
history.listen(route => {
  this.apps.forEach((app) => {
    app._route = route
  })
})
```

而 history.listen 方法定义在 src/history/base.js 中：

```
listen (cb: Function) {
  this.cb = cb
}
```

然后在 updateRoute 的时候执行 this.cb：

```
updateRoute (route: Route) {
  //. ..
  this.current = route
  this.cb && this.cb(route)
  // ...
}
```

也就是我们执行 transitionTo 方法最后执行 updateRoute 的时候会执行回调，然后会更新 this.apps 保存的组件实例的 `_route` 值，this.apps 数组保存的实例的特点都是在初始化的时候传入了 router 配置项，一般的场景数组只会保存根 Vue 实例，因为我们是在 new Vue 传入了 router 实例。`$route` 是定义在 Vue.prototype 上。每个组件实例访问 `$route` 属性，就是访问根实例的 `_route`，也就是当前的路由线路。

`<router-view>` 是支持嵌套的，回到 render 函数，其中定义了 depth 的概念，它表示 `<router-view>` 嵌套的深度。每个 `<router-view>` 在渲染的时候，执行如下逻辑：

```
data.routerView = true
// ...
while (parent && parent._routerRoot !== parent) {
  if (parent.$vnode && parent.$vnode.data.routerView) {
    depth++
  }
  if (parent._inactive) {
    inactive = true
  }
  parent = parent.$parent
}

const matched = route.matched[depth]
// ...
const component = cache[name] = matched.components[name]
```

`parent._routerRoot` 表示的是根 Vue 实例，那么这个循环就是从当前的 `<router-view>` 的父节点向上找，一直找到根 Vue 实例，在这个过程，如果碰到了父节点也是 `<router-view>` 的时候，说明 `<router-view>` 有嵌套的情况，depth++。遍历完成后，根据当前线路匹配的路径和 depth 找到对应的 RouteRecord，进而找到该渲染的组件。

除了找到了应该渲染的组件，还定义了一个注册路由实例的方法：

```
data.registerRouteInstance = (vm, val) => {
  const current = matched.instances[name]
  if (
    (val && current !== vm) ||
    (!val && current === vm)
  ) {
    matched.instances[name] = val
  }
}
```

给 vnode 的 data 定义了 registerRouteInstance 方法，在 src/install.js 中，我们会调用该方法去注册路由的实例：

```
const registerInstance = (vm, callVal) => {
  let i = vm.$options._parentVnode
  if (isDef(i) && isDef(i = i.data) && isDef(i = i.registerRouteInstance)) {
    i(vm, callVal)
  }
}

Vue.mixin({
  beforeCreate () {
    // ...
    registerInstance(this, this)
  },
  destroyed () {
    registerInstance(this)
  }
})
```

在混入的 beforeCreate 钩子函数中，会执行 registerInstance 方法，进而执行 render 函数中定义的 registerRouteInstance 方法，从而给 `matched.instances[name]` 赋值当前组件的 vm 实例。

render 函数的最后根据 component 渲染出对应的组件 vonde：

```
return h(component, data, children)
```

那么当我们执行 transitionTo 来更改路由线路后，组件是如何重新渲染的呢？在我们混入的 beforeCreate 钩子函数中有这么一段逻辑：

```
Vue.mixin({
  beforeCreate () {
    if (isDef(this.$options.router)) {
      Vue.util.defineReactive(this, '_route', this._router.history.current)
    }
    // ...
  }
})
```

由于我们把根 Vue 实例的 \_route 属性定义成响应式的，我们在每个 `<router-view>` 执行 render 函数的时候，都会访问 parent.\$route，如我们之前分析会访问 `this._routerRoot._route`，触发了它的 getter，相当于 `<router-view>` 对它有依赖，然后再执行完 transitionTo 后，修改 `app._route` 的时候，又触发了 setter，因此会通知 `<router-view>` 的渲染 watcher 更新，重新渲染组件。

Vue-Router 还内置了另一个组件 `<router-link>`， 它支持用户在具有路由功能的应用中（点击）导航。 通过 to 属性指定目标地址，默认渲染成带有正确链接的 `<a>` 标签，可以通过配置 tag 属性生成别的标签。另外，当目标路由成功激活时，链接元素自动设置一个表示激活的 CSS 类名。

`<router-link>` 比起写死的 `<a href="...">` 会好一些，理由如下：

无论是 HTML5 history 模式还是 hash 模式，它的表现行为一致，所以，当你要切换路由模式，或者在 IE9 降级使用 hash 模式，无须作任何变动。

在 HTML5 history 模式下，router-link 会守卫点击事件，让浏览器不再重新加载页面。

当你在 HTML5 history 模式下使用 base 选项之后，所有的 to 属性都不需要写（基路径）了。

那么接下来我们就来分析它的实现，它的定义在 src/components/link.js 中：

```
export default {
  name: 'RouterLink',
  props: {
    to: {
      type: toTypes,
      required: true
    },
    tag: {
      type: String,
      default: 'a'
    },
    exact: Boolean,
    append: Boolean,
    replace: Boolean,
    activeClass: String,
    exactActiveClass: String,
    event: {
      type: eventTypes,
      default: 'click'
    }
  },
  render (h: Function) {
    const router = this.$router
    const current = this.$route
    const { location, route, href } = router.resolve(
      this.to,
      current,
      this.append
    )

    const classes = {}
    const globalActiveClass = router.options.linkActiveClass
    const globalExactActiveClass = router.options.linkExactActiveClass
    // Support global empty active class
    const activeClassFallback =
      globalActiveClass == null ? 'router-link-active' : globalActiveClass
    const exactActiveClassFallback =
      globalExactActiveClass == null
        ? 'router-link-exact-active'
        : globalExactActiveClass
    const activeClass =
      this.activeClass == null ? activeClassFallback : this.activeClass
    const exactActiveClass =
      this.exactActiveClass == null
        ? exactActiveClassFallback
        : this.exactActiveClass

    const compareTarget = route.redirectedFrom
      ? createRoute(null, normalizeLocation(route.redirectedFrom), null, router)
      : route

    classes[exactActiveClass] = isSameRoute(current, compareTarget)
    classes[activeClass] = this.exact
      ? classes[exactActiveClass]
      : isIncludedRoute(current, compareTarget)

    const handler = e => {
      if (guardEvent(e)) {
        if (this.replace) {
          router.replace(location, noop)
        } else {
          router.push(location, noop)
        }
      }
    }

    const on = { click: guardEvent }
    if (Array.isArray(this.event)) {
      this.event.forEach(e => {
        on[e] = handler
      })
    } else {
      on[this.event] = handler
    }

    const data: any = { class: classes }

    const scopedSlot =
      !this.$scopedSlots.$hasNormal &&
      this.$scopedSlots.default &&
      this.$scopedSlots.default({
        href,
        route,
        navigate: handler,
        isActive: classes[activeClass],
        isExactActive: classes[exactActiveClass]
      })

    if (scopedSlot) {
      if (scopedSlot.length === 1) {
        return scopedSlot[0]
      } else if (scopedSlot.length > 1 || !scopedSlot.length) {
        if (process.env.NODE_ENV !== 'production') {
          warn(
            false,
            `RouterLink with to="${
              this.props.to
            }" is trying to use a scoped slot but it didn't provide exactly one child.`
          )
        }
        return scopedSlot.length === 0 ? h() : h('span', {}, scopedSlot)
      }
    }

    if (this.tag === 'a') {
      data.on = on
      data.attrs = { href }
    } else {
      // find the first <a> child and apply listener and href
      const a = findAnchor(this.$slots.default)
      if (a) {
        // in case the <a> is a static node
        a.isStatic = false
        const aData = (a.data = extend({}, a.data))
        aData.on = on
        const aAttrs = (a.data.attrs = extend({}, a.data.attrs))
        aAttrs.href = href
      } else {
        // doesn't have <a> child, apply listener to self
        data.on = on
      }
    }

    return h(this.tag, data, this.$slots.default)
  }
}
```

`<router-link>` 标签的渲染也是基于 render 函数，它首先做了路由解析：

```
const router = this.$router
const current = this.$route
const { location, route, href } = router.resolve(this.to, current, this.append)
```

router.resolve 是 VueRouter 的实例方法，它的定义在 src/index.js 中：

```
resolve (
  to: RawLocation,
  current?: Route,
  append?: boolean
): {
  location: Location,
  route: Route,
  href: string,
  normalizedTo: Location,
  resolved: Route
} {
  const location = normalizeLocation(
    to,
    current || this.history.current,
    append,
    this
  )
  const route = this.match(location, current)
  const fullPath = route.redirectedFrom || route.fullPath
  const base = this.history.base
  const href = createHref(base, fullPath, this.mode)
  return {
    location,
    route,
    href,
    normalizedTo: location,
    resolved: route
  }
}

function createHref (base: string, fullPath: string, mode) {
  var path = mode === 'hash' ? '#' + fullPath : fullPath
  return base ? cleanPath(base + '/' + path) : path
}
```

它先规范生成目标 location，再根据 location 和 match 通过 this.match 方法计算生成目标路径 route，然后再根据 base、fullPath 和 this.mode 通过 createHref 方法计算出最终跳转的 href。

解析完 router 获得目标 location、route、href 后，接下来对 exactActiveClass 和 activeClass 做处理，当配置 exact 为 true 的时候，只有当目标路径和当前路径完全匹配的时候，会添加 exactActiveClass；而当目标路径包含当前路径的时候，会添加 activeClass。

接着创建了一个守卫函数 ：

```
const handler = e => {
  if (guardEvent(e)) {
    if (this.replace) {
      router.replace(location)
    } else {
      router.push(location)
    }
  }
}

function guardEvent (e) {
  if (e.metaKey || e.altKey || e.ctrlKey || e.shiftKey) return
  if (e.defaultPrevented) return
  if (e.button !== undefined && e.button !== 0) return
  if (e.currentTarget && e.currentTarget.getAttribute) {
    const target = e.currentTarget.getAttribute('target')
    if (/\b_blank\b/i.test(target)) return
  }
  if (e.preventDefault) {
    e.preventDefault()
  }
  return true
}

const on = { click: guardEvent }
  if (Array.isArray(this.event)) {
    this.event.forEach(e => { on[e] = handler })
  } else {
    on[this.event] = handler
  }
```

最终会监听点击事件或者其它可以通过 prop 传入的事件类型，执行 hanlder 函数，最终执行 router.push 或者 router.replace 函数，它们的定义在 src/index.js 中：

```
push (location: RawLocation, onComplete?: Function, onAbort?: Function) {
  this.history.push(location, onComplete, onAbort)
}

replace (location: RawLocation, onComplete?: Function, onAbort?: Function) {
 this.history.replace(location, onComplete, onAbort)
}
```

实际上就是执行了 history 的 push 和 replace 方法做路由跳转。

最后判断当前 tag 是否是 `<a>` 标签，`<router-link>` 默认会渲染成 `<a>` 标签，当然我们也可以修改 tag 的 prop 渲染成其他节点，这种情况下会尝试找它子元素的 `<a>` 标签，如果有则把事件绑定到 `<a>` 标签上并添加 href 属性，否则绑定到外层元素本身。

#### 总结

那么至此我们把路由的 transitionTo 的主体过程分析完毕了，其他一些分支比如重定向、别名、滚动行为等同学们可以自行再去分析。

路径变化是路由中最重要的功能，我们要记住以下内容：路由始终会维护当前的线路，路由切换的时候会把当前线路切换到目标线路，切换过程中会执行一系列的导航守卫钩子函数，会更改 url，同样也会渲染对应的组件，切换完毕后会把目标线路更新替换当前线路，这样就会作为下一次的路径切换的依据。
