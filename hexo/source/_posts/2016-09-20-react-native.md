---
title: 基于Redux的ReactNative项目开发总结（二）
date: 2016-09-20 10:45:39
tags: [react, 'react-native']
categories: 移动开发
---

# 写在前面
> 自从上次写了此系列的一篇文章，公司App项目不幸被搁浅，我也转战到`React`组件库和`Node`后端项目的开发，本来的有些断章取义的想法没有很好地去整合，也就不丢出来献丑了。好在峰回路转，新的`ReactNative`项目袭来，重拾之前的架构，经过1个月的开发和思考，有丢弃也有创新。

# 版本更新带来的BUG修复
faceBook对`ReactNative`版本的更新速度实在是太快，还记得我开发第一款App的时候，用的还是`0.24`，现在已经更新到`0.33`。甚至在这个项目`init`的时候还是`0.31`，短短一个月，项目丢到别的机子上跑的时候已经更新到了`0.33`。
真是"三天不读书，赶不上刘少奇"，伴随着`ReactNative`版本快速的演进，的确是修复了很多的原先的不足（比如对原生组件功能的完善和部分跨平台BUG修复），但也带来了一些你不知道的问题和BUG。下面我先罗列几个我遇到的BUG和解决办法。

<!--more-->

## 让`Image`组件加载`http`的网络图片
这个问题出现在IOS里，xcode7以上的版本默认不支持`http`路径的网络图片，如果硬要使用，必须配置项目的`info`参数：

在`NSAppTransportSecurity`里添加属性`NSAllowsArbitraryLoads`为`true`，然后把`NSExceptionDomains` 下面的 `NSTemporaryExceptionAllowsInsecureHTTPLoads`也设为`true`

具体配置如下：

```XML
	<key>NSAppTransportSecurity</key>
	<dict>
		<key>NSAllowsArbitraryLoads</key>
		<true/>
		<key>NSExceptionDomains</key>
		<dict>
			<key>localhost</key>
			<dict>
				<key>NSTemporaryExceptionAllowsInsecureHTTPLoads</key>
				<true/>
			</dict>
		</dict>
	</dict>
```

或者通过Xcode修改：

