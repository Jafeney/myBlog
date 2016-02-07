title: express搭建nodeJS中间层（二）
date: 2016-01-10 11:22:39
tags: express 
categories: 大前端 
---

# 写在前面
上一篇写到用express搭建本地环境而且成功实现了路由和模板渲染，这一篇具体讲讲如何把一个原本用artTemplate渲染的前端页面用nodeJS渲染并返回给浏览器。

# 改造过程
> 上一篇已经在express里配置好了artTemplate模板引擎，所以这里的改造变得无比简单。

<!--more-->

## **删除不需要的依赖项**
之前在 `<head></head>`里引入的artTemplate类库存在的目的是在客户端用JavaScript调用template函数，对指定id的模板进行渲染，服务端这个步骤是不需要的，直接删除即可。

接下来再把这个结构删掉，因为服务端是直接把html文件作为模板引入的，不需要在单独声明 `script` 的类型和模板的id。
```
<script type="text/html" id="xxxx"></script>
```

最后一个依赖是用原本用JavaScript 调取接口获取data，然后用 template() 方法指定模板进行渲染这个过程，我们等下要把它放到服务端来完成，所以客户端里也不需要了。在我的设计模式里，只需要把 `fGetData()` 这个模块注释掉就行了。
```
/**
		 * [单例初始化入口]
		 * @return {[type]} [description]
		 */
		init:function(){
			var self=this;

			/*执行模板渲染模块*/
			self.fGetData(); //现在不需要了，把它注释掉吧！
			
			self.eventBind($,self);
			/*执行图片加载缓冲*/
			$('.lazyload').lazyload({effect : "fadeIn"});
		}
```


## **静态资源迁移**
我的项目是用gulp自动化管理的，只需要把 `builder` 、 `lib` 、`html` 这三个目录分别 copy到express工程 public目录下的 `JavaScripts` 、`stylesheets` ，根目录下的 `view` 文件夹里（文件位置具体怎么放看你的 个人爱好）。

当然这样迁移好还不够，模板文件里的路径也要改成迁移后的新路径。

> 注意：因为 之前我们再 app.js 里设置了静态资源的引用路径，所以路由 `/index` 下 静态资源 是可以直接访问的。比如 就像下面这样就能 引用到这2个 css文件了:
```
<head>
	<meta charset="utf-8" />
	<title>首页</title>
	<link rel="stylesheet" type="text/css" href="stylesheets/base.min.css"/>
	<link rel="stylesheet" type="text/css" href="stylesheets/style.min.css"/>
</head>
```

## **服务端渲染**
我们的数据本来是通过调取Java接口获取到的，所以这里 服务端 要用到 nodeJS的 `request` 模块，先安装它：
```
$ npm install request
```

有了request，我们就能在nodeJS里随意调用 Java写好的数据接口了。

接下里打开路由目录下的index.js 文件，在首页的路由里把模板渲染的工作 完成
```
var express = require('express');
var request= require('request');
var router = express.Router();

/* GET home page. */
router.get('/', function(req, res, next) {
	/*正式数据*/
  request('http://test.webapp.baai.com/hk/index.json',function(error,response,body){
    /*判断请求是否成功*/
    if (!error && response.statusCode == 200) {
      /*把字符串转换为json*/
      var data=JSON.parse(body); 
      /*渲染模板*/
      res.render('index', data);
    }
  });
});
```

 > 注意： 返回的数据默认格式是string，如果需要json格式，要用JSON.parse()进行处理，不然就会报错


## **部署到远程服务器**
这个过程 需要 懂一点 Linux，先用 文件服务工具  登录到 远程服务器，把 项目资源copy到线上服务器。 

