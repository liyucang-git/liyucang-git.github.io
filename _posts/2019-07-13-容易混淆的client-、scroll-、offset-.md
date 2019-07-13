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

offsetHeight 包括 padding、border、水平滚动条，但不包括 margin 的元素的高度。对于 inline 的元素这个属性一直是 0，单位 px，只读属性。

![](/img/localBlog/1563017558042.jpg)

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

### clientWidth & clientHeight

clientHeight 包括 padding 但不包括 border、水平滚动条、margin 的元素的高度。对于 inline 的元素这个属性一直是 0，单位 px，只读属性。

![](/img/localBlog/1563017407707.jpg)

### clientLeft & clientTop

返回元素的内边距的外边缘和他的边框的外边缘的水平距离和垂直距离，通常这些值就等于左边和上边的边框宽度。

在有滚动条时，并且浏览器将这些滚动条放置在左侧或顶部（反正我是没见过），clientLEft 和 clientTop 就包含这些滚动条的宽度。

## scroll

当元素的子元素比元素高且 overflow=scroll 时，元素会出现滚动条。

### scrollWidth & scrollHeight

因为子元素比父元素高，父元素不想被子元素撑的一样高就显示出了滚动条，在滚动的过程中本元素有部分被隐藏了，scrollHeight 代表包括当前不可见部分的元素的高度。

而可见部分的高度其实就是 clientHeight，也就是 scrollHeight>=clientHeight 恒成立。在有滚动条时讨论 scrollHeight 才有意义，在没有滚动条时 scrollHeight==clientHeight 恒成立。单位 px，只读元素。

![](/img/localBlog/1563017727539.jpg)

### scrollLeft & scrollTop

scrollTop 代表在有滚动条时，滚动条向下滚动的距离也就是元素顶部被遮住部分的高度。在没有滚动条时 scrollTop==0 恒成立。单位 px，可读可设置。

![](/img/localBlog/1563017856375.jpg)
