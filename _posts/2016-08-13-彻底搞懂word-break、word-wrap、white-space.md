---
layout:     post
title:      彻底搞懂word-break、word-wrap、white-space
# subtitle:
date:       2016-08-13
author:     Li Yucang
catalog: true
tags:
    - css
---

# 彻底搞懂word-break、word-wrap、white-space

今天我们来将一下css中三个让人迷惑而傻傻分不清的属性，在平时我们可能有用到过这些属性，但我们真的有把它们弄懂吗？今天我来带大家把这三个属性彻底搞清楚！

本篇文章使用的测试代码如下：

````
.box{
  width: 180px;
  border: 1px solid black;
}

<div class="box">
  Hi&nbsp;&nbsp;&nbsp;&nbsp;,
  I          am a boy who lovelovelovelovelovelove you.
  </br>
  你好&nbsp;&nbsp;&nbsp;&nbsp;，
  我          是一个超级喜欢你的男孩。
</div>
````
在不加任何样式时效果如下；

![](http://cdn.vivigo.xyz/blog/1536162047819_blog.png)

可以看到，nbsp;和</br>可以正常发挥作用，而连续的空格会被缩减成一个（比如This和is之间的三个空格变成了一个），换行符也全都无效。句子超过一行后会自动换行，而长度超过一行的单个单词会超出边界。接下来我们看下， 给它上面三个css属性赋值后会出现什么变化。

## white-space

正如它的名字，这个属性是用来控制空白字符的显示的，同时还能控制是否自动换行。它有五个值：normal | nowrap | pre | pre-wrap | pre-line。因为默认是normal，所以我们主要研究下其它四种值时的展现情况。（为了方便比较，下文所有图，左侧为normal即初始情况，右侧为赋上相应值时的情况）

### white-space:nowrap

先看下white-space:nowrap时的情况：

![](http://cdn.vivigo.xyz/blog/1536162148107_blog.png)

不仅空格被合并，换行符无效，连原本的自动换行都没了！只有</br>才能导致换行！所以这个值的表现还是挺简单的，我们可以理解为永不换行。

### white-space:pre

![](http://cdn.vivigo.xyz/blog/1536162322727_blog.png)

空格和换行符全都被保留了下来！不过自动换行还是没了。保留，所以pre其实是preserve的缩写，这样就好记了。

### white-space:pre-wrap

![](http://cdn.vivigo.xyz/blog/1536162409019_blog.png)

显然pre-wrap就是preserve+wrap，保留空格和换行符，且可以自动换行。

### white-space:pre-line

![](http://cdn.vivigo.xyz/blog/1536162474427_blog.png)

空格被合并了，但是换行符可以发挥作用,自动换行还在，所以pre-line其实是preserve+wrap+空格合并。

整理一下这几个属性：

![](http://cdn.vivigo.xyz/blog/1536163603920_blog.png)

## word-break

从这个名字可以知道，这个属性是控制单词如何被拆分换行的。它有三个值：normal | break-all | keep-all。

### word-break:keep-all

![](http://cdn.vivigo.xyz/blog/1536163733070_blog.png)

所有“单词”一律不拆分换行，注意，我这里的“单词”包括连续的中文字符（还有日文、韩文等），或者可以理解为只有空格可以触发自动换行

### word-break:break-all

![](http://cdn.vivigo.xyz/blog/1536163795278_blog.png)

所有单词碰到边界一律拆分换行，不管你是incomprehensibilities这样一行都显示不下的单词，还是long这样很短的单词，只要碰到边界，都会被强制拆分换行。所以用word-break:break-all时要慎重呀。

这样的效果好像并不太好呀，能不能就把incomprehensibilities拆一下，其它的单词不拆呢？那就需要下面这个属性了：

## word-wrap（overflow-wrap）

word-wrap又叫做overflow-wrap：

>word-wrap 属性原本属于微软的一个私有属性，在 CSS3 现在的文本规范草案中已经被重名为 overflow-wrap 。 word-wrap 现在被当作 overflow-wrap 的 “别名”。 稳定的谷歌 Chrome 和 Opera 浏览器版本支持这种新语法。

这个属性也是控制单词如何被拆分换行的，实际上是作为word-break的互补，它只有两个值：normal | break-word，那我们看下break-word：

![](http://cdn.vivigo.xyz/blog/1536163971265_blog.png)

终于达到了上文我们希望的效果，只有当一个单词一整行都显示不下时，才会拆分换行该单词。所以我觉得overflow-wrap更好理解好记一些，overflow，只有长到溢出的单词才会被强制拆分换行！（其实前面的word-break属性除了列出的那三个值外，也有个break-word值，效果跟这里的word-wrap:break-word一样，然而只有Chrome、Safari等部分浏览器支持）

## 总结

最后总结一下三个属性：

* white-space，控制空白字符的显示，同时还能控制是否自动换行。它有五个值：normal | nowrap | pre | pre-wrap | pre-line
* word-break，控制单词如何被拆分换行。它有三个值：normal | break-all | keep-all
* word-wrap（overflow-wrap）控制长度超过一行的单词是否被拆分换行，是word-break的补充，它有两个值：normal | break-word

相信读完了本文，你应该对white-space、word-break、word-wrap有比较系统的认识了吧，如果短时间还是记不住那就赶快收藏起来下次再来看一遍吧。