title: express搭建nodeJS中间层（三）
date: 2016-01-10 11:24:14
tags: express 
categories: 大前端 
---

# 写在前面
之前2篇已经比较详细地介绍用express搭建nodeJS中间层并部署到centOS服务器，用forever管理nodeJS进程，这一系列工作 都是基于 项目已经调试 完毕了。但实际开发过程中会发现 每次修改完 代码后 都需要关闭node进程然后 重启才能生效，特别麻烦，这篇就介绍一个 自动监听并同步的的 node工具——supervisor。

<!--more-->

# 全局安装 supervisor
直接用npm安装既可，输入指令 :
```
$ npm -g install supervisor
```
> 这里注意一点的就是，supervisor必须安装到全局，如果你不安装到全局，错误命令会提示你安装到全局。

如果不想安装到默认的全局,也可以自己修改全局路径到当前路径 :
```
$ npm config set prefix "路径"
```

# 用supervisor启动express项目
安装完以后就可以 用 supervisor 来启动 express项目了，进入项目根目录，执行：
```
$ supervisor ./bin/www
```
> 如果你之前已经 用forever 来托管这个进程了，在执行 supervisor之前 应该关闭 这个进程。

关闭所有forever托管的进程：
```
$ forever stopall
```
然后再执行supervisor的命令，这样你的项目一旦发生更改（只要不是语法错误），supervisor会帮你自动同步到该进程里来，不需要重新启动。

![这里写图片描述](http://img.blog.csdn.net/20151225164959721)

好了，顺便做个小测试吧，我们先在浏览器打开这个网站。数据已经监听到了：

![这里写图片描述](http://img.blog.csdn.net/20151225165151911)

随意修改一个 nodeJS文件，然后会看到：

![这里写图片描述](http://img.blog.csdn.net/20151225170051429)

node服务已经重新启动了，很方便对吧  ^_^ 。

# 让supervisor监听模板文件的改动
> 默认情况下，supervisor只能监听 nodeJS的文件，其他的文件改动它是不会捕捉到的。下面我们 通过添加 启动参数 的方式扩展这一功能。

首先 我们明确下 supervisor的几个 `options` 的用法：
```
//要监控的文件夹或js文件，默认为'.'
-w|--watch <watchItems>

//要忽略监控的文件夹或js文件  
-i|--ignore <ignoreItems>

//监控文件变化的时间间隔（周期），默认为Node.js内置的时间
-p|--poll-interval <milliseconds>

//要监控的文件扩展名，默认为'node|js'
-e|--extensions <extensions>

//要执行的主应用程序，默认为'node'
-x|--exec <executable>

//开启debug模式（用--debug flag来启动node）
--debug

//安静模式，不显示DEBUG信息

-q|--quiet
```

好了，看了上面的介绍，大家应该注意到 `--extensions <extensions>` 参数，对的，我们把 需要添加监听的文件名后缀 添加进去就可以了。我的项目里采用的ArtTemplate模板引擎，所有模板文件的后缀名是 `.art` ，所以我启动 supervisor的命令是这样的：

```
$ supervisor --extensions art ./bin/www
```

这样 模板文件 你更改后也能生效了，当然如果css文件也要同时添加监听，可以这么写：

```
$ supervisor --extensions art,css ./bin/www
```
运行的效果是这样的：

![这里写图片描述](http://img.blog.csdn.net/20151225172944056)

# 后话
当前 国内 关于express 4.x 搭建nodeJS中间层的 文档和一手资料不多，每当遇到问题时 就需要 去翻墙看看国外网站的或者翻译一些英文的帖子，着实不易啊。希望 我的这些 “战地笔记”能为有需要的人提供参考价值吧 ^ _ ^ 

> @参考 [《supervisor模块监控nodejs文件的变化并自动刷新》](http://www.w3cfuns.com/blog-5468606-5410375.html) 



