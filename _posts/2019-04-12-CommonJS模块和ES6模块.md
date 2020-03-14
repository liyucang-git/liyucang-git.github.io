---
layout: post
title: CommonJS模块和ES6模块
subtitle: 全面解析前端模块化
date: 2019-04-12
author: Li Yucang
catalog: true
tags:
  - es6
  - 模块
---

# CommonJS 模块和 ES6 模块

## Module 的语法

### 概述

历史上，JavaScript 一直没有模块（module）体系，无法将一个大程序拆分成互相依赖的小文件，再用简单的方法拼装起来。其他语言都有这项功能，比如 Ruby 的 require、Python 的 import，甚至就连 CSS 都有@import，但是 JavaScript 任何这方面的支持都没有，这对开发大型的、复杂的项目形成了巨大障碍。

在 ES6 之前，社区制定了一些模块加载方案，最主要的有 CommonJS 和 AMD 两种。前者用于服务器，后者用于浏览器。ES6 在语言标准的层面上，实现了模块功能，而且实现得相当简单，完全可以取代 CommonJS 和 AMD 规范，成为浏览器和服务器通用的模块解决方案。

ES6 模块的设计思想是尽量的静态化，使得编译时就能确定模块的依赖关系，以及输入和输出的变量。CommonJS 和 AMD 模块，都只能在运行时确定这些东西。比如，CommonJS 模块就是对象，输入时必须查找对象属性。

```
// CommonJS模块
let { stat, exists, readFile } = require('fs');

// 等同于
let _fs = require('fs');
let stat = _fs.stat;
let exists = _fs.exists;
let readfile = _fs.readfile;
```

上面代码的实质是整体加载 fs 模块（即加载 fs 的所有方法），生成一个对象（\_fs），然后再从这个对象上面读取 3 个方法。这种加载称为“运行时加载”，因为只有运行时才能得到这个对象，导致完全没办法在编译时做“静态优化”。

ES6 模块不是对象，而是通过 export 命令显式指定输出的代码，再通过 import 命令输入。

```
// ES6模块
import { stat, exists, readFile } from 'fs';
```

上面代码的实质是从 fs 模块加载 3 个方法，其他方法不加载。这种加载称为“编译时加载”或者静态加载，即 ES6 可以在编译时就完成模块加载，效率要比 CommonJS 模块的加载方式高。当然，这也导致了没法引用 ES6 模块本身，因为它不是对象。

由于 ES6 模块是编译时加载，使得静态分析成为可能。有了它，就能进一步拓宽 JavaScript 的语法，比如引入宏（macro）和类型检验（type system）这些只能靠静态分析实现的功能。

除了静态加载带来的各种好处，ES6 模块还有以下好处。

- 不再需要 UMD 模块格式了，将来服务器和浏览器都会支持 ES6 模块格式。目前，通过各种工具库，其实已经做到了这一点。
- 将来浏览器的新 API 就能用模块格式提供，不再必须做成全局变量或者 navigator 对象的属性。
- 不再需要对象作为命名空间（比如 Math 对象），未来这些功能可以通过模块提供。

### 严格模式

ES6 的模块自动采用严格模式，不管你有没有在模块头部加上"use strict";。

严格模式主要有以下限制。

- 变量必须声明后再使用
- 函数的参数不能有同名属性，否则报错
- 不能使用 with 语句
- 不能对只读属性赋值，否则报错
- 不能使用前缀 0 表示八进制数，否则报错
- 不能删除不可删除的属性，否则报错
- 不能删除变量 delete prop，会报错，只能删除属性`delete global[prop]`
- eval 不会在它的外层作用域引入变量
- eval 和 arguments 不能被重新赋值
- arguments 不会自动反映函数参数的变化
- 不能使用 arguments.callee
- 不能使用 arguments.caller
- 禁止 this 指向全局对象
- 不能使用 fn.caller 和 fn.arguments 获取函数调用的堆栈
- 增加了保留字（比如 protected、static 和 interface）

上面这些限制，模块都必须遵守。其中，尤其需要注意 this 的限制。ES6 模块之中，顶层的 this 指向 undefined，即不应该在顶层代码使用 this。

### export 命令

模块功能主要由两个命令构成：export 和 import。export 命令用于规定模块的对外接口，import 命令用于输入其他模块提供的功能。

一个模块就是一个独立的文件。该文件内部的所有变量，外部无法获取。如果你希望外部能够读取模块内部的某个变量，就必须使用 export 关键字输出该变量。下面是一个 JS 文件，里面使用 export 命令输出变量。

```
// profile.js
export var firstName = 'Michael';
export var lastName = 'Jackson';
export var year = 1958;
```

上面代码是 profile.js 文件，保存了用户信息。ES6 将其视为一个模块，里面用 export 命令对外部输出了三个变量。

export 的写法，除了像上面这样，还有另外一种。

```
// profile.js
var firstName = 'Michael';
var lastName = 'Jackson';
var year = 1958;

export {firstName, lastName, year};
```

上面代码在 export 命令后面，使用大括号指定所要输出的一组变量。它与前一种写法（直接放置在 var 语句前）是等价的，但是应该优先考虑使用这种写法。因为这样就可以在脚本尾部，一眼看清楚输出了哪些变量。

export 命令除了输出变量，还可以输出函数或类（class）。

```
export function multiply(x, y) {
  return x * y;
};
```

上面代码对外输出一个函数 multiply。

通常情况下，export 输出的变量就是本来的名字，但是可以使用 as 关键字重命名。

```
function v1() { ... }
function v2() { ... }

export {
  v1 as streamV1,
  v2 as streamV2,
  v2 as streamLatestVersion
};
```

上面代码使用 as 关键字，重命名了函数 v1 和 v2 的对外接口。重命名后，v2 可以用不同的名字输出两次。

需要特别注意的是，export 命令规定的是对外的接口，必须与模块内部的变量建立一一对应关系。

```
// 报错
export 1;

// 报错
var m = 1;
export m;
```

上面两种写法都会报错，因为没有提供对外的接口。第一种写法直接输出 1，第二种写法通过变量 m，还是直接输出 1。1 只是一个值，不是接口。正确的写法是下面这样。

```
// 写法一
export var m = 1;

// 写法二
var m = 1;
export {m};

// 写法三
var n = 1;
export {n as m};
```

上面三种写法都是正确的，规定了对外的接口 m。其他脚本可以通过这个接口，取到值 1。它们的实质是，在接口名与模块内部变量之间，建立了一一对应的关系。

同样的，function 和 class 的输出，也必须遵守这样的写法。

```
// 报错
function f() {}
export f;

// 正确
export function f() {};

// 正确
function f() {}
export {f};
```

另外，export 语句输出的接口，与其对应的值是动态绑定关系，即通过该接口，可以取到模块内部实时的值。

```
export var foo = 'bar';
setTimeout(() => foo = 'baz', 500);
```

上面代码输出变量 foo，值为 bar，500 毫秒之后变成 baz。

这一点与 CommonJS 规范完全不同。CommonJS 模块输出的是值的缓存，不存在动态更新。

最后，export 命令可以出现在模块的任何位置，只要处于模块顶层就可以。如果处于块级作用域内，就会报错，下一节的 import 命令也是如此。这是因为处于条件代码块之中，就没法做静态优化了，违背了 ES6 模块的设计初衷。

```
function foo() {
  export default 'bar' // SyntaxError
}
foo()
```

上面代码中，export 语句放在函数之中，结果报错。

### import 命令

使用 export 命令定义了模块的对外接口以后，其他 JS 文件就可以通过 import 命令加载这个模块。

```
// main.js
import {firstName, lastName, year} from './profile.js';

function setName(element) {
  element.textContent = firstName + ' ' + lastName;
}
```

