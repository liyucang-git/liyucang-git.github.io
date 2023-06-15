---
layout: post
title: JavaScript库打包指南
subtitle: 搞懂CJS、UMD、ESM
date: 2023-06-15
author: Li Yucang
catalog: true
tags:
  - npm包
  - js库
---

# JavaScript库打包指南

本指南旨在提供一些大多数库都应该遵循的一目了然的建议。以及一些额外的信息，用来帮助你了解这些建议被提出的原因，或帮助你判断是否不需要遵循某些建议。这个指南仅适用于 库（libraries），不适用于应用（app）。

要强调的是，这只是一些建议，并不是所有库都必须要遵循的。每个库都是独特的，它们可能有充足的理由不采用本文中的任何建议。

最后，这个指南不针对某一个特定的打包工具 —— 已经有许多指南来说明如何在配置特定的打包工具。相反我们聚焦于每个库和打包工具（或不用打包工具）都适用的事项。

## 关于模块化设计

**什么是模块化设计**

>模块化设计（Modular design），是一种将系统分解为更小的“模块”的生产方式。
这一思想广泛运用于机械制造、电子和软件工业中。

**代码的模块化**

常见的生产级编程语言都支持模块化,如 C++、Java、Python、PHP、JS 中都有 import 或 include 保留字。通常以单个文件作为模块的最小单元。

代码的模块化设计一般可抽象为三个部分：

* 输入（import）

* 计算（业务代码）

* 输出（export）

**为什么要用模块化**

* 把复杂问题分解成多个子问题

  * 关注点分离

* 大型软件开发的技术基础

  * 可扩展

  * 可替换

  * 代码重用

* 使多人并行开发成为可能

  * 面向接口开发（而不是面向实现开发）

**JS 中的模块化方案**

JS 的模块化经历了各种历史时期，在不同时期产生了不同的模块化方案。
到目前为止，对于编写源码来说，主流的方案只剩下两种。

* esm: 从 ES6 起官方规范自带的方案

* cjs: Node.js 使用的方案

但是为了支持不同的目标运行环境，需要编译成不同的输出格式（方案），了解不同的模块化方案是很有必要的。

