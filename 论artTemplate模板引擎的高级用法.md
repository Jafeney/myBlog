<p style="text-indent: 1.6em">天下武功，唯快不破”。对的，artTemplate模板引擎最大的特点就是一个“快”字，但是这套模板引擎官方github上的文档相当地精简，虽然有完整的demo，但是说真的，也不知道是不是我智商不够用，研究了一下午才把这套模板引擎应用到了我的web项目里。
</p>
###首先清楚artTemplate的几个表达式###
```html
    <div id="art-container"></div>
    <script id="art-main" type="text/html">
        <h3>大家好，我的名字叫做{{name}}</h3>
    </script>
    <script id="art-sub" type="text/html">
        <ul>
            {{each like value index}}
                <li> 我的第{{index＋1}}个爱好：{{value}} </li>
            {{/each}}
        </ul>
    </script>
```

```JavaScript
    var data={
        name:'Jafeney',
        like:['pingpeng','lol','programming','movie']
    }
    var art=template('art-main',data);
    document.getElementById('art-container').innerHtml=art;
```

-- 基本表达式
```JavaScript
    {{varible}}
```
-- 条件表达式
```JavaScript
    {{if true}}
        //do something
        {{else if true}}
            //do something
        {{else}}
            //do something
    {{/if}}
```
-- 遍历表达式
```JavaScript
    {{each like value index}}
        <li> 我的第{{index＋1}}个爱好：{{value}} </li>
    {{/each}}
    //也可以这么写
    {{each like}}
        <li> 我的第{{$index＋1}}个爱好：{{$value}} </li>
    {{/each}}
```
-- 包含表达式
```JavaScript
    {{include 'art-sub'}}
```