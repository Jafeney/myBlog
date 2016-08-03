title: webApp重构之路——性能和用户体验细节总结 
date: 2016-01-28 16:17:42 
tags: webApp  
categories: 移动开发 
---

# 写在前面
webApp开发有一段时间了，随着自己的知识面和技能水平不断提升，一个“重构“的想法变得越来越强烈，当然，重构之前，一个确切的方案是相当关键的，也可以说是重中之重， 吸取之前草率上阵的教训，这次要充分考虑可拓展性和可维护性，把模块化、性能优化、用户体验等做到我所能达到的极致。首先，我先总结开发过程中领悟到的提升webApp性能和用户体验的一些方案。

<!--more-->

# 性能优化
性能优化，顾名思义是要提升webApp的响应速度并减少不必要的内存和网络开销。这个有很多方案啊，[雅虎14条性能优化的方案](http://www.mahaixiang.cn/znseo/1056.html) 这里几乎都能用上，这里不抄书了，直接结合我的实战经验谈谈具体的做法。

## 压缩&合并
自从有了NodeJS的支持，前端静态资源压缩变得非常简单和快捷。这方面的技术很多，比如grunt、gulp、brower、webpack等，不同的工具API和功能也不尽相同。我用的比较熟练的是gulp，这里也抛砖引玉，具体介绍用gulp自动压缩和合并前端资源的策略：
> 
1、图片（压缩图片支持jpg、png、gif）
2、样式 （支持sass 同时支持合并、压缩、重命名）
3、javascript （检查、合并、压缩、重命名）
4、html （压缩）
5、客户端同步刷新显示修改
6、构建项目前清除发布环境下的文件（保持发布环境的清洁）

### 选择gulp组件
通过gulp plugins，寻找对于的gulp组件
> 
gulp-imagemin: 压缩图片
gulp-ruby-sass: 支持sass
gulp-minify-css: 压缩css
gulp-jshint: 检查js
gulp-uglify: 压缩js
gulp-concat: 合并文件
gulp-rename: 重命名文件
gulp-htmlmin: 压缩html
gulp-clean: 清空文件夹
gulp-livereload: 服务器控制客户端同步刷新（需配合chrome插件LiveReload及tiny-lr）

### 安装Gulp组件
```
npm install gulp-util gulp-imagemin gulp-ruby-sass gulp-minify-css gulp-jshint gulp-uglify gulp-rename gulp-concat gulp-clean gulp-livereload tiny-lr --save-dev
```
### 项目目录结构
project(项目名称)
```
|–.git 通过git管理项目会生成这个文件夹
|–node_modules 组件目录
|–dist 发布环境
    |–css 样式文件(style.css style.min.css)
    |–images 图片文件(压缩图片)
    |–js js文件(main.js main.min.js)
    |–index.html 静态文件(压缩html)
|–src 生产环境
    |–sass sass文件
    |–images 图片文件
    |–js js文件
    |–index.html 静态文件
|–.jshintrc jshint配置文件
|–gulpfile.js gulp任务文件
```
### gulp基础语法
gulp的语法相当简单，通过gulpfile文件来完成相关任务，因此项目中必须包含gulpfile.js。
gulp有五个方法：src、dest、task、run、watch 
>  
 |-src和dest：指定源文件和处理后文件的路径
 |- watch：用来监听文件的变化
 |- task：指定任务
 |- run：执行任务

### 编写gulp任务
```
/**
 * 组件安装
 * npm install gulp-util gulp-imagemin gulp-ruby-sass gulp-minify-css gulp-jshint gulp-uglify gulp-rename gulp-concat gulp-clean gulp-livereload tiny-lr --save-dev
 */

// 引入 gulp及组件
var gulp    = require('gulp'),                 //基础库
    imagemin = require('gulp-imagemin'),       //图片压缩
    sass = require('gulp-ruby-sass'),          //sass
    minifycss = require('gulp-minify-css'),    //css压缩
    jshint = require('gulp-jshint'),           //js检查
    uglify  = require('gulp-uglify'),          //js压缩
    rename = require('gulp-rename'),           //重命名
    concat  = require('gulp-concat'),          //合并文件
    clean = require('gulp-clean'),             //清空文件夹
    tinylr = require('tiny-lr'),               //livereload
    server = tinylr(),
    port = 35729,
    livereload = require('gulp-livereload');   //livereload

// HTML处理
gulp.task('html', function() {
    var htmlSrc = './src/*.html',
        htmlDst = './dist/';

    gulp.src(htmlSrc)
        .pipe(livereload(server))
        .pipe(gulp.dest(htmlDst))
});

// 样式处理
gulp.task('css', function () {
    var cssSrc = './src/scss/*.scss',
        cssDst = './dist/css';

    gulp.src(cssSrc)
        .pipe(sass({ style: 'expanded'}))
        .pipe(gulp.dest(cssDst))
        .pipe(rename({ suffix: '.min' }))
        .pipe(minifycss())
        .pipe(livereload(server))
        .pipe(gulp.dest(cssDst));
});

// 图片处理
gulp.task('images', function(){
    var imgSrc = './src/images/**/*',
        imgDst = './dist/images';
    gulp.src(imgSrc)
        .pipe(imagemin())
        .pipe(livereload(server))
        .pipe(gulp.dest(imgDst));
});

// js处理
gulp.task('js', function () {
    var jsSrc = './src/js/*.js',
        jsDst ='./dist/js';

    gulp.src(jsSrc)
        .pipe(jshint('.jshintrc'))
        .pipe(jshint.reporter('default'))
        .pipe(concat('main.js'))
        .pipe(gulp.dest(jsDst))
        .pipe(rename({ suffix: '.min' }))
        .pipe(uglify())
        .pipe(livereload(server))
        .pipe(gulp.dest(jsDst));
});

// 清空图片、样式、js
gulp.task('clean', function() {
    gulp.src(['./dist/css', './dist/js', './dist/images'], {read: false})
        .pipe(clean());
});

// 默认任务 清空图片、样式、js并重建 运行语句 gulp
gulp.task('default', ['clean'], function(){
    gulp.start('html','css','images','js');
});

// 监听任务 运行语句 gulp watch
gulp.task('watch',function(){

    server.listen(port, function(err){
        if (err) {
            return console.log(err);
        }

        // 监听html
        gulp.watch('./src/*.html', function(event){
            gulp.run('html');
        })

        // 监听css
        gulp.watch('./src/scss/*.scss', function(){
            gulp.run('css');
        });

        // 监听images
        gulp.watch('./src/images/**/*', function(){
            gulp.run('images');
        });

        // 监听js
        gulp.watch('./src/js/*.js', function(){
            gulp.run('js');
        });
    });

});
```
### LiveReload配置
1、安装Chrome LiveReload
2、通过npm安装http-server ，快速建立http服务
```
npm install http-server -g
```
3、通过cd找到发布环境目录dist
4、运行http-server，默认端口是8080
5、访问路径localhost:8080
6、再打开一个cmd，通过cd找到项目路径执行gulp，清空发布环境并初始化
7、执行监控 gulp
8、点击chrome上的LiveReload插件，空心变成实心即关联上，你可以修改css、js、html即时会显示到页面中

## 缓存 
在Web应用程序中，系统的瓶颈常在于系统的响应速度。如果系统响应速度过慢，用户就会出现埋怨情绪，系统的价值也因此会大打折扣。因此，提高系统响应速度，是非常重要的。缓存方案有2种，一种布在服务端，用于加快客户端请求的响应速度，比如 MemCache、Redis等；另一种是应用在客户端，目的是减少http请求，降低服务器压力，加快页面渲染速度。
### 服务端缓存 
#### **MemCache** 
memcache是一套分布式的高速缓存系统，由LiveJournal的Brad Fitzpatrick开发，但目前被许多网站使用以提升网站的访问速度，尤其对于一些大型的、需要频繁访问数据库的网站访问速度提升效果十分显著[1]  。这是一套开放源代码软件，以BSD license授权发布。

MemCache的工作流程如下：先检查客户端的请求数据是否在memcached中，如有，直接把请求数据返回，不再对数据库进行任何操作；如果请求的数据不在memcached中，就去查数据库，把从数据库中获取的数据返回给客户端，同时把数据缓存一份到memcached中（memcached客户端不负责，需要程序明确实现）；每次更新数据库的同时更新memcached中的数据，保证一致性；当分配给memcached内存空间用完之后，会使用LRU（Least Recently Used，最近最少使用）策略加上到期失效策略，失效数据首先被替换，然后再替换掉最近未使用的数据。
 
Memcache是一个高性能的分布式的内存对象缓存系统，通过在内存里维护一个统一的巨大的hash表，它能够用来存储各种格式的数据，包括图像、视频、文件以及数据库检索的结果等。简单的说就是将数据调用到内存中，然后从内存中读取，从而大大提高读取速度。

Memcache是danga的一个项目，最早是LiveJournal 服务的，最初为了加速 LiveJournal 访问速度而开发的，后来被很多大型的网站采用。Memcached是以守护程序(监听)方式运行于一个或多个服务器中，随时会接收客户端的连接和操作。

#### **Redis** 
redis是一个key-value存储系统。和Memcached类似，它支持存储的value类型相对更多，包括string(字符串)、list(链表)、set(集合)、zset(sorted set --有序集合)和hash（哈希类型）。这些数据类型都支持push/pop、add/remove及取交集并集和差集及更丰富的操作，而且这些操作都是原子性的。在此基础上，redis支持各种不同方式的排序。与memcached一样，为了保证效率，数据都是缓存在内存中。区别的是redis会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件，并且在此基础上实现了master-slave(主从)同步。

Redis 是一个高性能的key-value数据库。 redis的出现，很大程度补偿了memcached这类key/value存储的不足，在部 分场合可以对关系数据库起到很好的补充作用。它提供了Java，C/C++，C#，PHP，JavaScript，Perl，Object-C，Python，Ruby，Erlang等客户端，使用很方便。

Redis支持主从同步。数据可以从主服务器向任意数量的从服务器上同步，从服务器可以是关联其他从服务器的主服务器。这使得Redis可执行单层树复制。存盘可以有意无意的对数据进行写操作。由于完全实现了发布/订阅机制，使得从数据库在任何地方同步树时，可订阅一个频道并接收主服务器完整的消息发布记录。同步对读取操作的可扩展性和数据冗余很有帮助。

### 客户端缓存 
客户端缓存的方案常用的是 ajax缓存、资源离线存储、Storage缓存等。
#### **Ajax缓存** 
Ajax异步请求数据虽然不影响页面的同步加载，但是过多的ajax请求也会给服务器带来压力，因此我们的策略是缓存不必要或者相同的请求，把请求返回的数据用变量存在本地，然后给事件触发点设置 缓存标志，只要是相同的ajax请求就从本地取数据，或者直接`return`。另外再掺插一个用户体验的优化建议，就是ajax加载时的缓冲动画，动画可以用gif图片替代。

```
/**
* [开启缓冲动画]
* @return {[type]} [void]
*/
showLoadAnimate:function(){
	var self=this;
	_loadAnimate.show();
	_timer=window.setTimeout(function(){
		self.closeLoadAnimate();
		self.myalert('加载失败了')
	},2000);
},
/**
* [关闭缓冲动画]
* @return {[type]} [void]
*/
closeLoadAnimate:function(){
	clearTimeout(_timer);
	_loadAnimate.fadeOut('fast');
}
```

```
/*触发ajax*/
$('#btn').on('click',function(){
	if(!$('#find-shareBody').hasClass('isCahed')){
		loadData();
		$('#find-shareBody').addClass('isCahed');
	}
});
```

```
/*数据加载函数*/
function loadData(){
	_util.showLoadAnimate(); //加载缓冲动画
	$.ajax({
		url: _self.APIURL+'/product/productTypes.json',
		type: 'GET',
		dataType: 'json',
		data: {pId: _tId},
		success:function(res){
			var types=res.types,subTypes=res.subTypes;
			var tempStr="",len1=types.length,len2=subTypes.length;
			/*加载一级分类*/
			for(i=0;i<len1;i++){
				if(i===0){
					tempStr+="<li class='category-item active' data-id="+types[i].id+">"+types[i].typeName+"</li>";	
				}else{
					tempStr+="<li class='category-item' data-id="+types[i].id+">"+types[i].typeName+"</li>";
				}
			}
			_categoryWrap.html(tempStr);
			/*加载二级分类*/
			tempStr="";
			for(i=0;i<len2;i++){
				tempStr+="<li class='sub-category-item' data-id="+subTypes[i].id+"><img alt='"+subTypes[i].typeName+"' src="+subTypes[i].thumb+"_100 /><p>"+subTypes[i].typeName+"</p></li>";
			}
			_subCategoryWrap.html(tempStr);
			_util.closeLoadAnimate(); //关闭缓冲动画
		}
	});
}
```

#### **AppCache** 
如果你的Web应用中有一部分功能（或者整个应用）需要在脱离服务器的情况下使用，那么就可以通过AppCache来让你的用户在离线状态下也能使用。你所需要做的就是创建一个配置文件，在其中指定哪些资源需要被缓存，哪些不需要。此外，还能在其中指定某些联机资源在脱机条件下的替代资源。

AppCache的配置文件通常是一个以.appcache结尾的文本文件（推荐写法）。文件以CACHE MANIFEST开头，包含下列三部分内容：
> 
 - CACHE – 指定了哪些资源在用户第一次访问站点的时候需要被下载并缓存
 - NETWORK – 指定了哪些资源需要在联机条件下才能访问，这些资源从不被缓存
 - FALLBACK – 指定了上述资源在脱机条件下的替代资源

**举个例子** 
首先，你需要在页面上指定AppCache的配置文件：

```
<!DOCTYPE html>
<html manifest="manifest.appcache">
...
</html>
```
在这里千万记得在服务器端发布上述配置文件的时候，需要将MIME类型设置为text/cache-manifest，否则浏览器无法正常解析。

接下来是创建之前定义好的各种资源。我们假定在这个示例中，你开发的是一个交互类站点，用户可以在上面联系别人并且发表评论。用户在离线的状态下依然可以访问网站的静态部分，而联系以及发表评论的页面则会被其它页面替代，无法访问。

好的，我们这就着手定义那些静态资源：

```
CACHE MANIFEST
 
CACHE:
/about.html
/portfolio.html
/portfolio_gallery/image_1.jpg
/portfolio_gallery/image_2.jpg
/info.html
/style.css
/main.js
/jquery.min.js
```
> 注意：配置文件写起来有一点很不方便。举例来说，如果你想缓存整个目录，你不能直接在CACHE部分使用通配符（*），而是只能在NETWORK部分使用通配符把所有不应该被缓存的资源写出来。

你不需要显式地缓存包含配置文件的页面，因为这个页面会自动被缓存。接下来我们为联系和评论的页面定义FALLBACK部分：

```
FALLBACK:
/contact.html /offline.html
/comments.html /offline.html
```
最后我们用一个通配符来阻止其余的资源被缓存：

```
NETWORK:
*
```
最后的结果就是下面这样：

```
CACHE MANIFEST
 
CACHE:
/about.html
/portfolio.html
/portfolio_gallery/image_1.jpg
/portfolio_gallery/image_2.jpg
/info.html
/style.css
/main.js
/jquery.min.js
 
FALLBACK:
/contact.html /offline.html
/comments.html /offline.html
 
NETWORK:
*
```
还有一件很重要的事情要记得：你的资源只会被缓存一次！也就是说，如果资源更新了，它们不会自动更新，除非你修改了配置文件。所以有一个最佳实践是，在配置文件中增加一项版本号，每次更新资源的时候顺带更新版本号：

```
CACHE MANIFEST
 
# version 1
 
CACHE:
...
```

#### **Storage存储** 
如果你想在Javascript代码里面保存些数据，那么这两个东西就派上用场了。前一个可以保存数据，永远不会过期（expire）。只要是相同的域和端口，所有的页面中都能访问到通过LocalStorage保存的数据。举个简单的例子，你可以用它来保存用户设置，用户可以把他的个人喜好保存在当前使用的电脑上，以后打开应用的时候能够直接加载。后者也能保存数据，但是一旦关闭浏览器窗口（译者注：浏览器窗口，window，如果是多tab浏览器，则此处指代tab）就失效了。而且这些数据不能在不同的浏览器窗口之间共享，即使是在不同的窗口中访问同一个Web应用的其它页面。

> 注意：有一点需要提醒的是，LocalStorage和SessionStorage里面只能保存基本类型的数据，也就是字符串和数字类型。其它所有的数据可以通过各自的toString()方法转化后保存。如果你想保存一个对象，则需要使用JSON.stringfy方法。（如果这个对象是一个类，你可以复写它默认的toString()方法，这个方法会自动被调用）。

**举个例子**
我们不妨来看看之前的例子。在联系人和评论的部分，我们可以随时保存用户输入的东西。这样一来，即使用户不小心关闭了浏览器，之前输入的东西也不会丢失。对于jQuery来说，这个功能是小菜一碟。（注意：表单中每个输入字段都有id，在这里我们就用id来指代具体的字段）

```
$('#comments-input, .contact-field').on('keyup', function () {
   
// let's check if localStorage is supported
   if (window.localStorage) {
      localStorage.setItem($(this).attr('id'), $(this).val());
   }
});
```
每次提交联系人和评论的表单，我们需要清空缓存的值，我们可以这样处理提交（submit）事件：

```
$('#comments-form, #contact-form').on('submit', function () {
   
// get all of the fields we saved
   $('#comments-input, .contact-field').each(function () {
      
// get field's id and remove it from local storage
      localStorage.removeItem($(this).attr('id'));
   });
});
```
最后，每次加载页面的时候，把缓存的值填充到表单上即可：

```
// get all of the fields we saved
$('#comments-input, .contact-field').each(function () {
   
// get field's id and get it's value from local storage
   var val = localStorage.getItem($(this).attr('id'));
   
// if the value exists, set it
   if (val) {
      $(this).val(val);
   }
});
```

#### **IndexedDB**
在我个人看来，这是最有意思的一种技术。它可以保存大量经过索引（indexed）的数据在浏览器端。这样一来，就能在客户端保存复杂对象，大文档等等数据。而且用户可以在离线情况下访问它们。这一特性几乎适用于所有类型的Web应用：如果你写的是邮件客户端，你可以缓存用户的邮件，以供稍后再看；如果你写的是相册类应用，你可以离线保存用户的照片；如果你写的是GPS导航，你可以缓存用户的路线……不胜枚举。

IndexedDB是一个面向对象的数据库。这就意味着在IndexedDB中既不存在表的概念，也没有SQL，数据是以键值对的形式保存的。其中的键既可以是字符串和数字等基础类型，也可以是日期和数组等复杂类型。这个数据库本身构建于存储（store，一个store类似于关系型数据中表的概念）的基础上。数据库中每个值都必须要有对应的键。每个键既可以自动生成，也可以在插入值的时候指定，也可以取自于值中的某个字段。如果你决定使用值中的字段，那么只能向其中添加Javascript对象，因为基础数据类型不像Javascript对象那样有自定义属性。

**举个例子**
在这个例子中，我们用一个音乐专辑应用作为示范。不过我并不打算在这里从头到尾展示整个应用，而是把涉及IndexedDB的部分挑出来解释。如果大家对这个Web应用感兴趣的话，文章的后面也提供了源代码的下载。首先，让我们来打开数据库并创建store：

```
// check if the indexedDB is supported
if (!window.indexedDB) {
    throw 'IndexedDB is not supported!'; 
// of course replace that with some user-friendly notification
}
 
// variable which will hold the database connection
var db;
 
// open the database
// first argument is database's name, second is it's version (I will talk about versions in a while)
var request = indexedDB.open('album', 1);
 
request.onerror = function (e) {
    console.log(e);
};
 
// this will fire when the version of the database changes
request.onupgradeneeded = function (e) {
    
// e.target.result holds the connection to database
    db = e.target.result;
 
    
// create a store to hold the data
    
// first argument is the store's name, second is for options
    
// here we specify the field that will serve as the key and also enable the automatic generation of keys with autoIncrement
    var objectStore = db.createObjectStore('cds', { keyPath: 'id', autoIncrement: true });
 
    // create an index to search cds by title
    // first argument is the index's name, second is the field in the value
    
// in the last argument we specify other options, here we only state that the index is unique, because there can be only one album with specific title
    objectStore.createIndex('title', 'title', { unique: true });
 
    
// create an index to search cds by band
    
// this one is not unique, since one band can have several albums
    objectStore.createIndex('band', 'band', { unique: false });
};
```
相信上面的代码还是相当通俗易懂的。估计你也注意到上述代码中打开数据库时会传入一个版本号，还用到了onupgradeneeded事件。当你以较新的版本打开数据库时就会触发这个事件。如果相应版本的数据库尚不存在，则会触发事件，随后我们就会创建所需的store。接下来我们还创建了两个索引，一个用于标题搜索，一个用于乐队搜索。现在让我们再来看看如何增加和删除专辑：

```
// adding
$('#add-album').on('click', function () {
    
// create the transaction
    
// first argument is a list of stores that will be used, second specifies the flag
    
// since we want to add something we need write access, so we use readwrite flag
    var transaction = db.transaction([ 'cds' ], 'readwrite');
    transaction.onerror = function (e) {
        console.log(e);
    };
    var value = { ... }; 
// read from DOM
    
// add the album to the store
    var request = transaction.objectStore('cds').add(value);
    request.onsuccess = function (e) {
        
// add the album to the UI, e.target.result is a key of the item that was added
    };
});
 
// removing
$('.remove-album').on('click', function () {
    var transaction = db.transaction([ 'cds' ], 'readwrite');
    var request = transaction.objectStore('cds').delete(
/* some id got from DOM, converted to integer */
);
    request.onsuccess = function () {
        
// remove the album from UI
    }
});
```
是不是看起来直接明了？这里对数据库所有的操作都基于事务的，只有这样才能保证数据的一致性。现在最后要做的就是展示音乐专辑：

```
request.onsuccess = function (e) {
    if (!db) db = e.target.result;
 
    var transaction = db.transaction([ 'cds' ]); 
// no flag since we are only reading
    var store = transaction.objectStore('cds');
    
// open a cursor, which will get all the items from database
    store.openCursor().onsuccess = function (e) {
        var cursor = e.target.result;
        if (cursor) {
            var value = cursor.value;
            $('#albums-list tbody').append('
'+ value.title +''+ value.band +''+ value.genre +''+ value.year +'
'); // move to the next item in the cursor 
			cursor.continue(); 
		}
	}
}
```
这也不是十分复杂。可以看见，通过使用IndexedDB，可以很轻松的保存复杂对象，也可以通过索引来检索想要的内容：

```
function getAlbumByBand(band) {
    var transaction = db.transaction([ 'cds' ]);
    var store = transaction.objectStore('cds');
    var index = store.index('band');
    
// open a cursor to get only albums with specified band
    
// notice the argument passed to openCursor()
    index.openCursor(IDBKeyRange.only(band)).onsuccess = function (e) {
        var cursor = e.target.result;
        if (cursor) {
            
// render the album
            
// move to the next item in the cursor
            cursor.continue();
        }
    });
}
```
使用索引的时候和使用store一样，也能通过游标（cursor）来遍历。由于同一个索引值名下可能有好几条数据（如果索引不是unique的话），所以这里我们需要用到IDBKeyRange。它能根据指定的函数对结果集进行过滤。这里，我们只想根据指定的乐队进行检索，所以我们用到了only()函数。也能使用其它类似于lowerBound()，upperBound()和bound()等函数，它们的功能也是不言自明的。

## CDN 
CDN的全称是Content Delivery Network，即内容分发网络。其基本思路是尽可能避开互联网上有可能影响数据传输速度和稳定性的瓶颈和环节，使内容传输的更快、更稳定。通过在网络各处放置节点服务器所构成的在现有的互联网基础之上的一层智能虚拟网络，CDN系统能够实时地根据网络流量和各节点的连接、负载状况以及到用户的距离和响应时间等综合信息将用户的请求重新导向离用户最近的服务节点上。其目的是使用户可就近取得所需内容，解决 Internet网络拥挤的状况，提高用户访问网站的响应速度。

提供cdn的服务商很多，我所熟悉的是 [又拍云](https://console.upyun.com/#/login/)，它不仅提供cdn加速服务，还有云处理的能力，比如 图片压缩、打水印、锐化等 ，个人比较推荐使用这个。你可以把 项目发布的静态资源（图片、样式、脚本）托管到又拍云上，图片还可以添加云处理如打水印等，相当方便。

## 懒加载 
懒加载是一种资源加载方案，符合人脑认识事物的规则，用户滚动页面的时候自动获取更多的数据,而新得到的数据不会影响原有数据的显示,同时最大程度上减少服务器端的资源耗用。Web应用程序做的最多就是和后台数据库交互，而查询数据库是种非常耗时的过程。当数据库里记录过多时，查询优化更显得尤为重要。为了解决这种问题，有人提出了缓存的概念。缓存就是将用户频繁使用的数据放在内存中以便快速访问。在用户执行一次查询操作后，查询的记录会放在缓存中。当用户再次查询时，系统会首先从缓存中读取，如果缓存中没有，再查询数据库。缓存技术在一定程度上提升了系统性能，但是当数据量过大时，缓存就不太合适了。因为内存容量有限，把过多的数据放在内存中，会影响电脑性能。而另一种技术，懒加载可以解决这种问题。

### 图片懒加载
图片懒加载的实现思路很简单，一般是针对首页面内容比较长的页面，根据鼠标的滚动逐渐显示当屏的所有图片。你可能觉得这和 "瀑布流" 有点相似，但其实两者有本质差异。瀑布流是为了替代短分页，每次执行瀑布流会执行ajax操作。但是懒加载不一样，大家应该知道影响页面加载速度罪魁祸首就是图片，加载图片需要从服务端把图片download到浏览器然后渲染出来，这个过程非常耗时。但是 从服务端 下载图片的 url然后存储在DOM节点里是相当快速的。

上面说的比较拗口，还是来个例子演示一下。先[下载 jQuery的 lazyload插件](http://baaistatic.b0.upaiyun.com/Static/HK/lazyload.jquery.min.js)，我们把图片真正的url存在img标签的`data-original` 里，`src`属性 可以放一个 加载动画的gif图片的url，这样可以优化用户体验。  

```
<div class="new-img-box">
	<img data-original="{{item.picture}}" class="lazyload" src="http://baaistatic.b0.upaiyun.com/Static/HK/load.gif_400" alt="" />
</div>
```

```
/*执行图片加载缓冲*/
$('.lazyload').lazyload({effect : "fadeIn"});
```
这样，随着鼠标滚动栏滚动，进入当屏焦点的图片就会被 `渐变加载`。

# 用户体验 
用户体验优化，又叫交互优化，目的是让webApp更好用。主要是通过替换方案改变 用户感知到的响应速度，结合移动设备特点提供更有针对性、更好用的体验。
## 点击 
这个问题在我之前的文章里有详细介绍，感兴趣的同学可以 [戳这里](http://blog.csdn.net/u011413061/article/details/50071047) 。当然这里不做长篇大论复述，只说下原理和方案。

### 300ms延迟存在的理由
就目前而言，还有很多的站点没有做移动端分辨率兼容，因此部分区域需要放大、缩小来浏览。而移动端常用放大缩小的方案就是通过 "双击" ，然后在指定区域放大或缩小显示。那么问题来了，怎样区分用户是单击还是双击呢？PC端我们是通过鼠标连续两次点击的时间差来确定这个事件的触发，同样的道理，移动端我们也需要用户手指连续两次点击的这个时间差，也就是300ms延迟。

### tap事件的原理
解决300ms延迟，最简单的办法是引入移动端的框架如zepto，用它提供的tap事件替代click事件。当然我并不强制要求你用zepto框架来做webApp开发，这里我重点是想说 tap事件的原理。

首先我们要明确，tap事件不是移动端浏览器原生支持的事件，而是通过 touchstart －> touchmove －> touchend 这三个事件模拟出来的。基本条件有2个：
> 1）从接触到离开时间间隔短
    2）从起点到终点的距离小
 
只要能监听3个事件满足上面2个条件，你可以自己模拟出一个 tap 事件。比如我之前说的 [fastClick](%E6%B7%B1%E5%85%A5%E8%A7%A3%E6%9E%90FastClick%E8%A7%A3%E5%86%B3%E5%BB%B6%E8%BF%9F%E7%82%B9%E5%87%BB)。除此之外，如果你使用的是其他框架，有一个一劳永逸的方法就是拓展  `on` 方法，比如针对 jQuery：

```
    /**
	 * [针对移动端的click事件优化]
	 * @type {Boolean}
	 */
	var isTouch = ('ontouchstart' in document.documentElement) ? 'touchstart' : 'click', _on = $.fn.on;
	if(!$.fn.quickOn){
        $.fn.quickOn= function(){
            arguments[0] = (arguments[0] === 'click') ? isTouch: arguments[0];
            return $.fn.on.apply(this, arguments);    
        };
    }

	/*注意：iOS设备quickOn会有默认事件，因此用quickOn绑定的click事件需要阻止鼠标的默认事件*/
	$('#btn').quickOn('click',function(e){
		e.preventDefault();
		//....
	});
```

### 点击态
什么是点击态？就是 **给用户明确的点击反馈，提升用户体验**。用两种常见的方案：

1）使用`:active`伪类 提供一个点击态样式。这个是通过CSS这门语言本身的特性来解决的问题，最容易让人接受，但是它有一个致命的缺点——页面滚动的时候也会触发这个伪类的样式。

2）通过添加一个class类如 `.active` ，tap触发的时候添加这个样式，tap结束后通过 Javascript的`setTimeout` 延迟150ms后移除这个样式。比如：

```
$('#btn').on('tap',function(e){
	var _target=$(e.target);
	_target.addClass('active');

	setTimeout(function(){
		_target.removeClass('active');
	},150);
});
```

## 滚动
 **全局滚动**，滚动条在body节点或更顶层。
**局部滚动**，滚动条在body下的某一个DOM节点上。

### 弹性滚动

#### 针对IOS设备
IOS设备 **全局滚动** 默认支持 弹性滚动，局部滚动默认不支持 弹性滚动，所以需要通过css设置。

```
body{
	-webkit-overflow-scrolling:touch;
}

/*局部滚动的dom节点*/
.scroll-el{
	overflow:auto;
}
```
这个属性建议挂在body上，这样子节点可以继承这个属性，可以避免不必要的bug。

#### 针对Andriod设备
定制版本较多，表现各异，虽然默认没有弹性滚动的效果，但很多版本支持 力度滚动，而且也没有IOS上默认生硬的滚动效果效果。 `-webkit-overflow-scrolling` 默认浏览器是不支持的，所以不会起效果。

### IOS出界困扰 
什么时候会触发出界呢？
**全局滚动：**滚动到页面顶部（或底部）时继续向下（向上）滚动，就会出现。
**局部滚动：**滚动到页面顶部（或底部）时，手指离开停下，再继续向下（向上）滑动，就会出现。

#### 解决方案
**局部滚动：**使用 scrollFix 
```
if(startTopScroll<=0){
	elem.scrollTop=1;
}
if(startTopScroll+elem.offsetHeight>=elem.scrollHeight){
	elem.scrollTop=elem.scrollHeight-elem.offsetHeight-1;
}
```
除此之外，对于局部滚动页面的固定区域要禁止touchmove默认事件。

**全局滚动：** 暂时没有找到好的解决方法，可以考虑把全局滚动改成局部滚动。不过从iOS8开始Safari的出界部分的背景色和body的背景色保持一致。

### Android 蛋疼的局部滚动
andriod下使用局部滚动，会导致滚动条显示异常，且滚动不流畅。因此建议只使用全局滚动，固定元素可以通过定位在顶部或底部，然后给body添加内上边距或内下边距 模拟局部滚动的效果。

### 流畅滚动的N条军规
1）body上加上 `-webkit-overflow-scrolling:touch` 
2）iOS尽量使用局部滚动
3）iOS引进scrollFix避免出界
4）andriod下尽量使用全局滚动：尽量不用`overflow:auto` ,使用 `min-height:100%`代替 `height:100%`。
5）iOS下带有滚动条且`position:absolute`的节点不要设置背景色。

## 键盘定制
### 定制软键盘样式
可以通过设置`input`的 `type` 属性：

```
<form>
	<input type="email" />
	<input type="url" /> 
	<input type="tel" />
	<input type="number" />
	<input type="search" />
</form>
```
这里要注意一点，需要给form设置阻止默认onsubmit事件。

另外还有个纯数字键盘的方案，注意pattern的属性值只能是 `[0-9]*` 
```
<form>
	<input type="text" pattern="[0-9]*" />
</form>
```

### 定制软键盘行为
可以通过配置`input`节点的 `autocapitalize`、`autocorrect` 属性。

1）输入英文用户名首字母自动大写的别扭。
```
<input type="text" autocapitalize="off" />
```
2）自动纠错
```
<input type="text" autocorrect="on" />
```