流行打包工具 [Rollup.js](https://rollupjs.org/configuration-options/#output-format) 和 [Webpack](https://webpack.js.org/configuration/output/#outputlibrarytarget)都支持导出格式功能。

以 Rollup 文档为例子，一共有以下几种：

* cjs (CommonJS) — 适用于 Node 和其他打包工具（别名：commonjs）。

* amd (Asynchronous Module Definition，异步模块化定义) — 与 RequireJS 等模块加载工具一起使用。

* umd (Universal Module Definition，通用模块化定义) — amd，cjs 和 iife 包含在一个文件中。

* es — 将 bundle 保存为 ES 模块文件。适用于其他打包工具，在现代浏览器中用 `<script type=module>` 标签引入（别名：ems, module）。

* system — SystemJS 加载器的原生格式 （别名：systemjs）。

* iife — `<script>` 标签引入的自执行函数。如果你想为你的应用创建一个包，你需要用到的可能就是这种。

**代码分发**

上文提到，模块化的好处之一是具有可重用性，那么重用就会涉及到代码分发。

自己写的业务代码的本地源码中，模块关系很容易理解，但是注意也有其他的调用方式，比如 npm 和 CDN 分发。

相关的有一些工具和平台：

* [npm](https://www.npmjs.com/)

* [jsDelivr](https://www.jsdelivr.com/)

* [UNPKG](https://unpkg.com/)

* [BootCDN](https://www.bootcdn.cn/)

## 打包格式比较

可以直接使用 [Rollup](https://rollupjs.org/repl/?version=3.25.1&shareable=JTdCJTIyZXhhbXBsZSUyMiUzQW51bGwlMkMlMjJtb2R1bGVzJTIyJTNBJTVCJTdCJTIyY29kZSUyMiUzQSUyMmltcG9ydCUyMCU3QiUyMGV2ZXJ5JTIwJTdEJTIwZnJvbSUyMCdsb2Rhc2gnJTNCJTVDbiU1Q25jb25zdCUyMHJlc3VsdCUyMCUzRCUyMGV2ZXJ5KCU1QnRydWUlMkMlMjAxJTJDJTIwbnVsbCUyQyUyMCd5ZXMnJTVEJTJDJTIwQm9vbGVhbiklM0IlNUNuJTVDbmV4cG9ydCUyMGRlZmF1bHQlMjByZXN1bHQlM0IlMjIlMkMlMjJpc0VudHJ5JTIyJTNBdHJ1ZSUyQyUyMm5hbWUlMjIlM0ElMjJtYWluLmpzJTIyJTdEJTVEJTJDJTIyb3B0aW9ucyUyMiUzQSU3QiUyMm91dHB1dCUyMiUzQSU3QiUyMmZvcm1hdCUyMiUzQSUyMmVzJTIyJTdEJTJDJTIydHJlZXNoYWtlJTIyJTNBZmFsc2UlN0QlN0Q=) 来理解不同模块化方案在用法上的异同，下面是源代码：

```
import { every } from 'lodash';

const result = every([true, 1, null, 'yes'], Boolean);

export default result;
```

### IIFE

IIFE 是前端模块化早期的产物，它的核心思路是:

1、构建一个匿名函数

2、立刻执行这个匿名函数，对外部的依赖通过入参的形式传入

3、返回该模块的输出

下面是 Rollup 生成的 IIFE 格式的文件：

```
var myBundle = (function (lodash) {
	'use strict';

	const result = lodash.every([true, 1, null, 'yes'], Boolean);

	return result;

})(lodash);
```

**如何运行**

IIFE 的运行其实很容易，如果它没有其他依赖，只需要去引入文件，然后在 window 上取相应的变量即可。

如：

```
<script src="http://cdn.bootcss.com/jquery/3.3.1/jquery.min.js"></script>
<script> 
  // jquery 就是典型的自执行函数模式，当你引入后，他就会挂在到 window.$ 上
  window.$ // 这样就能取到 jquery 了
</script>
```

但是如果你像本 demo 中那样依赖了其他的模块，那你就必须保证以下两点才能正常运行：

1、此包所依赖的包，已在此包之前完成加载。

2、前置依赖的包，和 IIFE 只执行入参的变量命名是一致的。

以本 demo 的 IIFE 构建结果为例：

1、它前置依赖了 lodash，因此需要在它加载之前完成 lodash 的加载。

2、此 IIFE 的第二个入参是 lodash，作为前置条件，我们需要让 window.lodash 也指向 lodash。

因此，运行时，代码如下：

```
<head>
  <script src="https://cdn.bootcdn.net/ajax/libs/lodash.js/4.17.21/lodash.min.js"></script>
  <script>window.lodash = window._</script>  
  <script src="./bundle.js"></script>
</head>
<body>
  <script>
    console.log(window.myBundle)
  </script>
</body>
```

**优缺点**

优点:

* 通过闭包营造了一个“私有”命名空间，防止影响全局，并防止被从外部修改私有变量。

* 简单易懂

* 对代码体积的影响不大

缺点：

* 输出的变量可能影响全局变量；引入依赖包时依赖全局变量。

* 需要使用者自行维护 script 标签的加载顺序。

优点就不细说了，缺点详细解释一下。

缺点一：输出的变量可能影响全局变量；引入依赖包时依赖全局变量。

前半句：输出的变量可能影响全局变量; 其实很好理解，以上面 demo 的输出为例： window.Test 就已经被影响了。

这种明显的副作用在程序中其实是有隐患的。

后半句：引入依赖包时依赖全局变量； 我们为了让 demo 正常运行，因此加了一行代码让 window.lodash 也指向 lodash，但它确实是太脆弱了。

```
<!-- 没有这一行，demo就无法正常运行 -->
<script>window.lodash = window._</script>
```

你瞧，IIFE 的执行对环境的依赖是苛刻的，除非它完全不依赖外部包。（Jquery: 正是在下！）

虽然 IIFE 的缺点很多，但并不妨碍它在 Jquery 时代极大地推动了 web 开发的进程，因为它确实解决了 js 本身存在的很多问题。

那么？后续是否还有 更为优秀 的前端模块化方案问世呢？

当然有，往下看吧。

### CJS (CommonJS)

CJS 适用于浏览器之外的 Node 和其他生态系统。它在服务端被广泛使用。CJS 可以通过使用 require() 函数和 module.exports 来识别。require() 是一个可用于从另一个模块导入 symbols 到当前作用域的函数。 module.exports 是当前模块在另一个模块中引入时返回的对象。

CJS 模块的设计考虑到了服务器开发。这个 API 天生是同步的。换言之，在源文件中按 require 的顺序瞬时加载模块。

由于 CJS 是同步的且不能被浏览器识别，CJS 模块不能在浏览器端使用，除非它被转译器打包。像 Babel 和 Traceur 那样的转译器，是一种帮助我们在新版 JavaScript 中编码的工具。如果环境原生不支持新版本的 JavaScript，转译器将它们编译成支持的 JS 版本。

下面是 Rollup 生成的 CJS 格式的文件：

```
'use strict';

var lodash = require('lodash');

const result = lodash.every([true, 1, null, 'yes'], Boolean);

module.exports = result;
```

**特点**

* 由 Node.js 实现

* 多用在服务器端安装模块时

* 没有 runtime/async 模块

* 通过  require  导入模块

* 通过  module.exports  导出模块

* 无法使用摇树优化，因为当你导入时会得到一个模块时，得到的是一个对象，所以属性查找在运行时进行，无法静态分析

* 会得到一个对象的副本，因此模块本身不会实时更改

* 循环依赖的不能优雅处理

* 语法简单

**如何运行**

```
// run.js
const MyBundle = require('./bundle')
console.log(MyBundle)

# 执行脚本
node run.js 
```

可以看出，node.js 环境是天然支持 CommonJS 的。

**优缺点**

优点：

* 完善的模块化方案，完美解决了 IIFE 的各种缺点。

缺点：

* 不支持浏览器，执行后才能拿到依赖信息，由于用户可以动态 require（例如 react 根据开发和生产环境导出不同代码 的写法），无法做到提前分析依赖以及 Tree-Shaking 。

CommonJS 采用同步加载模块，而加载的文件资源大多数在本地服务器，所以执行速度或时间没问题。但是在浏览器端，限于网络原因，更合理的方案是使用异步加载。

### AMD

AMD 是"Asynchronous Module Definition"的缩写，意思就是"异步模块定义"。它采用异步方式加载模块，模块的加载不影响它后面语句的运行。所有依赖这个模块的语句，都定义在一个回调函数中，等到加载完成之后，这个回调函数才会运行。其中 RequireJS 是最佳实践者。

模块功能主要的几个命令：define、require、return和define.amd。define是全局函数，用来定义模块,define(id?, dependencies?, factory)。require 命令用于输入其他模块提供的功能，return 命令用于规范模块的对外接口，define.amd 属性是一个对象，此属性的存在来表明函数遵循 AMD 规范。

下面是 Rollup 生成的 AMD 格式的文件：

```
define(['lodash'], (function (lodash) { 'use strict';

	const result = lodash.every([true, 1, null, 'yes'], Boolean);

	return result;

}));
```

**特点**

* 由 RequireJs 实现

* 当你在客户端（浏览器）环境中，异步加载模块时使用

* 通过  require  实现导入

* 语法复杂

**如何运行**

```
<head>
  <!-- 1. 引入 require.js -->
  <script src="./require.js"></script>
  <!-- 2. 定义全局依赖 -->
  <script>
    window.requirejs.config({
      paths: {
        "lodash": "https://cdn.bootcdn.net/ajax/libs/lodash.js/4.17.21/lodash.min"
      }
    });
  </script>
  <!-- 3. 定义模块 -->
  <script src="./bundle.js"></script>
</head>
<body>
  <script>
    // 4. 开销模块
    window.requirejs(
      ['bundle'],
      function (myBundle) {
        console.log(myBundle)
      }
    );
  </script>
</body>
```

**优缺点**

优点：

* 解决了 CommonJS 的缺点

* 解决了 IIFE 的缺点

* 一套完备的浏览器里 js 文件模块化方案

缺点：

* 代码组织形式别扭，可读性差

但好在我们拥有了各类打包工具，浏览器内的代码可读性再差也并不影响我们写出可读性ok的代码。

现在，我们拥有了面向 node.js 的 CommonJs 和 面向浏览器的 AMD 两套标准。

如果我希望我写出的代码能同时被浏览器和nodejs识别，我应该怎么做呢？

### UMD

UMD(Universal Module Definition - 通用模块定义)模式，该模式主要用来解决 CommonJS 模式和 AMD 模式代码不能通用的问题，并同时还支持老式的全局变量规范。

1、判断define为函数，并且是否存在define.amd，来判断是否为 AMD 规范,

2、判断module是否为一个对象，并且是否存在module.exports来判断是否为CommonJS规范

3、如果以上两种都没有，设定为原始的全局变量规范。

下面是 Rollup 生成的 UMD 格式的文件：

```
(function (global, factory) {
	typeof exports === 'object' && typeof module !== 'undefined' ? module.exports = factory(require('lodash')) :
	typeof define === 'function' && define.amd ? define(['lodash'], factory) :
	(global = typeof globalThis !== 'undefined' ? globalThis : global || self, global.myBundle = factory(global.lodash));
})(this, (function (lodash) { 'use strict';

	const result = lodash.every([true, 1, null, 'yes'], Boolean);

	return result;

}));
```

**特点**

* CommonJs + AMD 的组合（即 CommonJs 的语法 + AMD 的异步加载）

* 可以用于 AMD/CommonJs 环境

* UMD 还支持全局变量定义，因此，UMD 模块能够在客户端和服务器上工作。

**如何运行**

在浏览器端，它的运行方式和 amd 完全一致。

在node.js端，它则和 CommonJS 的运行方式完全一致，在此就不赘述了。

**优缺点**

优点：

* 抹平了一个包在 AMD 和 CommonJS 里的差异

缺点：

* 会为了兼容产生大量不好理解的代码。（理解难度与包体积）

虽然在社区的不断努力下，CommonJS 、 AMD 、 UMD 都给业界交出了自己的答卷。

但很显然，它们都是不得已的选择。

浏览器应该有自己的加载标准。

ES6 草案里，虽然描述了模块应该如何被加载，但它没有 “加载程序的规范”。

### system

SystemJs 是一个通用的模块加载器，支持 CJS，AMD 和 ESM 模块。Rollup 可以将代码打包成 SystemJS 的原生格式。

下面是 Rollup 生成的 System 格式的文件：

```
System.register(['lodash'], (function (exports) {
	'use strict';
	var every;
	return {
		setters: [function (module) {
			every = module.every;
		}],
		execute: (function () {

			const result = exports('default', every([true, 1, null, 'yes'], Boolean));

		})
	};
}));
```

用的比较少，这里仅介绍。

### ESM (ES Module)

ES modules（ESM）是 JavaScript 官方的标准化模块系统。

1、它因为是标准，所以未来很多浏览器会支持，可以很方便的在浏览器中使用。(浏览器默认加载不能省略.js)

2、它同时兼容在 node 环境下运行。

3、模块的导入导出，通过import和export来确定。 可以和 Commonjs 模块混合使用。

4、ES modules 输出的是值的引用，输出接口动态绑定，而 CommonJS 输出的是值的拷贝

5、ES modules 模块编译时执行，而 CommonJS 模块总是在运行时加载

下面是 Rollup 生成的 ESM 格式的文件：

```
import { every } from 'lodash';

const result = every([true, 1, null, 'yes'], Boolean);

export { result as default };
```

**特点**

* 用于服务器/客户端

* 支持模块的  Runtime/static loading

* 当你导入时，获得是实际对象

* 通过  import  导入，通过  export  导出

* 静态分析——你可以决定编译时的导入和导出（静态），你只需要看源码，不需要执行它

* 由于 ES6 支持静态分析  ，因此摇树优化是可行的

* 始终获取实际值 ，以便实时更改模块本身

* 比 CommonJS 有更好的循环依赖管理

**如何运行**

部分现代浏览器已经开始实装 `<script type="module>`了，因此在浏览器上直接使用 esm 已成为现实。

```
<script src="./bundle.js" type="module"></script>
```

### 总结

分别适合在什么场景使用？

* IIFE: 适合部分场景作为SDK进行使用，尤其是需要把自己挂到 window 上的场景。

* CommonJS: 仅node.js使用的库。

* AMD: 只需要在浏览器端使用的场景。

* UMD: 既可能在浏览器端也可能在node.js里使用的场景。

* SystemJs: 和UMD类似。目前较出名的 Angular 用的就是它。

* ESM: 1. 还会被引用、二次编译的场景（如组件库等）；2.浏览器调试场景如 vite.js的开发时。3.对浏览器兼容性非常宽松的场景。

esm 被认为是“未来”，但 cjs 仍然在社区和生态系统中占有重要地位。esm 对打包工具来说更容易正确地进行 treeshaking，因此对于库来说，拥有这种格式很重要。或许在将来的某一天，你的库只需要输出 esm。

## 打包最佳实践

### Tree shaking

* Tree shaking是一个术语，通常用于描述移除 JavaScript 上下文中的未引用代码(dead-code)。它依赖于 ES2015 模块系统中的静态结构特性，例如  import  和  export。这个术语和概念实际上是兴起于 ES2015 模块打包工具  rollup。新的 webpack 4 正式版本，扩展了这个检测能力，通过  package.json  的  "sideEffects"  属性作为标记，向 compiler 提供提示，表明项目中的哪些文件是纯的 ES2015 模块，由此可以安全地删除文件中未使用的部分。

* webpack 和 Rollup 都支持摇树优化，这意味着我们需要牢记某些事情，以便我们的代码可被 Tree Shaking。

tree shaking 的实际例子

```
// main.js
import * as utils from "./utils";

const array = [1, 2, 3, 1, 2, 3];

console.log(utils.arrayUnique(array));
```

没有 Tree-shaking 的情况下，会将 utils 中的所有文件都进行打包，使得体积暴增。

ES Modules 之所以能 Tree-shaking 主要为以下四个原因:

1、import 只能作为模块顶层的语句出现，不能出现在 function 里面或是 if 里面。

2、import 的模块名只能是字符串常量。

3、不管 import 的语句出现的位置在哪里，在模块初始化的时候所有的 import 都必须已经导入完成。

4、import binding 是 immutable 的，类似 const。比如说你不能 import { a } from ‘./a’ 然后给 a 赋值个其他什么东西。

**tree shaking 应该注意什么**

没错，就是副作用，那么什么是副作用呢，请看下面的例子。

```
// effect.js
console.log(unused());
export function unused() {
  console.log(1);
}
```

```
// index.js
import { unused } from "./effect";
console.log(42);
```

此例子中 console.log(unused()); 就是副作用。在 index.js 中并不需要这一句 console.log。而 rollup 并不知道这个全局的函数去除是否安全。因此在打包地时候你可以显示地指定treeshake.moduleSideEffects 为 false，可以显示地告诉 rollup 外部依赖项没有其他副作用。

不指定的情况下的打包输出。 npx rollup index.js --file bundle.js

```
console.log(unused());

function unused() {
  console.log(1);
}

console.log(42);
```

指定没有副作用下的打包输出。npx rollup index.js --file bundle-no-effect.js --no-treeshake.moduleSideEffects

```
console.log(42);
```

当然以上只是副作用的一种，详情其他几种看查看 https://rollupjs.org/guide/en/

### 发布所有模块形态

* 我们应该发布所有模块形态，例如 UMD 和 ES Module ，因为我们永远不知道用户在哪个版本的浏览器或 webpack 中使用此库/包。

* 即使所有打包程序（如 webpack 和 Rollup）都能解析 ES Module ，但如果我们的使用者使用的是 webpack 1.x，则它无法解析 ES 模块。

使用现代的新特性，如果有需要，让开发者支持旧的浏览器：

1、当使用你的库时，能够让开发者去支持老版本的浏览器。

2、输出多个产出来支持不同版本的浏览器。

举个例子，如果你使用 TypeScript，你可以创建两个版本的包代码：

1、通过在 tsconfig.json 中设置 "target"="esnext"，生成一个用现代 JavaScript 的 esm 版本

2、通过在 tsconfig.json 中设置 "target"="es5" 生成一个兼容低版本 JavaScript 的 umd 版本

有了这些设置，大多数用户将获得现代版本的代码，但那些使用老的打包工具配置或使用 `<script> `加载代码的用户，将获得进行了额外编译来支持老版本浏览器的版本。

>鲜为人知的事实：webpack 使用 resolve.mainfields 确定检查  package.json  中的哪些字段。

>性能提示：由于所有现代浏览器现在都支持 ES 模块 ，因此也请务必发布 ES 版本的库/包。这样一来，可以减少编译次数，最终可以减少向用户交付的代码。 这将提高应用程序的性能。

### 编译（Babel-ify）源代码还是直接打包源代码

在构建我的 NPM 库时，我花费了大量时间来试图找出该问题（如何编译、如何打包）的答案。我开始挖掘自己的 node_modules，查找所有优秀的库并检查它们的构建系统。

![](/img/localBlog/68747470733a2f2f63646e2e6e6c61726b2e636f6d2f79757175652f302f323032302f6a7065672f3338323530342f313630313330373938333731372d34623864363831332d653134302d343038322d623964652d6364336231353065313035362e6a706567.jpeg)

在查看了不同 libraries/packages 的构建输出之后，我清楚地了解了这些库的作者在发布之前可能会想到的不同策略。 以下是我的观察。

如你在上图中所看到的，我已根据它们的特性将这些库/软件包分为两组：

* UI Libraries-UI 库（styled-components,  material-ui）

* Core Packages-核心包（react，react-dom）

你可能已经弄清楚了这两组之间的区别。

UI Libraries

* 有一个 dist 文件夹，该文件夹是针对 ES/UMD/CJS 模块系统  的打包和压缩版本。

* 有一个 lib 文件夹，用来存放被编译后的代码。

Core Packages

* 只有一个文件夹，其中包含针对 ES/UMD/CJS 模块系统的打包和压缩版本。

但是，为什么 UI Libraries 和 Core Packages 的构建输出有所不同？？

UI Libraries

想象一下，如果我们只是发布库的 bundled version 将其托管在 CDN 上，我们的用户将直接在 ` <script /> ` 标记中使用它。现在，如果使用者只想使用  `<Button /> ` 组件，则他们必须加载整个库。另外，在浏览器中，没有可以解决 tree shaking 的打包工具，最终我们会将整个库代码发送给我们的使用者。 因此，我们不能像如下代码引入整个库文件。

```
<script type="module">
  import { Button } from "https://unpkg.com/uilibrary/index.js";
</script>
```

现在，如果我们只是简单地将 src 转换为 lib 并将该 lib 托管在 CDN 上，那么我们的使用者实际上可以得到他们想要的任何东西而没有任何开销，“代码更少，加载更快”。  

```
<script type="module">
  import { Button } from "https://unpkg.com/uilibrary/lib/button.js";
</script>
```

Core Packages

Core Packages（核心包）永远不会通过  `<script /> ` 标记使用，因为它们必须是主应用程序的一部分。因此，我们可以安全地发布这些软件包的构建版本（  UMD，ES），并将构建后的系统交给用户使用。

例如，他们可以使用  UMD  而不使用摇树优化，或者如果打包器能够识别并获得摇树优化的好处，则可以使用  ES  。

```
// CJS require
const Button = require("uilibrary/button");
// ES import
import {Button} from "uilibrary";
```

但是……关于我们的问题：should we  transpile (Babelify) the source or bundle it？

对于 UI 库

* 当我们针对 es 模块系统构建时，需要 Babel 编译源代码，并将编译后的代码放置在 lib 文件夹中。我们甚至可以将 lib 托管在 CDN 上。

* 当我们针对 cjs/umd/es 模块系统 等多个模块系统构建时，需要  rollup  📦  打包和压缩代码。

对于 core packages

* 我们不需要 lib 版本。我们只需要针对 cjs/umd/es 模块系统，使用 rollup 进行  📦  打包和压缩源代码即可。

提示：对于愿意通过 ` <script /> ` 标记下载整个库/软件包的用户，我们也可以在 CDN 上托管 dist 文件夹 。

### 输出多文件

通过保留文件结构更好地支持 treeshaking

如果你对你的库使用了打包工具或编译器，可以对其进行配置以保留源文件目录结构。这样可以更容易地对特定文件进行 side effects 标记，有助于开发者的打包工具进行 threeshaking。

一个例外是，如果你要创建一个不依赖任何打包工具可以直接在浏览器中使用的产出（通常是 umd 格式，但也可能是现代的 esm 格式）。在这种情况下，最好让浏览器请求一个大文件，而不是请求多个小文件。此外，你应该进行代码压缩并为其创建 sourcemap。

### 要不要压缩代码

确定你期望的代码压缩程度

你可以将一些层面的代码压缩应用到你的库中，这取决于你对你的代码最终通过开发者的打包工具后的大小的追求程度。

例如，大多数编译器已经配置了删除空白符等其他简单的优化，即使是来自 NPM 模块的代码（在这里指的是你的库）。使用 terser —— 一个流行的 JavaScript 代码压缩工具 —— 这类压缩工具可以将包的最终大小减少 95%。在某些情况下，你可能会对这些优化感到满意，且不需要你来付出任何努力。

但如果在发布前对你的库进行代码压缩，这可以得到一些额外的好处，但需要深入了解压缩工具的配置和副作用。压缩工具通常不会将这类压缩用于 NPM 模块，因此，如果你不自己来做的话，你会错过这些节省。

最后，如果你正创建一个不依赖任何打包工具可以直接在浏览器中使用的产出（通常是 umd 格式，但也可以是现代的 esm 格式）。在这种情况下，你应该对代码进行压缩，并创建 sourcemap，并输出到一个单文件。

### 创建 sourcemap

当使用打包工具或编译器时，生成 sourcemap

对源代码进行任何形式的编译，都将导致未来某个异常的位置，无法与源码对应起来。为了帮助未来的自己，创建 sourcemap，即使只进行了很少的编译工作。

### 创建 TypeScript 类型

类型提升开发体验

随着使用 TypeScript 的开发者数量不断增长，将类型内置到你的库中将有助于改善开发体验 (DX)。此外，不使用 TypeScript 的开发者在使用支持类型的编辑器（例如 VSCode，它使用类型来支持其 Intellisense 功能）时也会获得更好的 DX。

但是，创建类型并不意味着你必须使用 TypeScript 来编写你的库。

* 一种选择是继续在源代码中使用 JavaScript，然后通过 JSDoc 注释来支持类型。然后，你可以将 TypeScript 配置为仅从你的 JavaScript 源代码中构建类型文件。

* 另一种选择是直接在 index.d.ts 文件中编写 TypeScript 类型文件。

获得类型文件后，请确保设置了 package.json#exports 和 package.json#types 字段.

### 外置框架

不要将 React、Vue 等框架打包在你的库中

当构建的库依赖某个框架（例如 React、Vue 等），或是作为另一个库的插件，你可能需要将框架配置到“externals”中。这可以使你的库引用这个框架，但不会将其打包到最终的产出中。这会避免产生一些 bug，并减少库的体积。

你应该还需要将框架添加到库的 package.json 的 peer dependencies 中，这将帮助开发者发现你依赖于某个框架。

### 必要的编译

编译 TypeScript、将 JSX 转换为函数调用

如果库的源码是需要进行编译的形式，如 TypeScript、React 或 Vue 组件等，那么你库需要输出的是编译后的代码。

例如：

* 你的 TypeScript 代码应该输出为 JavaScript。

* 你的 React 组件，例如 `<Example />`，应该在输出中使用 jsx() 或 createElement() 来替换 JSX 语法。

进行这样的编译时，请确保同时也创建 sourcemap

### 维护 changelog

记录更新和变更

只要能让开发者了解到有哪些变更和对他们的影响，至于是通过自动化工具还是通过亲自动手的方式来处理，这都无关紧要。理想情况下，库的每次版本变更都应该在 changelog 中进行相应的更新。

### 拆分出你的 CSS 文件

让开发者能够按需引入 CSS

如果你正在创建一个 CSS 库（如 Bootstrap、Tailwind 等），最简单的方式就是提供单一文件，包含库的所有功能。然而，在这种情况下，你的 CSS 产出最终可能会变得很大，影响开发者网站的性能。为了避免这种情况，库通常会提供自定义生成 CSS 产出的功能，让产出中只包含开发者正在使用的必要 CSS（例如，参考 [Bootstrap](https://getbootstrap.com/docs/5.2/customize/optimize/) 和 [Tailwind](https://tailwindcss.com/docs/optimizing-for-production) 是怎么做的）。

如果 CSS 只是你的库的一部分（例如，具有默认样式的组件库），那么最好将 CSS 按组件分离单独构建产出，在使用相应的组件时按需导入。这方面的一个例子是 [react-component](https://github.com/react-component/slider#usage)。

### 配置 package.json

package.json 中有许多重要的配置字段值得讨论；我在这里将着重讨论其中为重要的一些，这还有很多额外的字段，你同样可以进行配置。

**设置 name 字段**

给你的库取一个名字

name 字段将决定你的包在 npm 上的名字，开发者可以通过这个名字去安装并使用你的库。

注意，库的命名是有限制的，如果你的代码库属于某个组织，你还可以创建一个命名空间。更多细节可以参考 [name docs on npm](https://docs.npmjs.com/cli/v8/configuring-npm)。

name 和 version 的组合为库每次迭代创建一个唯一标识。

**设置 version 字段**

通过更改 version 来对你的库发布更新

正如 name 部分所说，name 和 version 的组合为你的库在 npm 上创建一个唯一标识。当你更新库中的代码时，你可以更新 version 字段并发布以允许开发者获取该新代码。

推荐使用 [semver](https://semver.org/) 版本控制策略，但要注意的是有些库选择 [calver](https://calver.org/) 或使用他们自己特有的版本控制策略。无论你选择使用哪种策略，都应该记录下来，以便开发者了解你的库是如何进行版本控制的。

你还应该在 changelog 中记录你的更改。

**定义你的 exports**

exports 为你的库定义公共 API

package.json 中的 exports 字段 - 有时被称为“package exports” - 是一个非常有用的补充，尽管它确实引入了一些复杂性。它做的最重要的两件事是：

1、定义哪些东西可以从你的库中导入，哪些则不可以，以及可导入的内容的名字。如果没有在 exports 中被列出，那么开发者就不可以 import 或 require 它们。换句话说，exports 的表现像是给你的库用户查看的公共 API，帮助定义哪些是外部的哪些是内部的。

2、允许你根据不同的条件（你可以定义）去选择那个文件是被导入的，例如“文件是被 import 还是被 require？开发人员需要的是 development 版本的库还是 production 版本等等。

关于这部分的内容NodeJS 团队和Webpack 团队提供了一些很优秀的文档。在此我列出一个涵盖大部分常见场景的例子：

```
{
  "exports": {
    ".": {
      "types": "index.d.ts",
      "module": "index.js",
      "import": "index.js",
      "require": "index.cjs",
      "default": "index.js"
    },
    "./package.json": "./package.json"
  }
}
```

让我们深入了解这些字段的含义以及我选择这个例子的原因：

* "." 表示你的库的默认入口

* 解析过程是从上往下的，并在找到匹配的字段后立即停止；所以入口的顺序是非常重要的

* types 字段应始终放在第一位，帮助 TypeScript 查找类型文件

* module 是一个“非官方”字段，它被 Webpack 和 Rollup 等打包工具所支持。它应该被放在 import 和 require 之前，并且指向 esm 格式的产出 -- 如果你的源代码是纯 esm 的，它也可以指向你的源代码。正如在格式部分中指出的那样，它旨在帮助打包工具只包含你的库的一个副本，无论它是通过 import 还是 require 方式引入的。

* import 用于当有人通过 import 使用你的库时

* require 用于当有人通过 require 使用你的库时

* default 字段用于兜底，在没有任何条件匹配时使用。虽然目前可能并不会匹配到它，但为了面对“未知的未来场景”，使用它是好的

当一个打包工具或者运行时支持 exports 字段的时候，那么 package.json 中的顶级字段 main、types、module 还有 browser 将被忽略，被 exports 取代。但是，对于尚不支持 exports 字段的工具或运行时来说，设置这些字段仍然很重要。

如果你有一个 "development" 和一个 "production" 的产出（例如，你有一些警告在 development 产出中有但在 production 产出中没有），那么你可以通过在 exports 字段中 "development" 和 "production" 来设置它们。注意一些打包工具例如 webpack 和 vite 将会自动识别这些导出条件，而 Rollup 也可以通过配置来识别它们，你需要提醒开发者在他们自己打包工具的配置中去做这些事。

**列出要发布的 files**

files 定义你的 NPM 包中要包含哪些文件

files 决定 npm CLI 在打包库时哪些文件和目录包含到最终的 NPM 包中。

例如，如果你将代码从 TypeScript 编译为 JavaScript，你可能就不想在 NPM 包中包含 TypeScript 的源代码。（相反，你应该包含 sourcemap）。

files 可以接受一个字符串数组（如果需要，这些字符串可以包含类似 glob 的语法），例如：

```
{
  "files": ["dist"]
}
```


注意，文件数组不接受相对路径表示；`"files": ["./dist"] `将无法正常工作。

验证你已正确设置 files 的一种好方法是运行 npm publish --dry-run，它将根据此设置列出将会包含的文件。

**为你的 JS 文件设置默认的模块 type**

type 规定你的 .js 文件使用哪个模块系统
运行时和打包工具需要一种方法来确定你的 .js 文件采用哪种模块系统 —— ESM 还是 CommonJS。因为 CommonJS 首先出现，所以它被打包工具视为默认的 - 但你可以通过在你的 package.json 中添加 "type" 来控制这种行为。

你可以选择 "type":"module" 或 "type":"commonjs"，也可以不添加该字段（默认为 CommonJS），但仍强烈建议你进行设置，显式地声明你正在使用哪一个。

请注意，你可以通过几个技巧在项目中混用模块类型：

* .mjs 文件总是 ESM 模块，即使你的 package.json 有 "type": "commonjs"（或者没有 type）

* .cjs 文件总是 CommonJS 模块，即使你的 package.json 有 "type": "module"

* 你可以在子目录下添加其他 package.json 文件；运行时和打包工具将向上遍历文件目录，直到寻找到最近的 package.json。这意味着你可以有两个不同的文件夹，都使用 .js 文件，但每个文件夹都有自己的 package.json 并设置为不同的 type 以获得基于 CommonJS 和 ESM 的文件夹。

**列出哪些模块有 sideEffects**

设置 sideEffects 来允许 treeshaking

创建一个“纯模块”带来的优点与创建一个纯函数十分类似；打包工具能够对你的库更好的进行 treeshaking。

通过设置 sideEffects 让打包工具知道你的模块是否是“纯”的。不设置这个字段，打包工具将不得不假设你所有的模块都是有副作用。

sideEffects 可以设为 false，表示没有任何模块具有副作用，也可以设置为字符串数组来列出哪些文件具有副作用。例如：

```
{
  // 所有模块都是“纯”的
  "sideEffects": false
}
```

或

```
{
  // 除了 "module.js"，所有模块都是“纯”的
  "sideEffects": ["module.js"]
}
```

所以，什么让一个模块具有副作用？例如修改一个全局变量，发送 API 请求，或导出 CSS，而且开发人员不需要做任何事情这些动作就会被执行。例如：

```
// 具有副作用的模块

export const myVar = "hello";

window.example = "testing";
```

导入 myVar 时，你的模块自动设置 window.example。

例如：

```
import { myVar } from "library";

console.log(window.example);
// 打印 "testing"
```

在某些情况下，如 polyfill，这种行为是有意的。然而，如果我们想让这个模块是“纯”的，我们可以将对 window.example 的赋值移动到一个函数中。例如：

```
// 一个“纯”模块

export const myVar = "hello";

export function setExample() {
  window.example = "testing";
}
```

现在这是一个“纯”模块。注意，从开发者的角度来看会有不同：

```
import { myVar, setExample } from "library";

console.log(window.example);
// 打印 "undefined"

setExample();

console.log(window.example);
// 打印 "testing"
```

**设置 main 字段**

main 定义 CommonJS 入口

main 是一个当打包工具或运行时不支持 package.json#exports 时的兜底方案；如果打包工具或运行时支持 package exports，则不会使用 main。

main 应该指向一个兼容 CommonJS 格式的产出；它应该与 package exports 中的 require 保持一致。

**设置 module 字段**

module 定义 ESM 入口

module 是一个当打包工具或运行时不支持 package.json#exports 时的兜底方案；如果打包工具或运行时支持 package exports，则不会使用 module。

module 应该指向一个兼容 ESM 格式的产出；它应该与 package exports 中的 module 或 import 保持一致。

**设置给 CDN 使用的附加字段**

支持 CDN，例如 unpkg 和 jsdelivr

为让你的库在 CDN 上“以默认的方式正常工作”，如 unpkg 和 jsdelivr，你可以设置它们的特定字段指向你的 umd 产出。例如：

```
{
  "unpkg": "./dist/index.umd.js",
  "jsdelivr": "./dist/index.umd.js"
}
```

**设置 browser 字段**

browser 指向能在浏览器中工作的产出

browser 是一个当打包工具或运行时不支持 package.json#exports 时的兜底方案；如果打包工具或运行时支持 package exports， 则不会使用 browser。

browser 应该指向能在浏览器中工作的 esm 产出。但是，只有在为浏览器和服务器（等其他非浏览器环境）创建不同的产出时，才需要设置该字段。如果你没有为多个环境创建多个产出，或者你的产出是“纯 JavaScript”或“通用”的，可以在任何 JavaScript 环境中运行，那么你就不需要设置 browser 字段。

注意，browser 字段不应该指向 umd 产出，因为那样的话，你的库就不会被打包工具（如 Webpack）进行 treeshaking，这些打包工具会优先考虑这个字段，而不是其他字段，比如 module 和 main。

**设置types字段**

types 定义 TypeScript 类型

types 是一个当打包工具或运行时不支持 package.json#exports 时的兜底方案； 如果打包工具或运行时支持 package exports，则不会使用 types。

types 应该指向你的 TypeScript 入口文件，例如 index.d.ts；它应该与 package exports 中的 types 字段指向同一个文件。

**列出 peerDependencies**

如果你依赖别的框架或库，将它设置为 peer dependency

你应该外置框架。然而，这样做后，你的库只有在开发人员自行安装你需要的框架后才能工作。设置 peerDependencies 让他们知道他们需要安装的框架。- 例如，如果你在创建一个 React 库：

```
{
  "peerDependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  }
}
```

你应该以书面形式来体现这些依赖；例如，npm v3-v6 不安装 peer dependencies，而 npm v7+ 将自动安装 peer dependencies。

**说明你的库使用哪个许可证**

保护你自己和其他的贡献者

>开源许可证用于保护贡献者和用户。没有这种保护，企业和有经验的开发者不会使用该项目。

上述引用自 [Choose a License](https://choosealicense.com/)，这也是一篇很好的文章，帮助你来决定哪个许可证适合你的项目。

当你决定了许可证，[关于许可证的 npm 文档](https://docs.npmjs.com/cli/v8/configuring-npm/package-json#license)中描述了许可证字段的格式。例如：

```
{
  "license": "MIT"
}
```

除此之外，你可以在项目的根目录下创建一个 LICENSE.txt 文件，并将许可证的文本复制到这里。





