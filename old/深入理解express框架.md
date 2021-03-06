title: 深入理解express框架
date: 2016-01-10 11:26:33
tags: express 
categories: 大前端 
---

# 写在前面
Express 是一个简洁而灵活的 node.js Web应用框架, 提供了一系列强大特性帮助你创建各种 Web 应用，和丰富的 HTTP 工具。使用 Express 可以快速地搭建一个完整功能的网站。

**Express 框架核心特性：**

 - 可以设置中间件来响应 HTTP 请求。 
 - 定义了路由表用于执行不同的 HTTP 请求动作。 
 - 可以通过向模板传递参数来动态渲染 HTML页面。

<!--more--> 
 
今天不介绍基本用法，如何用它搭建nodeJS中间层在我前面的文章里有比较详细的介绍，而这次主要是深入研究它的内部实现原理。

# 底层HTTP服务器
学习nodeJS基础的时候我们经常要用到 `http` 这个核心模块，它以极其简洁JavaScript代码 搭建出本地http服务.

<!--more-->


```
// 引入所需模块
var http = require("http");
 
// 建立服务器
var app = http.createServer(function(request, response) {
    response.writeHead(200, {
        "Content-Type": "text/plain"   
    });
    response.end("Hello world!\n");
});
 
// 启动服务器
app.listen(1337, "localhost");
console.log("Server running at http://localhost:1337/");
```

> **解析：** 第一行使用 `require` 函数引入Node内置模块 `http` 。然后存入名为 `http` 的变量中。然后我们使用 `http.createServer()` 将服务器保存至 app 变量。它将一个函数作为参数监听请求。稍后将会详细介绍它。最后我们要做的就是告诉服务器监听来自1337端口的请求，之后输出结果。然后一切完成。

# 中间件
通过中间件，搭建本地服务器的过程变得简单，比如最简单的connect
```
// 引入所需模块
var connect = require("connect");
var http = require("http");
 
// 建立app
var app = connect();
 
// 添加中间件
app.use(function(request, response) {
    response.writeHead(200, { "Content-Type": "text/plain" });
    response.end("Hello world!\n");
});
 
// 启动应用
http.createServer(app).listen(1337);
```

express 4.x 已经把connect中间件封装的express模块里。因此我们不需要依赖connect模块：
```
var express = require('express');
var app = express();

var server = app.listen(8081, function () {
  console.log("Server running at http://localhost:1337/")
});
```

## **什么是中间件**
一个最基本的中间件结构如下：
```
function myFunMiddleware(request, response, next) {
	// 对request和response作出相应操作
	// 操作完毕后返回next()即可转入下個中间件
    next();
}
```
当我们启动一个服务器，函数开始从顶部一直往下执行。还是还来个能跑的程序 

```
var connect = require("connect");
var http = require("http");
var app = connect();
 
app.use(connect.logger());
 
// Homepage
app.use(function(request, response, next) {
    if (request.url == "/") {
        response.writeHead(200, { "Content-Type": "text/plain" });
        response.end("Welcome to the homepage!\n");
        
// The middleware stops here.
    } else {
        next();
    }
});
 
// About page
app.use(function(request, response, next) {
    if (request.url == "/about") {
        response.writeHead(200, { "Content-Type": "text/plain" });
        response.end("Welcome to the about page!\n");
        
// The middleware stops here.
    } else {
        next();
    }
});
 
// 404'd!
app.use(function(request, response) {
    response.writeHead(404, { "Content-Type": "text/plain" });
    response.end("404 error!\n");
});
 
http.createServer(app).listen(1337);
```

是的，这样的过程略显繁琐，因此`express`把connect封装到了自己模块里，用express实现上面的过程简单了不少（往下看） 。


# 请求和响应
## **Request 对象**
> request 对象表示 HTTP 请求，包含了请求查询字符串，参数，内容，HTTP 头部等属性。常用属性和方法有16项

 - **`req.app`**   当callback为外部文件时，用req.app访问express的实例。
 -  **`req.baseUrl`**  获取路由当前安装的URL路径
 - **`req.body`   /  `req.cookies`**  获得「请求主体」/ Cookies
 - **`req.fresh`  /  `req.stale`**  判断请求是否还「新鲜」
 - **`req.hostname` / `req.ip`**   获取主机名和IP地址 
 - **`req.originalUrl` **   获取原始请求URL
 - **`req.params`**   获取路由的parameters
 - **`req.path`**   获取请求路径
 - **`req.protocol`**   获取协议类型
 - **`req.query`**  获取URL的查询参数串
 - **`req.route`**   获取当前匹配的路由
 - **`req.subdomains`**  获取子域名
 - **`req.accpets()`**  检查请求的Accept头的请求类型
 - **`req.acceptsCharsets` / `req.acceptsEncodings` / `req.acceptsLanguages`**
 - **`req.get()`**  获取指定的HTTP请求头
 - **`req.is()`**  判断请求头Content-Type的MIME类型