上面代码的 import 命令，用于加载 profile.js 文件，并从中输入变量。import 命令接受一对大括号，里面指定要从其他模块导入的变量名。大括号里面的变量名，必须与被导入模块（profile.js）对外接口的名称相同。

如果想为输入的变量重新取一个名字，import 命令要使用 as 关键字，将输入的变量重命名。

```
import { lastName as surname } from './profile.js';
```

import 命令输入的变量都是只读的，因为它的本质是输入接口。也就是说，不允许在加载模块的脚本里面，改写接口。

```
import {a} from './xxx.js'

a = {}; // Syntax Error : 'a' is read-only;
```

上面代码中，脚本加载了变量 a，对其重新赋值就会报错，因为 a 是一个只读的接口。但是，如果 a 是一个对象，改写 a 的属性是允许的。

```
import {a} from './xxx.js'

a.foo = 'hello'; // 合法操作
```

上面代码中，a 的属性可以成功改写，并且其他模块也可以读到改写后的值。不过，这种写法很难查错，建议凡是输入的变量，都当作完全只读，轻易不要改变它的属性。

import 后面的 from 指定模块文件的位置，可以是相对路径，也可以是绝对路径，.js 后缀可以省略。如果只是模块名，不带有路径，那么必须有配置文件，告诉 JavaScript 引擎该模块的位置。

```
import {myMethod} from 'util';
```

上面代码中，util 是模块文件名，由于不带有路径，必须通过配置，告诉引擎怎么取到这个模块。

注意，import 命令具有提升效果，会提升到整个模块的头部，首先执行。

```
foo();

import { foo } from 'my_module';
```

上面的代码不会报错，因为 import 的执行早于 foo 的调用。这种行为的本质是，import 命令是编译阶段执行的，在代码运行之前。

由于 import 是静态执行，所以不能使用表达式和变量，这些只有在运行时才能得到结果的语法结构。

```
// 报错
import { 'f' + 'oo' } from 'my_module';

// 报错
let module = 'my_module';
import { foo } from module;

// 报错
if (x === 1) {
  import { foo } from 'module1';
} else {
  import { foo } from 'module2';
}
```

上面三种写法都会报错，因为它们用到了表达式、变量和 if 结构。在静态分析阶段，这些语法都是没法得到值的。

最后，import 语句会执行所加载的模块，因此可以有下面的写法。

```
import 'lodash';
```

上面代码仅仅执行 lodash 模块，但是不输入任何值。

如果多次重复执行同一句 import 语句，那么只会执行一次，而不会执行多次。

```
import 'lodash';
import 'lodash';
```

上面代码加载了两次 lodash，但是只会执行一次。

```
import { foo } from 'my_module';
import { bar } from 'my_module';

// 等同于
import { foo, bar } from 'my_module';
```

上面代码中，**虽然 foo 和 bar 在两个语句中加载，但是它们对应的是同一个 my_module 实例。也就是说，import 语句是 Singleton 模式**。

### 模块的整体加载

除了指定加载某个输出值，还可以使用整体加载，即用星号（\*）指定一个对象，所有输出值都加载在这个对象上面。

下面是一个 circle.js 文件，它输出两个方法 area 和 circumference。

```
// circle.js

export function area(radius) {
  return Math.PI * radius * radius;
}

export function circumference(radius) {
  return 2 * Math.PI * radius;
}
```

现在，加载这个模块。

```
// main.js

import { area, circumference } from './circle';

console.log('圆面积：' + area(4));
console.log('圆周长：' + circumference(14));
```

上面写法是逐一指定要加载的方法，整体加载的写法如下。

```
import * as circle from './circle';

console.log('圆面积：' + circle.area(4));
console.log('圆周长：' + circle.circumference(14));
```

注意，模块整体加载所在的那个对象（上例是 circle），应该是可以静态分析的，所以不允许运行时改变。下面的写法都是不允许的。

```
import * as circle from './circle';

// 下面两行都是不允许的
circle.foo = 'hello';
circle.area = function () {};
```

### export default 命令

从前面的例子可以看出，使用 import 命令的时候，用户需要知道所要加载的变量名或函数名，否则无法加载。但是，用户肯定希望快速上手，未必愿意阅读文档，去了解模块有哪些属性和方法。

为了给用户提供方便，让他们不用阅读文档就能加载模块，就要用到 export default 命令，为模块指定默认输出。

```
// export-default.js
export default function () {
  console.log('foo');
}
```

上面代码是一个模块文件 export-default.js，它的默认输出是一个函数。

其他模块加载该模块时，import 命令可以为该匿名函数指定任意名字。

```
// import-default.js
import customName from './export-default';
customName(); // 'foo'
```

上面代码的 import 命令，可以用任意名称指向 export-default.js 输出的方法，这时就不需要知道原模块输出的函数名。需要注意的是，这时 import 命令后面，不使用大括号。

export default 命令用在非匿名函数前，也是可以的。

```
// export-default.js
export default function foo() {
  console.log('foo');
}

// 或者写成

function foo() {
  console.log('foo');
}

export default foo;
```

上面代码中，foo 函数的函数名 foo，在模块外部是无效的。加载的时候，视同匿名函数加载。

下面比较一下默认输出和正常输出。

```
// 第一组
export default function crc32() { // 输出
  // ...
}

import crc32 from 'crc32'; // 输入

// 第二组
export function crc32() { // 输出
  // ...
};

import {crc32} from 'crc32'; // 输入
```

上面代码的两组写法，第一组是使用 export default 时，对应的 import 语句不需要使用大括号；第二组是不使用 export default 时，对应的 import 语句需要使用大括号。

export default 命令用于指定模块的默认输出。显然，一个模块只能有一个默认输出，因此 export default 命令只能使用一次。所以，import 命令后面才不用加大括号，因为只可能唯一对应 export default 命令。

本质上，export default 就是输出一个叫做 default 的变量或方法，然后系统允许你为它取任意名字。所以，下面的写法是有效的。

```
// modules.js
function add(x, y) {
  return x * y;
}
export {add as default};
// 等同于
// export default add;

// app.js
import { default as foo } from 'modules';
// 等同于
// import foo from 'modules';
```

正是因为 export default 命令其实只是输出一个叫做 default 的变量，所以它后面不能跟变量声明语句。

```
// 正确
export var a = 1;

// 正确
var a = 1;
export default a;

// 错误
export default var a = 1;
```

上面代码中，export default a 的含义是将变量 a 的值赋给变量 default。所以，最后一种写法会报错。

同样地，因为 export default 命令的本质是将后面的值，赋给 default 变量，所以可以直接将一个值写在 export default 之后。

```
// 正确
export default 42;

// 报错
export 42;
```

上面代码中，后一句报错是因为没有指定对外的接口，而前一句指定对外接口为 default。

有了 export default 命令，输入模块时就非常直观了，以输入 lodash 模块为例。

```
import _ from 'lodash';
```

如果想在一条 import 语句中，同时输入默认方法和其他接口，可以写成下面这样。

```
import _, { each, forEach } from 'lodash';
```

对应上面代码的 export 语句如下。

```
export default function (obj) {
  // ···
}

export function each(obj, iterator, context) {
  // ···
}

export { each as forEach };
```

上面代码的最后一行的意思是，暴露出 forEach 接口，默认指向 each 接口，即 forEach 和 each 指向同一个方法。

export default 也可以用来输出类。

```
// MyClass.js
export default class { ... }

// main.js
import MyClass from 'MyClass';
let o = new MyClass();
```

### export 与 import 的复合写法

如果在一个模块之中，先输入后输出同一个模块，import 语句可以与 export 语句写在一起。

```
export { foo, bar } from 'my_module';

// 可以简单理解为
import { foo, bar } from 'my_module';
export { foo, bar };
```

