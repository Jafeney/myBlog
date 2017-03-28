title: 基于webpack的模块化构建
date: 2016-02-02 15:51:43
tags: 模块化 
categories: 前端 
---

# 写在前面
模块化构建会让项目的拓展性、代码复用性和可维护性大大提高，初期可能会增加一些管理的工作量。但是对长远来说绝对是值得的，一个良好的模块化方案会让维护工作变得轻松，这个好处项目越进展到后来越明显。而且模块化构建的框架和工具很多，`RequireJS`、`SeaJS`、`Grunt`、`Gulp`等，这些虽然成熟稳定但并不是我们今天的主题，既然是采用`react`开发webApp，那么模块化当然是选`webpack`。

<!--more-->

# webpack的优势
1、支持CommonJs和AMD模块，意思也就是我们基本可以无痛迁移旧项目
2、支持模块加载器和插件机制，可对模块灵活定制。babel-loader更是有效支持ES6。
3、可以通过配置，打包成多个文件。有效利用浏览器的缓存功能提升性能。
4、将样式文件和图片等静态资源也可视为模块进行打包。配合loader加载器，可以支持sass，less等CSS预处理器。
5、内置有source map，即使打包在一起依旧方便调试。

![这里写图片描述](http://img.blog.csdn.net/20160202152747754)

# webpack入门
首先介绍`webpack`的配置文件`webpack.config.js`
```
  var webpack=require('webpack');

  module.exports={
    entry:[
      'webpack/hot/onlu-dev-server',
      './js/app.js'
    ],
    output:{
      path:'./build',
      filename:'bundle.js'
    },
    module:{
      leaders:[
        {test:/\.js?$/,leaders:['react-hot','babel'],exclude:/node_modules/},
        {test:/\.js$/,exclude:/node_modules/,loader:/'babel-loader'},
        {test:/\.css$/,loader:'style!css'},
        {test:/\.less/,loader:'style-loader!css-loader!less-loader'}
      ],
    },
    resolve:{
      extensions:['','.js','.json']
    },
    plugins:[
      new webpack.NoErrorsPlugin()
    ]
  }
```

> `webpack.config.js` 文件通常放在项目的根目录里，它本身也是一个标准的`Common.js`规范的模块。在导出的配置对象中有几个关键的参数：

## entry
`entry`可以说个字符串或数组或者是对象。当`entry`是个字符串的时候用来定义入口文件。
```
  entry:'./js/main.js'
```
当`entry`是个数组的时候，里面同样包含入口文件，另外一个参数可以用来配置`webpack`提供的一个静态资源服务器如`webpack-dev-server`，它会监控项目中每一个文件的变化，实时进行构建，并且自动刷新页面，
```
  entry:[
    'webpack/hot/only-dev-server',
    './js/app.js'
  ]
```
当`entry`是一个对象的时候，我们可以将不同的文件构建成不同的文件，按需使用，比如：
```
  entry:{
    hello:'./js/hello.js',
    form:'./js/form.js'
  }
```

## output
`output`是个对象，用户定义构建后的文件的输出。其中包含`path`和`filename`:
```
  output:{
    path:'./build',
    filename:'bundle.js'
  }
```
当我们在`entry`中定义构建多个文件时，`filename`可以对应地更改为`[name].bundle.js`。

## module
关于模块的加载相关，我们就定义在`module.loaders`中，这里通过正则表达式去匹配不同后缀的文件名。然后它们定义不同的加载器，比如说给less文件定义三个串联的加载器（用`!`连接）
```
moudle:{
  loaders:[
    {test:/\.js?$/,loaders:['react-hot','babel'],exclude:/node_modules/},
    {test:/\.js$/,exclude:/node_modules/,loader:'babel-loader'},
    {test:/\.css$/,loader:'style!css'},
    {test:/\.less$/,loader:'style-loader!css-loader!less-loader'}
  ]
}
```
除此之外，我们还可以用来定义 `png`、`jpg`这样的图片资源在小于10k时自动处理为base64图片的加载器。
```
  {test:/\.(png|jpg)$/,loader:'url-loader?limit=10000'}
```
给`css`和`less`还有图片添加了`loader`之后，我们不仅可以像在`node`中那样`require`进来了。就像这样：
```
require('./bootstrap.css');
require('./myapp.less');
var img=document.createElement('img');
img.src=require('./glyph.png');
```
> 注意： 这样`require`进来的文件会内联到 `[name].bundle.js`中。如果我们需要把保留 `require`写法又想把`css`文件单独拿出来，可以使用`extract-text-webpack-plugin`插件。
```
$ npm install extract-text-webpack-plugin --save-dev
```
```
var ExtractTextPlugin =require('extract-text-webpack-plugin');
module.exports={
  plugins:[
    new ExtractTextPlugin("[name].css");
  ]   
}
```
在上面的示例代码中配置的第一个`loader`是`react-hot`的加载器，它可以实现`react`组件的热替换，我们已经在`entry`参数中配置了`webpack/hot/only-dev-server`,所以我们只要启动 `webpack`时开启`-hot`参数既可以使用`react-hot-loader`了，在`package.json`中这样定义：
```
  "script":{
    "start":"webpack-dev-server --hot --progress --colors",
    "build":"webpack --progress --colors"
  }
```

## resolve
`webpack`在构建包的时候会按目录的文件进行查找，`resolve`属性中的`extensions`数组中用于配置程序可有自动替换哪些文件的后缀,比如：
```
  resolve:{
    extensions:['','.js','.json']
  }
```
然后我们想要加载一个`js`文件时，只要`require('common')`就可以加载`common.js`文件了。

## plugin
`webpack`提供了丰富的组件来满足不同的需求，当然如果你足够牛也可以自己编写插件。这里介绍比较常用的 `NoErrorsPlugin`，它能跳过编译时出错的代码并记录，是编译后运行的包不会发生错误。
```
  plugins:[
    new webpack.NoErrorsPlugin()
  ]
```

## externals
当我们想在项目中`require`一些其他的类库或者API，而又不想让这些类库的源码构建到运行时的文件中，就可以通过配置`externals`参数来解决这个问题，比如jQuery类库
```
  externals:{
    "jquery":'jQuery'
  }
```
这样我们就可以放心在项目中使用这些API了
```
  var jQuery=require('jquery');
```

## context
当我们在require一个模块的时候，如果`require`中包含变量，想这样：
```
  require("./mods/"+name+".js");
```
那么在编译的时候是不能知道具体的模块的。但这个时候`webpack`也会帮我们分析：
1. 分析目录：`'./mods'`;
2. 提取正则表达式：`/^.*\.js$/`;
于是这个时候为了更好地配合`webpack`进行编译，我们可以给它指明路径，像在`cake-webpack-config`中所做的那样：
```
  var currentBase=progress.cwd();
  var context=abcOptions.options.context?abcOptions.options.context:path.isAbsolute(entryDir)?enentryDir:path.join(currentBase,entryDir);
```

# 让webpack和gulp强强联手
`webpack`与`gulp`并不矛盾，甚至一起使用会得到最大化的效益。大致的思路是这样：
使用`webpack`进行`assets`编译，使用`gulp`的流快速对其他资源进行打包。
## 关于工具的定位
`webpack`的定位是`module bundler`，作为模块化工具，它的竞争对手其实是`browserify`，而不是`gulp`。
## 功能和使用方式上的不同
webpack 提供了一些非常使用的功能，像我们之前用到的那些，如图片的处理、`resolve`的处理、分开构建等,配置相当简单。`gulp`想要使用这些功能的时候我们可以在`webpack`里先配置好，做到强强联手！

--- 

> @参考 [推酷 webpack前端模块加载工具](http://www.tuicool.com/articles/2qiE7jN) 
> @参考 [博客园 前端模块化工具-webpack](http://www.cnblogs.com/Leo_wl/p/4862714.html) 
> @我的技术博客 [Jafeney.com](http://jafeney.com/) 


