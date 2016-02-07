title: ArtTemplate helper函数的使用 
date: 2016-01-11 17:28:01 
tags: [NodeJS,"模板引擎"] 
categories: 大前端 
---



# 写在前面
”Art虐我千百遍，而我待她如初恋“，前端模板引擎很多，机缘巧合之下结识了这位美丽的Art，于是对她情有独钟，纵使开源社区薄弱，API文档稀缺，还是坚韧不拔地去 使用她。和她相处的几个月里 遇到的坑，趟过的枪实在是不计其数，但是没关系，既然选择了她，那就要一心一意，去克服困难，一起成长不是吗？

# nodeJS中使用
最初结识它是在 WebApp开发的时候，那时还不太会nodeJS，Art也仅仅是用在客户端 数据的渲染上，用的最多的是 `{{each}}`、`{{if}}` 、`{{include}}` 这些东西。在客户端JavaScript中 模块化有诸多限制，必须依托`AMD`或者`CMD`的方式来实现，而且客户端使用模板引擎对SEO的支持着实无助。当然这也算是自己趟过的一个坑吧，其实 Art真正实现价值还是在3.0之后 对nodeJS的支持，这样打破了传统的`express+jade` 组合模式，开启了另一个高性能的前后端可共用的模板引擎时代。

<!--more--> 

## 安装
使用npm之前先确保已经安装了nodeJS，使用 `node -v` 测试一下，然后：
```
npm install art-template
```

## 使用
```
var template = require('art-template');
var data = {list: ["aui", "test"]};
var html = template(__dirname + '/index/main', data);
```

## 配置
NodeJS 版本新增了如下默认配置：
| 字段 | 类型 | 默认值 | 说明 |
| ---:--- |---:---|---:---| ---:---|
| base | String | `''` | 指定模板目录 |
| extname| String | `'.html'` | 指定模板后缀名 |
| encoding | String | `'utf-8'` | 指定模板编码 |

注意：配置base指定模板目录可以缩短模板的路径，并且能够避免include语句越级访问任意路径引发安全隐患，例如：

```
template.config('base', __dirname);
var html = template('index/main', data)
```


## NodeJS + Express
这个在之前的文章里已经详细介绍了，配置方法大致长这样。
```
var template = require('art-template');
template.config('base', '');
template.config('extname', '.html');
app.engine('.html', template.__express);
app.set('view engine', 'html');
//app.set('views', __dirname + '/views');
```
> 还不清楚的 用力戳 [这里](http://jafeney.com/2016/01/10/express%E6%90%AD%E5%BB%BAnodeJS%E4%B8%AD%E9%97%B4%E5%B1%82%EF%BC%88%E4%B8%80%EF%BC%89/)

## helper函数
helper函数的API很简单，但它的用处实在不小。也正是因为它的存在使得 Art变得非常地灵活——这也完全取决于你的自己。
```
template.helper(name, callback)
```
先看一下官方 [yanis.wang](https://github.com/yaniswang) 写的例子，他老人家故意整个正则和参数形式的回调 来提高逼格，小子我来回看了好几遍才搞懂（加了些注释进去）  ~_~

所以看之前，对 replace函数不熟悉的请 [戳这里](http://www.w3school.com.cn/jsref/jsref_replace.asp)

### html部分
```
<!--数据容器-->
<div id="content"></div>
<!--/数据容器-->

<script id="test" type="text/html">
	/*对time对象进行格式化*/
	{{time | dateFormat:'yyyy年 MM月 dd日 hh:mm:ss'}}
</script>

```
### JavaScript部分
```
/** 
 * 对日期进行格式化， 
 * @param date 要格式化的日期 
 * @param format 进行格式化的模式字符串
 *     支持的模式字母有： 
 *     y:年, 
 *     M:年中的月份(1-12), 
 *     d:月份中的天(1-31), 
 *     h:小时(0-23), 
 *     m:分(0-59), 
 *     s:秒(0-59), 
 *     S:毫秒(0-999),
 *     q:季度(1-4)
 * @return String
 * @author yanis.wang
 * @see	http://yaniswang.com/frontend/2013/02/16/dateformat-performance/
 */
template.helper('dateFormat', function (date, format) {

    date = new Date(date); //新建日期对象
    
    /*日期字典*/
    var map = {
        "M": date.getMonth() + 1, //月份 
        "d": date.getDate(), //日 
        "h": date.getHours(), //小时 
        "m": date.getMinutes(), //分 
        "s": date.getSeconds(), //秒 
        "q": Math.floor((date.getMonth() + 3) / 3), //季度 
        "S": date.getMilliseconds() //毫秒 
    };
    
	/*正则替换*/
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
    
    /*返回自身*/
    return format;
});

/*数据*/
var data = {
	time: (new Date).toString(),
};
/*渲染*/
var html = template('test', data);
/*绑定*/
document.getElementById('content').innerHTML = html;

```

> 当然了，看不懂这个例子也没关系，我扔块 “低小下”的砖头给你。

### 一块丑丑的砖头
#### app.js
```
/*引用artTemplate模板*/
var template=require('art-template');

template.render('transNumber',function(number){
	swich(number){
		case 0:
			return "zero";
			break;
		case 1:
			return "none";
			break;
		//...more and more
		default:
			break;
	}
});
```

### test.art
```
<div>{{number | transNumber:number}}</div>
```

没错，这么搞就可以了，回调函数 按照实际需求爱怎么写就怎么写。


#最后
欢迎关注我的个人博客： http://jafeney.com  ^_^