上面代码中，export 和 import 语句可以结合在一起，写成一行。但需要注意的是，**写成一行以后，foo 和 bar 实际上并没有被导入当前模块，只是相当于对外转发了这两个接口，导致当前模块不能直接使用 foo 和 bar**。

模块的接口改名和整体输出，也可以采用这种写法。

```
// 接口改名
export { foo as myFoo } from 'my_module';

// 整体输出
export * from 'my_module';
```

默认接口的写法如下。

```
export { default } from 'foo';
```

具名接口改为默认接口的写法如下。

```
export { es6 as default } from './someModule';

// 等同于
import { es6 } from './someModule';
export default es6;
```

同样地，默认接口也可以改名为具名接口。

```
export { default as es6 } from './someModule';
```

下面三种 import 语句，没有对应的复合写法。

```
import * as someIdentifier from "someModule";
import someIdentifier from "someModule";
import someIdentifier, { namedIdentifier } from "someModule";
```

为了做到形式的对称，现在有提案，提出补上这三种复合写法。

```
export * as someIdentifier from "someModule";
export someIdentifier from "someModule";
export someIdentifier, { namedIdentifier } from "someModule";
```

### 模块的继承

模块之间也可以继承。

假设有一个 circleplus 模块，继承了 circle 模块。

```
// circleplus.js

export * from 'circle';
export var e = 2.71828182846;
export default function(x) {
  return Math.exp(x);
}
```

上面代码中的 export *，表示再输出 circle 模块的所有属性和方法。注意，export *命令会忽略 circle 模块的 default 方法。然后，上面代码又输出了自定义的 e 变量和默认方法。

这时，也可以将 circle 的属性或方法，改名后再输出。

```
// circleplus.js

export { area as circleArea } from 'circle';
```

上面代码表示，只输出 circle 模块的 area 方法，且将其改名为 circleArea。

加载上面模块的写法如下。

```
// main.js

import * as math from 'circleplus';
import exp from 'circleplus';
console.log(exp(math.e));
```

上面代码中的 import exp 表示，将 circleplus 模块的默认方法加载为 exp 方法。

### 跨模块常量

本书介绍 const 命令的时候说过，const 声明的常量只在当前代码块有效。如果想设置跨模块的常量（即跨多个文件），或者说一个值要被多个模块共享，可以采用下面的写法。

```
// constants.js 模块
export const A = 1;
export const B = 3;
export const C = 4;

// test1.js 模块
import * as constants from './constants';
console.log(constants.A); // 1
console.log(constants.B); // 3

// test2.js 模块
import {A, B} from './constants';
console.log(A); // 1
console.log(B); // 3
```

如果要使用的常量非常多，可以建一个专门的 constants 目录，将各种常量写在不同的文件里面，保存在该目录下。

```
// constants/db.js
export const db = {
  url: 'http://my.couchdbserver.local:5984',
  admin_username: 'admin',
  admin_password: 'admin password'
};

// constants/user.js
export const users = ['root', 'admin', 'staff', 'ceo', 'chief', 'moderator'];
```

然后，将这些文件输出的常量，合并在 index.js 里面。

```
// constants/index.js
export {db} from './db';
export {users} from './users';
```

使用的时候，直接加载 index.js 就可以了。

```
// script.js
import {db, users} from './constants/index';
```

### import()

#### 简介

前面介绍过，import 命令会被 JavaScript 引擎静态分析，先于模块内的其他语句执行（import 命令叫做“连接” binding 其实更合适）。所以，下面的代码会报错。

```
// 报错
if (x === 2) {
  import MyModual from './myModual';
}
```

上面代码中，引擎处理 import 语句是在编译时，这时不会去分析或执行 if 语句，所以 import 语句放在 if 代码块之中毫无意义，因此会报句法错误，而不是执行时错误。也就是说，import 和 export 命令只能在模块的顶层，不能在代码块之中（比如，在 if 代码块之中，或在函数之中）。

这样的设计，固然有利于编译器提高效率，但也导致无法在运行时加载模块。在语法上，条件加载就不可能实现。如果 import 命令要取代 Node 的 require 方法，这就形成了一个障碍。因为 require 是运行时加载模块，import 命令无法取代 require 的动态加载功能。

```
const path = './' + fileName;
const myModual = require(path);
```

上面的语句就是动态加载，require 到底加载哪一个模块，只有运行时才知道。import 命令做不到这一点。

因此，有一个提案，建议引入 import()函数，完成动态加载。

```
import(specifier)
```

上面代码中，import 函数的参数 specifier，指定所要加载的模块的位置。import 命令能够接受什么参数，import()函数就能接受什么参数，两者区别主要是后者为动态加载。

import()返回一个 Promise 对象。下面是一个例子。

```
const main = document.querySelector('main');

import(`./section-modules/${someVariable}.js`)
  .then(module => {
    module.loadPageInto(main);
  })
  .catch(err => {
    main.textContent = err.message;
  });
```

import()函数可以用在任何地方，不仅仅是模块，非模块的脚本也可以使用。**它是运行时执行，也就是说，什么时候运行到这一句，就会加载指定的模块。另外，import()函数与所加载的模块没有静态连接关系，这点也是与 import 语句不相同**。**import()类似于 Node 的 require 方法，区别主要是前者是异步加载，后者是同步加载**。

#### 适用场合

下面是 import()的一些适用场合。

（1）按需加载。

import()可以在需要的时候，再加载某个模块。

```
button.addEventListener('click', event => {
  import('./dialogBox.js')
  .then(dialogBox => {
    dialogBox.open();
  })
  .catch(error => {
    /* Error handling */
  })
});
```

上面代码中，import()方法放在 click 事件的监听函数之中，只有用户点击了按钮，才会加载这个模块。

（2）条件加载

import()可以放在 if 代码块，根据不同的情况，加载不同的模块。

```
if (condition) {
  import('moduleA').then(...);
} else {
  import('moduleB').then(...);
}
```

上面代码中，如果满足条件，就加载模块 A，否则加载模块 B。

（3）动态的模块路径

import()允许模块路径动态生成。

```
import(f())
.then(...);
```

上面代码中，根据函数 f 的返回结果，加载不同的模块。

#### 注意点

**import()加载模块成功以后，这个模块会作为一个对象，当作 then 方法的参数**。因此，可以使用对象解构赋值的语法，获取输出接口。

```
import('./myModule.js')
.then(({export1, export2}) => {
  // ...·
});
```

上面代码中，export1 和 export2 都是 myModule.js 的输出接口，可以解构获得。

如果模块有 default 输出接口，可以用参数直接获得。

```
import('./myModule.js')
.then(myModule => {
  console.log(myModule.default);
});
```

上面的代码也可以使用具名输入的形式。

```
import('./myModule.js')
.then(({default: theDefault}) => {
  console.log(theDefault);
});
```

如果想同时加载多个模块，可以采用下面的写法。

```
Promise.all([
  import('./module1.js'),
  import('./module2.js'),
  import('./module3.js'),
])
.then(([module1, module2, module3]) => {
   ···
});
```

import()也可以用在 async 函数之中。

```
async function main() {
  const myModule = await import('./myModule.js');
  const {export1, export2} = await import('./myModule.js');
  const [module1, module2, module3] =
    await Promise.all([
      import('./module1.js'),
      import('./module2.js'),
      import('./module3.js'),
    ]);
}
main();
```

## Module 的加载实现

### 浏览器加载

#### 传统方法

HTML 网页中，浏览器通过`<script>`标签加载 JavaScript 脚本。

```
<!-- 页面内嵌的脚本 -->
<script type="application/javascript">
  // module code
</script>

<!-- 外部脚本 -->
<script type="application/javascript" src="path/to/myModule.js">
</script>
```

上面代码中，由于浏览器脚本的默认语言是 JavaScript，因此 type="application/javascript"可以省略。

