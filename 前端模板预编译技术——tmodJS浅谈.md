title: 前端模板预编译技术——tmodJS浅谈
date: 2016-01-14 11:57:02
tags: [模板引擎,NodeJS] 
categories: 大前端  
---

# 什么是前端模板预编译
前端模板预编译通过预编译技术让前端模板突破浏览器限制，实现后端模板一样的同步“文件”加载能力。它采用目录来组织维护前端模板，从而让前端模板实现工程化管理，最终保证前端模板在复杂单页 web 应用下的可维护性。同时预编译输出的代码经过多层优化，能够在最大程度节省客户端资源消耗。

<!--more-->

## **按文件与目录组织模板**
```
template('tpl/home/main', data)
```

## **模板支持引入子模板**
```
{{include '../public/header'}}
```

>TmodJS 一经启动，就无需人工干预，每次模板创建与更新都会自动编译，引入一个 js 即可使用template(path)接口调用本地模板文件，直到正式上线都无需对代码进行任何修改，整个过程简单自然。

# 强大之处
1. 编译模板为不依赖引擎的 js 文件
2. 前端模板按文件与目录组织
3. 模板之间支持引入外部模板
4. 使用同步模板加载接口
5. 可选多种规范的模块输出：AMD、CMD、CommonJS
6. 支持作为 GruntJS 的插件构建项目
7. 模板目录可被前后端共享
8. 支持检测修改即时编译
9. 支持模板运行时调试
10. 配置文件支持多人共享

# 使用tmodJS
## **全局安装**
```
npm install -g tmodjs
```
## **编写模板**
TmodJS 的前端模板不再耦合在业务页面中，而是和后端模板一样有专门的目录管理。目录名称只支持英文、数字、下划线的组合，一个模板对应一个.html文件。
## **模板语法**
和artTemplate一脉相承，建议协同使用。
### 表达式
{{ 与 }} 符号包裹起来的语句则为模板的逻辑表达式。
### 输出表达式
对内容编码输出：
```
{{content}}
```
不编码输出（编码可以防止数据中含有 HTML 字符串，避免引起 XSS 攻击）

```
{{#content}}
```
### 条件表达式

```
{{if admin}}
    <p>admin</p>
{{else if code > 0}}
    <p>master</p>
{{else}}
    <p>error!</p>
{{/if}}
```
### 遍历表达式
无论数组或者对象都可以用 each 进行遍历。

```
{{each list as value index}}
    <li>{{index}} - {{value.user}}</li>
{{/each}}
```
亦可以被简写：

```
{{each list}}
    <li>{{$index}} - {{$value.user}}</li>
{{/each}}
```

### 模板包含表达式
用于嵌入子模板：
```
{{include 'template_name'}}
```
子模板默认共享当前数据，亦可以指定数据：

```
{{include 'template_name' news_list}}
```
### include 路径规范约定
1. 路径不能带后缀名
2. 路径不能够进行字符串运算
3. 路径不能使用变量代替
4. 必须使用以.开头的相对路径
### 辅助方法
```
{{time | dateFormat:'yyyy-MM-dd hh:mm:ss'}}
```
支持传入参数与嵌套使用：
```
{{time | say:'cd' | ubb | link}}
```
>为了模板可维护，模板本身是不能随意访问外部数据的，它所有的语句都将运行在一个沙箱中。如果需要访问外部对象可以注册辅助方法，这样所有的模板都能访问它们。

**新建一个辅助方法文件配置**
在模板目录新建一个 config/template-helper.js 文件，然后编辑 package.json 设置"helpers": "./config/template-helper.js"。

**编写辅助方法**
在 config/template-helper.js 中声明辅助方法。

以日期格式化为例：
```
template.helper('dateFormat', function (date, format) {

    date = new Date(date);

    var map = {
        "M": date.getMonth() + 1, //月份 
        "d": date.getDate(), //日 
        "h": date.getHours(), //小时 
        "m": date.getMinutes(), //分 
        "s": date.getSeconds(), //秒 
        "q": Math.floor((date.getMonth() + 3) / 3), //季度 
        "S": date.getMilliseconds() //毫秒 
    };
    format = format.replace(/([yMdhmsqS])+/g, function(all, t){
        var v = map[t];
        if(v !== undefined){
            if(all.length > 1){
                v = '0' + v;
                v = v.substr(v.length-2);
            }
            return v;
        }
        else if(t === 'y'){
            return (date.getFullYear() + '').substr(4 - all.length);
        }
        return all;
    });
    return format;
});
```
调用：
```
{{time | dateFormat:'yyyy-MM-dd hh:mm:ss'}}
```

