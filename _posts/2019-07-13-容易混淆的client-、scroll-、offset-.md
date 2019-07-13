---
layout: post
title: 容易混淆的client-、scroll-、offset-
subtitle: js获取DOM元素位置和尺寸大小
date: 2019-07-13
author: Li Yucang
catalog: true
tags:
  - css
---

# 容易混淆的 client-、scroll-、offset-

在一些复杂的页面中经常会用 JavaScript 处理一些 DOM 元素的动态效果，这种时候我们经常会用到一些元素位置和尺寸的计算，浏览器兼容性问题也是不可忽略的一部分，要想写出预想效果的 JavaScript 代码，我们需要了解一些基本知识。

首先，每个 HTML 元素都有下列属性：

| offset       | client       | scroll       |
| ------------ | ------------ | ------------ |
| offsetWidth  | clientWidth  | scrollWidth  |
| offsetHeight | clientHeight | scrollHeight |
| offsetLeft   | clientLeft   | scrollLeft   |
| offsetTop    | clientTop    | scrollTop    |

这些属性和元素的高度、滚动、位置有关，单凭单词很难搞清楚分别代表什么意思之间有什么区别。

## offset

### offsetWidth & offsetHeight

offsetHeight 和 offsetWidth 用于描述元素外尺寸，是指 元素内容+内边距+滚动条+边框，不包括外边距部分。

对于 inline 的元素这个属性一直是 0，单位 px，只读属性。

![](/img/localBlog/1563017558042.jpg)

### offsetLeft & offsetTop

offsetTop 和 offsetLeft 表示该元素的左上角（边框外边缘）与已定位的父容器（offsetParent 对象）左上角的距离。

所有 HTML 元素拥有 offsetLeft 和 offsetTop 属性来返回元素的 X 和 Y 坐标：

1. 相对于已定位元素的后代元素和一些其他元素（表格单元），这些属性返回的坐标是相对于祖先元素

2. 一般元素，则是相对于文档，返回的是文档坐标

offsetParent 对象是指元素最近的定位（relative,absolute）祖先元素，递归上溯，如果没有祖先元素是定位的话，会返回 null，则这些属性都是相对文档坐标。

```
//用offsetLeft和offsetTop来计算e的位置
function getElementPosition(e){
    var x = 0,y = 0;
    while(e != null) {
        x += e.offsetLeft;
        y += e.offsetTop;
        e = e.offsetParent;
    }
    return {
        x : x,
        y : y
    };
}
```

## client

### clientWidth & clientHeight

clientHeight 和 clientWidth 用于描述元素内尺寸，是指 元素内容+内边距 大小，不包括边框（IE 下实际包括）、外边距、滚动条部分

对于 inline 的元素这个属性一直是 0，单位 px，只读属性。

![](/img/localBlog/1563017407707.jpg)

### clientLeft & clientTop

clientTop 和 clientLeft 返回内边距的边缘和边框的外边缘之间的水平和垂直距离，也就是左，上边框宽度。

在有滚动条时，并且浏览器将这些滚动条放置在左侧或顶部（反正我是没见过），clientLeft 和 clientTop 就包含这些滚动条的宽度。

## scroll

### scrollWidth & scrollHeight

因为子元素比父元素高，父元素不想被子元素撑的一样高就显示出了滚动条，在滚动的过程中本元素有部分被隐藏了，scrollHeight 代表包括当前不可见部分的元素的高度。

而可见部分的高度其实就是 clientHeight，也就是 scrollHeight >= clientHeight 恒成立。在有滚动条时讨论 scrollHeight 才有意义，在没有滚动条时 scrollHeight == clientHeight 恒成立。单位 px，只读元素。

![](/img/localBlog/1563017727539.jpg)

### scrollLeft & scrollTop

scrollTop 代表在有滚动条时，滚动条向下滚动的距离也就是元素顶部被遮住部分的高度。在没有滚动条时 scrollTop==0 恒成立。单位 px，可读可设置。

![](/img/localBlog/1563017856375.jpg)

## 相对于文档与视口的坐标

当我们计算一个 DOM 元素位置也就是坐标的时候，会涉及到两种坐标系，文档坐标和视口坐标。