默认情况下，浏览器是同步加载 JavaScript 脚本，即渲染引擎遇到`<script>`标签就会停下来，等到执行完脚本，再继续向下渲染。如果是外部脚本，还必须加入脚本下载的时间。

如果脚本体积很大，下载和执行的时间就会很长，因此造成浏览器堵塞，用户会感觉到浏览器“卡死”了，没有任何响应。这显然是很不好的体验，所以浏览器允许脚本异步加载，下面就是两种异步加载的语法。

```
<script src="path/to/myModule.js" defer></script>
<script src="path/to/myModule.js" async></script>
```

上面代码中，`<script>`标签打开 defer 或 async 属性，脚本就会异步加载。渲染引擎遇到这一行命令，就会开始下载外部脚本，但不会等它下载和执行，而是直接执行后面的命令。

defer 与 async 的区别是：**defer 要等到整个页面在内存中正常渲染结束（DOM 结构完全生成，以及其他脚本执行完成），才会执行**；**async 一旦下载完，渲染引擎就会中断渲染，执行这个脚本以后，再继续渲染**。一句话，defer 是“渲染完再执行”，async 是“下载完就执行”。另外，如果有多个 defer 脚本，会按照它们在页面出现的顺序加载，而多个 async 脚本是不能保证加载顺序的。

#### 加载规则

浏览器加载 ES6 模块，也使用`<script>`标签，但是要加入 type="module"属性。

```
<script type="module" src="./foo.js"></script>
```

上面代码在网页中插入一个模块 foo.js，由于 type 属性设为 module，所以浏览器知道这是一个 ES6 模块。

**浏览器对于带有 type="module"的`<script>`，都是异步加载，不会造成堵塞浏览器，即等到整个页面渲染完，再执行模块脚本，等同于打开了`<script>`标签的 defer 属性**。

```
<script type="module" src="./foo.js"></script>
<!-- 等同于 -->
<script type="module" src="./foo.js" defer></script>
```

如果网页有多个`<script type="module">`，它们会按照在页面出现的顺序依次执行。

`<script>`标签的 async 属性也可以打开，这时只要加载完成，渲染引擎就会中断渲染立即执行。执行完成后，再恢复渲染。

```
<script type="module" src="./foo.js" async></script>
```

一旦使用了 async 属性，`<script type="module">`就不会按照在页面出现的顺序执行，而是只要该模块加载完成，就执行该模块。

ES6 模块也允许内嵌在网页中，语法行为与加载外部脚本完全一致。

```
<script type="module">
  import utils from "./utils.js";

  // other code
</script>
```

对于外部的模块脚本（上例是 foo.js），有几点需要注意。

- 代码是在模块作用域之中运行，而不是在全局作用域运行。模块内部的顶层变量，外部不可见。
- 模块脚本自动采用严格模式，不管有没有声明 use strict。
- 模块之中，可以使用 import 命令加载其他模块（.js 后缀不可省略，需要提供绝对 URL 或相对 URL），也可以使用 export 命令输出对外接口。
- 模块之中，顶层的 this 关键字返回 undefined，而不是指向 window。也就是说，在模块顶层使用 this 关键字，是无意义的。同一个模块如果加载多次，将只执行一次。

下面是一个示例模块。

```
import utils from 'https://example.com/js/utils.js';

const x = 1;

console.log(x === window.x); //false
console.log(this === undefined); // true
```

利用顶层的 this 等于 undefined 这个语法点，可以侦测当前代码是否在 ES6 模块之中。

```
const isNotModuleScript = this !== undefined;
```

### ES6 模块与 CommonJS 模块的差异

讨论 Node 加载 ES6 模块之前，必须了解 ES6 模块与 CommonJS 模块完全不同。

它们有两个重大差异。

- CommonJS 模块输出的是一个值的拷贝，ES6 模块输出的是值的引用。
- CommonJS 模块是运行时加载，ES6 模块是编译时输出接口。第二个差异是因为 CommonJS 加载的是一个对象（即 module.exports 属性），该对象只有在脚本运行完才会生成。**而 ES6 模块不是对象，它的对外接口只是一种静态定义，在代码静态解析阶段就会生成。**

下面重点解释第一个差异。

**CommonJS 模块输出的是值的拷贝，也就是说，一旦输出一个值，模块内部的变化就影响不到这个值**。请看下面这个模块文件 lib.js 的例子。

```
// lib.js
var counter = 3;
function incCounter() {
  counter++;
}
module.exports = {
  counter: counter,
  incCounter: incCounter,
};
```

上面代码输出内部变量 counter 和改写这个变量的内部方法 incCounter。然后，在 main.js 里面加载这个模块。

```
// main.js
var mod = require('./lib');

console.log(mod.counter);  // 3
mod.incCounter();
console.log(mod.counter); // 3
```

上面代码说明，lib.js 模块加载以后，它的内部变化就影响不到输出的 mod.counter 了。这是因为 mod.counter 是一个原始类型的值，会被缓存。除非写成一个函数，才能得到内部变动后的值。

```
// lib.js
var counter = 3;
function incCounter() {
  counter++;
}
module.exports = {
  get counter() {
    return counter
  },
  incCounter: incCounter,
};
```

上面代码中，输出的 counter 属性实际上是一个取值器函数。现在再执行 main.js，就可以正确读取内部变量 counter 的变动了。

```
$ node main.js
3
4
```

**ES6 模块的运行机制与 CommonJS 不一样。JS 引擎对脚本静态分析的时候，遇到模块加载命令 import，就会生成一个只读引用。等到脚本真正执行时，再根据这个只读引用，到被加载的那个模块里面去取值**。换句话说，ES6 的 import 有点像 Unix 系统的“符号连接”，原始值变了，import 加载的值也会跟着变。因此，ES6 模块是动态引用，并且不会缓存值，模块里面的变量绑定其所在的模块。

还是举上面的例子。

```
// lib.js
export let counter = 3;
export function incCounter() {
  counter++;
}

// main.js
import { counter, incCounter } from './lib';
console.log(counter); // 3
incCounter();
console.log(counter); // 4
```

上面代码说明，ES6 模块输入的变量 counter 是活的，完全反应其所在模块 lib.js 内部的变化。

再举一个例子。

```
// m1.js
export var foo = 'bar';
setTimeout(() => foo = 'baz', 500);

// m2.js
import {foo} from './m1.js';
console.log(foo);
setTimeout(() => console.log(foo), 500);
```

上面代码中，m1.js 的变量 foo，在刚加载时等于 bar，过了 500 毫秒，又变为等于 baz。

让我们看看，m2.js 能否正确读取这个变化。

```
$ babel-node m2.js

bar
baz
```

上面代码表明，ES6 模块不会缓存运行结果，而是动态地去被加载的模块取值，并且变量总是绑定其所在的模块。

由于 ES6 输入的模块变量，只是一个“符号连接”，所以这个变量是只读的，对它进行重新赋值会报错。

```
// lib.js
export let obj = {};

// main.js
import { obj } from './lib';

obj.prop = 123; // OK
obj = {}; // TypeError
```

上面代码中，main.js 从 lib.js 输入变量 obj，可以对 obj 添加属性，但是重新赋值就会报错。因为变量 obj 指向的地址是只读的，不能重新赋值，这就好比 main.js 创造了一个名为 obj 的 const 变量。

最后，export 通过接口，输出的是同一个值。不同的脚本加载这个接口，得到的都是同样的实例。

```
// mod.js
function C() {
  this.sum = 0;
  this.add = function () {
    this.sum += 1;
  };
  this.show = function () {
    console.log(this.sum);
  };
}

export let c = new C();
```

上面的脚本 mod.js，输出的是一个 C 的实例。不同的脚本加载这个模块，得到的都是同一个实例。

```
// x.js
import {c} from './mod';
c.add();

// y.js
import {c} from './mod';
c.show();

// main.js
import './x';
import './y';
```

