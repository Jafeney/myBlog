title: JavaScript Constructor(构造器)模式 
date: 2016-01-10 15:18:36 
tags: [Javascript,"设计模式"]  
categories: 前端  
---


# 前言
在经典的面相对象语言编程中，Constructor是一种在内存已分配给该对象的情况下，用于初始化新创建对象的特殊方法。在Javascript中，几乎所有东西都是对象，我们通常最感兴趣的是object构造器。

object构造器用于创建特定类型的对象——准备好对象以备使用，同时接收构造器可以使用的参数，以在第一次创建对象时，设置成员属性和方法的值。

![这里写图片描述](http://img.blog.csdn.net/20151209234517042)

<!--more-->

# 基本Constructor
Javascript虽然不支持类的概念，但它确实支持与对象一起用的特殊constructor函数，通过在构造器前面加new关键字，告诉Javascript像使用构造器一样实例化一个新对象，并且对象成员由该函数定义。

在构造器内，this关键字引用新创建的对象。回顾对象创建，基本的构造器看起来可能是这样的：

```
function Car(model,year,miles){
	this.model=model;
	this.year=year;
	this.miles=miles;
	
	this.toString=function(){
		return this.model+"has done "+this.miles+"miles";
	};
}

var civic=new Car("Honda Civic",2009,20000);
var mondeo=new Car("Ford Mondeo",2010,5000);

console.log(civic.toString());
console.log(mondeo.toString());
```
> 
分析：上面是一个简单的构造器模式版本，但它有2个问题。

1.  它使得继承变得困难；

2.  toString()这样的函数是为每个使用Car构造器创建的新对象而分别重新定义的，这不是最理想的构造方式，因为**这种函数应该在所有Car类型实例之间共享**。

# 带原型的Constructor
Javascript的prototype属性相信大家都知道，调用Javascript构造器创建一个对象后，新对象就会具有构造器原型的所有属性。通过这种方式，可以创建多个Car对象，并访问相同的原型。如下：

```
function Car(model,year,miles){
	this.model=model;
	this.year=year;
	this.miles=miles;
}
/*使用Object.prototype.newMethod而不是Object.prototype可以避免重新定义prototype对象*/
Car.protoType.toString=function(){
	return this.model+"has done "+this.miles+"miles";
};

var civic=new Car("Honda Civic",2009,20000);
var mondeo=new Car("Ford Mondeo",2010,5000);

console.log(civic.toString());
console.log(mondeo.toString());
```
>这样toString()的当以实例就能够在所有Car对象之间共享了^_^