我们经常用到的 document 就是整个页面部分，而不仅仅是窗口可见部分，还包括因为窗口大小限制而出现滚动条的部分，它的左上角就是我们所谓相对于文档坐标的原点。

视口是显示文档内容的浏览器的一部分，它不包括浏览器外壳（菜单，工具栏，状态栏等），也就是当前窗口显示页面部分，不包括滚动条。

如果文档比视口小，说明没有出现滚动，文档左上角和视口左上角相同，一般来讲在两种坐标系之间进行切换，需要加上或减去滚动的偏移量（scroll offset）。

为了在坐标系之间进行转换，我们需要判定浏览器窗口的滚动条位置。window 对象的 pageXoffset 和 pageYoffset 提供这些值，IE 8 及更早版本除外。也可以通过 scrollLeft 和 scrollTop 属性获得滚动条位置，正常情况下通过查询文档根节点（document.documentElement）来获得这些属性值，但在怪异模式下必须通过文档的 body 上查询。

```
function getScrollOffsets(w) {
    var w = w || window;
    if (w.pageXoffset != null) {
        return {
            x: w.pageXoffset,
            y: pageYoffset
        };
    }
    var d = w.document;
    if (document.compatMode == "CSS1Compat")
        return {
            x: d.documentElement.scrollLeft,
            y: d.documentElement.scrollTop
        };
    return {
        x: d.body.scrollLeft,
        y: d.body.scrollTop
    };
}
```

有时候能够判断视口的尺寸也是非常有用的

```
function getViewPortSize(w) {
    var w = w || window;
    if (w.innerWidth != null)
        return {
            w: w.innerWidth,
            h: w.innerHeight
        };
    var d = w.document;
    if (document.compatMode == "CSS1Compat")
        return {
            w: d.documentElement.clientWidth,
            h: d.documentElement.clientHeight
        };
    return {
        w: d.body.clientWidth,
        h: d.body.clientHeight
    };
}
```

### 文档坐标

任何 HTML 元素都拥有 offectLeft 和 offectTop 属性返回元素的 X 和 Y 坐标，对于很多元素，这些值是文档坐标，但是对于以定位元素后代及一些其他元素（表格单元），返回相对于祖先的坐标。我们可以通过简单的递归上溯累加计算

```
function getElementPosition(e) {
    var x = 0,
        y = 0;
    while (e != null) {
        x += e.offsetLeft;
        y += e.offsetTop;
        e = e.offsetParent;
    }
    return {
        x: x,
        y: y
    };
}
```

尽管如此，这个函数也不总是计算正确的值，当文档中含有滚动条的时候这个方法就不能正常工作了，我们只能在没有滚动条的情况下使用这个方法，不过我们用这个原理算出一些元素相对于某个父元素的坐标。

### 视口坐标

计算视口坐标就相对简单了很多，可以通过调用元素的 getBoundingClientRect 方法。方法返回一个有 left、right、top、bottom 属性的对象，分别表示元素四个位置的相对于视口的坐标。getBoundingClientRect 所返回的坐标包含元素的内边距和边框，不包含外边距。兼容性很好，非常好用

### 元素尺寸

由上面计算坐标方法，我们可以方便得出元素尺寸。在符合 W3C 标准的浏览器中 getBoundingClientRect 返回的对象还包括 width 和 height,但在原始 IE 中未实现，但是通过返回对象的 right-left 和 bottom-top 可以方便计算出。

## 滚动加载

移动端加载数据时，由于数据太多，不会一次性全部加载出来。有些会采用 pc 端那样用分页码的形式，但是更多的确实滑动滚动条到内容最后，加载更多内容出来。

```
scrollHeight === clientHeight + scrollTop
```

需要三个高度：scrollHeight（文档内容实际高度，包括超出视窗的溢出部分）、scrollTop（滚动条滚动距离）、clientHeight（窗口可视范围高度）。当 clientHeight + scrollTop >= scrollHeight 时，表示已经抵达内容的底部了，可以加载更多内容。

```
scrollDiv.addEventListener('scroll', event => {
      const element = event.target;
      const scrollHeight = element.scrollHeight;
      const scrollTop = element.scrollTop;
      const clientHeight = element.clientHeight;
      const isToBottom = scrollHeight === clientHeight + scrollTop;
      if (isToBottom) {
        // do something
      }
    });
```
