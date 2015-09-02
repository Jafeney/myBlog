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








