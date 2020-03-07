---
layout: post
title: Javascript作用域和变量提升
subtitle: 探索变量的生命周期
date: 2017-04-11
author: Li Yucang
catalog: true
tags:
  - js
---

# Javascript 作用域和变量提升

## js 作用域

### js 作用域分析

ES5 只有全局作用域和函数作用域，没有块级作用域，这带来很多不合理的场景：

- 内层变量可能会覆盖外层变量。
- 用来计数的循环变量泄露为全局变量。

ES6 在这个基础上引申出来一个叫做“块级作用域”的概念，即“ {} 中间的部分是一个块级作用域”。例如：for 循环、 if 逻辑判断、while 循环等语句后面的 {} 都是一个块级作用域，出于兼容性考虑，块作用域主要体现在 let、const、class、function 上，而对于 var 依旧不存在块作用域。

首先来解释一下什么是没有块级作用域?

```
var i = 1;
if (true) {
  var a = 2;
}
console.log(a); // 2
```

这里变量 a 是声明在 if 的{}里,但在 js 中对于 var 声明不存在块作用域,此时 a 的作用域是全局作用域。

再看几个例子：

```
var a = 10;
function fn() {
  console.log(a); // undefined
  var a = 20;
  console.log(a); // 20
}
fn();
console.log(a); // 10
```

这里在函数 fn 中，局部变量 a 会提升到函数最顶部,而在退出函数作用域回到全局作用域时，函数作用域中声明的变量 a 不会在影响全局作用域中 a 的值。

我们把上面的例子稍加改动，看看结果有什么不同：

```
var a = 10;
function fn() {
  console.log(a); // 10
  a = 20;
  console.log(a); // 20
}
fn();
console.log(a); // 20
```

结果竟然不一样！这里 fn 函数内的 a 没有变量声明，则是直接访问的全局变量中的 a，之后对值做的修改也会改变全局变量中 a 的值。

再来一个例子：

```
var a = 10;
if (true) {
  console.log(a); // 10
  var a = 20;
  console.log(a); // 20
}
console.log(a); // 20
```

注意这里，在 js 中 var 是不存在块级作用域的，所以这里相当于重复声明了两次，那么第二次声明会被忽略，仅当作赋值。

### 全局作用域绑定

在全局作用域中，var 声明的变量会成为全局对象（浏览器环境中的 window）的属性。这意味着 var 很可能会无意中覆盖一个已经存在的全局变量。

```
var Test=1;
window.Test === Test; // true
```

如果在全局作用域中使用 let 或 const，会在全局作用域下创建一个新的绑定，但该绑定不会添加为全局对象的属性。换句话说，用 let 或 const 不能覆盖全局变量，而只能遮蔽它。

```
const foo = 1;
window.foo = 2;
console.log(foo); // 1
console.log(window.foo); // 2
```

### ES5 中的块作用域

我们知道，ES5 环境中只有全局作用域和函数作用域，那么要想实现类似块级作用域的效果，我们通常可以通过封装函数或封装匿名自调用立即执行函数来实现。例如：

```
//函数作用域
function testFn() {
    var a = 'apple';
}
console.log(a); //Error: a is not defined

//匿名自调用实现类似“块级作用域”的效果
;(function () {
    var b = 'banana';
})();

console.log(b); //Error: b is not defined

//es5实现多重“块级作用域”类似效果
;(function () {
    var a = 'apple';

    ;(function () {
        var a = 'aaa';

        ;(function () {
            var a = 'AAA';
        })();
    })();
})();
```

### 块级绑定最佳实践的进化

ECMAScript6 标准尚在开发中时，人们普遍认为应该默认使用 let 而不是 var。对很多 JavaScript 开发者而言，let 实际上与他们想要的 var 一样，直接替换符合逻辑。这种情况下，对于需要些保护的变量则要使用 const。

然而，当更多开发者迁移到 ECMAScript6 后，另一种做法日益普及：默认使用 const，只有确实需要改变变量的值时使用 let。因为大部分变量的值在初始化后不应再改变，而预料外的变量值的改变是很多 bug 的源头。

## js 变量提升

有时候我会看见一个奇怪的实践，变量 var varname 和函数 function funcName() {...}在作用域任意地方声明：

```
// var hoisting
num;     // => undefined
var num;
num = 10;
num;     // => 10
// function hoisting
getPi;   // => function getPi() {...}
getPi(); // => 3.14
function getPi() {
  return 3.14;
}
```

num 变量在 var num 声明之前被访问，所以被认定为 undefined。

函数 function getPi() {...}被定义在末尾。由于它被提升到作用域的顶部，所以函数能在定义 getPi()之前被调用。这就是典型的提升。

事实证明，先使用后声明变量或函数可能会造成混乱。假设您在滚动查看一个大文件，突然看到一个未声明的变量...它是怎么出现在这里，又是在哪里定义的？

