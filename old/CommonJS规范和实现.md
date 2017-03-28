title: CommonJS规范和实现
date: 2016-01-10 12:02:27
tags: 模块化 
categories: 大前端 
---


# 写在前面
> 一言以蔽：**CommonJS是服务器端模块的规范，Node.js采用了这个规范。**
# commonJS简介
根据CommonJS规范，一个单独的文件就是一个模块。加载模块使用`require`方法，该方法读取一个文件并执行，最后返回文件内部的exports对象。

举个例子 `example.js`

<!--more-->

```
console.log("evaluating example.js");
var invisible = function () {
  console.log("invisible");
}
exports.message = "hi";
exports.say = function () {
  console.log(message);
}
```
我们可以通过 `require` 加载这个模块
```
var example = require('./example.js');
```
这时，变量`example`就对应模块中的`exports`对象，于是就可以通过这个变量，使用模块提供的各个方法。
```
{
  message: "hi",
  say: [Function]
}
```
`require`方法默认读取js文件，所以可以省略js后缀名。

```
var example = require('./example');
```
> **注意** js文件名前面需要加上路径，可以是相对路径（相对于使用`require`方法的文件），也可以是绝对路径。如果省略路径，node.js会认为，你要加载一个核心模块，或者已经安装在本地 node_modules 目录中的模块。如果加载的是一个目录，node.js会首先寻找该目录中的 package.json 文件，加载该文件 main 属性提到的模块，否则就寻找该目录下的 index.js 文件。

## 【例】最简单的模块  
模块 `addition.js`
```
exports.do = function(a, b){ return a + b };
```
上面的语句定义了一个加法模块，做法就是在`exports`对象上定义一个`do`方法，那就是供外部调用的方法。使用的时候，只要用`require`函数调用即可。
```
var add = require('./addition');
add.do(1,2);   // 输出3
```

## 【例】稍复杂些的例子
模块 `foobar.js`

```
function foobar(){
	this.foo = function(){
		console.log('Hello foo');
    }
    this.bar = function(){
        console.log('Hello bar');
    }
}
exports.foobar = foobar;
```
调用该模块的方法如下：

```
var foobar = require('./foobar').foobar,
    test   = new foobar();
test.bar(); // 输出 'Hello bar'
```
有时，不需要`exports`返回一个对象，只需要它返回一个函数。这时，就要写成`module.exports` 

```
module.exports = function () {
  console.log("hello world")
}
```

# AMD规范与CommonJS规范的兼容性
> CommonJS规范加载模块是同步的，也就是说，只有加载完成，才能执行后面的操作。AMD规范则是非同步加载模块，允许指定回调函数。由于Node.js主要用于服务器编程，模块文件一般都已经存在于本地硬盘，所以加载起来比较快，不用考虑非同步加载的方式，所以CommonJS规范比较适用。但是，如果是浏览器环境，要从服务器端加载模块，这时就必须采用非同步模式，因此浏览器端一般采用AMD规范。

AMD规范使用`define`方法定义模块（最典型的就是`requireJS`了）
```
define(['package/lib'], function(lib){
    function foo(){
        lib.log('hello world!');
    } 
    return {
        foo: foo
    };
});
```
当然，AMD规范允许输出的模块兼容`CommonJS`规范，这时`define`方法需要写成下面这样：

```
define(function( require, exports, module )
    var someModule = require( "someModule" );
    var anotherModule = require( "anotherModule" );    

    someModule.doTehAwesome();
    anotherModule.doMoarAwesome();

    exports.asplode = function() {
        someModule.doTehAwesome();
        anotherModule.doMoarAwesome();
    };
});
```

# Package.json定义
根据commonJS规范，每个nodeJS模块都必须有个`package.json`文件，也方便分享到`npm`社区。

