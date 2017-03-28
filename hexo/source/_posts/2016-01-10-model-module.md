---
title: JavaScript Module模式
date: 2016-01-10 15:21:14
tags: [Javascript,"设计模式"]  
categories: 前端
---


# 写在前面
Module模式最初被定义为一种在传统软件工程中为类提供私有和公有封装的方法。而在Javascript中，Module模式用于进一步模拟类的概念，通过这种方式，能够使一个单独的对象拥有公有/私有的方法和变量，从而屏蔽来自全局作用域的特殊部分。产生的结果是：函数名与在页面上其他脚本定义的函数冲突的可能性降低。
> 应当注意的一点是：在Javascript没有private访问修饰符因此算不得真正的私有，而是通过函数作用域来模拟私有这个概念。在Module模式内由于闭包的存在，声明的变量和方法只在该模式内部可用，但在返回对象上定义的变量和方法是可以对外访问的。

<!--more-->

# 示例
```
var testModule=(function(){
	var counter=0;
	return{
		incrementCounter:function(){
			return ++counter;
		},
		resetCounter:function(){
			console.log("counter value prior to reset:"+counter);
			counter=0;
		}
	};
})();
//增加计数器
testModule.incrementCounter();
//加长计数器值并重置
testModule.resetCounter();
```


# Module模式变化
## 引入混入
这种变化允许我们把全局变量（如jQUery、Underscore）作为参数传递给模块的匿名函数，并按照我们所希望的为它们取个本地别名。

```
var myModule=(function(jQ,_){
	function privateMethod1(){
		jQ(".container").html("test");
	}
	function privateMethod2(){
		console.log(_.min([10,5,100,2,1000]));
	}
	return{
		publicMethod:function(){
			privateMethod1();
		}
	};
//引入jQuery对象和underscore对象
})(jQuery,_);
```
## 引出
这种变化允许我们声明全局变量，而不需要实现它们，并可以同样地支持上一个实例中全局引入的概念。

```
var myModule=(function(){
	//模块对象
	var module={},
	privateVariable="Hello World";
	function privateMethod(){
		//....
	}

	module.publicProperty="Foobar";
	module.publicMethod=function(){
		console.log(privateVariable);
	}
	return module;
})();
```

# 总结
## 优点
我们已经了解单例模式如何使用，但为什么Module模式是一个好的选择呢？首先，相比真正封装的思想，它对于很多拥有面向对象背景的开发人员来说更加整洁，至少是从Javascript的角度。

其次，他支持私有数据，因此在Module模式中代码的公有部分能够接触私有部分，然而外界无法接触类的私有部分。

## 缺点
Module模式的缺点是：由于我们访问公有和私有成员的方式不同，当我们想改变可见性时，实际上我们必须要修改每一个曾经使用该成员的地方。

我们也无法访问那些之后再方法里添加的私有成员。也就是说在很多情况下，如果正确使用Module模式任然是当然有用的，肯定可以改进应用程序的结构。

其他缺点包括：无法为私有成员创建自动化单元测试，bug需要修正补丁时会增加额外的复杂性。为私有方法打补丁是不可能的。相反，我们必须覆盖所有bug的私有方法进行交互的公有方法。另外开发人员也无法轻易地拓展私有方法，所以要记住**私有方法并不想它们最初显示出来那么灵活**。