![这里写图片描述](http://img.blog.csdn.net/20160919113407953)

## 真机调试IP无法修改问题解决
这个问题是`0.29`以后带来的，本来如果需要真机调试（以IOS为例），我们需要手动修改`AppDelegate.m`和`RCTWebSocketExecutor.m`这两个文件里的`localhost`为我们需要的IP地址。

可能是意识到这样繁琐操作的带来的麻烦，`0.29`以后`faceBook`为我们节省了这一步，自动去获取项目运行的IP地址，而不需要我们手动去修改。OK，他们确实做到了，但是不尽人意的是带来了新的问题：

电脑换一个网络，项目运行后无法开启`debug`模式，仔细察看发现运行的居然是`pre-bundled`版本，就是预构建的版本，而不是即时运行的版本。甚至更坑的是居然还会时不时地连不上本地的Node进程。

那怎么**手动设定IP地址**呢？查阅资料获悉：

React Native iOS在0.29.0版本中BundleURL加载方法做了重大改变，新增了RCTBundleURLProvider单例类专门处理BundleURL，使用NSUserDefaults保存配置信息。

> 默认加载方式
在Debug模式下，执行react-native-xcode.sh编译脚本会自动获取当前网卡en0的IP地址，并打入App包中一个配置文件ip.txt，App运行时会读取ip文件，自动生成Developer Server URL，通过这种加载方式，我们不再需要手动去把”localhost”改成Mac的IP了，每次编译都会读取当前最新的IP。

非`Debug`模式时，没有ip.txt文件，会直接读取本地jsbundle文件，和以前版本的Load from `pre-bundled file on disk`方式相同。
但是我经过测试发现，en0是Wifi的网络，如果关闭Wifi，使用网线端口连接网络，en0默认就是inactive，没有对应的IP。

> 手动设置IP
`RCTBundleURLProvider`在接口中暴露了`jsLocation`属性，可以通过`setJsLocation`手动设置IP。

```ObjectC
NSURL *jsCodeLocation;

[[RCTBundleURLProvider sharedSettings] setDefaults];
#if DEBUG
[[RCTBundleURLProvider sharedSettings] setJsLocation:@"192.168.1.101"];
#endif
jsCodeLocation = [[RCTBundleURLProvider sharedSettings] jsBundleURLForBundleRoot:@"index.ios" fallbackResource:nil];
```

另需要在Info设置`NSAppTransportSecurity`的`NSAllowsArbitraryLoads`为`true`即可（这一步同上图）。

## XCode8项目运行报错解决
前天升到Xcode8，莫名其妙地发现项目运行居然报错了：

```JavaScript
Code signing is required for product type 'Unit Test Bundle' in SDK 'iOS 10.0'
```

解决方法如下：

把`Code Singing Identity`设置为 `IOS Developer`

![这里写图片描述](http://img.blog.csdn.net/20160919142105716)

# 一些心得
这个部分主要是一些项目的架构设计中觉得比较合理的地方。

## Style运用技巧
`ReactNative`里的`style`有点类似`css`，但又不是我们熟悉的`css`，可以理解为 "用`JavaScript`编写的阉割版`CSS`" ，即每个组件支持的Style属性是固定的，不能把Web里的 "`div`化万物" 的理念拿过来用。比如，文本必须用 `Text` 组件包裹，其他组件如`View`、`Image`是没有和`font`相关的属性的。 所以我们要对常用组件的`style`属性了然于心。

然后说到技巧，因为这里`style`已经变成了一个`JavaScript`对象，那么我们就应该用对象的思想来处理它。

### Global模块
这个模块的作用和`less`里的全局变量是一样的，我们把项目使用的色调、字体大小、最细边框、窗口大小等变量暴露给全局的 `container` 调用：

```JavaScript
import { PixelRatio, Dimensions } from 'react-native'

// 全局颜色
export const Color = {
    primary: '#f75a47',    // App主色调
    white: '#ffffff',      // 白色
    navBar: '#f9f9f9',     // NavBar的背景颜色
    text: {                // 给文本使用
        grey1: '#494949',
        grey2: '#686868',
        grey3: '#9B9B9B',
        grey4: '#B5B5B5',
        red: '#FC4586',
    },
    bg: {                 // 背景色
        grey1: '#ddd',
        grey2: '#eee',
        grey3: '#f7f7f7',
        red: '#FC4586',
        pink: '#F984AD',
    },
    border: {            // 边框颜色
        grey1: '#ddd',
        grey2: '#eee',
        grey3: '#f4f4f4',
        red: '#FC4586',
    },
};

// 全局字体大小
export const Size = {
    xxsmall: 10,
    xsmall: 12,
    small: 14,
    default: 16,
    large: 18,
    xlarge: 20,
    xxlarge: 24,
    pixel: 1/PixelRatio.get(),  // 最细边框
}

// 全局Window尺寸
export const Window = {
    width: Dimensions.get('window').width,
    height: Dimensions.get('window').height,
}
```

然后在 `container`里，我们可以直接引入。这样的好处是容易做到APP整体样式的统一

```
import { Size, Color, Window } from './global'
```

### 样式组合与覆盖
以前写Web的时候，为了减少代码量，我们很习惯去编写组件或者单元，这些组件融入到页面里就需要组合特定的样式，或者覆盖一些样式。这一点 `ReactNative`里也可以做到。我们可以把多个样式对象放在一个数组里使用：

```	JavaScript
<View style={[style1, style2, style3]}></View>
```

 看到这里，很多有洁癖的朋友估计会去纠结 应该编写多么小的单元，组合多少次比较合适。其实2次足矣，因为`ReactNative`天然的组件化特性，让我们可以把组件一层层剖开，我的建议也是多写子组件，合理使用同级之间的组合。

## 组件化设计技巧
`ReactNative`组件化开发能力浑然天成，语言本身优势明显，至于技巧，我觉得可以体现在下面三个方面：

### 组件设计模版
`ReactNative`组件其实就是`React`组件融入了原生App的`Native`能力。我们可以像写一个`React`组件一样去设计它，它完全支持`ES6＋`语法，这是我的书写模版，可供大家参考：

```JavaScript
// 导入React核心模块
import React, { Component, PropTypes, } from 'react'
// 导入组件使用到的Native依赖模块
import { View, StyleSheet, Text, TouchableOpacity, } from 'react-native'

// 定义并默认导出自己的component
export default class myComp extends Component {

  // 入参类型验证
  static propTypes = {
    // 如果类型是style
    containerStyle: View.propTypes.style,   
    // 如果类型是bool
	selected: PropTypes.bool,
	// 如果类型是string
	text: PropTypes.string,  
	// 如果类型是function
	onPress: PropTypes.func,
	// 如果类型是object
	options: PropTypes.object,
	// 如果类型是array
	source: PropTypes.array,
	// 如果类型是number
	num: PropTypes.number,
  }

  // 入参默认值（不设置则为undefined）
  static defaultProps = {
    selected: true,
    num: 5,
  }

  // 构造函数
  constructor(props) {
    // 继承父类的this对象和传入的外部属性
    super(props)
    // 设置初始状态
    this.state = {
	    selected: props.selected,
	    num: props.num,
	    source: props.source,
    }
  }

  // 遍历的部分可以写成子渲染函数
  _renderList(data) {
    if (Array.isArray(data)) {
      // 推荐这种写法
      return data.map((item, i) => {
        return (<Text key={i}>{item}</Text>)
      })
    }
  }

  // 事件处理句柄（触发处用匿名函数包裹以匹配当前的上下文对象）
  handlePress() {
    // 组件外部传入的回调函数先验证再触发
    this.props.onPress && this.props.onPress()
  }

  // 主渲染函数
  render() {
    // 推荐写法
	let { selected, num, source } = this.state;
	let { source, containerStyle } = this.props;
    return (
      <View style={[styles.container, containerStyle]}>
        // 匿名函数方式触发回调，避免使用bind
	    <TouchableOpacity onPress={()=>this.handlePress()}>
	      <Text>Hello World</Text>  
	    </TouchableOpacity>
	    <View>
		  { this._renderList(source) }
	    </View>
      </View>
    )
  }
}

// style写在最下面
const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
})
```

### 自成一体, 拒绝依赖  
在我的架构里，`React`组件分为`components` 和 `containers`。

 - `containers`顾名思义是页面的容器，每个App都有独立的一套`containers`，具有特定的样式和数据输入，无法迁移。
 - `components`下面组件都是相互独立的，除了`Native`和`React`的核心模块，拒绝任何外部依赖。对于`icon`素材、内部的子组件等建议用文件夹包裹成一个文件单元，相关的资源文件跟着组件走，让这些组件完全可以脱离具体的App项目独立存在，自成一体，可以随意迁移。（类似`Node_module`的设计思想）


> 大家知道，React的组件式开发就好比搭积木，如果我们不断积累并形成了一套完整的组件库，那么以后开发类似的App简直是如有神助。这是多么美妙的一件事！

### 属性命名要贴近官方

比如点击触发的回调函数，相信大家都有自己的命名习惯：`callback`、`handlePress`、`cb`、`onPress`.......

我的建议是沿用官方的`onPress`命名。遵循这条命名规范，是为了组件的推广和国际化做的考虑，建议平时查阅`ReactNative`官方文档的时候，记忆官方组件的属性命名方式，编写自己组件时候类似的属性名尽量和官方保持一致。

---

@欢迎关注我的 [github](https://github.com/jafeney) 和 [个人博客 －Jafeney](http://jafeney.com)