现在执行 main.js，输出的是 1。

```
$ babel-node main.js
1
```

这就证明了 x.js 和 y.js 加载的都是 C 的同一个实例。

### Node 加载

#### 概述

Node 对 ES6 模块的处理比较麻烦，因为它有自己的 CommonJS 模块格式，与 ES6 模块格式是不兼容的。目前的解决方案是，将两者分开，ES6 模块和 CommonJS 采用各自的加载方案。

Node 要求 ES6 模块采用.mjs 后缀文件名。也就是说，只要脚本文件里面使用 import 或者 export 命令，那么就必须采用.mjs 后缀名。require 命令不能加载.mjs 文件，会报错，只有 import 命令才可以加载.mjs 文件。反过来，.mjs 文件里面也不能使用 require 命令，必须使用 import。

目前，这项功能还在试验阶段。安装 Node v8.5.0 或以上版本，要用--experimental-modules 参数才能打开该功能。

```
$ node --experimental-modules my-app.mjs
```

为了与浏览器的 import 加载规则相同，Node 的.mjs 文件支持 URL 路径。

```
import './foo?query=1'; // 加载 ./foo 传入参数 ?query=1
```

上面代码中，脚本路径带有参数?query=1，Node 会按 URL 规则解读。同一个脚本只要参数不同，就会被加载多次，并且保存成不同的缓存。由于这个原因，只要文件名中含有:、%、#、?等特殊字符，最好对这些字符进行转义。

目前，Node 的 import 命令只支持加载本地模块（file:协议），不支持加载远程模块。

如果模块名不含路径，那么 import 命令会去 node_modules 目录寻找这个模块。

```
import 'baz';
import 'abc/123';
```

如果模块名包含路径，那么 import 命令会按照路径去寻找这个名字的脚本文件。

```
import 'file:///etc/config/app.json';
import './foo';
import './foo?search';
import '../bar';
import '/baz';
```

如果脚本文件省略了后缀名，比如 import './foo'，Node 会依次尝试四个后缀名：./foo.mjs、./foo.js、./foo.json、./foo.node。如果这些脚本文件都不存在，Node 就会去加载./foo/package.json 的 main 字段指定的脚本。如果./foo/package.json 不存在或者没有 main 字段，那么就会依次加载./foo/index.mjs、./foo/index.js、./foo/index.json、./foo/index.node。如果以上四个文件还是都不存在，就会抛出错误。

最后，Node 的 import 命令是异步加载，这一点与浏览器的处理方法相同。

#### 内部变量

ES6 模块应该是通用的，同一个模块不用修改，就可以用在浏览器环境和服务器环境。为了达到这个目标，Node 规定 ES6 模块之中不能使用 CommonJS 模块的特有的一些内部变量。

首先，就是 this 关键字。ES6 模块之中，顶层的 this 指向 undefined；CommonJS 模块的顶层 this 指向当前模块，这是两者的一个重大差异。

其次，以下这些顶层变量在 ES6 模块之中都是不存在的。

- arguments
- require
- module
- exports
- \_\_filename
- \_\_dirname

如果你一定要使用这些变量，有一个变通方法，就是写一个 CommonJS 模块输出这些变量，然后再用 ES6 模块加载这个 CommonJS 模块。但是这样一来，该 ES6 模块就不能直接用于浏览器环境了，所以不推荐这样做。

```
// expose.js
module.exports = {__dirname};

// use.mjs
import expose from './expose.js';
const {__dirname} = expose;
```

上面代码中，expose.js 是一个 CommonJS 模块，输出变量**dirname，该变量在 ES6 模块之中不存在。ES6 模块加载 expose.js，就可以得到**dirname。

#### ES6 模块加载 CommonJS 模块

CommonJS 模块的输出都定义在 module.exports 这个属性上面。Node 的 import 命令加载 CommonJS 模块，Node 会自动将 module.exports 属性，当作模块的默认输出，即等同于 export default xxx。

下面是一个 CommonJS 模块。

```
// a.js
module.exports = {
  foo: 'hello',
  bar: 'world'
};

// 等同于
export default {
  foo: 'hello',
  bar: 'world'
};
```

import 命令加载上面的模块，module.exports 会被视为默认输出，即 import 命令实际上输入的是这样一个对象{ default: module.exports }。

所以，一共有三种写法，可以拿到 CommonJS 模块的 module.exports。

```
// 写法一
import baz from './a';
// baz = {foo: 'hello', bar: 'world'};

// 写法二
import {default as baz} from './a';
// baz = {foo: 'hello', bar: 'world'};

// 写法三
import * as baz from './a';
// baz = {
//   default: {foo: 'hello', bar: 'world'},
// }
```

下面是一些例子。

```
// b.js
module.exports = null;

// es.js
import foo from './b';
// foo = null;

import * as bar from './b';
// bar = { default:null };
```

上面代码中，es.js 采用第二种写法时，要通过 bar.default 这样的写法，才能拿到 module.exports。

```
// c.js
module.exports = function two() {
  return 2;
};

// es.js
import foo from './c';
foo(); // 2

import * as bar from './c';
bar.default(); // 2
bar(); // throws, bar is not a function
```

上面代码中，bar 本身是一个对象，不能当作函数调用，只能通过 bar.default 调用。

CommonJS 模块的输出缓存机制，在 ES6 加载方式下依然有效。

```
// foo.js
module.exports = 123;
setTimeout(() => module.exports = null);
```

上面代码中，对于加载 foo.js 的脚本，module.exports 将一直是 123，而不会变成 null。

由于 ES6 模块是编译时确定输出接口，CommonJS 模块是运行时确定输出接口，所以采用 import 命令加载 CommonJS 模块时，不允许采用下面的写法。

```
// 不正确
import { readFile } from 'fs';
```

上面的写法不正确，因为 fs 是 CommonJS 格式，只有在运行时才能确定 readFile 接口，而 import 命令要求编译时就确定这个接口。解决方法就是改为整体输入。

```
// 正确的写法一
import * as express from 'express';
const app = express.default();

// 正确的写法二
import express from 'express';
const app = express();
```

#### CommonJS 模块加载 ES6 模块

CommonJS 模块加载 ES6 模块，不能使用 require 命令，而要使用 import()函数。ES6 模块的所有输出接口，会成为输入对象的属性。

```
// es.mjs
let foo = { bar: 'my-default' };
export default foo;

// cjs.js
const es_namespace = await import('./es.mjs');
// es_namespace = {
//   get default() {
//     ...
//   }
// }
console.log(es_namespace.default);
// { bar:'my-default' }
```

上面代码中，default 接口变成了 es_namespace.default 属性。

下面是另一个例子。

```
// es.js
export let foo = { bar:'my-default' };
export { foo as bar };
export function f() {};
export class c {};

// cjs.js
const es_namespace = await import('./es');
// es_namespace = {
//   get foo() {return foo;}
//   get bar() {return foo;}
//   get f() {return f;}
//   get c() {return c;}
// }
```

### 循环加载

“循环加载”（circular dependency）指的是，a 脚本的执行依赖 b 脚本，而 b 脚本的执行又依赖 a 脚本。

```
// a.js
var b = require('b');

// b.js
var a = require('a');
```

通常，“循环加载”表示存在强耦合，如果处理不好，还可能导致递归加载，使得程序无法执行，因此应该避免出现。

但是实际上，这是很难避免的，尤其是依赖关系复杂的大项目，很容易出现 a 依赖 b，b 依赖 c，c 又依赖 a 这样的情况。这意味着，模块加载机制必须考虑“循环加载”的情况。

对于 JavaScript 语言来说，目前最常见的两种模块格式 CommonJS 和 ES6，处理“循环加载”的方法是不一样的，返回的结果也不一样。

#### CommonJS 模块的加载原理

