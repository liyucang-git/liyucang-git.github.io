---
layout: post
title: 剖析JavaScript类型转换
subtitle: 带你走出类型转换的迷宫
date: 2018-02-22
author: Li Yucang
catalog: true
tags:
    - js
---

# 剖析JavaScript类型转换 

js在使用`==`进行比较的时候，如果`==`两边类型不同则会发生隐式类型转换。很多初学者对这块理解的不够，今天就让我们一起把它弄清楚吧，先来一段典型的具有迷惑性的代码:

````
[]==[]
//false
[]==![]
//true
{}==!{}
//false
{}==![]
//VM1896:1 Uncaught SyntaxError: Unexpected token ==
![]=={}
//false
[]==!{}
//true
undefined==null
//true
````
看了这些比较结果是不是一脸懵逼呢，不要慌，接着往下看。我们就从[] == []和[] == ![]例子切入分析一下为什么输出的结果是true而不是其它的呢?

## ECMAScript规范里面的==

我们查阅规范文档找到其中关于`==`的内容：

* 当比较数字和字符串时，字符串会转换成数字值。 JavaScript 尝试将数字字面量转换为数字类型的值。 首先, 一个数学上的值会从数字字面量中衍生出来，然后得到被四舍五入后的数字类型的值。
* 如果其中一个操作数为布尔类型，那么布尔操作数如果为true，那么会转换为1，如果为false，会转换为整数0，即0。
* 如果一个对象与数字或字符串相比较，JavaScript会尝试返回对象的默认值。操作符会尝试通过方法valueOf和toString将对象转换为其原始值（一个字符串或数字类型的值）。如果尝试转换失败，会产生一个运行时错误。
* 注意：当且仅当与原始值比较时，对象会被转换为原始值。当两个操作数均为对象时，它们作为对象进行比较，仅当它们引用相同对象时返回true。

我们整理规范如下：

1. 如果一个运算数是 Boolean 值，在检查相等性之前，把它转换成数字值。false 转换成 0，true 为 1。
2. 如果一个运算数是字符串，另一个是数字，在检查相等性之前，要尝试把字符串转换成数字。这里注意字符串是能通过 Number(str) 转化为数字，若果不符合 Number 函数要求的格式会转化为 NaN。
3. 如果一个对象与数字或字符串相比较，JavaScript会尝试通过toPrimitive将对象转换为其原始值，具体的后面细说。
4. 值 null 和 undefined 相等。且在检查相等性时，不能把 null 和 undefined 转换成其他值，即null合undefined与其他值都是不相等的。
5. NaN与所有值都不相等，即使两个数都是 NaN，等号仍然返回 false，因为根据规则，NaN 不等于 NaN。
6. 如果两个运算数都是对象，那么比较的是它们的引用值。如果两个运算数指向同一对象，那么等号返回 true，否则两个运算数不等。

## 从[] == ![]切入分析

把规范一看大家是不是有了点眉目，我们就从 [] == ![]例子切入分析一下为什么输出的结果是true。

### 运算优先级

首先在进行比较之前由于 ! 的优先级高于 ==，所以会先对 ![]求值，这里我们了解一下常见运算符的优先级：

