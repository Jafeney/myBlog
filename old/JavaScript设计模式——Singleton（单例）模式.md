title: JavaScript Singleton（单例）模式
date: 2016-01-10 15:29:05 
tags: [Javascript,"设计模式"]  
categories: 前端 
---


# 写在前面
singleton模式是被熟知的原因是因为它限制了类的实例化次数只能一次。从经典意义上来说，singleton模式在该实例不存在的情况下，可以通过一个方法创建一个类来实现创建类的新实例；如果实例已经存在，它会简单返回该对象的引用。

singleton不同于静态类(或对象)，因为我们可以推迟它们的初始化，这通常是因为它们需要一些信息，而这些信息在初始化期间可能无法获得、对于没有察觉到之前的引用的代码，它们不会提供方便检索的方法。这是因为它既不是对象，也不是由一个singleton返回的“类”；它是一个结构。

思考一下**闭包变量为何实际上并不是闭包**，而提供闭包的函数作用域是闭包。在Javascript中，singleton充当共享资源命名空间，从全局命名空间中隔离出代码实现，从而为函数提供单一访问点。

<!--more-->

# 来个例子

```
var mySingleton=(function(){
	//实例保持了Singleton的一个引用
	var instance;
	function init(){
		//Singleton
		//私有方法和变量
		function privateMethod(){
			console.log("I am private");
		}
		var privateVariable="I am also private";
		var privateRandomNumber=Math.random();
		return{
			//公有方法和变量
			publicMethod:function(){
				console.log("The public can see me!");
			},
			publicProperty:"I am also public",
			getRandomNumber:function(){
				return privateRandomNumber;
			}
		};
	}
	return{
		//获取singleton的实例，如果存在则返回，不存在就创建新实例
		getInstance:function(){
			if(!instance){
				instance=init();
			}
			return instance;
		}
	}
})();

var myBadSingleton=(function(){
	//实例保存了singleton的一个引用
	var instance;
	function init(){
		//Singleton
		var privateRandomNumber=Math.random();
		return{
			getRandomNumber:function(){
				return privateRandomNumber;
			}
		};
	}
	return{
		//每次都创建新实例
		getInstance:function(){
			instance=init();
			return instance;
		}
	};
})();

var singleA=mySingleton.getInstance();
var singleB=mySingleton.getInstance();
console.log(singleA.getRandomNumber()===SingleB.getRandomNumber()); //true
var badSingleA=myBadSingleton.getInstance();
var badSingleB=myBadSingleton.getInstance();
console.log(badSingleA.getRandomNumber()!==badSingleB.getRandomNumber()); //true
```
# 分析
好了，写了这么多代码，我们先明确一个问题——是什么使Singleton成为实例的全局访问入口（通常通过MySingleton.getInstance()）?

> 因为我们没有（至少在静态语言中）直接调用新的MySingleton()。然而，这在Javascript中是可能的。

## **适用的场景**
1、当类只能有一个实例而且客户可以从一个众所周知的访问点访问它时。
2、该唯一的实例应该是通过子类化可扩展的，并且客户应该无需更改代码就能使用一个扩展的实例时。
## **延迟执行**
我们看下面代码：

```
mySingleton.getInstance=function(){
	if(this._instance==null){
		if(isFoo()){
			this._instance=new FooSingleton();
		}else{
			this._instance=new BasicSingleton();
		}
	}
	return this._instance;
};
```
在这里，getInstance变得有点像Factory（工厂）方法，当访问它时，我们不需要更新代码中的每个访问点。FooSingleton（上面）将是一个BasicSingleton的子类，并将实现相同的接口。

**那么问题来了，为什么要延迟执行Singleton呢？**

在C++中，Singleton负责隔绝动态初始化顺序的不可预知性，将控制权归还给程序员。

值得注意的是类的静态实例（对象）和Singleton之间的区别：当Singleton可以作为一个静态的实例实现时，它也可以延迟构建，直到需要使用静态实例时，无需使用资源或内存。

如果我们有一个可以直接被初始化的静态对象，需要确保执行代码的顺序总是相同的（例如：在初始化期间objCar需要objWheel的情况），当我们有大量的源文件时，它并不能伸缩。

Singleton和静态对象都是有用的，但是我们不应当以同样的方式过度使用它们，也不应该过度地使用其他模式。

## **最佳实践**
在实践中，当在系统中确实需要一个对象来协调其他对象时，Singleton模式是很有用的，在这里，大家可以看到在这个上下文中模式的使用：

```
var SingletonTester=(function(){

	//options:包含Singleton所需配置信息的对象
	//@eg: var options={name:"test",pointX:5};
	function Singleton(options){
		//如果未提供options则设置为空对象
		options=options || {};
		//未Singleton设置一些属性
		this.name="SingletonTester";
		this.pointX=options.pointX||6;
		this.pointY=options.pointY||10;
	}

	//实例持有者
	var instance;
	//静态变量和方法的模拟
	var _static={
		name:"SingletonTester",
		//获取实例的方法，返回Singleton对象的Singleton实例
		getInstance:function(options){
			if(instance===undefined){
				instance=new Singleton(options);
			}
			return instance;
		}
	};
	return _static;
})(); 

var singletonTest=SingletonTester.getInstance({
	pointX:5
});

//记录pointX的输出以便验证
//输出：5
console.log(singletonTest.pointX);
```
# 总结
Singleton很有使用价值，通常当发现在Javascript中需要它的时候，则表示我们可能需要重新评估我们的设计。Singleton的存在往往表面系统中的模块要么是系统紧密耦合，要么是其逻辑国语分散在代码库的多个部分。**由于一系列的问题：从隐藏的依赖到创建多个实例的难度、底层依赖的难度等等，Singleton的测试会更加困难。**

