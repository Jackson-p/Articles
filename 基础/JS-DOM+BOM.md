# Js-DOM+BOM

__前言:__

本来这只是一篇js DOM编程艺术的读书笔记，但是看到了一种分类方法是javascript由ECMAScript、DOM、BOM组成，觉得很合理，重视一下DOM和BOM问题，尤其是事件那一部分是通过DOM这个API来实现的

* BOM与DOM

```txt
BOM：浏览器对象模型，Browser Object Model。window.open,window.blur等；BOM会因浏览器的内部实现不同而不同
DOM：把文档当成一棵树模型，将HTML节点转成js的变量；W3C的标准所有浏览器公共遵守的标准
```

[参考](https://blog.csdn.net/anythings/article/details/51240133)

* DOM文档加载顺序

```txt
(1) 解析HTML结构。
(2) 加载外部脚本和样式表文件。
(3) 解析并执行脚本代码。
(4) 构造HTML DOM模型。//ready
(5) 加载图片等外部文件。
(6) 页面加载完毕。//load
```

* DOM 组成

```txt
元素节点，文本节点，属性节点,css层叠样式表
元素节点：p
文本节点：bulaililiaoliao
属性节点：title = "sth test"
```

* window.onload技巧

```js
假设有两个函数，firstFunction与secondFunction都在页面加载时执行，如果写作：
window.onload=firstFunction;
window.onload=secondFunction;
只会加载第二个函数指令，所以往往我们可以用匿名函数的小技巧绕过这个问题
window.onload=function() {
    firstFunction();
    secondFunction();
}
实际生产中我们可以封装如下：
function addLoadEvent(func) {
    var oldonload = window.onload;
    if(typeof(oldonload) != "function") {
        window.onload = func;
    }else {
        window.onload = function() {
            oldonload();
            func();
        }
    }
}
这样完成同样的效果只需：
addLoadEvent(firstFunction);
addLoadEvent(secondFunction);
```

* onkeypress与onclick

```txt
用tab切换光标到某链接上，按回车也会触发onclick事件，总体上说，附加onkeypress引来的问题比解决的问题多。
```

* DOM Core编写代码会简化些

```js
var source = whichpic.getAttribute("href");
var source = whichpic.href;
```

* DOM的一些操作

>属性：childNodes,parentNode,nextSibling
>方法：appendChild(),insertBefore()
>nodeType共有12种属性值，其中的3种有实用价值

```txt
1、元素节点
2、属性节点
3、文本节点（用nodeValue改变文本节点的值）
```

* DOM与CSS结构：

```txt
通常页面结构分为三层：
1、HTML搭建的结构层
2、CSS渲染的表示层
3、js和DOM主宰的的行为层
```

* DOM操作CSS须知：

```txt
DOM的style属性只能返回那些内嵌在HTML里的样式信息，不能用来检索在外部CSS文件里声明的样式,然而我们一般不会使用内联的方法去使用CSS样式，那么DOM会处理怎么样的CSS样式呢？
```

* DOM操作表示层场景：

```txt
1、根据元素之间的相对位置找出或修饰某特定元素
2、解决浏览器对CSS的兼容问题（感觉2018年的现在CSS支持很好了）
注：使用DOM该改变CSS可以考虑用自己用过的JS改变className如addClass
```

>addClass函数封装

```js
function addClass(element,value) {
    if(!element.className) {
        element.className = value;
    } else {
        element.className = element.className + " "+value;
    }
}
//DOM编程艺术上的代码感觉不太好，新引进变量，还是全局变量orz。。。
function addClass(element,value) {
    if(!element.className) {
        element.className = value;
    } else {
        newClassName = element.className;
        newClassName+=" ";
        newClassName+= value;
        element.className = newClassName;
    }
}
```

* Ajax使用：循序渐进开发原则，先完成网站的整体层次和核心功能，再附加Ajax层作为优化，也就是传说中的（Hijax).

>Ajax 基础代码解析

```js
function getHTTPObject() {
    //IE
    if(window.ActiveXObject) {
        var waystation = new ActiveXObect("Microsoft.XMLHTTP");
    }else if(window.XMLHttpRequest) {
        var waystation = new XMLHttpRequest();
    }else {
        var waystation = false;
    }
    return waystation;
}

request = getHTTPObject();
request.open("GET","example.txt", true);
request.onreadystatechange = doSomething;
request.send(null);
/*
readyState属性值：
0: 尚未初始化
1: 正在加载
2: 加载完毕
3: 正在处理
4: 处理完毕
*/
function doSomething() {
    if(request.readyState == 4) {
        alert(request.responseText);
    }
}
```

接下来补充两个DOM操作CSS的实例
>书上好实例1:DOM操作表格实现隔行上色

```HTML
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <script>
        function stripeTables() {
            if(!document.getElementsByTagName) return false;
            var tables = document.getElementsByTagName('table');
            for(var i=0;i<tables.length;i++) {
                var odd = false;
                var rows = tables[i].getElementsByTagName('tr');
                for(var j=0;j<rows.length;j++) {
                    if(odd == true) {
                        rows[j].style.backgroundColor = "#ffc";//addClass(rows[j],"odd");
                        odd = false;
                    } else {
                        odd = true;
                    }
                }
            }
        }
    </script>
</head>
<body>
    <table>
        <caption>Itinerary</caption>
        <thead>
            <tr>
                <th>when</th>
                <th>where</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td>June 9th</td>
                <td>Portland,<abbr title="Oregon">oR</abbr></td>
            </tr>
            <tr>
                <td>June 10th</td>
                <td>Seattle,<abbr title="Washington">WA</abbr></td>
            </tr>
            <tr>
                <td>June 12th</td>
                <td>Sacramento,<abbr title="California">CA</abbr></td>
            </tr>
        </tbody>
    </table>
    <button id="btn" onclick = "stripeTables()">change</button>
</body>
</html>

```

