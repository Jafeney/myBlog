---
title: ReactNative坑点——Date对象正确使用
date: 2016-11-16 11:28:33
tags: react-native
categories: 移动开发
---

# 写在前面
昨天遇到了一个非常诡异的场景，`ReactNative`写的倒计时组件线上版本无法运行，但本地测试却是正常的。我一度以为是`ReactNative`转换的时候出了问题，不知道从何下手。最后盘查了一圈，发现了一个不得了的事！——`ReactNative`的JS解析，当打开`chrome`进行`Debug`时，用的的确是`chrome`的内核，但对于转换好的版本，如IOS版本用的是`safari`的内核，`Android`版本也会随着操作系统的不同而存在差异。

# 定位出错点
上面发现的问题有点类似以前经常碰到的浏览器兼容问题。`Get`到这层意思，马上就发现下面这种写法存在兼容问题：

```JavaScript
var date = new Date('2016-12-15 10:20')
```
`Chrome`浏览器里当然是正确的，但是在`Safari`和`Firefox`里是`date`的值是 `Invalid Date`。

<!--more-->

## Chrome浏览器里的结果
 ![这里写图片描述](http://img.blog.csdn.net/20161116103245001)

## Safari浏览器里的结果
![Safari浏览器里的结果](http://img.blog.csdn.net/20161116102850046)

## Firefox浏览器的结果
![Firefox浏览器的结果](http://img.blog.csdn.net/20161116102928640)

# 解决办法
介于上面的兼容性问题，需要对 `Date()`这个构造方法做处理：

```JavaScript
export function parseDate(date) {
    var isoExp, parts;
    isoExp = /^\s*(\d{4})-(\d\d)-(\d\d)\s(\d\d):(\d\d):(\d\d)\s*$/;
    try {
        parts = isoExp.exec(date);
    } catch(e) {
        return null;
    }
    if (parts) {
        date = new Date(parts[1], parts[2] - 1, parts[3], parts[4], parts[5], parts[6]);
    } else {
        return null;
    }
    return date;
}
```
> 上面用到了 `export` 关键字，你可以把它放到 `mixins` 里全局调用。

以后就用 `parseDate` 替代`new Date()`，就避开了兼容问题。

```JavaScript
import { parseDate } from './mixins/helper'

// ... 省略

let date = parseDate('2016-11-15 10:20')
```
