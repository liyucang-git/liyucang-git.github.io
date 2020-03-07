---
layout: post
title: 简单明了区分escape、encodeURI和encodeURIComponent
subtitle: js编码的几种方式
date: 2016-10-22
author: Li Yucang
catalog: true
tags:
  - js
  - api
---

# 简单明了区分 escape、encodeURI 和 encodeURIComponent

## 前言

讲这 3 个方法区别的文章太多了，但是大部分写的都很绕。本文试图从实践角度去讲这 3 个方法。

## escape

该特性已经从 Web 标准中删除，虽然一些浏览器目前仍然支持它，但也许会在未来的某个时间停止支持，请尽量不要使用该特性。所以就不讨论了。

## encodeURI(URI)

由该方法定义可知，入参是一个完整的 uri(统一资源标识符)。encodeURI 会替换所有的字符，但不包括以下字符，即使它们具有适当的 UTF-8 转义序列：

| 类型         | 包含                          |
| ------------ | ----------------------------- |
| 保留字符     | ; , / ? : @ & = + \$          |
| 非转义的字符 | 字母 数字 - \_ . ! ~ \* ' ( ) |
| 数字符号     | #                             |

我们发现，例如对于 XMLHTTPRequests, "&", "+", 和 "=" 不会被编码，然而在 GET 和 POST 请求中它们是特殊字符。

## encodeURIComponent(Component)

encodeURIComponent()是对统一资源标识符（URI）的**组成部分**进行编码的方法。它使用一到四个转义序列来表示字符串中的每个字符的 UTF-8 编码（只有由两个 Unicode 代理区字符组成的字符才用四个转义字符编码）。

encodeURIComponent 转义除了字母、数字、(、)、.、!、~、\*、'、-和\_之外的所有字符。

## 总结

对 URL 编码是常见的事，所以这两个方法应该是实际中要特别注意的。它们都是编码 URL，唯一区别就是编码的字符范围，encodeURIComponent 比 encodeURI 编码的范围更大。

来个例子：

```
encodeURI("http://www.cnblogs.com/season-huang/some other thing");
// "http://www.cnblogs.com/season-huang/some%20other%20thing";

encodeURIComponent("http://www.cnblogs.com/season-huang/some other thing");
// "http%3A%2F%2Fwww.cnblogs.com%2Fseason-huang%2Fsome%20other%20thing"

var param = "http://www.cnblogs.com/season-huang/"; //param为参数
param = encodeURIComponent(param);
var url = "http://www.cnblogs.com?next=" + param;
//"http://www.cnblogs.com?next=http%3A%2F%2Fwww.cnblogs.com%2Fseason-huang%2F"
```

so:

- 如果你需要编码整个 URL，然后需要使用这个 URL，那么用 encodeURI。

- 当你需要编码 URL 中的参数的时候，那么 encodeURIComponent 是最好方法。