>书上好实例2:DOM改变相邻第一个样式

```HTML
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
  <script>
      function getNextElement(node) {
        if(node.nodeType == 1 ) {
          return node;
        }
        if(node.nextSibling) {
          return getNextElement(node.nextSibling);
        }
        return null;
      }
      function styleHeaderSiblings() {
          if(!document.getElementsByTagName) return false;
          var headers = document.getElementsByTagName('h1');
          for(var i=0;i<headers.length;i++) {
            var elem = getNextElement(headers[i].nextSibling);
            elem.style.fontWeight = "bold";
            elem.style.fontSize = "1.2em";
          }
      }
    </script>
</head>

<body>
        <button id="btn" onclick="styleHeaderSiblings()">change</button>
        <h1>Hold the front page</h1>
        <p>This first paragraph leads you in</p>
        <p>now you get the nitty</p>
        <p>the most important information</p>
        <h1>Extral! Extral!</h1>
        <p>Further developments are unfolding</p>
        <p>you can read all about it</p>
</body>
</html>
```

>书上好实例3:DOM完成动画效果（感觉用CSS3可以很轻松地完成）

```HTML
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <style>
        #slideshow {
            width:100px;
            height:100px;
            position: relative;
            /* overflow: hidden; */
        }
        #preview {
            position: absolute;
        }
    </style>
    <script>
        function insertAfter(newElement,targetElement) {
            var parent = targetElement.parentNode;
            if(parent.lastChild == targetElement) {
                parent.appendChild(newElement);
            } else {
                parent.insertBefore(newElement,targetElement.nextSibling);
            }
        }
        function moveElement(elementID, final_x, final_y, interval) {
            if(!document.getElementById) return false;
            if(!document.getElementById(elementID)) return false;
            var elem = document.getElementById(elementID);
            if(elem.movement) {
                clearTimeout(elem.movement);
            }
            if(!elem.style.left) {
                elem.style.left = "0px";
            }
            if(!elem.style.top) {
                elem.style.top = "0px";
            }
            var xpos = parseInt(elem.style.left);
            var ypos = parseInt(elem.style.top);
            if(xpos == final_x && ypos ==final_y) {
                return true;
            }
            if(xpos < final_x) {
                var dist = Math.ceil((final_x - xpos)/10);
                xpos = xpos + dist;
            }
            if(xpos > final_x) {
                var dist = Math.ceil((xpos - final_x)/10);
                xpos = xpos - dist;
            }
            if(ypos < final_y) {
                var dist = Math.ceil((final_y - ypos)/10);
                ypos = ypos + dist;
            }
            if(ypos >final_y) {
                var dist = Math.ceil((ypos - final_y)/10);
                ypos = ypos - dist;
            }
            elem.style.left = xpos + "px";
            elem.style.right = ypos + "px";
            elem.movement = setTimeout(moveElement(elementID,final_x,final_y,interval),interval);
        }
        function prepareSlideshow() {
            if(!document.getElementsByTagName) return false;
            if(!document.getElementById) return false;
            if(!document.getElementById('linklist')) return false;
            var slideshow = document.createElement('div');
            slideshow.setAttribute("id","slideshow");
            var preview = document.createElement("img");
            preview.setAttribute("src","1.jpeg");
            preview.setAttribute("alt","building blocks of web design");
            preview.setAttribute("id","preview");
            slideshow.appendChild(preview);
            var list = document.getElementById("linklist");
            insertAfter(slideshow,list);
            var links = list.getElementsByTagName('a');
            links[0].onmouseover = function() {
                moveElement("preview",100,0,10);
            }
            links[1].onmouseover = function() {
                moveElement("preview",200,0,10);
            }
            links[2].onmouseover = function() {
                moveElement("preview",300,0,10);
            }
        }
    </script>

</head>

<body>
<h1>Web Design</h1>
<p>These are the things you should know</p>
<ol id="linklist">
    <li>
        <a href="structure.html">Structure</a>
    </li>
    <li>
        <a href="presentation.html">Presentation</a>
    </li>
    <li>
        <a href="#">Behavior</a>
    </li>
</ol>
<button onclick="prepareSlideshow()">change</button>
</body>

</html>
```

>常见dom操作集锦(现在写的只是样例)

```js

    （1）创建新节点
    createDocumentFragment()    //创建一个DOM片段
    createElement()   //创建一个具体的元素
    createTextNode()   //创建一个文本节点
  （2）添加、移除、替换、插入
    appendChild()
    removeChild()
    replaceChild()
    insertBefore() //在已有的子节点前插入一个新的子节点
  （3）查找
    getElementsByTagName()    //通过标签名称
    getElementsByName()    //通过元素的Name属性的值(IE容错能力较强，会得到一个数组，其中包括id等于name值的)
    getElementById()    //通过元素Id，唯一性
```