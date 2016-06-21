---
title: 当React.js遇到ES6  
date: 2016-06-18 14:57:27  
tags: 
- react.js
- es6  
categories: 
- 学习
- front-end
- react.js
---

虽然很早以前就知道了React.js，但是最近才下手使用。结果一上来就直接晕菜（ es5 or es6 ）  
都2016年了，况且还是自己瞎捣鼓的项目，怎么可能还用es5？  
所以，......  

component & 组件
===

React.js就是将一个个功能点封装成component，即组件，然后像普通HTML标签一样使用。在网上看到的大部分教程里都还是使用`React.createClass()`来创建组件  

<!--more-->

``` javascript
//ES5
class Photo extends React.Component {  
	handleSomething: function(e) { ... },
	render: function() { ... },
}

//ES6
class Photo extends React.Component {
	handleSomething(e) { ... }
	render() { ... }
}
```

生命周期
===

React.js将组建划分为三种状态：  
1. Mounting：已插入真实 DOM  
2. Updating：正在被重新渲染  
3. Unmounting：已移出真实 DOM  

每种状态对应两个处理函数，分为进入状态前调用--will，进入状态后调用--did：  
1. componentWillMount()  
2. componentDidMount()
3. componentWillUpdate(object nextProps, object nextState)  
4. componentDidUpdate(object prevProps, object prevState)  
5. componentWillUnmount()  

所有函数在es5和es6中都可以直接使用，不过`componentWillMount`在es6中可被`constructor`代替

``` javascript
// The ES5 way
var EmbedModal = React.createClass({
  	componentWillMount: function() { … },
});

// The ES6+ way
class EmbedModal extends React.Component {
	constructor(props) {
		super(props);
		// 通常在componentwillmount当中的操作，可放到这里...
	}
}
```

## 有几个关键点  
1. componentWillMount 用来处理render之前的逻辑  
2. 不要在render中处理业务逻辑，即一定不能出现改变 state 的操作  
3. componentDidMount 中处理和真实DOM相关的逻辑，典型的场景：在这里调用ajax，使用jQuery  

## 实例化组件的过程  

1. getInitialState: 获取 this.state 的默认值
2. componentWillMount: 在render之前调用此方法，在render之前需要做的事情就在这里处理
3. render: 渲染并返回一个虚拟DOM
4. componentDidMount: 在render之后，react会使用render返回的虚拟DOM来创建真实DOM，完成之后调用此方法。  

## 更新组件的过程   

1. componentWillRecieveProps:   父组件或者通过组件的实例调用 setProps 改变当前组件的 props 时调用。
2. shouldComponentUpdate: 是否需要更新，慎用
3. componentWillUpdate: 调用 render方之前
4. render: 
5. componentDidUpdate: 真实DOM已经完成更新。  

属性初始化
===

React.js的组件有两个属性 **props** ， **state** 。  
> 原文博主划分的更细：  
> **props** 分为属性（prop type），默认属性（default prop）。  
> **state** 分为初始状态(initial state)。  

区别：  
**props** 代表固有属性，改变时不会触发视图重新渲染，可通过es6类中的`static`来声明。  
**state** 代表组件状态，对于**初始状态（initial state）**可以通过ES7当中的属性初始化（property initializers）来实现，并在在随后的组件使用中，不允许直接对**state**进行修改（除了初始化阶段），只能通过`this.setState() { ... }`来更新状态  

``` javascript
// The ES5 way
var Video = React.createClass({
    //初始化默认props
    getDefaultProps: function() {
        return {
            autoPlay: false,
            maxLoops: 10,
        };
    },
    //初始化state
    getInitialState: function() {
        return {
            loopsRemaining: this.props.maxLoops,
        };
    },
    propTypes: {
        autoPlay: React.PropTypes.bool.isRequired,
        maxLoops: React.PropTypes.number.isRequired,
        posterFrameSrc: React.PropTypes.string.isRequired,
        videoSrc: React.PropTypes.string.isRequired,
    },
});

// The ES6+ way
class Video extends React.Component {
    //初始化默认props
    static defaultProps = {
        autoPlay: false,
        maxLoops: 10,
    }
    static propTypes = {
        autoPlay: React.PropTypes.bool.isRequired,
        maxLoops: React.PropTypes.number.isRequired,
        posterFrameSrc: React.PropTypes.string.isRequired,
        videoSrc: React.PropTypes.string.isRequired,
    }
    //初始化state
    state = {
        loopsRemaining: this.props.maxLoops,
    }
}
```

对于`static`这里有点说明，它属于es7的标准里的方式，属于超前使用。  

> ES7 中在构造函数（ constructor ）下的属性初始化操作中的 this 指向的是类的实例，所以初始状态（ initial state ）可以通过 this.prop （ 即传入的参数 ）来设定。    

``` javascript
/** 这两种形式等价 */

// The ES6 way
class Video extends React.Component {
    //somethings
}
Video.defaultProps = {
    autoPlay: false,
    maxLoops: 10,
}

// The ES7 way
class Video extends React.Component {
    //初始化默认props
    static defaultProps = {
        autoPlay: false,
        maxLoops: 10,
    }
}
```