当然一个经验丰富的 JavaScript 开发者不会用这种方式写代码，但是在成千上万的 JavaScript GitHub 仓库中很有可能会面对这样的代码。

即使看着上面给出的代码示例，也很难理解代码中的声明流程。自然地，首先声明或描述一个未知的词语，然后才使用它来编写短语。let 鼓励遵循这种方式来使用变量。

### 探索底层：变量的生命周期

当引擎访问变量的时候，它们的生命周期包括下面几个阶段：

1. 声明阶段：在作用域中注册一个变量
2. 初始化阶段：分配内存，给作用域中的变量创建绑定。在这个阶段，变量自动地被初始化为 undefined
3. 赋值阶段：给已经初始化过的变量赋值

通过声明阶段但是没有到达初始化阶段的变量是处于未定义的状态。

![](/img/localBlog/1552642288519_5405.png)

注意，根据变量的生命周期，声明阶段和一般说的变量声明是不同的术语。简言之，引擎在三个阶段处理变量声明：声明阶段、初始化阶段和赋值阶段。

### var 变量的生命周期

熟悉了生命周期阶段，我们用它们来描述引擎是怎样处理 var 变量的。

![](/img/localBlog/1552642288798_5553.png)

假设一个场景，当 JavaScript 遇到一个作用域里有一个 var 声明的变量的函数。在任何语句执行之前，这个变量就通过了声明阶段和初始化阶段（第一步）。 var variable 在函数作用域中声明的位置不影响声明和初始化阶段。在声明和初始化之后，赋值阶段之前，变量值为 undefined 并且可以被访问使用。在赋值阶段，variable = 'value'，变量会得到它的初始值（第二步）。严格来说，提升的概念是在函数作用域的顶部声明和初始化变量。在声明和初始化阶段之间没有间隙。来研究一个例子。下面的代码创建了一个带有 var 变量的函数：

```
function multiplyByTen(number) {
  console.log(ten); // => undefined
  var ten;
  ten = 10;
  console.log(ten); // => 10
  return number * ten;
}
multiplyByTen(4); // => 40
```

当 JavaScript 开始执行 multipleByTen(4)的时候，它就进入了 multipleByTen 的函数作用域，在第一条语句之前，变量 ten 完成了声明和初始化阶段。所以调用 console.log(ten)会打印出 undefined。语句 ten = 10 分配了一个初始值。分配之后，console.log(ten)正确地输出了 10。

### 函数声明的生命周期

假设一个函数声明语句 function funName() {...}，这更加容易。

![](/img/localBlog/1552642289250_8506.png)

声明、初始化、赋值阶段在函数作用域的开始立刻执行（只有一步）。funcName()可以在该作用域的任何地方调用，不依赖于声明语句的位置（甚至可以在最后）。

下面的代码示例展示了函数提升：

```
function sumArray(array) {
  return array.reduce(sum);
  function sum(a, b) {
    return a + b;
  }
}
sumArray([5, 10, 8]); // => 23
```

当 JavaScript 调用 sumArray([5, 10, 8])的时候，它就进入了 sumArray 的函数作用域。在作用域里面，在所有语句执行之前，sum 通过了三个阶段：声明、初始化、赋值。这种方式，array.reduce(sum)甚至可以在声明语句 function sum(a, b) {...}之前使用 sum。

#### 块级作用域中的函数声明

ES5 规定，函数只能在全局作用域和函数作用域之中声明，不能在“块级作用域”中声明。事实上，各大浏览器并没有遵守这个规则。

为此，ES6 引入了块级作用域，明确允许在块级作用域之中声明函数。在 ES6 的块级作用域之中，函数声明语句的行为类似于 let，在块级作用域之外不可引用；但又有别于 let 命令，允许重复声明同名函数且存在函数变量提升。

块级作用域中的函数特征：

1. 允许在块级作用域内声明函数。
2. 在块级作用域之外不可引用，内层块作用域声明的函数不干扰外层作用域的函数。
3. 允许重复声明同名函数且函数变量提升到块作用域顶部(包括声明、初始化、赋值)。

允许块级作用域内声明函数：

```
if (true) {
    function testFn(){
        //handle
    }
}
```

同样存在函数变量提升：

```
if (true) {
    testFn(); //这里正常输出“hehehe”，表明testFn函数变量产生提升

    function testFn() {
        console.log('hehehe')
    }
}
```

内层作用域声明的同名函数不干扰外层作用域的同名函数：

```
{
    testFn(); //I am outside.

    {
        testFn(); //I am inside.

        function testFn() {
            console.log('I am inside.')
        }
    }

    function testFn() {
        console.log('I am outside.')
    }

    testFn(); //I am outside.
}
```