介绍 ES6 如何处理“循环加载”之前，先介绍目前最流行的 CommonJS 模块格式的加载原理。

CommonJS 的一个模块，就是一个脚本文件。require 命令第一次加载该脚本，就会执行整个脚本，然后在内存生成一个对象。

```
{
  id: '...',
  exports: { ... },
  loaded: true,
  ...
}
```

上面代码就是 Node 内部加载模块后生成的一个对象。该对象的 id 属性是模块名，exports 属性是模块输出的各个接口，loaded 属性是一个布尔值，表示该模块的脚本是否执行完毕。其他还有很多属性，这里都省略了。

以后需要用到这个模块的时候，就会到 exports 属性上面取值。即使再次执行 require 命令，也不会再次执行该模块，而是到缓存之中取值。也就是说，CommonJS 模块无论加载多少次，都只会在第一次加载时运行一次，以后再加载，就返回第一次运行的结果，除非手动清除系统缓存。

#### CommonJS 模块的循环加载

CommonJS 模块的重要特性是加载时执行，即脚本代码在 require 的时候，就会全部执行。一旦出现某个模块被"循环加载"，就只输出已经执行的部分，还未执行的部分不会输出。

让我们来看，Node 官方文档里面的例子。脚本文件 a.js 代码如下。

```
exports.done = false;
var b = require('./b.js');
console.log('在 a.js 之中，b.done = %j', b.done);
exports.done = true;
console.log('a.js 执行完毕');
```

上面代码之中，a.js 脚本先输出一个 done 变量，然后加载另一个脚本文件 b.js。注意，此时 a.js 代码就停在这里，等待 b.js 执行完毕，再往下执行。

再看 b.js 的代码。

```
exports.done = false;
var a = require('./a.js');
console.log('在 b.js 之中，a.done = %j', a.done);
exports.done = true;
console.log('b.js 执行完毕');
```

上面代码之中，b.js 执行到第二行，就会去加载 a.js，这时，就发生了“循环加载”。系统会去 a.js 模块对应对象的 exports 属性取值，可是因为 a.js 还没有执行完，从 exports 属性只能取回已经执行的部分，而不是最后的值。

a.js 已经执行的部分，只有一行。

```
exports.done = false;
```

因此，对于 b.js 来说，它从 a.js 只输入一个变量 done，值为 false。

然后，b.js 接着往下执行，等到全部执行完毕，再把执行权交还给 a.js。于是，a.js 接着往下执行，直到执行完毕。我们写一个脚本 main.js，验证这个过程。

```
var a = require('./a.js');
var b = require('./b.js');
console.log('在 main.js 之中, a.done=%j, b.done=%j', a.done, b.done);
```

执行 main.js，运行结果如下。

```
$ node main.js

在 b.js 之中，a.done = false
b.js 执行完毕
在 a.js 之中，b.done = true
a.js 执行完毕
在 main.js 之中, a.done=true, b.done=true
```

上面的代码证明了两件事。一是，在 b.js 之中，a.js 没有执行完毕，只执行了第一行。二是，main.js 执行到第二行时，不会再次执行 b.js，而是输出缓存的 b.js 的执行结果，即它的第四行。

```
exports.done = true;
```

总之，CommonJS 输入的是被输出值的拷贝，不是引用。

另外，由于 CommonJS 模块遇到循环加载时，返回的是当前已经执行的部分的值，而不是代码全部执行后的值，两者可能会有差异。所以，输入变量的时候，必须非常小心。

```
var a = require('a'); // 安全的写法
var foo = require('a').foo; // 危险的写法

exports.good = function (arg) {
  return a.foo('good', arg); // 使用的是 a.foo 的最新值
};

exports.bad = function (arg) {
  return foo('bad', arg); // 使用的是一个部分加载时的值
};
```

上面代码中，如果发生循环加载，require('a').foo 的值很可能后面会被改写，改用 require('a')会更保险一点。

#### ES6 模块的循环加载

ES6 处理“循环加载”与 CommonJS 有本质的不同。ES6 模块是动态引用，如果使用 import 从一个模块加载变量（即 import foo from 'foo'），那些变量不会被缓存，而是成为一个指向被加载模块的引用，需要开发者自己保证，真正取值的时候能够取到值。

请看下面这个例子。

```
// a.mjs
import {bar} from './b';
console.log('a.mjs');
console.log(bar);
export let foo = 'foo';

// b.mjs
import {foo} from './a';
console.log('b.mjs');
console.log(foo);
export let bar = 'bar';
```

上面代码中，a.mjs 加载 b.mjs，b.mjs 又加载 a.mjs，构成循环加载。执行 a.mjs，结果如下。

```
$ node --experimental-modules a.mjs
b.mjs
ReferenceError: foo is not defined
```

上面代码中，执行 a.mjs 以后会报错，foo 变量未定义，这是为什么？

让我们一行行来看，ES6 循环加载是怎么处理的。首先，执行 a.mjs 以后，引擎发现它加载了 b.mjs，因此会优先执行 b.mjs，然后再执行 a.mjs。接着，执行 b.mjs 的时候，已知它从 a.mjs 输入了 foo 接口，这时不会去执行 a.mjs，而是认为这个接口已经存在了，继续往下执行。执行到第三行 console.log(foo)的时候，才发现这个接口根本没定义，因此报错。

解决这个问题的方法，就是让 b.mjs 运行的时候，foo 已经有定义了。这可以通过将 foo 写成函数来解决。

```
// a.mjs
import {bar} from './b';
console.log('a.mjs');
console.log(bar());
function foo() { return 'foo' }
export {foo};

// b.mjs
import {foo} from './a';
console.log('b.mjs');
console.log(foo());
function bar() { return 'bar' }
export {bar};
```

这时再执行 a.mjs 就可以得到预期结果。

```
$ node --experimental-modules a.mjs
b.mjs
foo
a.mjs
bar
```

这是因为函数具有提升作用，在执行 import {bar} from './b'时，函数 foo 就已经有定义了，所以 b.mjs 加载的时候不会报错。这也意味着，如果把函数 foo 改写成函数表达式，也会报错。

```
// a.mjs
import {bar} from './b';
console.log('a.mjs');
console.log(bar());
const foo = () => 'foo';
export {foo};
```

上面代码的第四行，改成了函数表达式，就不具有提升作用，执行就会报错。

我们再来看 ES6 模块加载器 SystemJS 给出的一个例子。

```
// even.js
import { odd } from './odd'
export var counter = 0;
export function even(n) {
  counter++;
  return n === 0 || odd(n - 1);
}

// odd.js
import { even } from './even';
export function odd(n) {
  return n !== 0 && even(n - 1);
}
```

上面代码中，even.js 里面的函数 even 有一个参数 n，只要不等于 0，就会减去 1，传入加载的 odd()。odd.js 也会做类似操作。

运行上面这段代码，结果如下。

```
$ babel-node
> import * as m from './even.js';
> m.even(10);
true
> m.counter
6
> m.even(20)
true
> m.counter
17
```

上面代码中，参数 n 从 10 变为 0 的过程中，even()一共会执行 6 次，所以变量 counter 等于 6。第二次调用 even()时，参数 n 从 20 变为 0，even()一共会执行 11 次，加上前面的 6 次，所以变量 counter 等于 17。

这个例子要是改写成 CommonJS，就根本无法执行，会报错。

```
// even.js
var odd = require('./odd');
var counter = 0;
exports.counter = counter;
exports.even = function (n) {
  counter++;
  return n == 0 || odd(n - 1);
}

// odd.js
var even = require('./even').even;
module.exports = function (n) {
  return n != 0 && even(n - 1);
}
```

上面代码中，even.js 加载 odd.js，而 odd.js 又去加载 even.js，形成“循环加载”。这时，执行引擎就会输出 even.js 已经执行的部分（不存在任何结果），所以在 odd.js 之中，变量 even 等于 undefined，等到后面调用 even(n - 1)就会报错。