箭头函数（Arrow function）
===

对于这一特性的使用，属于有得有失。因为React.js在原先的`React.createClass`方法中进行了很多额外属性的绑定，其中之一：`this`在`React.createClass`方法中指向的是组件的实例化对象

``` javascript
var Video = React.createClass({
    //初始化默认props
    handleSomething: function(e) { 
    	//this指向的是Video的实例对象
		this.setState({ open: false })
	}
});
```

但是，当我们使用ES6的语法来定义类后，React.js在`React.createClass`方法中进行的属性绑定都会没有，只能自己进行`this`绑定

``` javascript
// The ES6 way
class Video extends React.Component {
    constructor(props) {
        super(props);
        // 手动绑定
        this.handleOptionsButtonClick = this.handleSomething.bind(this);
    }
    handleSomething(e) {
        // this指向的是Video的实例对象
        this.setState({ open: true });
    }
}
```

不过，可以利用ES6中的箭头函数（Arrow function）和属性初始化（ property initializers ）这俩新特性来进行`this`

``` javascript
// The ES6 way
class Video extends React.Component {
    constructor(props) {
        super(props);
        // ...
    }
    handleSomething = (e) => {
        // 这里还有点不明白，箭头函数的this绑定？
        this.setState({ open: true });
    }
}
```

析构 & 扩展运算符
===

总感觉这个特性是ES6里头最麻烦的，这个特性一般使用于属性的传递时。

``` javascript
class Video extends React.Component {
	handlerSomething = (e) => { ... }
    render() {
        let {
            className,
            ...others,  // 包括this.props中除了calssName外的所有属性
        } = this.props;
        
        return (
            <div className={className}>
                <PostsGrid {...others} />
                <button onClick={this. handlerSomething}>Load more</button>
            </div>
        );
    }
}
```

有一个小技巧，可以利用**扩展运算符属性**和**普通的属性**优先级，来进行属性覆盖  

``` HTML
//最终得到的class为override，即使this.props中也存在className属性
<div {...this.props} className="override"> ... </div>
```

字符串模板
===

感觉这是平常最有用的一个特性，毕竟经常会碰到字符串拼接的需求

``` javascript
let inputName = 'name';

let className = inputName + 'Value';
//上下两种形式等价
let className = `${inputName}Value`;
```

其他
===

一些关于React.js的属性说明  

## this.props.children  

本身`this.props`对象的属性与组件的属性一一对应，但是有一个例外，就是 `this.props.children`属性。它表示组件的所有子节点。
`this.props.children`的值可能有三种可能：  
1. 如果当前组件没有子节点，它就是 `undefined`。  
2. 如果有一个子节点，数据类型是 `object`。  
3. 如果有多个子节点，数据类型就是 `array`。  
所以，最好使用React.js提供的`React.Children`来处理`this.props.children`，`React.Children`永远都是 `array`。

## PropTypes

组件的属性验证

``` javascript
// The ES5 way
var Video = React.createClass({
    propTypes: {
        autoPlay: React.PropTypes.bool.isRequired,
        maxLoops: React.PropTypes.number.isRequired,
        posterFrameSrc: React.PropTypes.string.isRequired,
        videoSrc: React.PropTypes.string.isRequired,
    },
});

// The ES6+ way
class Video extends React.Component {
    static propTypes = {
        autoPlay: React.PropTypes.bool.isRequired,
        maxLoops: React.PropTypes.number.isRequired,
        posterFrameSrc: React.PropTypes.string.isRequired,
        videoSrc: React.PropTypes.string.isRequired,
    }
}
```  

## 获取真实的DOM节点

组件并不是真实的 DOM 节点，而是React.js中最核心的虚拟 DOM (virtual DOM)，只有当它插入HTML文档后，才会变成真是的DOM，对应React.js中的 `componentDidMount` 

``` javascript
var Vedio = React.createClass({
    handleClick: function() {
        this.refs.myTextInput.focus();
    },
    render: function() {
	    return (
	        <div>
	            <input type="text" ref="myTextInput" />
	            <input type="button" value="Focus the text input" onClick={this.handleClick} />
	        </div>
	    );
    }
});
```

## 表单
表单填入这个行为属于用户跟组件的互动，所以不能用 `this.props` 

``` javascript
class Video extends React.Component {
    state = { value: 'Hello!' }
    handleChange = (e) => { 
        this.setState({ value: e.target.value }) 
    }
    render: function () {
        var value = this.state.value;
        return (
            <div>
                <input type="text" value={value} onChange={this.handleChange} />
                <p>{value}</p>
            </div>
        );
    }
}
```

参考
===

1. [http://blog.mcbird.cn/2015/09/11/react-on-es6-plus/](http://blog.mcbird.cn/2015/09/11/react-on-es6-plus/)  
2. [http://blog.csdn.net/lihongxun945/article/details/46334379/](http://blog.csdn.net/lihongxun945/article/details/46334379/)
3. [http://www.ruanyifeng.com/blog/2015/03/react](http://www.ruanyifeng.com/blog/2015/03/react)