一切似乎都很美好，然而我们看这么一段代码：

```
function testFn() {
    console.log('outside')
}

(() => {
    if (false) {
        function testFn() {
            console.log('inside')
        }
    }

    testFn(); // Error: testFn is not a function
})();
```

ES6 理论上会得到“I am outside!”。因为块级作用域内声明的函数类似于 let，对作用域之外没有影响。但是，如果你真的在 ES6 浏览器中运行一下上面的代码，是会报错的，这是为什么呢？

经查验相关资料，原因是若改变了块级作用域内声明的函数的处理规则，显然会对老代码产生很大影响。为了减轻因此产生的不兼容问题，ES6 规定，浏览器的实现可以不遵守上面的规定，允许有自己的行为方式：

1. 允许在块级作用域内声明函数。
2. 函数声明类似于 var，即会提升到全局作用域或函数作用域的头部。
3. 同时，函数声明还会提升到所在的块级作用域的头部。

oh my god! 如果我们的代码在**非严格模式**下运行，我们的代码不会完全按照 es6 的规范运行。根据这三条规则，在浏览器的 ES6 环境中，块级作用域内声明的函数，行为类似于 var 声明的变量。上面的报错代码其实运行的是：

```
// 浏览器的 ES6 环境
function testFn() {
    console.log('outside')
}

(() => {
    var testFn = undefined;
    if (false) {
        function testFn() {
            console.log('inside')
        }
    }

    testFn(); // Error: testFn is not a function
})();
```

考虑到环境导致的行为差异太大，应该避免在块级作用域内声明函数。建议,若需要在块级作用域内声明函数，应写成函数表达式，而不是函数声明语句。

##### 严格模式下

在定义函数的代码块内，块级函数会提升至顶部：

```
"use strict";
if (1) {
    console.log(typeof doSomething);    // function
    function doSomething() { }
    doSomething();
}
console.log(typeof doSomething);    // undefined
```

在块作用域中的函数声明不会影响到块作用域之外：

```
"use strict";
function testFn() {
    console.log('outside')
}

(() => {
    if (false) {
        function testFn() {
            console.log('inside')
        }
    }

    testFn(); // 'outside'
})();
```

##### 非严格模式下

与严格模式下稍有不同，这些函数不在提升至代码块的顶部，而是提升至外围函数或全局作用域的顶部：

```
if (true) {
    console.log(typeof doSomething);    // function
    function doSomething() { }
    doSomething();
}
console.log(typeof doSomething);        // function
```

```
function testFn() {
    console.log('outside')
}

(() => {
    if (false) {
        function testFn() {
            console.log('inside')
        }
    }

    testFn(); //Error: testFn is not a function
})();
```

### let 变量的生命周期

let 变量的处理方式和 var 不同。最主要的区别就是声明和初始化阶段被分开了。

![](/img/localBlog/1552642289501_7079.png)

现在假设解释器进入了一个块级作用域，作用域包含一个 let variable 语句。变量立刻通过了声明阶段，在作用域注册了它的名字（步骤 1）。

接着解释器一行一行的解析语句。

如果在这个阶段尝试访问变量 variable，JavaScript 会抛出 ReferenceError: variable is not defined。因为变量的状态是未定义的，variable 在暂时性死区。

当解释器到达 let variable 语句的时候，初始化阶段通过（步骤 2）。现在变量的状态是已定义并且访问它会得到 undefined。

变量退出了暂时性死区。

稍后，当赋值语句 variable = 'value'出现，就通过了赋值阶段（步骤 3）。

如果 JavaScript 遇到 let variable = 'value'，那么初始化和赋值会发生在这一条语句上。

来看一个例子。在块级作用域里面用 let 声明一个变量 variable：

```
let condition = true;
if (condition) {
  // console.log(number); // => Throws ReferenceError
  let number;
  console.log(number); // => undefined
  number = 5;
  console.log(number); // => 5
}
```

当 JavaScript 进入 if (condition) {...}块级作用域的时候，number 立刻通过了声明阶段。

因为 number 处于未定义的状态，处于暂时性死区，试图访问这个变量会抛出 ReferenceError: number is not defined。

后来，语句 let number 完成了初始化。现在这个变量可以访问了，但是它的值是 undefined。

赋值语句 number = 5 当然就是完成了赋值阶段。

const 和 class 类型和 let 有相同的生命周期，除了赋值只能发生一次。

如上所述，提升就是变量在作用域顶部进行声明和初始化。但是 let 的生命周期将声明和初始化两个阶段解耦了。解耦让提升这个术语失效了。

两个阶段中间的间隙创建了暂时性死区，在这里，变量不能被访问。

以一种科幻的风格，在 let 生命周期中，提升这个术语的崩塌创造了暂时性死区。

## 结语

### 回顾

讲了这么多，我们再来几个例子帮大家回顾一下：

