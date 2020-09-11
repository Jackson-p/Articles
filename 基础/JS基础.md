# Js基础

__前言:__

校招时写的，算是js入门知识了，博客迁移来的

## es5

### js数据类型

栈：原始数据类型undefined，Null，Boolean，Number(含NaN)，String(字符串不可修改))

堆：引用数据类型对象，数组，函数，Date

[基本数据类型和引用数据类型的区别](https://www.cnblogs.com/cxying93/p/6106469.html)

> typeof简单判定（无法区分null，数组和对象）

    typeof 2 //number
    typeof '222' //string
    typeof true //boolean
    typeof (function{}) //function
    typeof undefined //undifined
    typeof null //object
    typeof {}   //object
    typeof [] //object

>原生原型拓展函数判定

    var gettype = Object.prototype.toString
    gettype.call('aaaa')        //[object String]

    gettype.call(2222)          //[object Number]

    gettype.call(true)          //[object Boolean]

    gettype.call(undefined)     //[object Undefined]

    gettype.call(null)          //[object Null]

    gettype.call({})            //[object Object]
    gettype.call([])            //[object Array]
    gettype.call(function(){})  //[object Function]

注意两种判定方法的大小写。。。

>判断{}空对象

```js
var test = {};
if(JSON.stringify(test) == '{}'){
	console.log("方法1");
}
try{
	for(var i in test){
		throw new Error('不是空对象');
	}
	console.log("方法2");//此方法后被封装到jQuery的$.isEmptyObject
}catch(e){
	console.log(e);
}
if(!(Object.getOwnPropertyNames(test).length)){console.log("方法3");}
if(!(Object.keys(test).length)){console.log("方法4");}//es6

```

### 类型转化问题

这个问题在《你不知道的JS》中有较为详尽的解释，当初看的时候也是一扫而过觉得精彩，直到突然有一天在面试中被问到 [] == true 和 {} == true 才突然觉得这里可以搞。

#### 引例

在说类型转化之前先说下基本类型和引用类型，js中六大基本数据类型都是保存在栈中，而引用类型如数组、函数或者Object都是保存在堆中，当我们赋予a = obj 这种情况时，实际上a是只是一个指向obj的指针，嗯。。。下面的例子蛮明晰的我觉得

```js
var a = function() {console.log(11)};
var b = function() {console.log(11)};
console.log(a == b);

var a = {
    name:"liao",
    say: function(){
        console.log("i'm", this.name);
    }
}
var b = {
    name:"liao",
    say: function(){
        console.log("i'm", this.name);
    }
}

console.log(a == b);//false
console.log({} == {});//false
console.log([] == []);//false
console.log(new String(123) == new String(123));//false
console.log(1 == 1);//true
console.log("123" == "123");//true
console.log("123" == new String(123));//true

```

#### 理论实践结合，看看中间算法

[实践查表](https://dorey.github.io/JavaScript-Equality-Table/)

[理论算法](https://blog.csdn.net/wmaoshu/article/details/69676896)

### 变量声明之var,let,const

[讲的挺全的](https://blog.csdn.net/zhouzuoluo/article/details/80724033)

### 变量提升

>在定义或定义赋值之前引用，会只提变量，为undefined

    console.log(foo); //undefined
    var foo = 2;

>举个详细的例子

```js

    var tmp = new Date();
    function f() {
    console.log(tmp);
        if (false) {
            var tmp = 'hello world';
        }
    }
    f(); // undefined
```
这段代码很经典，出现情况是定义赋值前引用，如果我们把后面的赋值去掉，根据前面的规则，结果还是undefined，如果只保留tmp = 'hello world'那么结果会正常输出,我觉得这种变量提升和函数作用域中的作用域链有一定的关系，注意这个和函数先解析没关系，，为了避免误解再来两个。

```js

var a = 2; 
var a = function a(){ console.log('1') }; 
console.log(a);//[Function: a]

-------------------

var a = 2;
function a(){
    console.log(1);
}
console.log(a);

```

>函数同样存在变量提升？

    function getValue() {
        return 'c';
    }
    function functions(flag) {
        if (flag) {
            function getValue() { return 'a'; }
        } else {
            function getValue() { console.log(1);return 'b'; }
        }
        return getValue();
    }
    console.log(functions(true));
    这里本以为这一段代码最后总是会返回b，牛客上的题也是这么设计的，但后来的node环境可能做了些修正，这里其实会返回a的。。。,而且没输出1说明第二个函数解析都没解析，所以个人对函数的理解是
    ->1、函数解析本身并不存在“变量提升”
    紧接着第二个测试
    function doSth(func,a,b){
        function func(a,b) {
            return a-b;
        }
        return func(a,b);
    }
    function func(a,b) {
        return a+b;
    }
    console.log(doSth(func,1,2));

    ->2、函数的调用沿作用域链查找

    紧接着看了网上大牛的3个测试

    (function(){
    function a(){};
        var a;
        console.log(typeof a); //function
    })();
    (function(){
        console.log(typeof a);//function
        function a(){};
        var a = 1;
    })();
    (function(){
        function a(){};
        var a = 1;
        console.log(typeof(a)); //number
    })();

    ->3、js先解析再执行(会被覆盖)

>词法分析一个

    function a(){
        var b = 'a';
        function b(){
            console.log('b')
        }
        console.log(b)
    }
    a()
    //解析时
    function a(){
        var b = 'a';
        b = function b(){console.log('b')};
        console.log(b);
    }
    //执行时
    function a(){
        b = function b(){console.log('b')};
        var b = 'a';
        console.log(b);
    }
    a();
>总结：

    ES5只有全局作用域和函数作用域，没有块级作用域，因此es6新增了let，来引入块级作用域

------------

### 对闭包的理解及常见应用场景

> 闭包的三个特点

```txt
1、函数嵌套函数
2、函数内部可以引用外部的参数和变量
3、参数和变量不会被垃圾回收机制回收
```

>闭包的缺点

```txt
1、会造成内存泄漏
2、在函数中创建函数是不明智，闭包对脚本性能有负面影响，包括处理速度和内存消耗
```

以下举几个闭包典型例子～

>闭包应用场景1:保存变量

```js
先看一段代码：
function tell(){
    var a=[];
    for(var i=0;i<10;i++)//如果是块级作用域，也就你把var改成let就不会需要闭包啦
    {
        a[i]=function(){
            return i;
        }
    }
    // for(i=0;i<10;i++)
    //     console.log(a[i]());
    console.log(a[2]());
}
tell();
经典的例子，结果是10，因为作用域的问题，i循环后再调用function的时候，已经是10了。再举个跟上面很像的例子，这回用闭包解决
function count(){
    var arr = [];
    for(var i=0;i<10;i++)
    {
        arr.push(function(){
            return i*i;
        });
    }
    console.log(arr[1]());
    return arr;
}
count();
使用闭包后的修正版，闭包保存了立即函数的变量
function count(){
    var arr = [];
    for(var i=0;i<10;i++)
    {
        arr.push((function(n){
            return function(){
                return n*n;
            }
        })(i));
    }
    return arr;
}
var result = count();
console.log(result[0]());
console.log(result[5]());
```

>闭包应用场景2:函数式编程

1、高阶函数

像sort，map，filter，reduce他们就是不错的例子,正则里的replace附带函数,

```js
num.sort(function(x,y){
    return x-y;
})
```

2、函数科里化

```js
function addsome(x){
    return function(y){
        return x+y;
    };
}
var add5 = addsome(5);
var add10 = addsome(10);
console.log(add5(1));
console.log(add10(1));
输出6和11
```

```js
//上面的例子其实不算是严格的函数柯里化的思想，或者说比较基础吧，上一个比较经典的面试题
function f(a, b){
    if(arguments.length===2){
        return a+b;
    }else{
        let args = arguments;
        return function(){
            return args[0] + arguments[0];
        }
    }
}
console.log(f(1)(2));
console.log(f(1,2));
```

3、偏函数

个人理解是借用函数的前一部分形参作为输入（一般不做输出），后一部分可输入可输出，来产生一个新的定制函数的形式(选自朴灵)

```js
var isType = function(type){
    return function(obj){
        return Object.prototype.toString.call(obj) == '[object ' + type + ']';
    }
}
var isString = isType('String');
console.log(isString("123"));
```

>闭包应用场景3:实现变量的私有化和公有化（用那个经典的立即执行函数当然是可以的）

```js
function create_timer(){
    var x=0;
    return {
        say:function add(){
            console.log(x++);
        }
    };
}
var said = create_timer();
for(var i=0;i<10;i++)
    said.say();
```

>闭包应用场景4:超多的回调，差不多回调的最简含义如下

```js
function dosomething(callback,x,y){
    return callback(x,y);
}
function addd(x,y){
    return x+y;
}
console.log(dosomething(addd,1,2));
```

>闭包应用场景5:jQuery插件书写中，避免在插件内部使用$作为jQuery对象，而使用
完整的jQuery来表示，可以用闭包来回避这个问题

```js
;(function($){
    //sth
})(jQuery);
```


### js单线程

#### 异步与回调

>异步防止阻塞 [参考](https://www.cnblogs.com/dong-xu/p/7000163.html)

```js
//举一个同步遍历打印数组和异步回调打印数组的例子～
var arr = new Array(200);
//arr.fill(1);
for(var i=0;i<arr.length;i++)
    arr[i]=i;
function ascyprint(arr,handle){
    var t = setInterval(function(){
        if(!arr.length){
            clearInterval(t);
        }else{
            handle(arr.shift());
        }
    },0);
}
ascyprint(arr,function(value){
    console.log(value);
});
arr.forEach(function(index,value,arr){
    console.log(value);
});
```

>页面上回调使用实例

```HTML
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
    <p id="test">大家好，我是布莱利liaoliao</p>
    <button type="submit">12px</button>
    <button type="submit">24px</button>
    <button type="submit">48px</button>
    <script>
        var pp = document.getElementById('test')
        function changesize(x){
            return function(){
                pp.style.fontSize = x+"px";
            };
        }
        var size1 = changesize(12);
        var size2 = changesize(24);
        var size3 = changesize(48);
        var btn=document.getElementsByTagName('button');
        if(btn[0].type == 'submit')
            console.log(1);
        btn[0].onclick = size1;
        btn[1].onclick = size2;
        btn[2].onclick = size3;
        </script>
</body>
</html>
```

>js的四种异步方式

[参考](https://www.cnblogs.com/zuobaiquan01/p/8477322.html)

#### 事件循环与任务队列

>事件循环

JS 会创建一个类似于 while (true) 的循环，每执行一次循环体的过程称之为 Tick。每次 Tick 的过程就是查看是否有待处理事件，如果有则取出相关事件及回调函数放入执行栈中由主线程执行。待处理的事件会存储在一个任务队列中，也就是每次 Tick 会查看任务队列中是否有需要执行的任务。

>任务队列

异步操作会将相关回调添加到任务队列中。而不同的异步操作添加到任务队列的时机也不同，如 onclick, setTimeout, ajax 处理的方式都不同，这些异步操作是由浏览器内核的 webcore 来执行的，webcore 包含上图中的3种 webAPI，分别是 DOM Binding、network、timer模块。
    
举例：

* onclick 由浏览器内核的 DOM Binding 模块来处理，当事件触发的时候，回调函数会立即添加到任务队列中

* setTimeout 会由浏览器内核的 timer 模块来进行延时处理，当时间到达的时候，才会将回调函数添加到任务队列中

* ajax 则会由浏览器内核的 network 模块来处理，在网络请求完成返回之后，才将回调添加到任务队列中。

>主线程 

[参考](https://www.cnblogs.com/allenlily5200/p/7096711.html)

 JS 只有一个线程，称之为主线程。而事件循环是主线程中执行栈里的代码执行完毕之后，才开始执行的。所以，主线程中要执行的代码时间过长，会阻塞事件循环的执行，也就会阻塞异步操作的执行。只有当主线程中执行栈为空的时候（即同步代码执行完后），才会进行事件循环来观察要执行的事件回调，当事件循环检测到任务队列中有事件就取出相关回调放入执行栈中由主线程执行

>我们用常见的写一个定时器来分析这个问题

```js
//定时器
function timer(period){
    for(var i=0;i<period;i++)
    {
        setTimeout(function(){
            return (function(n){
                console.log(n);
            })(i);
        },1000);
    }
}
timer(5);
// 结果并不会是我们期待的1,2,3,4,而是在一秒钟后输出5个5，调试后发现for循环很快设置了5个定时器，他们统
// 一会在1s后执行，设置完定时器后，当前所在事件也就是这个函数结束了，这个时候开始执行setTimeout定时器
// 内部的函数，因为i已经是5了，所以所有的定时器里都变成了5，并一起输出了出来，所以想正经写一个定时器还是
// 要用下递归
var t;
function timer(time){
    console.log(time);
    if(time==50)
    {
        return clearTimeout(t);
    }
    t=setTimeout(()=>timer(time+1),1000);
}
timer(1);
// 这个思路就是每个函数都会定义一个定时器，函数结束的时候正好运行
// setTimeout函数，也就是正好开启下一个函数，这样就实现了定时器雏形
//以下是对es 6 相对失败的思考orz
function timer(){
    var i=0;
    var p=new Promise((resolve,reject) =>{
        setTimeout(()=>{
            console.log("计时开始");
            resolve(i+1);
        },1000);
    });
    while(i<50)
    {
        p.then((val) =>{
            console.log(val);
        });
    }
    return;
}
timer();
// 这段代码就很玄妙了，因为setTimeout函数会放在事件队列里，等当前函数执行完了才会设定，所以会进入死循环。。
setTimeout(function(){console.log(4)},0);
new Promise(function(resolve){
    console.log(1)
    for( var i=0 ; i<10000 ; i++ ){
        i==9999 && resolve()
    }
    console.log(2)
}).then(function(){
    console.log(5)
});
console.log(3);
// 看到知乎上经典的问题，答案是1，2，3，5，4.
// 又分析了去掉while的代码，差不多得出了一个有关setTimeout和Promise的规律
// setTimeout是本轮事件结束下轮事件开始时运行，promise是本轮事件结束时运行，也就是说就算是
// then也是在setTimeout之前的
//这里学了一下回来补充，所谓的promise与setTimeout执行顺序就是传说中的宏任务与微任务的区别，微任务比较少基本上也就promise和process.nextTick()会涉及
```

补充我个人觉得的话上述那个例子作为微任务宏任务的简化例子就差不多了

[上大牛](http://www.cnblogs.com/jiasm/p/9482443.html)

> setTimeout+同步异步队列

```js
for(var i=1;i<=4;i++){
	var time=setTimeout(function(i){
		clearTimeout(time);
                console.log(i);
	},1000,i);
}
//输出结果1,2,3

```

------------

### 原型与原型链与构造函数

##### new、_proto_、prototype、constructor

>简单版理解

我觉得这种概念用图说话最好，这个图简直比红宝书还要简洁明了

![简图](https://user-images.githubusercontent.com/20417123/51168232-a0725100-18e3-11e9-9bb7-6f9ec4170535.png)


>复杂版理解

这个就涉及到了Object.prototype了

![复图](https://user-images.githubusercontent.com/20417123/51168261-b54ee480-18e3-11e9-9bfb-29ee76eed559.png)

>分析：new的过程

```js
var obj = new Base();//Base 为某构造函数
等同于
var obj = {};
obj.__proto__ = Base.prototype;
Base.call(obj);

```

### 函数与作用域

#### this的理解

[原理层面](http://www.ruanyifeng.com/blog/2018/06/javascript-this.html)

[实战层面(cherry的文章)](https://juejin.im/post/59bfe84351882531b730bac2)

修正补充一下，cherry的文章适合用es5的this，箭头函数中确实this也是指向引用它的那个环境，可是这个环境官方是有所定义的。

重要的几点：

* 箭头函数的this是在定义函数时绑定而不是执行时绑定
* 箭头函数的this实际上是继承父级上下文（注意对象不是上下文，应该指的是函数作用域和全局作用域）
* 箭头函数本身无this，因此无法做构造函数

[箭头函数参考](https://www.cnblogs.com/var-chu/p/8476464.html)

#### 函数参数与arguments

[例子讲解～](http://www.cnblogs.com/lwbqqyumidi/archive/2012/12/03/2799833.html)

#### 作用域链的理解

一般是出现在闭包研究问题的这个过程里吧哈哈，这里不是很难，记录一篇有关作用域的文章，单纯觉得写得很有趣

[皮](https://segmentfault.com/a/1190000015779676)


------------

### 面向对象与继承

注：面向对象是一项非常有用的模式，js在初生时并没有考虑太多这方面的问题，后来无数js大牛创造出了这种模式，感
觉js创建对象的方法也有很多，工厂模式，原型模式，构造函数模式等等。。。。各大教科书高程什么的讲得很全，
个人筛选出了可能比较好的两种模式，毕竟没法记住所有的方法。。

#### 创建对象

>法一：组合使用构造函数模式和原型模式

```js
function Person(name,sex,height){
    this.name = name;
    this.sex = sex;
    this.height = height;
    this.friend = ["liao","liaoliao"];
}
Person.prototype = {
    construtor :Person,
    say : function(){
        console.log("i'm "+this.name);
    }
}
// 个人的想法是，构造函数中定义好这个对象所含有的变量，而原型方面定义好这个对象的函数，从而实现了“属性”和
// “功能”的分离，创建多个对象时也不会影响这里的friends数组，不是引用而是创建
```

>法二：稳妥构造函数（不是寄生式构造，寄生式构造不能用instanceof确定对象类型，不建议使用寄生式构造)

```js
function Person(name,sex,height){
    var o=new Object();
    var name = name;
    var sex = sex;
    var height = height;
    var friend = ["liao","liaoliao"];
    o.say = function(){
        console.log("i'm "+name);
    }
    return o;
}

// 这种构造方式最大的特点就是安全，封装性很好，不可以改变定义好对象的内部变量，但要注意三点（在某博客上
// 看到的）

// 注意： （以下3点）
// 1. 在稳妥构造函数中变量不能挂到要返回的对象o中
// 2. 在稳妥构造函数中的自定义函数操作元素时使用不要用this

// 3. 在函数外部使用稳妥构造函数时不用new。
```

>完整的带继承的测试代码

```js
function Person(name,sex,height){
    this.name = name;
    this.sex = sex;
    this.height = height;
    this.friend = ["liao","liaoliao"];
}
Person.prototype = {
    construtor :Person,
    say : function(){
        console.log("i'm "+this.name);
    }
}
//寄生式构造
// function Person(name,sex,height){
//     var o = new Object();
//     o.name = name;
//     o.sex = sex;
//     o.height = height;
//     o.friend = ["liao","liaoliao"];
//     o.say = function(){
//         console.log("i'm "+this.name);
//     }
//     return o;
// }
//稳妥构造
// function Person(name,sex,height){
//     var o=new Object();
//     var name = name;
//     var sex = sex;
//     var height = height;
//     var friend = ["liao","liaoliao"];
//     o.say = function(){
//         console.log("i'm "+name);
//     }
//     return o;
// }
// var person1 = Person("liao","boy",150);
// person1.say();
// console.log(person1.friend);
// person1.name="gg";
// person1.say();
function Superman(){
    var superman = new Person();
    superman.name = "wyp";
    superman.fly = function(){
        superman.say.call(this);
        console.log(this.name+" is flying");
    };
    return superman;
}
var person1=new Person("yao","公",150);
var person2=new Person("ji","母",160);
person1.say(); //i'm yao
console.log(person1.friend); //['liao','liaoliao']
person1.friend.push("bulaili");
console.log(person1.friend); //['liao','liaoliao','bulaili']
console.log(person2.friend); //['liao','liaoliao']
person2.say(); //i'm ji
person1.name="2333";
person1.say(); //i'm 2333
var wyp=new Superman();
wyp.fly(); //i'm wyp \n wyp is flying
wyp.name = "daer";
wyp.fly(); //i'm daer \n daer is flying
// 上述用的是寄生式继承，个人觉得传统的原型链+组合函数-》组合继承，比较散化封装性不太好，写多了容
// 易逻辑混乱

// 然后看了看寄生式继承与寄生式组合继承，两者非常像：思路都是创建对象->增强对象->返回对象，于是没有就着高
// 程上所写的把构造函数借用和原型链操作分开，而是做了个封装觉得这样的风格最好吧～，虽然只是个寄生式
// 继承orz
```

#### 继承

> 感觉必须要知道的组合继承

```js
function Person(name,height){
    this.name = name;
    this.height = height;
    this.friends=["liao1","liao2"];
}
Person.prototype.say = function () {
    console.log("I'm "+this.name);
}
function child(name,height,sex){
    Person.apply(this);
    this.name=name;
    this.height = height;
    this.sex = sex;
}
child.prototype = new Person();
var hh=new child("liao",150,"boy");
hh.say();
console.log(hh.friends);

```

这种继承的缺点是调用了两次父类构造函数性能消耗大一点，但是相对于其他的继承方案而言已经是比较不错的继承方法了

> 有所升级的寄生组合继承

```js
function Person(name,height){
    this.name = name;
    this.height = height;
    this.friends=["liao1","liao2"];
}
Person.prototype.say = function () {
    console.log("I'm "+this.name);
}
function child(name,height,sex){
    Person.apply(this);
    this.name=name;
    this.height = height;
    this.sex = sex;
}
(function(){
    var F = function(){};
    F.prototype = Person.prototype;
    child.prototype = new F();
})()
var hh=new child("liao",150,"boy");
hh.say();
console.log(hh.friends);
```

F算是对红宝书的致敬吧，上面记得用的就是这个F，当时老实讲还没看懂orz，现在理解了其实就是有一次构造函数的调用变成一个啥也没有的构造函数的调用（这样会不会使原型链稍显混乱？233）说个有趣的事情吧，其实Object.create()就是上述的中间层替换的语法糖，看下MDN的polyfill（）2333

```js
function F() {}
    F.prototype = proto;
    return new F();
```

但是这种构造函数的情况下不太适合写Object.create()哈，一般他和对象字面量结合在一起玩～

```js
//这段代码是网上某老哥的
var Parent = {
    getName: function() {
        return this.name;
    },
    getSex: function() {
        return this.sex;
    }
}
 
var Child = Object.create(Parent, {
    name: { value: "Benjamin"},
    url : { value: "http://www.zuojj.com"}
});
 
var SubChild = Object.create(Child, {
    name: {value: "zuojj"},
    sex : {value: "male"}
})
 
//Outputs: http://wwww.zuojj.com
console.log(SubChild.url);
 
//Outputs: zuojj
console.log(SubChild.getName());
 
//Outputs: undefined
console.log(Child.sex);
 
//Outputs: Benjamin
console.log(Child.getName());
```

### 对象的深浅拷贝

>前提

在讨论深浅拷贝问题，先思考一下这个问题的实际出现场景，我们这里用到了最简单的变量，函数，和没有函数的字面量对象三种情况：

```js
var a = 3;
var b = a;
b = 4;
console.log(b);//4
console.log(a);//3
var c = function(){
    console.log("hey jude");
}
var d = c;
d = function() {
    console.log("Brelly liaoliao");
}
c();//hey jude
d();//Brelly liaoliao ,注意这里表面上是函数实际上也是简单变量造成深拷贝
var obj1 = {
    name:"liaoliao",
    height:150
};
var obj2 = obj1;
obj2.name = "俊";
console.log(obj1.name);
console.log(obj2.name);//这里我们的obj2的改动影响的obj1的改动
```

所以得出出现深浅拷贝问题的情形在于对象，对象赋值或者说对象引用后改动的影响

>浅拷贝概念

    像前提中出现的我们定义一个对象，并直接赋值给一个变量的时候，我们改变这个变量也会改变愿对象，这是因为变量其实没有重新开辟一片空间去保存一个对象的副本，而只是一个单纯的指向原对象的一个引用，所以我们改变变量的时候当然也会改变原有的对象

>深拷贝概念

    这样看来深拷贝就好理解了，重新开辟一个空间，用来做原对象的副本，改变这个副本并不会影响原来的对象

>深拷贝方法

先给出这样的场景：

```js
var object1 = {
    apple: 0,
    banana: {
        weight: 52,
        price: 100
    },
    cherry: 97
};
var object2 = {
    banana: {
        price: 200
    },
    durian: 100
};
var object3 = {
    name:"俊",
    say:function(){
        console.log("我是"+this.name);
    }
};
```

__方法一:__ json转换（因为底层是二进制）

这个就是会吧原来的数据转化为字符串，这是针对对象的所有引用关系就不复存在了，然后再转化回来就是
    一个全新的对象。不在出现新对象改动污染原始对象的问题了.

```js
var b = JSON.parse(JSON.stringify(object1));
b.apple = 2;
console.log(object1.apple); //0
console.log(b.apple); //2
```

缺点：对象里不能含有函数，也就是说只适于json支持的数据格式

```js
var b = JSON.parse(JSON.stringify(object3));
b.name = "liao";
object3.say();
b.say();
//这样的拷贝就成功。。。报了错， b.say() is not a function
```

__方法二:__ 手写个clone函数吧

```js
function isArray(obj) {
    return Object.prototype.toString.call(obj) == '[object Array]';
}
var clone = function(v) {
    var o = isArray(v) ? [] : {};
    for(var i in v) {
        o[i] = typeof(v[i]) === 'object' ? clone(v[i]) : v[i];
    }
    return o;
};
var a = object3;
var b = clone(object3);
b.name = "liao";
a.say();
b.say();
```

__探索:__ jQuery源码中的extend

感觉好难，自己模仿着写了一个不知道对不对

```js

//默认情况浅拷贝
//object1--->{"apple":0,"banana":{"price":200},"cherry":97,"durian":100}
//object2的banner覆盖了object1的banner，但是weight属性未被继承
//$.extend(object1, object2);

//深拷贝
//object1--->{"apple":0,"banana":{"weight":52,"price":200},"cherry":97,"durian":100}
//object2的banner覆盖了object1的banner，但是weight属性也被继承了呦
// $.extend(true,object1, object2);

// console.log('object1--->'+JSON.stringify(object1));

//以下为jQuery的extend仿写版
function extend(deep, target, obj) {
    var first = arguments[0] || {};
    var i = 0;
    var src, copy, clone,name,copyIsArray;
    if (typeof deep == "boolean") {
        deep = deep;
        i = 1;
    }
    else
        deep = false;
    var canshu = [].slice.call(arguments, i);
    var len = canshu.length;
    if (!len)
        return;
    if (len == 1)
        return target;
    //canshu[0]=== target
    if (typeof (target) !== "object" && !isFunction(target))
        target = {};
    for (var i = 1; i < len; i++) {
        for (name in canshu[i]) {
            //根据被扩展对象的键获得目标对象相应值，并赋值给src
            src = target[name];
            //得到扩展对象的值
            copy = canshu[i][name];
            if (target === copy) {
                continue;
            }
            if (deep && copy && ((typeof copy === "object") || (copyIsArray = isArray(copy)))) {
                if (copyIsArray) {
                    //将copyIsArray重新设置为false，为下次遍历做准备
                    copyIsArray = false;
                    // 判断被扩展的对象中src是不是数组
                    clone = src && isArray(src) ? src : [];
                } else {
                    // 判断被扩展的对象中src是不是纯对象
                    clone = src && typeof (src) == "object" ? src : {};
                }
                target[name] = extend(deep, clone, copy);
                // 如果不需要深度复制，则直接把copy（第i个被扩展对象中被遍历的那个键的值
            } else if (copy !== undefined) {
                target[name] = copy;
            }
        }
    }
    return target;
}
//console.log(extend(false,object1,object2));
//console.log(extend(true, object1, object2));
```


### 跨域
跨域问题一直是前端非常重要的一点，会出现在像ajax这种向其他网站发送请求的过程中（出于安全性考虑，浏览器不允许ajax跨域获取数据，当你访问地址的时候，实际上数据已经从别的域名拿过来了(可以从network上的接收包查看)，但是浏览器为了保障不被信用的数据注入本地，进行了拦截），写一个小文章简单研究一下吧～

#### 同源策略
说到跨域必须要先提的就是浏览器的同源策略，不符合同源策略的请求就叫跨域请求。如果两个网页的域名、协议、端口均相同他们就是同源的，反之就是非同源的，此时不会执行跨域的脚本。

#### 跨域的方法
[参考](https://github.com/happylindz/blog/issues/3)
+红宝书上一个图像ping

### 通信
由对跨域的分析看了红宝书之后觉得通信部分也值得一写。主要分为页面向服务器请求数据的技术Ajax，服务器向页面推送数据的技术Comet，全双工通信和页面间的通信方案。

#### Ajax（页面->服务器)
[完整的ajax封装](https://www.jianshu.com/p/d644c398be06)

#### Comet(服务器->页面)

##### 实现方式
[自己在csdn上的文章](https://blog.csdn.net/jikexueyuan5555/article/details/79343919)
[大佬的即时通讯node代码](http://luoxia.me/code/2016/10/16/%E6%95%B0%E6%8D%AE%E6%8E%A8%E9%80%81%E4%B8%9A%E5%8A%A1%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88/)

#### Web Sockets（全双工通信）

>特点：

使用自定义协议，连接为ws://和wss://(加密)

>优点:

能够在客户端和服务器之间发送非常少量的数据，而不必担心HTTP那样字节级的开销。

>缺点:

官方制定协议的事件比制定js 的api的时间还要慢，不支持DOM2级语法，所以要用socket.open=function()这种

>readyState值：

0:正在建立连接
1:已经建立连接
2:正在关闭连接
3:已经关闭连接

#### 页面间通信

>localstorage

原理：“当同源页面的某个页面修改了localStorage,其余的同源页面只要注册了storage事件，就会触发”；

标签页1

```HTML
<input id="name">
    <input type="button" id="btn" value="提交">
    <script type="text/javascript">
        var btn = document.getElementById('btn');
        var inpu = document.getElementById('name');
        var name;
        function detectInput(){
            setTimeout(function(){
                name = inpu.value;
                if(!name){
                    detectInput();
                }
            },5000);
        }
        detectInput();
        
        btn.onclick = function(){
            localStorage.setItem("Brelly",name);
        }
```
标签页2

```HTML
<script type="text/javascript">
		window.addEventListener("storage", function(event){  
			console.log(event.key + "=" + event.newValue);  
		});   
</script>

```
测试：需要http-server下两个网页所在目录，不能在本地打开，这样当一个页面的值发生变化时，另一个页面也跟着发生变化

>postMessage

```HTML
//a.com/index.html
<iframe src="b.com/index.html" id="ifr">
</iframe>
<script>
window.onload = function(){
  var iframe = document.getElementById('ifr');
  var targetOrigin = 'http://b.com'; // 若写成'http://b.com/c/proxy.html'效果一样
                                        // 若写成'http://c.com'就不会执行postMessage了
  iframe.contentWindow.postMessage('data to send',targetOrigin);
}
</script>



// b.com/index.html
<script type="text/javascript">
  window.addEventListener('message',function(event){
    // 通过origin属性判断消息来源地址
    if(event.origin == 'http://a.com'){
      console.log(event.data);
      console.log(event.source);
    }
  },false);
</script>
```
> websocket通过服务器广播（以下是犀牛书上的代码）

客户端：
```HTML
<input type="text" id="input" style="width:100%;">
    <script>
        window.onload = function(){
            var nick = prompt("enter your name");
            var input  = document.getElementById('input');
            input.focus();
            var socket = new WebSocket("ws://"+location.host+"/");
            socket.onmessage = function(event){//从服务器获取信息
                var msg = event.data;
                var node = document.createTextNode(msg);
                var div = document.createElement('div');
                div.appendChild(node);
                document.body.insertBefore(div,input);
                input.scrollIntoView();
            }
            //通过WebSocket发送消息给服务器端
            input.onchange = function(){
                var msg = nick+ ":" + input.value;
                socket.send(msg);
                input.value = "";
            }
        };
        
    </script>
```

服务端

```js
var http = require('http');
var ws = require('websocket-server');

var clientui = require('fs').readFileSync("test.html");

var httpserver = new http.Server();
httpserver.on("request",function(request,response){
    if(request.url == '/'){
        response.writeHead(200,{"Content-Type":"text/html"});
        response.write(clientui);
        response.end();
    }
    else{
        response.writeHead(404);
        response.end();
    }
});
//在http服务器上包装一个websocket服务器
var wsserver = ws.createServer({server:httpserver});
wsserver.on("connection",function(socket){
    socket.send("Welcome to the chat room.");
    socket.on("message",function(msg){
        wsserver.broadcast(msg);
    })
})
wsserver.listen(8000);
```

---

## es6

### es6的新特性
[请见](https://blog.csdn.net/u012468376/article/details/54565068)
相对比较全的包括第六变量类型symbol、箭头函数独特之处和解构赋值等）

#### 补充

1、我觉得箭头函数就是用来解决闭包第二个return指向windows的这种情况

2、箭头函数用完即丢感觉有点儿爽。。。。

3、写了一下箭头函数顺便也把this的问题解决了，其实在出现箭头函数问题之前都是用that解决的2333

```js

sth = 456;
var thistest = {
    sth:123,
    dothis:function(){
        return (function(){
            console.log(this.sth);
        })();
    },
    dothat:function(){
        console.log(this.sth);
    },
    doArrow:() => console.log(sth),//箭头函数会默认把所在函数环境的this绑在自己身上
    dothatArrow:function(){
        return (() => console.log(this.sth))();
    }
};
thistest.dothis();//456
thistest.dothat();//123
thistest.doArrow();//456
thistest.dothatArrow();//123
```

### Promise专题

>Promise状态

* pending

* fulfilled

* rejected

resolve便指的是pending->fulfilled || pending->rejected,当状态改变后便定型便是resolved，所以注意Promise是三个状态

>race与all 小尝试(all是并行请求)

```js
// var p1=Promise.resolve(43);
// var p2=Promise.resolve("hello liao");
// var p3=Promise.reject("Oops");
// Promise.race([p1,p2,p3])
// .then( function(msg){
//     console.log(msg);
// });
// Promise.all( [p1,p2,p3])
// .catch( function(err){
//     console.log(err);
// });
// Promise.all( [p1,p2])
// .then( function(msg){
//     console.log(msg);
// })
function getY(x) {
    return new Promise(function(resolve,reject){
        setTimeout( function (){
            resolve(2*x+1);
        },1000);
    });
}
function foo(bar,baz){
    var x=bar*baz;
    return getY(x)
           .then( function(y){
               return [x,y];
           });
}

foo(10,20)
.then( function(msgs){
    var x=msgs[0];
    var y=msgs[1];
    console.log( x,y );
});
```

>值得回味的阿里笔试有关于promise发送串行请求

```js
const timeout = ms => new Promise((resolve, reject) => {
	setTimeout(() => {
		resolve();
	}, ms);
});

const ajax1 = () => timeout(2000).then(() => {
	console.log('1');
	return 1;
});

const ajax2 = () => timeout(1000).then(() => {
	console.log('2');
	return 2;
});

const ajax3 = () => timeout(2000).then(() => {
	console.log('3');
	return 3;
});

const mergePromise = ajaxArray => {
    var data = [];
    var total = Promise.resolve();
    ajaxArray.map((fn) => {
          total = total.then(fn).then(res => {
                   data.push(res);
                   return data;
          });
    });
    return total;
};

mergePromise([ajax1, ajax2, ajax3]).then(data => {
	console.log('done');
	console.log(data); // data 为 [1, 2, 3]
});

// 分别输出
// 1
// 2
// 3
// done
// [1, 2, 3]
```

>promise 发送串行并行请求

```js
/**
 * 创建promise
 * @param {Number} value 
 */
function makePromise (value) {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve(value);
    }, Math.random() * 1000)
  })
}
/**
 * 打印结果
 * @param {Number} value 
 */
function print (value) {
  return value
}

let promises = [1, 3, 4, 5, 6].map((item, index) => {
  return makePromise(item)
});

// 并行执行
Promise.all(promises)
.then(() => {
  console.log('done')
})
.catch(() => {
  console.log('error')
})

// 串行执行
let parallelPromises = promises.reduce(
  (total, currentValue) => total.then(() => currentValue.then(print)),Promise.resolve()
)
//老实讲这个串行的写法不是很懂。。。

parallelPromises
.then(() => {
  // console.log('done')
})
.catch(() => {
  console.log('done')
})
```

[Promise基础手册](http://es6.ruanyifeng.com/#docs/promise)

### generator迭代器

>generator发送串行并行请求与自动化执行

同理也能在es 6入门那书中找到答案(串行并行请求没有233)

```js
function run(fn) {
  var gen = fn();

  function next(err, data) {
    var result = gen.next(data);
    if (result.done) return;
    result.value(next);
  }

  next();
}

function* g() {
  // ...
}

run(g);
```

[Generator函数](http://es6.ruanyifeng.com/#docs/generator)

### async/await


[async/await基础](https://blog.csdn.net/juhaotian/article/details/78934097)

>async/await发送串行并行请求

觉得比较合理的想法是如果发送的是串行请求的话，那么用async,并行的话用Promise.all，至于以前想的async自动化执行的问题，他本来不就是把generator函数和自动执行器包装在一个函数里吗。。

[async函数](http://es6.ruanyifeng.com/#docs/async)

------------

## 代码风格与规范

### 代码风格

#### es6 风格（阮一峰 es6入门）

#### 对象关联风格代替原型链风格 （选自《你不知道的JS》)

>原型风格

```js
function Foo(who){
    this.me=who;
}
Foo.prototype.identyyify = function() {
    return "i'm "+this.me;
}
function Bar(who){
    Foo.call(this,who);
}
Bar.prototype=Object.create(Foo.prototype);
Bar.prototype.speak = function(){
    console.log("hello, "+this.identify()+".");
};
var b1=new Bar("b1");
var b2=new Bar("b2");
b1.speak();
b2.speak();
```

>对象关联风格

```js
Foo={
    init: function(who){
        this.me = who;
    },
    identify: function(){
        return "I am "+this.me;
    }
};
Bar = Object.create(Foo);
Bar.speak = function(){
    console.log("hello, "+this.identify()+".");
};
var b1=Object.create(Bar);
b1.init("b1");
var b2=Object.create(Bar);
b2.init("b2");
b1.speak();
b2.speak();
```

#### jQuery链式风格

#### html与js逻辑分离的风格

#### 能用css完成的动画效果不用js完成

### 代码规范

命名小写pc与phone下划线，css类名中划线，函数单驼峰，React组件双驼峰