## **Response 对象**
> response 对象表示 HTTP 响应，即在接收到请求时向客户端发送的 HTTP 响应数据。常见属性有以下17项。

 - **`res.app`**  同req.app一样
 - **`res.append()`** 追加指定HTTP头
 -  **`res.set()`** 在`res.append()`后将重置之前设置的头
 - **`res.cookie(name，value [，option])`**  设置Cookie
 - **`opition` **  `domain` / `expires` / `httpOnly` / `maxAge` / `path` / `secure` / `signed`
 - **`res.clearCookie()`** 清除Cookie
 - **`res.download()`** 传送指定路径的文件
 - **`res.get()`**  返回指定的HTTP头
 - **`res.json()`** 传送JSON响应
 - **`res.jsonp()`** 传送JSONP响应
 - **`res.location()`** 只设置响应的Location HTTP头，不设置状态码或者close response
 - **`res.redirect()`** 设置响应的Location HTTP头，并且设置状态码302
 - **`res.send()`** 传送HTTP响应
 - **`res.sendFile(path [，options] [，fn])`**  传送指定路径的文件 -会自动根据文件extension设定Content-Type
 - **`res.set()`**  设置HTTP头，传入object可以一次设置多个头
 - **`res.status()`**  设置HTTP状态码
 - **`res.type()`**   设置Content-Type的MIME类型

# 路由

最原始的路由：
```
var http = require("http");
http.createServer(function(req, res) {
  
// Homepage
    if(req.url == "/") {
        res.writeHead(200, { "Content-Type": "text/html" });
        res.end("Welcome to the homepage!");
    }
 
// About page
    else if (req.url == "/about") {
        res.writeHead(200, { "Content-Type": "text/html" });
        res.end("Welcome to the about page!");
    }
 
// 404'd!
    else {
        res.writeHead(404, { "Content-Type": "text/plain" });
        res.end("404 error! File not found.");
    }
 
}).listen(1337, "localhost");
```

express封装的路由：
```
var express = require('express');
var app = express();

//  主页输出 "Hello World"
app.get('/', function (req, res) {
   console.log("主页 GET 请求");
   res.send('Hello GET');
})

//  POST 请求
app.post('/', function (req, res) {
   console.log("主页 POST 请求");
   res.send('Hello POST');
})

//  /del_user 页面响应
app.delete('/del_user', function (req, res) {
   console.log("/del_user 响应 DELETE 请求");
   res.send('删除页面');
})

//  /list_user 页面 GET 请求
app.get('/list_user', function (req, res) {
   console.log("/list_user GET 请求");
   res.send('用户列表页面');
})

// 对页面 abcd, abxcd, ab123cd, 等响应 GET 请求
app.get('/ab*cd', function(req, res) {   
   console.log("/ab*cd GET 请求");
   res.send('正则匹配');
})

var server = app.listen(8081, function () {
  var host = server.address().address
  var port = server.address().port
  console.log("应用实例，访问地址为 http://%s:%s", host, port)
});
```

# 视图
express结合前端模板引擎可以在服务端实现视图的渲染，这里还是以 jade模板引擎为例：
```
// 启动Express
var express = require("express");
var app = express();
 
// 设置view目录
app.set("views", __dirname + "/views");
 
// 设置模板引擎
app.set("view engine", "jade");
```
> 开头部分的代码和前面基本一样。之后我们指定视图文件所在目录。然后告诉Express我们要使用 Jade作为模板引擎。

接下来 我们建立一个名为 index.jade 的文件并把它放入 views 目录。代码如下：
```
doctype 5
html
  body
    h1 Hello, world!
    p= message
```
我们需要从Express中渲染这个视图。代码如下：
```
app.get("/", function(request, response) {
    response.render("index", { message: "I love anime" });    
});
```
Express为 response 对象添加了一个 render 方法。这个方法可以处理很多事情，但最主要的还是加载模板引擎和对应的视图文件，之后渲染成普通的HTML文档，例如这里的 index.jade.


# 参考资料
> @参考 [《深入理解 Express.js》](http://blog.jobbole.com/41325/)