```
var foo = 1;
function bar() {
    if (!foo) {
        var foo = 10;
    }
    console.log(foo);
}
bar();
```

有人认为这里!foo 是 false,那么赋值语句应该不执行.那么输出结果应该是 1,however,结果是 10！

再来一个例子：

```
var a = 1;
function b() {
    a = 10;
    return;
    function a() {}
}
b();
console.log(a);
```

看到这个例子,大家会认为,哎,这里里面的 a = 10 会覆盖外面的 a = 1,觉得答案是 10.但是~,答案是 1！！哈哈哈，答错了别慌，重头看一遍本文吧，知识要慢慢沉淀哟。

在 Javascript 中,变量进入一个作用域可以通过下面四种方式：

1. 语言自定义变量:所有的函数作用域中都存在 this 和 arguments 这两个默认变量
2. 函数形参:函数的形参存在函数作用域中
3. 函数声明:function foo() {}
4. 变量定义:var foo

JS 代码真正执行过程中,在代码运行前，函数声明和变量定义通常会被解释器移动到其所在作用域的最顶部，这就是所谓的变量提升。

对于 var a = 1 这句话而言,我们可以拆分成两部分来看变量定义与变量赋值.不管 var a = 1 在作用域的什么地方,变量定义会被提升到到当前作用域的顶部来执行。

对于 JS 中函数定义,我们可以采取 function foo(){}或者采取 var foo = function(){}(这里我们不谈 var foo = new Function()这么烦的方式).这两种方式有什么区别?

```
function test() {
    foo(); // TypeError "foo is not a function"
    bar(); // "this will run!"
    var foo = function () { // function expression assigned to local variable 'foo'
        console.log("this won't run!");
    }
    function bar() { // function declaration, given the name 'bar'
        console.log("this will run!");
    }
}
test();
```

这里例子等价于：

```
function test() {
	//hoist variable foo and function bar
	var foo;	//undefined
	function bar() {
		console.log("this will run!");
	}
	foo();
	bar();
	foo = function () {
		console.log("this won't run!");
	}
}
test();
```

function foo(){}提升的是整个函数定义。var foo = function(){}提升的是 foo 变量的定义。

结合刚刚说的，我们回头来看之前引子中的例子:

```
var foo = 1;
function bar() {
    if (!foo) {
        var foo = 10;
    }
    console.log(foo);
}
bar();
```

这里等价于：

```
var foo;
foo = 1;
function bar(){
	var foo;//undefined
	if(!foo){
		foo = 10;
	}
	console.log(foo);
}
bar();
```

在函数 bar 外面 foo 为 1,但在函数 bar 内,定义一个新的 foo,这个时候 foo 是 undefined,！foo 就变成了 true,执行赋值操作,foo=10;这个时候输出的 foo 是函数 bar 内刚定义的 foo,输出的值为 10;

另外一个例子：

```
var a = 1;
function b() {
    a = 10;
    return;
    function a() {}
}
b();
console.log(a);
```

等价于：

```
var a;
a = 1;
function b(){
	function a(){};
	a = 10;
	return;
}
b();
console.log(a);
```

这里函数 b 的外部定义了 a,值为 1.函数 b 内,声明函数 a 的过程被提升(hoist),之后 a 被赋值为 10;由于是函数 b 中新定义了 a,所以对函数 b 外部的 a 并不造成影响,最后得 console.log(a)输出的值还是 1;

### 使用 es6 语法避免可能问题

再 es6 出来以后，我们通过 es6 语法糖可以避免一些作用域以及变量提升带来的问题.比如 let 的使用.这里举两个例子,一个关于变量的,一个关于函数的。

#### ES6 中关于变量的例子

用 let 替换之前例子中的 var：

```
let foo = 1;
function bar() {
    if (!foo) {
        let foo = 10;
    }
    console.log(foo);
}
bar();
```

babel 编译后生成:

```
var foo = 1;
function bar() {
    if (!foo) {
        var _foo = 10;
    }
    console.log(foo);
}
bar();
```

执行结果是 1;

#### ES6 中关于函数的例子

```
function f() { console.log('I am outside!'); }
if(true) {
   // 重复声明一次函数f
   function f() { console.log('I am inside!'); }
}
f();
```

ES5 执行结果是 I am inside!,两次 function f(){}的声明都被提升了,赋值取最后一次赋值.所以执行结果是 I am inside! ES6 以 babel 编译为例,编译出来结果是：

```
'use strict';

function f() {
   console.log('I am outside!');
}
if (true) {
   // 重复声明一次函数f

   var _f = function _f() {
      console.log('I am inside!');
   };
}
f();
```

babel 编译自动把里面同名函数 f 变成了\_f.执行结果是 I am outside!注意这里要加严格模式，原因上文有说。
