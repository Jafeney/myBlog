##使用最新的jQuery类库##
##使用合适的选择器##
> (1) 使用id来定位DOM元素是最高效的方式，如果不能直接用id找到你需要的元素，可以考虑用find()方法。
> (2) 标签选择器的性能也是不错的，它是性能优化的第二选择，因为jQuery将直接调用本地方法document.getElementByTagName()来定位DOM元素。
> (3) 对于类选择器，现代浏览器和IE9+ 支持本地方法document.getElementByClassName(),而对于老的浏览器如IE8 以及以前的版本，只能靠使用DOM 搜索方式来实现，这无疑对性能产生较大的影响，所以建议大家有选择性的使用它。
> (4) 属性选择器无法直接实现，大多都是使用DOM搜索方式来达到效果，很多现代浏览器支持querySelectorAll()方法，但是不同的浏览器的性能还是不一样的，总的来说，使用这种方式性能并不是非常理想。所以尽量避免使用这种有害性能的方式。
> (5) 伪类选择器也同样无法直接实现，jQuery需要搜索每一个元素来定位这个元素，这将是对性能造成较大的消耗，尽量不要使用，如果非用不可，就先用ID 选择器定位父元素，然后再使用该选择器。
> 注意：尽量使用ID选择器，尽量给选择器指定上下文。
##缓存对象##
> (1) 推荐使用jQuery的链式方式，这样可以最大程度上发挥缓存变量的优点，有效提高代码运行性能。
> (2) 如果一个jQuery对象被多次使用，那么可以把它们缓存到全局环境中。记住，永远不要让相同的选择器在大马里出现多次。
##循环时的DOM操作##
> 使用jQuery可以很方便地添加、删除或者修改DOM节点，但是在一些循环，例如for()、while()或者$.each() 中处理节点时得值得注意，因为这样很消耗性能，最好的方式是 先把元素字符串全部创建好 再插入到DOM中去。 
##数组方式使用jQuery对象##
> (1) 使用jQuery选择器获取结果是一个jQuery对象。然而jQuery类库回让你感觉你正在使用一个定义了索引和长度的数组。再性能方面，建议使用简单for和while循环来处理，而不是$.each()，这样能使你的代码更快。

``` JavaScript
   $.each(array,function(i){
       array[i]=i;
   });
   //替换为：
   var array=new Array();
   for(var i=0;i<array.length;i++){
       array[i]=i;
   }
```
> (2) 检查长度也是一个检查jQuery对象是否存在的方式，下面一段代码通过length属性检查页面中是否存在ID为“content”的元素：
``` JavaScript
    var $content=$('#content');
    if($content){
        //总是true
        //Do something
    }
    if($content.length){
        //拥有元素才返回true
        //Do something
    }
```
##事件代理##
> 每一个javascript事件都会冒泡到父级节点。当我们需要给多个元素调用同个函数时这点会很有用。比如，我们要为一个表格绑定这样的行为：点击td后，把背景设置为红色，代码如下：
``` JavaScript
    $('#myTable td').click(function(){
        $(this).css('background','red');
    });
```
> 假定有100个td元素，在使用以上方式的时候，你绑定了100个事件，这将带来很负面的性能影响。更好的方式是：向它们的父级节点绑定一次事件，然后通过 event.target 获取到点击的当前元素，代码如下：
``` JavaScript
    $('#myTable').click(function(e){
        var $clicked=$(e.target);   // e.target 捕捉到触发的目标元素
        $clicked.css('background','red');
    });
```
> 在改进方式中，你只为一个元素绑定了一个事件。显然，这种方式的属性有优于之前那种，同时在1.7版本中提供了一个新的方式 on()，来帮助你将整个事件监听封装到一个便利方法中，如：
``` JavaScript
    $('#myTable').on('click','td',function(){
        $(this).css('background','red');
    });
```
##将你的代码转换成jQuery插件##
> 如果你的代码需要反复重用，有一种很优雅的写法就是转变成jQuery插件，格式如下：
``` JavaScript
    (function($){
        $.fn.yourPluginName=function(){
            // your code goes here 
            return this;
        }
    })(jQuery);
```
##使用join()来拼接字符串##
> 也许你之前一直使用“＋”来拼接长字符串，现在你可以改改了。虽然它可能会是有点奇怪，但它确实有助于优化性能，尤其是长字符串处理的时候。
> 首先创建一个数组，然后循环，最后使用join()把数组转化为字符串，代码如下：
``` JavaScript
    var array=[];
    for(var i=0;i<=10000;i++){
        array[i]='<li>'+i+'</li>';
    }
    $('#list').html(array.join(''));
```
##合理使用HTML5的Data属性##
> data()方法可以获取 data- 格式的 属性值。规则如下：
``` HTML
    <div id='d1' data-role='page' data-last-value='43' data-options='{"name":"John"}'></div>
```
``` JavaScript
    $('#d1').data('role');  //"page"
    $('#d1').data('lastvalue'); //43
    $('#d1').data('options').name;  //"John"
```
##尽量使用原生的JavaScript方法##
> 下面一段代码，它用来判断多选框是否被选中：
``` JavaScript
    var $cr=$('#cr');   //jQuery对象
    $cr.on('click',function{
        if($cr.is(":checked")){
            //jQuery方式判断
            alert("感谢你的支持，你可以继续操作");
        }
    });
```
> 以上方式可以直接用JavaScript方法来实现：
``` JavaScript
    var $cr=$('#cr');   //jQuery对象
    var cr=$cr.get(0);  //DOM对象，获取$cr[0]
    $cr.on('click',function(){
        if(cr.checked){
            //原生的JavaScript方式判断
            alert("感谢你的支持，你可以继续操作");
        }
    });
```
> 毋庸置疑，第二种方式效率高于第一种方式，因为它不需要拐弯抹角的去调用许多函数。还有更多类似的操作，把如下代码：
``` JavaScript
    $(this).css('color','red');
    //优化成：
    this.style.color="red";
    $('<p></p>');
    //优化成：
    $(document.createElement('p'));
```
##压缩JavaScript##















