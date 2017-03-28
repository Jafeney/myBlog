---
title: JavaScript Revealing Module（揭示模块）模式
date: 2016-01-10 15:23:44
tags: [Javascript,"设计模式"]  
categories: 前端
---


# 前言
在对Module模式有个熟悉的了解之后，我们来认识一个稍有改进的版本——ChristianHeilmann的Revealing Module模式。
# 模式的由来
原来的Module模式可能无法实现这样的需求：
当我们从另一个方法调用一个公有方法或者访问公有变量时，必须要重复主对象的名称。而且使用Module时必须要切换到对象字面量表示法来让某种方法变成公有方法。

我们需要的可能是这样的一个模式：
能够在私有范围内简单定义所有的函数和变量，并返回一个匿名对象，它拥有指向私有函数的指针，该函数是它希望展示为公有的方法。

有点拗口，还是上代码吧 (☆_☆)

<!--more-->

# 代码

```
var myRevealingModule=(function(){
	var privateVar="Ben Cherry",
		publicVar="Hey there!";

	function privateFunction(){
		console.log("Name："+privateVar);
	}

	function publicSetName(strName){
		privateName=strName;
	}

	function publicGetName(){
		privateFunction();
	}
	/*将暴露的公有指针指向到私有函数和属性上*/
	return{
		setName:publicSetName,
		greeting:publicVar,
		getName:publicGetName
	};
})();

myRevealingModule.setName("Paul Kinlan");
```

# 优点
该模式可以使脚本语法更加一致，在模块代码底部，它也会很容易指出哪些函数和变量可以被公开访问，从而改善可读性。

# 缺点
该模式的一个缺点是：如果一个私有函数引用一个公有函数，在需要打补丁时，公有函数是不能被覆盖的。这时因为私有函数将继续引用私有实现，该模式并不适用于公有成员，只适用于函数。

引用私有变量的公有对象成员也遵守无补丁规则，正因为如此，采用Revealing Module模式创建的模块可能比那些采用原始Module模式创建的模块更加脆弱，所以在使用时应该特别小心。
