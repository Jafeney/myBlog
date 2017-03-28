---
title: 理解Javascript的prototype
date: 2016-02-23 15:19:00
tags: Javascript
categories: 前端
---

# 写在前面
废话不多说，请看下面3道题，把6个`console.log()`的答案写下来，然后对照着在`Console`控制台里敲一遍，校验一下结果。

<!--more-->

# 代码片段

```
var A = function() {};

a.prototype = {
    num : 1,
    text : 'aaa'
};

var x = new A();

// 第一题
console.log(x.num);
console.log(x.text);

// --这里是分割线--

var y = new A();
A.prototype = {
    num : 2
};
// 第二题
console.log(y.num);
console.log(y.text);

// --这里是分割线--

y.num = 3;
var z=new A();
// 第三题
console.log(z.num);
console.log(z.text);
```

# 答题时间
题目都看明白了吧，给你10秒钟赶紧在纸上把6个`console.log()`的结果写下来吧。

计时开始咯
... 1 ...
... 2 ...
... 3 ...
... 4 ...
... 5 ...
... 6 ...
... 7 ...
... 8 ...
... 9 ...
... 10 ...

# 公布答案
6个`console.log()`的结果如下：
```
1
aaa
1
aaa
2
undefined
```

好了，你对了几题呢？如果都对了说明你已经理解`prototype`的精髓了，可以离开这篇文章了。但如果你没有全对，还是建议看一下下面的解析。


# 答案解析
## 第一题
`x`对象是A类的一个实例，`x`继承A的原型，所以它具备了A的`num`和`text`属性，因此输出的结果是  `1`和`aaa`。

> 注意： 其实JavaScript并不是严格的面相对象语言，它没有类的概念，所谓的面向对象是用函数模拟出来的，这里暂且引用Java里面向对象的理论帮助大家更好理解。

## 第二题
`y`对象是A类的一个实例，`y`继承A的原型，所以它的`num`和`text`属性值也是`1`和`aaa`。
> 注意：`y`实例化之后虽然对A的原型进行了操作，但是并不会影响到`y`，而会影响到第三题的`z`对象。

## 第三题
上一题已经说到A的原型已经发生了重新定义，相比之前，缺少了对`text`的定义，因此`z`对象的`num`属性为 `2`，而`text`属性未定义是 `undefined`。
