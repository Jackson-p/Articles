# JS实现倒计时

## 青铜

倒计时大家可能觉得很容易

```js
var total = 60000;
var interval = 1000;
var t = setInterval(function(){
    total -= interval;
    if(!total){
        clearInterval(t);
        return;
    }
    console.log('剩余时间为' + total/1000 + 's');
},interval)
```

可通常setInterval有很多弊端

1.setInterval对自己调用的代码是否报错漠不关心。即使调用的代码报错了，它依然会持续的调用下去 

2.setInterval无视网络延迟。在使用ajax轮询服务器是否有新数据时，必定会有一些人会使用setInterval，然而无论网络状况如何，它都会去一遍又一遍的发送请求，如果网络状况不良，一个请求发出，还没有返回结果，它会坚持不懈的继续发送请求，最后导致的结果就是请求堆积。 
3.setInterval并不定时。如果它调用的代码执行的时间小于定时的时间，它会跳过调用，这就导致无法按照你需要的执行次数或无法得到你想要的结果。


好。。。那我用setTimeout匀速触发。。。。

```js
var total = 60000;
var interval = 1000;
var t = setTimeout(countdown,interval);
function countdown(){
    total -= interval;
    console.log("剩余时间为" + total/1000 + '秒');
    if(!total){
        clearTimeout(t);
        return;
    }
    t = setTimeout(countdown,interval);
}
```


但是写完代码，我们发现其实时间稍久误差就蛮大的了，原因无非是首先浏览器在计算这个1s的时候就可能会有偏差，如果主进程或者渲染进程占用的时间会久了话就更难控制在1s之内了。。。所以上述写法有点欠缺orz

## 黄金

所以针对以上情况，我们可以在倒计时的过程中加入“偏差纠正”，这样的话就可以不断调整1s之内的误差，可以解决平时比较基础的情况

```js
var total = 60000;
var interval = 1000;
var t;
var cnt = 0; //用数学计算有多少周期来求得严格意义上的时间
var begin_time = new Date().getTime();
if(total > 0){
    t = setTimeout(startCountDown,interval);
}
function startCountDown(){
    cnt++;
    var now_time = new Date().getTime();
    var offset = now_time - (begin_time + cnt * interval);
    var rest_time;
    offset = offset?offset:0;//正常来说有延迟会导致get出来的time要比准确时间大的。。
    total -= interval;// 我们只是会矫正使总体保持1次执行/s
    console.log(`时间偏差为${offset}` + '剩余时间:' + total/interval + '秒' );
    rest_time = interval - offset > 0 ? interval - offset : 0;//如果延迟太久甚至超越了单次周期就立刻立即执行吧

    if(total <= 0){
        clearTimeout(t);
        return;
    }else{
        t = setTimeout(startCountDown,rest_time);
    }

}
```


## 铂金

所以真正意义上的矫正，还是前端从后台提取到标准的时间，以达成同步？也就是传说中的用ajax来提取，但是这中间造成的网络延迟又怎么算orz。

## 钻石

就算解决了，可是上述方法还是会存在问题，因为在浏览器中待使用的tab标签页优先级会比较低，也就是说我们在切换tab页的时候，计时有可能会延误很大（运行在后台的tab页，定时器延迟毫秒可能设置>=1000ms,oh,IE不存在这个问题hhh)

上个方法做了tab切换测试，果然延迟提高。。。

<img width="179" alt="2018-11-13 9 34 37" src="https://user-images.githubusercontent.com/20417123/48492903-cc327580-e865-11e8-9aed-cf3afb6501ae.png">


所以可使用visibilitychange 事件来处理切换 tab 页以及浏览器最小化时的倒计时误差修正

```js
// 处理页面可见属性的改变
document.addEventListener('visibilityChange', () => {
    if (!document.hidden) {
      // get newest downtime
    }
});
```

## 王者?

[Web Worker](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Workers_API/Using_web_workers)

注意下Chrome从本地文件运行脚本时不允许加载web worker.

所以用python3 搭了下服务器，这个其实还蛮简单的,后来觉得有点儿傻了，直接用node.js本家的http-server就好了。。。。

到指定目录下python3 -m http.server 3000

然后访问服务器下的文件就可以看到神奇的Worker工作了（话说这个数据通信的过程怎么和Electron那么像。。。）

```HTML
<!DOCTYPE html>
<html lang="en">

<head>
	<meta charset="UTF-8">
	<meta name="viewport" content="width=device-width, initial-scale=1.0">
	<meta http-equiv="X-UA-Compatible" content="ie=edge">
	<title>Worker-倒计时</title>
	<style>
		#container {
			text-align: center;
		}

		#beginbtn {
			padding: 5px;
			background-color: cornsilk;
			width: 100px;
			font-weight: bold;
			font-size: 16px;
			outline: none;
		}
	</style>
</head>

<body>
	<div id="container">
		<h1>liaoliao</h1>
		<button id="beginbtn">计时开始</button>
	</div>
	<script>
		var beginbtn = document.getElementById('beginbtn');
		var content = document.getElementsByTagName('h1')[0];
		beginbtn.onclick = function () {
			if (window.Worker) {
				console.log("Worker works");
				var cntdown = new Worker('http://0.0.0.0:3000/test.js');
				cntdown.onmessage = function (e) {
					var cnt_time = e.data;
					content.innerHTML = cnt_time;
				}
				beginbtn.disabled = true;
			} else {
				console.log('err');
			}
		}
	</script>
</body>

</html>
```

然后是Worker.js

```js
var total = 60000;
var interval = 1000;
var t = setInterval(function(){
    total -= interval;
    postMessage(total/1000);
    if(t <= 0){
        clearInterval(t);
        return;
    }
},1000)
```

PS:段位就当是瞎说的了，大佬觉得有问题欢迎批评指正，但求别打orz