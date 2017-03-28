title: JavaScript call和apply的区别
date: 2016-01-10 14:53:32
tags: Javascript 
categories: 前端 
---


# 写在前面
   很多来面试前端工程师的人说自己精通Javascript语言，问他call()和apply()这两个方法有什么区别，结果往往是一问三不知。
    其实区分 apply和call就一句话：
```Javascript
    foo.call(this,arg1,arg2,arg3)==foo.apply(thsi,arguments)==this.foo(arg1,arg2,arg3);
```

<!--more-->

# 两者区别
> 
call 和 apply都属于Function.prototype的一个方法，它是Javascript引擎内在实现的，因为属于Function.prototype对象的实例，也就是每个方法都有call,apply属性，这两个方法很容易混淆，因为他们的作用一样，只是使用方式不同（传递的参数不同）。

## 不同点分析
    我们针对上面的foo.call(this,arg1,arg2,arg3)展开分析：<br/>
    foo 是一个方法，this是方法执行时上下文相关对象，永远指向当前代码所处在的对象中。arg1,arg2,arg3是传给foo方法的参数。
```JavaScript
    function A(){
        var message="a";
        return{
            getMessage:function(){return this.message}
        }
    }
    function B(){
        var message="b";
        return{
            setMessage:function(message){this.message=message}
        }
    }
    var a=new A();
    var b=new B();
    b.setMessage.call(a,"a的消息");
    alert(a.getMessage()); //这里将输出“a的消息”
```
这就是动态语言Javascript call的威力所在，简直是无中生有，对象的方法可以任意指派，而对象本身是没有这个方法的。注意，指派通俗地讲就是把方法借给另一个对象调用，原理上时方法执行时上下文的对象改变了。

所以，b.setMessage.call(a,"a 的消息");就等于用a做执行时上下文对象调用b对象的setMessage()方法，而这个过程与b一点关系都没有。作用等效于a.setMessage()

下面说一下apply的使用场景
```JavaScript
    function print(a,b,c,d){
        console.log(a+b+c+d);
    }
    function example(a,b,c,d){
        //用call方式借用print，参数显式打散传递
        print.call(this,a,b,c,d);
        //apply方式借用print，参数作为一个数组传递
        print.apply(this,arguments);
        //或者这样写
        print.apply(this,[a,b,c,d]);
    }
    example('我','不','开','心');
```
从上面的例子我们发现，call和apply方法除了第一个参数相同，call方法的其他参数一次传递给借用的方法做参数，而apply就两个参数，第二个参数是借用方法的参数组成的数组。 **总结一下，当参数明确时可用call，当参数不明确时用apply并结合arguments**