![这里写图片描述](http://img.blog.csdn.net/20151222174919033)

接下来就是 安装nodeJS环境了，这个步骤网上教程太多了，不同的系统不一样（不会的自己 [百度](http://www.baai.com) 吧）。我用的服务器是阿里云的 centOS，本身自带nodeJS环境，所以不用从头安装，不过node的版本比较低，所以我简单升级了一下环境。

1、检查 Node的当前版本，使用命令
```
$ node -v
$ npm -v
```
2、清除npm cache
```
$ sudo npm cache clean -f 
```
3、全局安装 Node Binary管理模块“n”	
```
$ npm install n -g
```
4、升级到最新版本（该步骤可能需要花费一些时间）
```
$ sudo n stable  
```
5、查看Node的版本，检查升级是否成功
```
$ node -v  
$ npm -v
```

## **Nginx站点与NodeJS站点共存的配置**
首先是网站入口问题，Nginx使用了80端口，NodeJS使用8080端口。我们利用Nginx的“proxy_pass”将对80端口NodeJS站点的访问导向8080端口，在LuManager中，这个配置十分简单：

1、进入LuManager后台，用“快速建站”建立NodeJS网站，如testnodejs.com网站，这里也可建立多个NodeJS网站，网站域名按如下方式填写即可：
```
testnodejs.com *.testnodejs.com testnodejs2.com *.testnodejs2.com
```
使他们指向共同的NodeJS网站群根目录，如nodejs目录。

2、然后为NodeJS网站配置Nginx，修改该NodeJS网站配置，进入选填项，在Nginx扩展设置(location段)**添加如下代码：
```
proxy_pass http://127.0.0.1:8080;
```
如此一来，外部对testnodejs.com、testnodejs2.com等NodeJS网站的访问全部转向了8080端口，NodeJS监听8080端口即可。而该NodeJS网站群的根目录即上面设置的nodejs目录，我们可在该目录中再搭设testnodejs.com、testnodejs2.com等虚拟站点。
 
## **安装Forever后台管理器**
我们不可能直接通过node命令来管理远程站点，这样无法保证网站的可持续运行。我们用Forever来解决这个问题，它可以将NodeJS应用以后台守护进程的方式运行，我们还可以将NodeJS应用设成随系统启动而自动运行。
首先，全局安装Forever：
```
$ npm install forever -g
```
安装 完后我们 就要让 express项目在后台自行运行了：

进入到 你的项目的根目录。原本 express项目的启动命令是 `npm start`，这里我用forever启动肯定不能这么搞了，我们打开 项目的package.json文件：

```
{
  "name": "node_hk",
  "version": "0.0.0",
  "private": true,
  "scripts": {
    "start": "node ./bin/www"
  },
  "dependencies": {
    "body-parser": "~1.13.2",
    "cookie-parser": "~1.3.5",
    "debug": "~2.2.0",
    "express": "~4.13.1",
    "art-template": "~3.0.3",
    "morgan": "~1.6.1",
    "serve-favicon": "~2.3.0"
  }
}
```
注意到 “start”脚本对应的shell指令是 `node ./bin/www` ,换句话说 `npm start` 实际上执行的就是这个 shell命令，那么我们很容易想到 项目启动的命令 实际上就是 它了，只是 express 4.x 把这个工作 托管给了 npm。

于是，forever启动项目的命令就是：
```
$ forever start ./bin/www
```

好了，执行这个命令后，你的项目就托管给 forever在后台自动运行了，你关闭了 窗口也不会影响 node服务。然后再说说 关闭命令：

先查看当前交给 forever 托管的node进程：
```
$ forever list
```
这个命令会列出当前 被 forever托管的 nodeJS进程，如果想关闭那个，把它的id找到，执行：
```
$ forever stop [id]
```
如果你懒得找进程，那干脆全部关闭吧，执行：

```
$ forever stopall
```

###**Forever使用介绍**
**子命令actions：**
```
start: 启动守护进程
stop: 停止守护进程
stopall: 停止所有的forever进程
restart: 重启守护进程
restartall: 重启所有的foever进程
list: 列表显示forever进程
config: 列出所有的用户配置项
set <key> <val>: 设置用户配置项
clear <key>: 清楚用户配置项
logs: 列出所有forever进程的日志
logs <script|index>: 显示最新的日志
columns add <col>: 自定义指标到forever list
columns rm <col>: 删除forever list的指标
columns set<cols>: 设置所有的指标到forever list
cleanlogs: 删除所有的forever历史日志
```
**配置参数options：**
```
-m MAX: 运行指定脚本的次数
-l LOGFILE: 输出日志到LOGFILE
-o OUTFILE: 输出控制台信息到OUTFILE
-e ERRFILE: 输出控制台错误在ERRFILE
-p PATH: 根目录
-c COMMAND: 执行命令，默认是node
-a, append: 合并日志
-f, fifo: 流式日志输出
-n, number: 日志打印行数
pidFile: pid文件
sourceDir: 源代码目录
minUptime: 最小spinn更新时间(ms)
spinSleepTime: 两次spin间隔时间
colors: 控制台输出着色
plain: no-colors的别名，控制台输出无色
-d, debug: debug模式
-v, verbose: 打印详细输出
-s, silent: 不打印日志和错误信息
-w, watch: 监控文件改变
watchDirectory: 监控顶级目录
watchIgnore: 通过模式匹配忽略监控
-h, help: 命令行帮助信息
```

> @参考 [《Nodejs进程管理模块forever详解_node.js_》](http://www.html5cn.com.cn/shili/javascripts/4385.html) 

> @参考    zensh[《阿里云主机Nginx下配置NodeJS、Express和Forever》](https://cnodejs.org/topic/5059ce39fd37ea6b2f07e1a3) 