## name
必须字段，在package.json中最重要的就是name和version字段。他们都是必须的，如果没有就无法install。name和version一起组成的标识在假设中是唯一的。改变包应该同时改变version。
name是这个东西的名字。注意：
1、不要把node或者js放在名字中。因为你写了package.json它就被假定成为了js，不过你可以用"engine"字段指定一个引擎（见后文）。
2、这个名字会作为在URL的一部分、命令行的参数或者文件夹的名字。任何non-url-safe的字符都是不能用的。
3、这个名字可能会作为参数被传入require()，所以它应该比较短，但也要意义清晰。
4、在取名字之前，你最好去 [npm registry](http://registry.npmjs.org/) 查看一下这个名字是否已经被使用了。 

## version
必须字段，在`package.json`中最重要的就是`name`和`version`字段。他们都是必须的，如果没有就无法`install`。`name`和`version`一起组成的标识在假设中是唯一的。改变包应该同时改变version。

> version必须能被 node-semver解析，它被包在npm的依赖中。（要自己用可以执行npm install semver）

## description
可选字段，模块描述，必须是字符串。`npm search`的时候会用到。

## keywords
可选字段，放简介，字符串。方便屌丝们在 `npm search`中搜索。

## homepage
可选字段，项目官网的url。
> **注意 ** 这和“url”不一样。如果你放一个“url”字段，registry会以为是一个跳转到你发布在其他地方的地址，然后喊你滚粗。 嗯，滚粗，没开玩笑。

## bugs
可选字段，你项目的提交问题的url和（或）邮件地址。这对遇到问题的屌丝很有帮助。
差不多长这样：
```
{ 
	"url" : "http://github.com/owner/project/issues",
	 "email" : "project@hostname.com"
}
```
你可以指定一个或者指定两个。如果你只想提供一个url，那就不用对象了，字符串就行。如果提供了url，它会被`npm bugs`命令使用。

## License
可选字段，你应该要指定一个许可证，让人知道使用的权利和限制的。
最简单的方法是，假如你用一个像BSD或者MIT这样通用的许可证，就只需要指定一个许可证的名字，像这样：
```
{ "license" : "BSD" }
```
如果你有更复杂的许可条件，或者想要提供给更多地细节，可以这样:

```
"licenses" : [
  { "type" : "MyLicense"
  , "url" : "http://github.com/owner/project/path/to/license"
  }
]
```
在根目录中提供一个许可证文件也蛮好的。

## people fields: author, contributors
都是可选字段。author是一个人。contributors是一堆人的数组。person是一个有name字段，可选的有url、email字段的对象，像这样：
```
{ 
	"name" : "Barney Rubble",
	"email" : "b@rubble.com",
	"url" : "http://barnyrubble.tumblr.com/"
}
```
或者可以把所有的东西都放到一个字符串里，npm会给你解析：
```
"Barney Rubble <b@rubble.com> (http://barnyrubble.tumblr.com/)
```
email和url在两种形式中都是可选的。也可以在你的npm用户信息中设置一个顶级的maintainers字段。

## Files
可选字段，files是一个包含项目中的文件的数组。如果命名了一个文件夹，那也会包含文件夹中的文件。（除非被其他条件忽略了）你也可以提供一个`.npmignore`文件，让即使被包含在files字段中得文件被留下。其实就像 `.gitignore`一样。

## main
可选字段。main字段配置一个文件名指向模块的入口程序。如果你包的名字叫foo，然后用require("foo")，main配置的模块的exports对象会被返回。这应该是一个相对于根目录的文件路径。对于大多数模块，它是非常有意义的，其他的都没啥。

## bin
可选字段。很多的包都会有执行文件需要安装到PATH中去。这个字段对应的是一个Map，每个元素对应一个{ 命令名：文件名 }。
```
{ "bin" : { "npm" : "./cli.js" } }
```
## directories
用于指示包的目录结构
## directories.lib
指示库文件的位置。
## directories.bin
和前面的bin是一样的，但如果前面已经有bin，那么这个就无效。除了以上两个，还有Directories.doc& Directories.man & Directories.example。
## repository
可选字段。用于指示代码存放的位置。
```
"repository" :
  { 
	  "type" : "git",
	  "url" : "http://github.com/npm/npm.git"
  }
 
"repository" :
  { 
	  "type" : "svn",
	  "url" : "http://v8.googlecode.com/svn/trunk/"
  }
```
## scripts
可选字段，object。Key是生命周期事件名，value是在事件点要跑的命令。参考npm-scripts。
## config
可选字段，object。Config对象中的值在Scripts的整个周期中皆可用，专门用于给Scripts提供配置参数。
## dependencies
可选字段，指示当前包所依赖的其他包。
```
{ 
	"dependencies" :{ 
	  "foo" : "1.0.0 - 2.9999.9999",
	  "bar" : ">=1.0.2 <2.1.2",
	  "baz" : ">1.0.2 <=2.3.4",
	  "boo" : "2.0.1",
	  "qux" : "<1.0.0 || >=2.3.1 <2.4.5 || >=2.5.2 <3.0.0",
	  "asd" : "http://asdf.com/asdf.tar.gz",
	  "til" : "~1.2",
	  "elf" : "~1.2.3",
	  "two" : "2.x",
	  "thr" : "3.3.x",
  }
}
```
版本格式可以是下面任一种：

```
version 完全匹配

>version 大于这个版本

>=version大于或等于这个版本

<version

<=version

~version 非常接近这个版本

^version 与当前版本兼容

1.2.x X代表任意数字，因此1.2.1, 1.2.3等都可以

http://... Unix系统下使用的tarball的URL。

* 任何版本都可以

""任何版本都可以

version1 - version2  等价于 >=version1 <=version2.

range1 || range2 满足任意一个即可

git... Git地址

user/repo
```

## devDependencies
可选字段。如果只需要下载使用某些模块，而不下载这些模块的测试和文档框架，放在这个下面比较不错。

## peerDependencies
可选字段。兼容性依赖。如果你的包是插件，适合这种方式。
## bundledDependencies
可选字段。发布包时同时打包的其他依赖。
## optionalDependencies
可选字段。如果你想在某些依赖即使没有找到，或则安装失败的情况下，npm都继续执行。那么这些依赖适合放在这里。
## engines
可选字段。既可以指定node版本：
```
{ "engines" : {"node" : ">=0.10.3 <0.12" } }
```
也可以指定npm版本：
```
{ "engines" : {"npm" : "~1.0.20" } }
```
## engineStrick
可选字段，布尔值。如果你肯定你的程序只能在制定的engine上运行，设置为true。

## os
可选字段。指定模块可以在什么操作系统上运行：
```
"os" : [ "darwin","linux" ]
"os" : [ "!win32" ]
```
## cpu
可选字段。指定CPU型号。
```
"cpu" : [ "x64","ia32" ]
"cpu" : [ "!arm","!mips" ]
```
## preferGlobal
可选字段，布尔值。如果你的包是个命令行应用程序，需要全局安装，就可以设为true。
## private
可选字段，布尔值。如果private为true，npm会拒绝发布。这可以防止私有repositories不小心被发布出去。
## publishConfig
可选字段。发布时使用的配置值放这。
## 默认值
```
"scripts":{"start": "node server.js"}
```
如果你的包里有server.js文件，npm默认将执行： node server.js.
```
"scripts":{"preinstall":"node-gyp rebuild"}
```
如果包里有binding.gyp，npm默认在preinstall命令时，使用node-gyp做编译。


# 参考资料 
> @参考 [《CommonJS规范》](http://m.blog.csdn.net/blog/buyaore_wo/17385065)
> @参考 [《Specifics of npm's package.json handling》](http://www.mujiang.info/translation/npmjs/files/package.json.html)
> @参考 [《package.json 字段全解析》](http://my.oschina.net/u/1992917/blog/523046)


