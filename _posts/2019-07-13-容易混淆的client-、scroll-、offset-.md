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

任何 HTML 元素的只读属性 offsetWidth 和 offsetHeight 已 CSS 像素返回它的屏幕尺寸，返回的尺寸包干元素的边框和内边距（width/height + border + padding），和滚动条。

### offsetLeft & offsetTop

所有 HTML 元素拥有 offsetLeft 和 offsetTop 属性来返回元素的 X 和 Y 坐标：

1. 相对于已定位元素的后代元素和一些其他元素（表格单元），这些属性返回的坐标是相对于祖先元素

2. 一般元素，则是相对于文档，返回的是文档坐标

offsetParent 属性指定这些属性所相对的父元素，如果 offsetParent 为 null，则这些属性都是文档坐标

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

client 是一种间接指代，它就是 web 浏览器客户端，专指它定义的窗口或视口。

### clientWidth & clientHeight

clientWidth 和 clientHeight 类似于 offsetWidth 和 offsetHeight，不同的是不包含边框大小（width/height + padding）。同时在有滚动条的情况下，clientWidth 和 clientHeight 在其返回值中也不包含滚动条。

对于内联元素，总是返回 0

### clientLeft & clientTop

返回元素的内边距的外边缘和他的边框的外边缘的水平距离和垂直距离，通常这些值就等于左边和上边的边框宽度。

在有滚动条时，并且浏览器将这些滚动条放置在左侧或顶部（反正我是没见过），clientLEft 和 clientTop 就包含这些滚动条的宽度。

## scroll

### scrollWidth & scrollHeight

这两个属性是元素的内容区域加上内边距，在加上任何溢出内容的尺寸.

因此，如果没有溢出时，这些属性与 clientWidth 和 clientHeight 是相等的。

### scrollLeft & scrollTop

指定的是元素的滚动条的位置

scrollLeft 和 scrollTop 都是可写的属性，通过设置它们来让元素中的内容滚动。