```
$ node
> var m = require('./even');
> m.even(10)
TypeError: even is not a function
```

## webpack 与 babel 在模块化中的作用

自从使用了 es6 的模块系统后，各种地方愉快地使用 import export default，但也会在老项目中看到使用 commonjs 规范的 require module.exports。甚至有时候也会常常看到两者互用的场景。使用没有问题，但其中的关联与区别不得其解，使用起来也糊里糊涂。比如：

1. 为何有的地方使用 require 去引用一个模块时需要加上 default？ require('xx').default
2. 经常在各大 UI 组件引用的文档上会看到说明 import { button } from 'xx-ui' 这样会引入所有组件内容，需要添加额外的 babel 配置，比如 babel-plugin-component？
3. 为什么可以使用 es6 的 import 去引用 commonjs 规范定义的模块，或者反过来也可以又是为什么？
4. 我们在浏览一些 npm 下载下来的 UI 组件模块时（比如说 element-ui 的 lib 文件下），看到的都是 webpack 编译好的 js 文件，可以使用 import 或 require 再去引用。但是我们平时编译好的 js 是无法再被其他模块 import 的，这是为什么？
5. babel 在模块化的场景中充当了什么角色？以及 webpack ？哪个启到了关键作用？
6. 听说 es6 还有 tree-shaking 功能，怎么才能使用这个功能？

### webpack 模块化的原理

webpack 本身维护了一套模块系统，这套模块系统兼容了所有前端历史进程下的模块规范，包括 amd commonjs es6 等，本文主要针对 commonjs es6 规范进行说明。模块化的实现其实就在最后编译的文件内。

我编写了一个 demo 更好的展示效果。

```
// webpack

const path = require('path');

module.exports = {
  entry: './a.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js',
  }
};


// a.js
import a from './c';

export default 'a.js';
console.log(a);


// c.js

export default 333;
```

下面这段 js 就是使用 webpack 编译后的代码（经过精简），其中就包含了 webpack 的运行时代码，其中就是关于模块的实现。

```
(function(modules) {

  function __webpack_require__(moduleId) {
    var module =  {
      i: moduleId,
      l: false,
      exports: {}
    };
    modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
    return module.exports;
  }

  return __webpack_require__(0);
})([
  (function (module, __webpack_exports__, __webpack_require__) {

    // 引用 模块 1
    "use strict";
    Object.defineProperty(__webpack_exports__, "__esModule", { value: true });
    /* harmony import */ var __WEBPACK_IMPORTED_MODULE_0__c__ = __webpack_require__(1);

/* harmony default export */ __webpack_exports__["default"] = ('a.js');
console.log(__WEBPACK_IMPORTED_MODULE_0__c__["a" /* default */]);

  }),
  (function (module, __webpack_exports__, __webpack_require__) {

    // 输出本模块的数据
    "use strict";
    /* harmony default export */ __webpack_exports__["a"] = (333);
  })
]);
```

我们再精简下代码，会发现这是个自执行函数。

```
(function(modules) {

})([]);
```

自执行函数的入参是个数组，这个数组包含了所有的模块，包裹在函数中。

自执行函数体里的逻辑就是处理模块的逻辑。关键在于 **webpack_require** 函数，这个函数就是 require 或者是 import 的替代，我们可以看到在函数体内先定义了这个函数，然后调用了他。这里会传入一个 moduleId，这个例子中是 0，也就是我们的入口模块 a.js 的内容。

我们再看 **webpack_require** 内执行了

```
modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
return module.exports；
```

即从入参的 modules 数组中取第一个函数进行调用，并入参

- module
- module.exports
- webpack_require

我们再看第一个函数（即入口模块）的逻辑（精简）：

```
function (module, __webpack_exports__, __webpack_require__) {

/* harmony import */ var __WEBPACK_IMPORTED_MODULE_0__c__ = __webpack_require__(1);

    /* harmony default export */ __webpack_exports__["default"] = ('a.js');
    console.log(__WEBPACK_IMPORTED_MODULE_0__c__["a" /* default */]);

  }
```

我们可以看到入口模块又调用了 **webpack_require**(1) 去引用入参数组里的第 2 个函数。

然后会将入参的 **webpack_exports** 对象添加 default 属性，并赋值。

这里我们就能看到模块化的实现原理，这里的 **webpack_exports** 就是这个模块的 module.exports 通过对象的引用传参，间接的给 module.exports 添加属性。

最后会将 module.exports return 出来。就完成了 **webpack_require** 函数的使命。

比如在入口模块中又调用了 **webpack_require**(1)，就会得到这个模块返回的 module.exports。

但在这个自执行函数的底部，webpack 会将入口模块的输出也进行返回

```
return __webpack_require__(0);
```

目前这种编译后的 js，将入口模块的输出（即 module.exports） 进行输出没有任何作用，只会作用于当前作用域。这个 js 并不能被其他模块继续以 require 或 import 的方式引用。

### babel 的作用

按理说 webpack 的模块化方案已经很好的将 es6 模块化转换成 webpack 的模块化，但是其余的 es6 语法还需要做兼容性处理。babel 专门用于处理 es6 转换 es5。当然这也包括 es6 的模块语法的转换。

其实两者的转换思路差不多，区别在于 webpack 的原生转换 可以多做一步静态分析，使用 tree-shaking 技术（下面会讲到）

babel 能提前将 es6 的 import 等模块关键字转换成 commonjs 的规范。这样 webpack 就无需再做处理，直接使用 webpack 运行时定义的 **webpack_require** 处理。

那么 babel 是如何转换 es6 的模块语法呢？

**导出模块**

es6 的导出模块写法有

```
export default 123;

export const a = 123;

const b = 3;
const c = 4;
export { b, c };
```

babel 会将这些统统转换成 commonjs 的 exports。

```
exports.default = 123;
exports.a = 123;
exports.b = 3;
exports.c = 4;
exports.__esModule = true;
```

babel 转换 es6 的模块输出逻辑非常简单，即将所有输出都赋值给 exports，并带上一个标志 \_\_esModule 表明这是个由 es6 转换来的 commonjs 输出。

babel 将模块的导出转换为 commonjs 规范后，也会将引入 import 也转换为 commonjs 规范。即采用 require 去引用模块，再加以一定的处理，符合 es6 的使用意图。

**引入 default**

对于最常见的

```
import a from './a.js';
```

在 es6 中 import a from './a.js' 的本意是想去引入一个 es6 模块中的 default 输出。

通过 babel 转换后得到 var a = require(./a.js) 得到的对象却是整个对象，肯定不是 es6 语句的本意，所以需要对 a 做些改变。

我们在导出提到，default 输出会赋值给导出对象的 default 属性。

```
exports.default = 123;
```

所以 babel 加了个 \_interopRequireDefault 函数。

```
function _interopRequireDefault(obj) {
    return obj && obj.__esModule
        ? obj
        : { 'default': obj };
}

var _a = require('assert');
var _a2 = _interopRequireDefault(_a);

var a = _a2['default'];
```

所以这里最后的 a 变量就是 require 的值的 default 属性。如果原先就是 commonjs 规范的模块，那么就是那个模块的导出对象。

**引入 \* 通配符**

我们使用 import \* as a from './a.js' es6 语法的本意是想将 es6 模块的所有命名输出以及 defalut 输出打包成一个对象赋值给 a 变量。

已知以 commonjs 规范导出：

```
exports.default = 123;
exports.a = 123;
exports.b = 3;
exports.__esModule = true;
```

那么对于 es6 转换来的输出通过 var a = require('./a.js') 导入这个对象就已经符合意图。

所以直接返回这个对象。

```
if (obj && obj.__esModule) {
   return obj;
}
```

