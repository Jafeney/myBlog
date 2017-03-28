title: Node爬坑记——伪造cookie 
date: 2016-01-21 23:33:59 
tags: [NodeJS,cookie] 
categories: 大前端  
---

# 写在前面
在没有引入`NodeJS`层之前，客户端和服务端之前的数据传输可以用`Ajax`来完成，而且服务端可以直接读取客户端请求头携带的`cookie`，这个直接走 `HTTP` 协议，没有任何问题。但是当引入了一个`NodeJS`作为中间层，通过中间层调用服务层（比如`Java`的数据接口层）的接口时，你会发现 直接走`http`协议，Java层根本无法读取到 原本从客户端发送过来的`cookie`。这个时候 就需要在中间层 伪造一个`cookie`了。

<!--more-->

# 客户端cookie

在切入正题之前不妨先谈谈客户端`cookie`的情况，相信这也是你很想明白的东西。

## Ajax如何跨域发送cookie

刚才虽然说这个很容易，其实对于初学者来说还是有个陷阱在里面的，这个场景当然是发生在跨域请求里。不跨域的`ajax`请求原则上是可以携带`cookie`的，跨域了之后就需要设置`XMLHttpRequest`对象的一些参数了。

### 跨域的几种方式浅淡 
我所了解的跨域方案有5种，这里不做详细介绍。如果感兴趣，请用力 [戳这里](http://www.cnblogs.com/rainman/archive/2011/02/20/1959325.html)
#### document.domain+iframe的设置 
对于主域相同而子域不同的例子，可以通过设置document.domain的办法来解决。具体的做法是可以在http://www.a.com/a.html和http://script.a.com/b.html两个文件中分别加上document.domain = ‘a.com’；然后通过a.html文件中创建一个iframe，去控制iframe的contentDocument，这样两个js文件之间就可以“交互”了。当然这种办法只能解决主域相同而二级域名不同的情况，如果你异想天开的把script.a.com的domian设为alibaba.com那显然是会报错地！

#### 动态创建script 
虽然浏览器默认禁止了跨域访问，但并不禁止在页面中引用其他域的JS文件，并可以自由执行引入的JS文件中的function（包括操作cookie、Dom等等）。根据这一点，可以方便地通过创建script节点的方法来实现完全跨域的通信。

#### 利用iframe和location.hash 
这个办法比较绕，但是可以解决完全跨域情况下的脚步置换问题。原理是利用location.hash来进行传值。在url： http://a.com#helloword中的‘#helloworld’就是location.hash，改变hash并不会导致页面刷新，所以可以利用hash值来进行数据传递，当然数据容量是有限的。假设域名a.com下的文件cs1.html要和cnblogs.com域名下的cs2.html传递信息，cs1.html首先创建自动创建一个隐藏的iframe，iframe的src指向cnblogs.com域名下的cs2.html页面，这时的hash值可以做参数传递用。cs2.html响应请求后再将通过修改cs1.html的hash值来传递数据（由于两个页面不在同一个域下IE、Chrome不允许修改parent.location.hash的值，所以要借助于a.com域名下的一个代理iframe；Firefox可以修改）。同时在cs1.html上加一个定时器，隔一段时间来判断location.hash的值有没有变化，一点有变化则获取获取hash值。 

#### window.name实现的跨域数据传输 
文章较长列在此处不便于阅读，你可以[戳这里](http://www.cnblogs.com/rainman/archive/2011/02/21/1960044.html) 。

#### 使用HTML5 postMessage 
具体使用这个方法：otherWindow.postMessage(message, targetOrigin); 
> otherWindow: 对接收信息页面的window的引用。可以是页面中iframe的contentWindow属性；window.open的返回值；通过name或下标从window.frames取到的值。
message: 所要发送的数据，string类型。
targetOrigin: 用于限制otherWindow，“*”表示不作限制

### 用原生JavaScript Cookie操作

```
//COOKIE功能检查
function fCheckCookie(){
    if(!navigator.cookieEnabled){
        alert("您好，您的浏览器设置禁止使用cookie\n请设置您的浏览器，启用cookie功能，再重新登录。");
    }
}

//获取Cookie
function fGetCookie(sName){
   var sSearch = sName + "=";
   if(document.cookie.length > 0){
      offset = document.cookie.indexOf(sSearch)
      if(offset != -1){
         offset += sSearch.length;
         end = document.cookie.indexOf(";", offset)
         if(end == -1) end = document.cookie.length;
         return unescape(document.cookie.substring(offset, end))
      }
      else return ""；
   }
}

//设置Cookie
function fSetCookie(name, value, isForever, domain){
    var sDomain = ";domain=" + (domain || gOption["sCookieDomain"] );
    document.cookie = name + "=" + escape(value) + sDomain + (isForever?";expires="+  (new Date(2099,12,31)).toGMTString():"");
}

//跨域调用方法
function fGetScript(sUrl){
    var oScript = document.createElement("script");
    oScript.setAttribute("type", "text/javascript");
    oScript.setAttribute("src", sUrl);
    try{oScript.setAttribute("defer", "defer");}catch(e){}
    window.document.body.appendChild(oScript);
}
```
### 用jQuery实现跨域调用cookie
下面这段代码是个跨域读取cookie的例子：
```
$.ajax({
	type:'get',
	url:_self.APIURL+'/activity/isAttended.json',
	dataType:'jsonp',
	data:{
		activityId:_activityId,
		callback:1
	},
	xhrFields: {
		withCredentials: true
	},
	crossDomain: true,
	success:function(res){
		if(res.isAttended==1){
			$('#indexMask').show();
		}else{
			$('#indexMask').hide();
		}
	}
});
```

# 中间层cookie

## 用原生http模块实现

### 伪造cookie 
引入http模块（NodeJS的核心模块，可以直接调用）
```
var http=require('http');
```
路由方法（这里以购物车为例）
```
/*GET shopCar page.*/
router.get('/shopCar',function(req,res,next){}
```
request方法的options参数配置（这一步即可在headers里伪造cookie）
```
var options = {  
      hostname: 'api.baai.hk',  
      port: 80,  
      path: '/cart.shtml',  
      method: 'GET',  
      headers: {  
          'Content-Type': 'application/x-www-form-urlencoded; charset=UTF-8',
          'Cookie': 'baai_user_token='+req.cookies.baai_user_token
      }  
  };  
```
### 模板渲染
接着上面的路由方法，headers里的cookie伪造好之后就可以调用request请求了。
```
/*声明一个空变量缓存返回的数据流（这一步很关键哦）*/
var temp='';

/*发起http请求*/
var req = http.request(options, function (_res) {  
    console.log('STATUS: ' + res.statusCode);  
    console.log('HEADERS: ' + JSON.stringify(res.headers));  
    _res.setEncoding('utf8');  
    _res.on('data', function (chunk) {   
	    console.log(chunk);
	    /*把数据流拼接起来得到完整数据*/
        _temp+=chunk; 
    });  
	_res.on('end', function(){
		/*模板渲染*/
        var data=JSON.parse(_temp);
        res.render('shopCar', data); 
    });
});  

/*关闭http请求*/
req.end();
```

## 用三方的request模块实现 
三方有个`request`模块，封装了原生http模块的一些功能，比较好用，也比较推荐使用这个。
### 安装request模块
```
npm install request --save
```
### 调用模块
```
var request=require('request');
```
> 这里注意一下，request为了安全默认是禁用`cookie` 的，需要手动开启。在`defaults`或`options`将`jar`设为`true`，使后续的请求都使用`cookie`。

```
var request = request.defaults({jar: true});
request('http://www.google.com', function (_res) {
	request('http://images.google.com');
});
```
通过创建`request.jar()`的新实例，可以使用定制的`cookie`，而不是`request`全局的`cookie jar`，看下面这个实例。

```
var j = request.jar();
var cookie = request.cookie('baai_user_token');
j.setCookie(cookie, 'http://www.baai.hk', function (err, cookie){
	console.log('cookie set succeeded');
});
request({url: 'http://www.baai.hk/cart.shtml', jar: j}, function (_res) {
	/*模板渲染*/
	var data=JSON.parse(_res);
    res.render('shopCar', data);
});
```

# 结语

就目前而言，`NodeJS`在国内互联网圈子里的地位还是属于刚起家的状况，这门技术应用的领域也相对单一，虽然阿里对`NodeJS`（尤其像淘宝、天猫）相当重视，今年双十一和双十二也充分尝到了技术进步带来的甜头。但是 `NodeJS` 作为中间层开发方面的文章、帖子、社区还是相对薄弱些，不过相信会一天天好起来吧。小子我虽初入茅庐，也很愿意花些业余时间为`NodeJS` 的开源工作 做些小小的贡献。

--- 
欢迎大家关注我的技术个人博客： http://jafeney.com   ^_^









