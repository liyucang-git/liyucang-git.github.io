---
layout: post
title: :first-of-type的误用
subtitle: 探索css伪类选择符
date: 2019-07-16
author: Li Yucang
catalog: true
tags:
  - css
  - css伪类选择符
---

# :first-of-type 的误用

近期在使用:first-of-type + class 定位元素的时候发现不起作用，所以翻阅资料小小的整理总结一下。

相信大家对 E:first-child 、E:nth-child 这种伪类选择符 Pseudo-Classes Selectors 的写法并不陌生，但其实伪类选择符不仅仅只有你经常用的这几种写法，如下表：

| 语法关键词：匹配父元素内的 | 语法关键词：匹配父元素内同类型标签元素中的 |
| -------------------------- | ------------------------------------------ |
| E:first-child              | E:first-of-type                            |
| E:last-child               | E:last-of-type                             |
| E:only-child               | E:only-of-type                             |
| E:nth-child(n)             | E:nth-of-type(n)                           |
| E:nth-last-child(n)        | E:nth-last-of-type(n)                      |

观察上面表格可发现，我们发现左侧伪类选择符跟右侧伪类选择符是一一对应的，单词拼写无非就是将-child 部分替换成-of-type,这就意味着不用一个个去死记硬背所有的伪类选择符，只要拿其中一对伪类选择符 做下用法对比就可以掌握左右两侧所有伪类选择符的用法区别，下面我们以 E:first-child vs E:first-of-type 为例说明两者之间的用法区别，来个例子试试看吧！

```
<section class="section section1">
    <header>header</header>
    <p class="section1-item">111</p>
    <p class="section1-item">222</p>
    <p class="section1-item">333</p>
    <p class="section1-item">444</p>
    <footer>footer</footer>
</section>
```

**E:first-child**

匹配父元素的第一个子元素 E：

```
header:first-child {
  color: green;
}
// 找到第一个子元素，并且匹配header标签，即<header>header</header>
```

如果 E 不是第一个子元素，则会匹配失败：

```
.section1-item:first-child {
  color: green;
}
// 找到第一个子元素，该元素不匹配.section1-item，匹配失败
```

**E:first-of-type**

匹配父元素内同类型标签元素中的第一个元素 E：

```
p:first-of-type{
  color: green;
}
// 在子元素中找出所有p标签，匹配第一个p标签，即<p class="section1-item">111</p>
```

一切都如我们设想的一样，不过，我们改一下上面的代码片段：

```
<section class="section section1">
    <header class="section1-item">header</header>
    <p class="section1-item">111</p>
    <p class="section1-item">222</p>
    <p class="section1-item">333</p>
    <p class="section1-item">444</p>
    <footer>footer</footer>
</section>

.section1-item:first-of-type{
  color: green;
}
//匹配到了<header class="section1-item">header</header>和<p class="section1-item">111</p>
```

问题来了，居然给两个元素都加上了绿色！

我们继续把上面的 HTML 代码片段进行修改，改成：

```
<section class="section section1">
    <header class="section1-item">test</header>
    <header class="section1-item">header</header>
    <p class="section1-item">111</p>
    <p class="section1-item">222</p>
    <p class="section1-item">333</p>
    <p class="section1-item">444</p>
    <footer>footer</footer>
</section>

.section1-item:first-of-type{
  color: green;
}
//匹配到了<header class="section1-item">test</header>和<p class="section1-item">111</p>
```

这下答案就很明显了：

E:first-of-type 不仅与 someselector 这个选择器相关，还与符合该选择器的元素的标签相关。

再看一下 W3C 的说明：

> The :last-of-type pseudo-class represents an element that is the last sibling of its type in the list of children of its parent element.

这里所谓 the last sibling of its type 其实讲得并不是很清楚，但仔细琢磨，这里的 type 的意思应该就是不仅仅与选择器相关，还与标签相关。

**总结**

好了，我们用下面的代码再仔细的讲解一遍：

```
<section class="section section1">
    <header class="section1-item">test</header>
    <header class="section1-item">header</header>
    <p class="section1-item">111</p>
    <p class="section1-item">222</p>
    <p class="section1-item">333</p>
    <p class="section1-item">444</p>
    <footer>footer</footer>
</section>

.section1-item:first-of-type{
  color: green;
}
//匹配到了<header class="section1-item">test</header>和<p class="section1-item">111</p>
```

css 匹配步骤：

1. 找到所有 class 为 section1-item 的标签，上面的 Dom 结构里有`<header>`、`<p>`;

2. 在子元素中找到所有的 header、p 标签，分别记作 list1，list2；

3. 如果 list1、list2 中第一个标签的 class 是 section1-item，则字体改为 blue;

**在 .class:first-of-type 的匹配过程中，会找到所有匹配 .class 的多个标签 list，然后依次判断第一个元素是否符合 E，如果符合则匹配成功。:first-of-type 的本质是搜索对应的标签 list，并匹配处于特定位置的元素。**

如果我们需要匹配处于特殊位置的子元素，我们有时候可以用:nth-child 来替代:first-of-type：

```
<section class="section section1">
    <div>div</div>
    <header class="section1-item">header</header>
    <p class="section1-item">111</p>
    <p class="section1-item">222</p>
    <p class="section1-item">333</p>
    <p class="section1-item">444</p>
    <footer>footer</footer>
</section>

.section1-item:nth-child(3)
或
.section1-item:nth-of-type(2)

匹配到<p class="section1-item">222</p>
```
