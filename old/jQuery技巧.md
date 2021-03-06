##禁用页面的右击菜单##
``` JavaScript
    $(function(){
        $(document).on('contextmenu',function(e){
           return false; 
        });
    });      
```
##新窗口打开页面##
``` JavaScript
    $(function(){
        //例子1:href="http://"的超链接将会在新窗口打开链接
        $('a[href^="http://"]').attr('target','_blank');

        //例子2:rel="external"的超链接将会在新窗口打开链接
        $('#a[rel$="external"]').on('click',function(){
            this.target="_blank";
        });
    });
```
##判断浏览器类型##
``` JavaScript
    $(function(){
        //Firefox 2 and above
        if($.support.mozilla && $.support.version>='1.8'){
            //do something
        }
        //Safari
        if($.support.safari){
            //do something
        }
        //Chrome
        if($.support.chrome){
            //do something
        }
        //Opera
        if($.support.opera){
            //do something
        }
        //IE6 and below
        if($.support.msie && $.support.version<=6){
            //do something
        }
        //anything above IE6
        if($.support.msie && $.support.version>6){
            //do something
        }
    });
```
##输入框文字获取和失去焦点##
``` JavaScript
    $(function(){
        $('input.text1').val("Enter your search text here");
        textFill($('input.text1'));
    });
    //input focus text function
    function textFill(input){
        var originalvalue=input.val();
        input.focus(function(){
            if($.trim(input.val())===originalvalue){
                input.val();
            }
        }).blur(function(){
            if($.trim(input.val()==='')){
                input.val(originalvalue);
            }
        });
    }
```
##返回头部滑动动画##
```JavaScript
    jQuery.fn.scrollTo=function(speed){
        var targetOffset=$(this).offset().top;
        $('html.body').stop().animate({scrollTop:targetOffset},speed);
        return this;
    }
    // use
    $('#goheader').click(function(){
        $('body').scrollTo(500);
        return false;
    });
```
##获取鼠标位置##
```JavaScript
    $(document).ready(function(){
        $(document).mousemove(function(e){
            $('#XY').html("X:"+e.pageX+" | Y:"+e.pageY);
        });
    });
```
##判断元素是否存在##
```JavaScript
    $(document).ready(function(){
        if($('#id').length){
            //do something
        }
    });
```
##点击div也可以跳转##
```JavaScript
    $('div').click(function(){
        window.location=$(this).find('a').attr('href');
        return false;
    });
    // use
    <div><a href="#">home</a></div>
```
##设置div在屏幕中央##
```JavaScript
    $(document).ready(function(){
        jQuery.fn.center=function(){
            this.css({
                'position':’absolute‘,
                'top':($(window).height()-this.height())/2+$(window).scrollTop()+"px",
                'left':($(window).width()-this.width())/2+$(window).scrollLeft()+"px"
            });
        }
        //use
        $('#XY').center();
    });
```
##根据浏览器的大小添加不同的样式##
```JavaScript
    $(function(){
        function checkWindowSize(){
            if($(window).width()>1200){
                $('body').addClass('large');
            }else{
                $('body').removeClass('large');
            }
        }
        $(window).resize(checkWindowSize);
    });
```
##创建自己的选择器##
```JavaScript
    $(function(){
        $.extend($.expr[':'],{
            moreThen500px:function(a){
                return $(a).width()>500
            }
        });
        $('.box:moreThen500px').click(function(){
            //...
        });
    });
```
##关闭所有的动画效果##
```JavaScript
    $(function(){
        jQuery.fn.off=true;
    });
```
##检测鼠标的右键还是左键##
```JavaScript
    $(function(){
        $('#XY').mousedown(function(e){
            alert(e.which) //1:左键 2:中键 3:右键
        });
    });
```
##回车提交表单##
```JavaScript
    $(function(){
        $('input').keyup(function(e){
            if(e.which==='13'){
                alert('回车提交！');
            }
        });
    });
```
##设置全局Ajax参数##
```JavaScript
    $(function(){
        $('#load').ajaxStart(function(){
            showLoading();  //显示loading
            disableButtons();   //禁用按钮
        });
        $('#load').ajaxComplete(function(){
            hideLoading();  //隐藏 loading
            enableButtons();    //启用按钮
        });
    });
```
##获取选中的下拉框##
```JavaScript
    $('#someElement').find('option:selected');
    $('#someElement option:selected');
```
##切换复选框##
```JavaScript
    var tog=false;
    $('button').click(function(){
        $('input[type=checkbox]').attr('checked',!tog);
        tog=!tog;
    });
```
##使用siblings()来选择同辈元素##
```JavaScript
    //不这样做
    $('#nav li').click(function(){
        $('#nav li').removeClass('active');
        $(this).addClass('active');
    });
    //替代的做法
    $('#nav li').click(function(){
        $(this).addClass('active').siblings().removeClass('active');
    });
```
##在一段时间之后自动隐藏或者关闭元素##
```JavaScript
    $('div').slideUp(300).delay(3000).fadeIn(400);
```
##使用Firefox和Firebug来记录事件日志##
```JavaScript
    jQuery.log=jQuery.fn.log=function(msg){
        if(console){
            console.log('%s:%o',msg,this);
        }
        return this;
    }
```
##为任何与选择器相匹配的元素绑定事件##
```JavaScript
    $('table').on('click','td',function(){
        $(this).toggleClass('hover');
    });
```
##使用CSS钩子##
```JavaScript
    $.cssHooks['borderRadius']={
        get:function(elem,computed,extra){
            //depending on browser,read the value of -moz-,-webkit-,-o-...
        }
        set:function(elem,value){
            //set the appropriate css3 property
        }
    };
    //use
    $('#reset').css('borderRadius',5);
```
##$.proxy()的使用##
使用回调方法的缺点之一是当执行类库中的方法后，上下文对象this被设置到另外一个元素，而使用$.proxy()可以解决这个问题，代码如下：
```JavaScript
    $('#panel').fadeIn(function(){
        //Using $.proxy:
        $('#panel button').click($.proxy(function(){
            //this 指向 #panel
            $(this).fadeOut();
        },this));
    });
```
##限制Text-Area域中的字符个数##
```JavaScript
    jQuery.fn.maxLength=function(max){
        this.each(function(){
            var type=this.tagName.toLowerCase();
            var inputType=this.type?this.type.toLowerCase():null;
            if(type==="input" && inputType==="text" || inputType==="password"){
                //应用标准的maxLength
                this.maxLength=max;e
            }else if(type==="textarea"){
                this.onkeypress=function(e){
                    var ob=e||event;
                    var keyCode=ob.keyCode;
                    var hasSlection=document.selection?document.selection.createRange().text.length>0:this.selectionStart!==this.selectionEnd;
                    return !(this.value.length>==max && (keyCode>50 || keyCode===32 || keyCode===0 || keyCode===13) && !ob.ctrlKey && !ob.altKey && !hasSlection);
                };
                this.onkeyup=function(){
                    if(this.value.length>max){
                        this.value=this.value.substring(0,max);
                    }
                };
            }
        });
    };
    //use
    $('#mytextarea').maxLength(10);
```
##本地存储##
```JavaScript
    localStorage.someData="This is going to be saved";
```
##解析json数据时报parseError错误##
```JavaScript
    //1.4之前版本，key没引号，这样没问题
    {
        key:"123",
        status:"0"
    }
    //升级成jQuery1.4之后，都必须加上双引号，格式如下：
    {
        "key":"123",
        "status":"0"
    }
```
##从元素中除去HTML##
```JavaScript
    (function($){
        $.fn.stripHtml=function(){
            var regExp=/<("[^"]*"|'[^']*'|[^'">])*>/gi;
            this.each(function(){
                $(this).html($(this).html().replace(regExp,''));
            });
            return $(this);
        }
    })(jQuery);
    //use
    $('div').stripHtml();
```
##拓展String对象的方法##
```JavaScript
    $.extend(String.prototype,{
        isPositiveInteger:function(){
            return (new RegExp(/^[1-9]\d/).test(this));
        },
        isInteger:function(){
            return (new RegExp(/^\d+$/).test(this));
        },
        isNumber:function(value,element){
            return (new RegExp(/^-?(?:\d+|\d{1,3}(?:,\d{3})+)(?:\.\d+)?$/).test(this));
        },
        trim:function(){
            return this.replace(/(^\s*)|(\s*$)|\r|\n/g,"");
        },
        trans:function(){
            return this.replace(/&lt;/g,'<').replace(/&gt;/g,'>').replace(/&quot;/g,'"');
        },
        replaceAll:function(os,ns){
            return this.replace(new RegExp(os,"gm"),ns);
        },
        skipChar:function(ch){
            if(!this || this.length===0){return '';}
            if(this.charAt(0)===ch){return this.substring(1).skipChar(ch);}
            return this;
        },
        isValidPwd:function(){
            return (new RegExp(/^([_]|[a-zA-Z0-9]){6,32}$/).test(this));
        },
        isValidMail:function(){
            return (new RegExp(/^\w+((-\w+)|(\.\w+))*\@[A-Za-z0-9]+((\.|-)[A-Za-z0-9]+)*\.[A-Za-z0-9]+$/).test(this.trim()));
        },
        isSpaces:function(){
            for(var i=0,length=this.length;i<length;i++){
                var ch=this.charAt(i);
                if(ch!=' ' && ch!="\n" && ch!="\t" && ch!="\r"){
                    return false;
                }
                return true;
            }
        },
        isPhone:function(){
            return (new RegExp(/(^([0-9]{3,4}[-])?\d{3,8}(-\d{1,6})?$)|(^\([0-9]{3,4}\)\d{3,8}(\(\d{1,6}\))?$)|(^\d{3,8}$)/).test(this));
        },
        isUrl:function(){
            return (new RegExp(/^[a-zA-Z]+:\/\/([a-zA-Z0-9\-\.]+)(-\w .\/?%&=:]*)$/).test(this));
        },
        isExternalUrl:function(){
            return this.isUrl() && this.indexOf("://"+document.domain)===-1;
        }
    });
    //use
    $('input').val().isInteger();
```








