title: 断点调试你的express项目
date: 2016-01-16 15:07:33
tags: [NodeJS,express] 
categories: 大前端 
---


# 写在前面
前端工程师接触最多的是JavaScript，JavaScript程序可以通过浏览器进行调试，比如chrome的调试工具、Firefox的FireBug等。现在大前端趋势下前端工程师开始接触NodeJS这个JavaScript的孪生兄弟，俗话说殊途同归，NodeJS当然也可以通过chrome的调试工具进行断点调试。这篇文章着重介绍一下这个技术。

<!--more-->
# node-inspector
## 全局安装node-inspector 
执行这一步之前请确保已成功安装NodeJS环境，然后执行如下代码安装 node-inspector模块。
```
npm install -g node-inspector
```
## 以debug模式开启express服务
我的Node项目习惯用express搭建，然后用supervisor自动监听改动并重启服务。如果对这个模式不太了解，请用力戳 [这里](http://jafeney.com/2016/01/10/express%E6%90%AD%E5%BB%BAnodeJS%E4%B8%AD%E9%97%B4%E5%B1%82%EF%BC%88%E4%B8%89%EF%BC%89/) 。

```
supervisor --debug ./bin/www 
```
![这里写图片描述](http://img.blog.csdn.net/20160116144740669)

## 启动node-inspector
接着再打开一个命令窗口，执行下面的命令
```
node-inspector &
```
![这里写图片描述](http://img.blog.csdn.net/20160116145143011)

这样 node-inspector服务就启动了，打开chrome浏览器（或者chrome内核的浏览器），输入 
```
http://localhost:8080/?ws=localhost:8080&port=5858
```
> **注意**  如果不想默认监听8080端口，可以手动设置端口号，比如 
```
node-inspector & -p 8888
```

## 调试NodeJS程序
接下来我们就可以 在chrome里 像之前调试 JavaScript一样 调试我们的NodeJS程序了 

![这里写图片描述](http://img.blog.csdn.net/20160116150401537)