如果本来就是 commonjs 规范的模块，导出时没有 default 属性，需要添加一个 default 属性，并把整个模块对象再次赋值给 default 属性。

```
function _interopRequireWildcard(obj) {
    if (obj && obj.__esModule) {
        return obj;
    }
    else {
        var newObj = {}; // (A)
        if (obj != null) {
            for (var key in obj) {
                if (Object.prototype.hasOwnProperty.call(obj, key))
                    newObj[key] = obj[key];
            }
        }
        newObj.default = obj;
        return newObj;
    }
}
```

**import { a } from './a.js'**

直接转换成 require('./a.js').a 即可。

**总结**

经过上面的转换分析，我们得知即使我们使用了 es6 的模块系统，如果借助 babel 的转换，es6 的模块系统最终还是会转换成 commonjs 的规范。所以我们如果是使用 babel 转换 es6 模块，混合使用 es6 的模块和 commonjs 的规范是没有问题的，因为最终都会转换成 commonjs。

### babel5 & babel6

我们在上文 babel 对导出模块的转换提到，es6 的 export default 都会被转换成 exports.default，即使这个模块只有这一个输出。

我们经常会使用 es6 的 export default 来输出模块，而且这个输出是这个模块的唯一输出，我们会误以为这种写法输出的是模块的默认输出。

```
// a.js

export default 123;

// b.js 错误

var foo = require('./a.js')
```

在使用 require 进行引用时，我们也会误以为引入的是 a 文件的默认输出。

结果这里需要改成 var foo = require('./a.js').default

这个场景在写 webpack 代码分割逻辑时经常会遇到。

```
require.ensure([], (require) => {
   callback(null, [
     require('./src/pages/profitList').default,
   ]);
 });
```

这是 babel6 的变更，在 babel5 的时候可不是这样的。

在 babel5 时代，大部分人在用 require 去引用 es6 输出的 default，只是把 default 输出看作是一个模块的默认输出，所以 babel5 对这个逻辑做了 hack，如果一个 es6 模块只有一个 default 输出，那么在转换成 commonjs 的时候也一起赋值给 module.exports，即整个导出对象被赋值了 default 所对应的值。

这样就不需要加 default，require('./a.js') 的值就是想要的 default 值。

但这样做是不符合 es6 的定义的，在 es6 的定义里，default 只是个名字，没有任何意义。

```
export default = 123;
export const a = 123;
```

这两者含义是一样的，分别为输出名为 default 和 a 的变量。

还有一个很重要的问题，一旦 a.js 文件里又添加了一个具名的输出，那么引入方就会出麻烦。

```
// a.js

export default 123;

export const a = 123; // 新增

// b.js

var foo = require('./a.js');

// 由之前的 输出 123
// 变成 { default: 123, a: 123 }
```

所以 babel6 去掉了这个 hack，这是个正确的决定，升级 babel6 后产生的不兼容问题 可以通过引入 babel-plugin-add-module-exports 解决。

### webpack 编译后的 js，如何再被其他模块引用

通过 webpack 配置编译后的 js 是无法被其他模块引用的，webpack 提供了 output.libraryTarget 配置指定构建完的 js 的用途。

**默认 var**

如果指定了 output.library = 'test' 入口模块返回的 module.exports 暴露给全局 var test = returned_module_exports

**commonjs**

如果 library: 'spon-ui' 入口模块返回的 module.exports 赋值给 exports['spon-ui']

**commonjs2**

入口模块返回的 module.exports 赋值给 module.exports

所以 element-ui 的构建方式采用 commonjs2 ，导出的组件的 js 最后都会赋值给 module.exports，供其他模块引用。

### 模块依赖的优化

#### 按需加载的原理

我们在使用各大 UI 组件库时都会被介绍到为了避免引入全部文件，请使用 babel-plugin-component 等 babel 插件。

```
import { Button, Select } from 'element-ui'
```

由前文可知 import 会先转换为 commonjs， 即

```
var a = require('element-ui');
var Button = a.Button;
var Select = a.Select;
```

var a = require('element-ui'); 这个过程就会将所有组件都引入进来了。

所以 babel-plugin-component 就做了一件事，将 import { Button, Select } from 'element-ui' 转换成了

```
import Button from 'element-ui/lib/button'
import Select from 'element-ui/lib/select'
```

即使转换成了 commonjs 规范，也只是引入自己这个组件的 js，将引入量减少到最低。

所以我们会看到几乎所有的 UI 组件库的目录形式都是

```
|-lib
||--component1
||--component2
||--component3
|-index.common.js
```

index.common.js 给 import element from 'element-ui' 这种形式调用全部组件。

lib 下的各组件用于按需引用。

#### tree-shaking

webpack2 开始引入 tree-shaking 技术，通过静态分析 es6 的语法，可以删除没有被使用的模块。他只对 es6 的模块有效，所以一旦 babel 将 es6 的模块转换成 commonjs，webpack2 将无法使用这项优化。所以要使用这项技术，我们只能使用 webpack 的模块处理，加上 babel 的 es6 转换能力（需要关闭模块转换）。

最方便的使用方法为修改 babel 的配置。

```
// webpack

const path = require('path');

module.exports = {
  entry: './a.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js',
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /(node_modules|bower_components)/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: [['babel-preset-es2015', {modules: false}]],
          }
        }
      }
    ]
  }
};
```

```
// a.js
import a from './c';

export default 'a.js';
console.log(a);


// c.js
export default 333;

const foo = 123;
export { foo };
```

修改的点在于增加了 babel，并关闭其 modules 功能。然后在 c.js 中增加一个输出 export { foo }，但是 a.js 中并不引用它。

最后在编译出的 js 中，c.js 模块如下:

```
"use strict";
/* unused harmony export foo */
/* harmony default export */ __webpack_exports__["a"] = (333);

var foo = 123;
```

foo 变量被标记为没有使用，在最后压缩时这段会被删除。

需要说明的是，即使在引入模块时使用了 es6 ，但是引入的那个模块却是使用 commonjs 进行输出，这也无法使用 tree-shaking。

而第三方库大多是遵循 commonjs 规范的，这也造成了引入第三方库无法减少不必要的引入。

所以对于未来来说第三方库要同时发布 commonjs 格式和 es6 格式的模块。es6 模块的入口由 package.json 的字段 module 指定。而 commonjs 则还是在 main 字段指定。

## 总结

CommonJS

1. 对于基本数据类型，属于复制。即会被模块缓存。同时，在另一个模块可以对该模块输出的变量重新赋值。
2. 对于复杂数据类型，属于浅拷贝。由于两个模块引用的对象指向同一个内存空间，因此对该模块的值做修改时会影响另一个模块。
3. 当使用 require 命令加载某个模块时，就会运行整个模块的代码。
4. 当使用 require 命令加载同一个模块时，不会再执行该模块，而是取到缓存之中的值。也就是说，CommonJS 模块无论加载多少次，都只会在第一次加载时运行一次，以后再加载，就返回第一次运行的结果，除非手动清除系统缓存。
5. 循环加载时，属于加载时执行。即脚本代码在 require 的时候，就会全部执行。一旦出现某个模块被"循环加载"，就只输出已经执行的部分，还未执行的部分不会输出。

ES6 模块

1. ES6 模块中的值属于【动态只读引用】。
2. 对于只读来说，即不允许修改引入变量的值，import 的变量是只读的，不论是基本数据类型还是复杂数据类型。当模块遇到 import 命令时，就会生成一个只读引用。等到脚本真正执行时，再根据这个只读引用，到被加载的那个模块里面去取值。
3. 对于动态来说，原始值发生变化，import 加载的值也会发生变化。不论是基本数据类型还是复杂数据类型。
4. 循环加载时，ES6 模块是动态引用。只要两个模块之间存在某个引用，代码就能够执行。