![](http://cdn.vivigo.xyz/blog/1552642292601_7741.jpeg)

### toBoolean

!称为逻辑非运算符，会先对目标值进行布尔类型转化（toBoolean），至于怎么转我们看下图：

![](http://cdn.vivigo.xyz/blog/1552642292934_1357.png)

[]是一个对象,所以对应转换成Boolean对象的值为true;那么![]对应的Boolean值就是false。

### toNumber

根据我们整理的规范条件1，`==`运算符会将`true`转化（toNumber）为 1。我们平时使用 + 或者 Number 函数来进行数字转化就是根据toNumber规则，具体转化规则如下：

![](http://cdn.vivigo.xyz/blog/1552642293375_788.png)

注意表格中的String转化Number时，对字符串有严格要求，像：'3f'、'7.3.4'等不符合规范的值都会转化为NaN。

由此得出false被转化成了0，此时的比较变成了[]==0。

### toPrimitive

在此处因为 [] 是对象，0是数字Number，根据我们整理的规范3返回比较toPrimitive([]) == 0的结果。

再来看看ECMAScript标准怎么定义toPrimitive方法的:

![](http://cdn.vivigo.xyz/blog/1552642294227_1782.png)

是不是看了这个定义,还是一脸懵逼,toPrimitive这尼玛什么玩意啊?这不是等于没说吗?别慌，经过翻阅资料，上面要说的可以概括成:


````
toPrimitive(obj,preferredType)

JS引擎内部转换为原始值toPrimitive(obj,preferredType)函数接受两个参数，第一个obj为被转换的对象，第二个
preferredType为希望转换成的类型（默认为空，接受的值为Number或String）。

在执行toPrimitive(obj,preferredType)时如果第二个参数为空并且obj为Date的实例时，此时preferredType会
被设置为String，其他情况下preferredType都会被设置为Number。

如果preferredType为Number，toPrimitive执行过程如下：
1. 如果obj为原始值，直接返回；
2. 否则调用 obj.valueOf()，如果执行结果是原始值，返回之；
3. 否则调用 obj.toString()，如果执行结果是原始值，返回之；
4. 否则抛异常。

如果preferredType为String，将上面的第2步和第3步调换，即：
1. 如果obj为原始值，直接返回；
2. 否则调用 obj.toString()，如果执行结果是原始值，返回之；
3. 否则调用 obj.valueOf()，如果执行结果是原始值，返回之；
4. 否则抛异常。
````

首先我们要明白obj.valueOf()和 obj.toString()还有原始值分别是什么意思,这是弄懂上面描述的前提之一。

### valueOf

JavaScript调用valueOf方法将对象转换为原始值。你很少需要自己调用valueOf方法；当遇到要预期的原始值的对象时，JavaScript会自动调用它。

默认情况下，valueOf方法由Object后面的每个对象继承。 每个内置的核心对象都会覆盖此方法以返回适当的值。如果对象没有原始值，则valueOf将返回对象本身。

JavaScript的许多内置对象都重写了该函数，以实现更适合自身的功能需要。因此，不同类型对象的valueOf()方法的返回值和返回值类型均可能不同，具体见下图：

![](http://cdn.vivigo.xyz/blog/1552642295073_5002.jpg)

````
// Array：返回数组对象本身
var array = ["ABC", true, 12, -5];
console.log(array.valueOf() === array);   // true

// Date：当前时间距1970年1月1日午夜的毫秒数
var date = new Date(2013, 7, 18, 23, 11, 59, 230);
console.log(date.valueOf());   // 1376838719230

// Number：返回数字值
var num =  15.26540;
console.log(num.valueOf());   // 15.2654

// 布尔：返回布尔值true或false
var bool = true;
console.log(bool.valueOf() === bool);   // true

// new一个Boolean对象
var newBool = new Boolean(true);
// valueOf()返回的是true，两者的值相等
console.log(newBool.valueOf() == newBool);   // true
// 但是不全等，两者类型不相等，前者是boolean类型，后者是object类型
console.log(newBool.valueOf() === newBool);   // false

// Function：返回函数本身
function foo(){}
console.log( foo.valueOf() === foo );   // true
var foo2 =  new Function("x", "y", "return x + y;");
console.log( foo2.valueOf() );
/*
ƒ anonymous(x,y
) {
return x + y;
}
*/

// Object：返回对象本身
var obj = {name: "张三", age: 18};
console.log( obj.valueOf() === obj );   // true

// String：返回字符串值
var str = "http://www.xyz.com";
console.log( str.valueOf() === str );   // true

// new一个字符串对象
var str2 = new String("http://www.xyz.com");
// 两者的值相等，但不全等，因为类型不同，前者为string类型，后者为object类型
console.log( str2.valueOf() === str2 );   // false
````

### toString

toString用来返回对象的字符串表示，每个对象都有一个toString()方法，当该对象被表示为一个文本值时，或者一个对象以预期的字符串方式引用时自动调用。默认情况下，toString()方法被每个Object对象继承。如果此方法在自定义对象中未被覆盖，toString() 返回 "[object type]"，其中type是对象的类型。

Array对象覆盖了Object的 toString 方法。对于数组对象，toString 方法连接数组并返回一个字符串，其中包含用逗号分隔的每个数组元素。

Date对象也覆盖了Object的 toString 方法，返回一个时间字符串。

````
var obj = {};
console.log(obj.toString());//[object Object]

var arr2 = [];
console.log(arr2.toString());//""空字符串
  
var date = new Date();
console.log(date.toString());//Sun Feb 28 2016 13:40:36 GMT+0800 (中国标准时间)
````

### 原始值

原始值指的是'Null','Undefined','String','Boolean','Number'五种基本数据类型之一。

### 回到[] == ![]

上面说了那么多，如果觉得描述还不好明白,一大堆描述晦涩又难懂,我们用代码说话，简单实现一个toPrimitive：

````
const toPrimitive = (obj, preferredType='Number') => {
    let Utils = {
        typeOf: function(obj) {
            return Object.prototype.toString.call(obj).slice(8, -1);
        },
        isPrimitive: function(obj) {
            let types = ['Null', 'String', 'Boolean', 'Undefined', 'Number'];
            return types.indexOf(this.typeOf(obj)) !== -1;
        }
    };
   
    if (Utils.isPrimitive(obj)) {
        return obj;
    }
    
    preferredType = (preferredType === 'String' || Utils.typeOf(obj) === 'Date') ?
     'String' : 'Number';

    if (preferredType === 'Number') {
        if (Utils.isPrimitive(obj.valueOf())) {
            return obj.valueOf()
        };
        if (Utils.isPrimitive(obj.toString())) {
            return obj.toString()
        };
    } else {
        if (Utils.isPrimitive(obj.toString())) {
            return obj.toString()
        };
        if (Utils.isPrimitive(obj.valueOf())) {
            return obj.valueOf()
        };
    }
}

var a={};
toPrimitive(a);
````

接着比较toPrimitive([]) == 0，现在我们知道toPrimitive([])="",也就是空字符串。按照我们规范的第二条，将字符串转化为数字：toNumber("")==0，根据之前的toNumber规范，空字符串转化为0，最后成了0 == 0，答案显而易见为true，一波三折。

## 总结

### 关于toPrimitive的更深层次理解

这里大家注意一点，前面提到的toPrimitive中会使用到的toString和valueOf方法都是可以进行覆盖的。

valueOf返回为原始值，toPrimitive直接取valueOf值：

````
var x = {
  toString() {
    return 3;
  },
  valueOf() {
    return 2;
  }
}

x == 2; // true
````

valueOf返回不为原始值，toString返回为原始值，toPrimitive取toString值：

````
var x = {
  toString() {
    return 3;
  },
  valueOf() {
    return [];
  }
}

x == 3; // true
````

当valueOf和toString返回不为原始值时，进行比较会报错：

````
var x = {
  toString() {
    return {};
  },
  valueOf() {
    return [];
  }
}

x == 3; // VM1873:10 Uncaught TypeError: Cannot convert object to primitive value
````

注意，根据我们上文说的，null、undefined、Boolean也都属于原始值，我们来尝试验证一下：

````
var x = {
  toString() {
    return 3;
  },
  valueOf() {
    return null;
  }
}

x == 3; // false 说明没有采用toString返回的3，而是采用了valueOf的返回值null

x == null; //false 此时直接与null比较，x作为对象不会进行类型转换，直接返回false

x == undefined; //false 与null同理

x != x; // false 这里是两个对象进行比较并不会进行类型转换，直接比较引用

+x; // 0 相当于toPrimitive(x,'Number')，再次验证确实返回了null

我们把返回改成undefined：
var x = {
  toString() {
    return 3;
  },
  valueOf() {
    return undefined;
  }
}
+x; // NaN 相当于toPrimitive(x,'Number')，ok，看来确实返回了undefined

再试试Boolean：
var x = {
  toString() {
    return 3;
  },
  valueOf() {
    return true;
  }
}
+x; // 1 true -> 1，完美
````

好了，说了那么多我们来总结一下：

![](http://cdn.vivigo.xyz/blog/1552642295348_7237.jpg)

根据我们得到的最终的图，我们总结一下==运算的规则：

1. undefined == null，结果是true。且它俩与所有其他值比较的结果都是false。

2. String == Boolean，需要两个操作数同时转为Number。

3. String/Boolean == Number，需要String/Boolean转为Number。

4. Object == Primitive，需要Object转为Primitive(具体通过valueOf和toString方法)。

瞧见没有，一共只有4条规则！是不是很清晰、很简单。

### 实战建议

理解js `==`运算符的运作规则固然很重要，但在我们真正的项目中建议大家使用`===`来替代`==`，全等运算符`===`不会进行类型转换，能使我们的代码更加清晰哦！
