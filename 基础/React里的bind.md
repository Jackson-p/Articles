# 为何React的函数要加bind

这个问题呢，其实来自于一次开发时那个时候需要给一个组件加一个绑定事件，并传递一个参数。当时的写法是

```js
onClick = {this.handleChange(6).bind(this)
```

现在看来是不是特zz。。。。，当然会报错了，正确地利用bind的传参方法

```js
onClick = {this.handleChange.bind(this,6)}
```

然后遇到了这个错误的时候我就在想，写Vue的时候咋就没思考过这个问题。为什么React的组件里面绑定方法就一定要加这个bind呢？而同样通过箭头函数的方式又为什么可以呢？上网看了很多前辈的说法，感觉都模糊其词，只是大体说了下在React的事件系统中，如果不bind就会使this丢失。所以我模拟分析了一下这其中的原因。

## 开始代码

一开始我使用一个对象来模拟这个过程的，感觉咋的都不太对。后来突然想到我们定义一个class类，比如class a的时候，他的type实质上是个函数。好我们完全用简化的手段去模仿React实现场景，并解释为啥要bind。

```js
function _class(){
    this.name = 'liaoliao';//组件内变量值
    this.saybind = function(){ //需要bind的es5写法
        console.log(this.name);
    }
    this.sayarrow = () => console.log(this.name); // 不需要bind的箭头函数写法
    return {//因为是模拟，所以就不实际实现jsx了。。。这里代表三种写法
        onClick1:this.saybind,
        onClick2:this.saybind.bind(this),
        onClick3:this.sayarrow
    }
}
//以下模拟生成实例后，用户在三种不同情况下点击绑定了事件的组件
var test = new _class();
test.onClick1();
test.onClick2();
test.onClick3();
```

## 开始解释

emmm，接下来就好说了。

情况一的慢动作拆解:

```js
function _class(){
    this.name = 'liaoliao';
    this.saybind = function(){
        console.log(this.name);
    }
    this.sayarrow = () => console.log(this.name);
}
var test = new _class();
var onClick = test.saybind;
onClick();
```

这是学js中经典的丢失this指向问题。。。。，此时调用this的是onClick()而他却在全局域中，自然undefined.

情况二的慢动作解析

```js
function _class(){
    this.name = 'liaoliao';
    this.saybind = function(){
        console.log(this.name);
    }
    this.sayarrow = () => console.log(this.name);
}
var test = new _class();
var onClick = test.saybind.bind(test);
onClick();
```

虽然是通过外界触发，但是已经把this保住了。

情况三的慢动作解析

```js
function _class(){
    this.name = 'liaoliao';
    this.saybind = function(){
        console.log(this.name);
    }
    this.sayarrow = () => console.log(this.name);
}
var test = new _class();
var onClick = test.sayarrow;
onClick();
```

这个是我之前相对最难理解的，因此还特意温习了一下箭头函数，都有点儿忘了。

箭头函数为什么不用bind？箭头函数是在函数定义时就确定了的，本身无this，this直接继承父级上下文中的this，箭头函数的父级上下文是谁，是_class函数作用域。故这个this也保住了，通过this.name自然能引出想要的name了。

## PS

补充一下，React的实现太复杂了，只是个人从相对简单的角度分析了下这个问题。熟读源码的大佬欢迎指教。若有错误，欢迎指出～（感谢感谢）