## **编译模板**
只需要运行tmod这个命令即可，默认配置参数可以满足绝大多数项目。
```
tmod [模板目录] [配置参数]
```
> 注意：模板目录必须是模板的根目录，若无参数则为默认使用当前工作目录，tmodjs 会监控模板目录修改，每次模板修改都会增量编译。

## **配置参数**

 - `--debug` 输出调试版本
 - `--charset value` 定义模板编码，默认`utf-8`
 -  `--output value` 定义输出目录，默认`./build`
 - `--type value` 定义输出模块格式，默认`default`，可选`cmd`、`amd`、`commonjs`
 - `--no-watch` 关闭模板目录监控
 -  `--version` 显示版本号
 -  `--help` 显示帮助信息

> 注意：配置参数将会保存在模板目录配置文件中，下次运行无需输入配置参数（--no-watch 与 --debug 除外）。
### 举个例子
```
tmod ./tpl --output ./build
```

## **使用模板**
根据编译的 `type` 的配置不同，会有两种不同使用方式：
### 使用默认的格式
TmodJS 默认将整个目录的模板压缩打包到一个名为 template.js 的脚本中，可直接在页面中使用它：
```
<script src="tpl/build/template.js"></script>
<script>
    var html = template('news/list', _list);
    document.getElementById('list').innerHTML = html;
</script>
```
> RequireJS、SeaJS、NodeJS 加载 在线实例 http://aui.github.io/tmodjs/test/index.html 

### 指定格式（amd / cmd / commonjs)
此时每个模板就是一个单独的模块，无需引用 template.js：

```
var render = require('./tpl/build/news/list');
var html = render(_list);
```
> 注意：模板路径不能包含模板后缀名

## **演示**
TmodJS 源码包中test/tpl是一个演示项目的前端模板目录，基于默认配置。切换到源码目录后，编译：
```
tmod test/tpl
```
编译完毕后你可以在浏览器中打开 `test/index.html` 查看如何使用编译后的模板。

## **配置**
TmodJS 的项目配置文件保存在模板目录的 package.json 文件中：

```
{
    "name": "template",
    "version": "1.0.0",
    "dependencies": {
        "tmodjs": "1.0.0"
    },
    "tmodjs-config": {
        "output": "./build",
        "charset": "utf-8",
        "syntax": "simple",
        "helpers": null,
        "escape": true,
        "compress": true,
        "type": "default",
        "runtime": "template.js",
        "combo": true,
        "minify": true,
        "cache": false
    }
}
```
![这里写图片描述](http://img.blog.csdn.net/20151220142609220)

### gulp配置
让 TmodJS 作为 Gulp 的一个插件使用：
```
npm install gulp-tmod --save-dev
```
由@lichunqiang开发，项目主页：https://github.com/lichunqiang/gulp-tmod

## **常见Q&A**
问：TmodJS 需要部署到服务器中吗？
> 不需要，这是本地工具，基于 NodeJS 编写是为了实现跨平台。

问：如何将每个模板都编译成单独的 amd/cmd 模块输出？
> 指定 type 参数即可，如--type cmd则可以让每个模板都支持 RequireJS/SeaJS 调用。

问：如何将模板编译成 NodeJS 的模块？
> 指定 type 参数即可，如--type commonjs。

问：线上运行的模板报错了如何调试？
> 开启 debug 模式编译，如--debug，这样会输出调试版本，可以让你快速找到模板运行错误的语句以及数据。

问：如何不压缩输出 js？
> 编辑配置文件，设置"minify": false。

问：如何修改默认的输出目录？
> 指定 output 参数即可，如--output ../../build 

如何让模板访问全局变量？
> 具体参考上面的 **辅助方法**。

问：可以使用使用类似 tmpl 那种的 js 原生语法作为模板语法吗？
> 可以。编辑配置文件，设置"syntax": "native"即可，目前 TmodJS 默认使用的是 simple 语法。

问：如何兼容旧版本 atc 的项目？
> 编辑配置文件，分别设置"type": "cmd"、"syntax": "native"、"output": "./"

问：如何迁移原来写在页面中的 artTemplate 模板，改为 TmodJS 这种按按文件存放的方式？
> 参考 [《页面中的模板迁移指南》](https://github.com/aui/tmodjs/wiki/%E9%A1%B5%E9%9D%A2%E4%B8%AD%E7%9A%84%E6%A8%A1%E6%9D%BF%E8%BF%81%E7%A7%BB%E6%8C%87%E5%8D%97)

问：我需要手动合并模板，如何让 tmodjs 不合并输出？
> 编辑配置文件，设置combo:false。

---

> @参考 [《前端模板外置解决方案》](https://github.com/aui/tmodjs/)
