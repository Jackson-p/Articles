# 移动端web调试方法

在很多时候，通过chrome浏览器进行调试和时候无论是样式还是功能都会与真机有一定出入，所以移动端调试web页面比较重要，记录三种方法先。

## 连线

方法如名，直接将手机和电脑相连再通过chrome的dev调试。大牛总结很多了，这里贴个链接。
[机连调试方法](https://segmentfault.com/a/1190000005964730)

## 移动端访问ip

先在电脑上启动本地应用，然后ifconfig（mac命令）查看ip，假如是在本地localhost:8000启的应用，直接在手机浏览器里输入ip:8000就可以了。缺点就是如果某些app需要端内调试，以ip开头的地址不一定会被识别。

## charles代理

这个个人觉得是最好用的办法了，先打开charles，手机连wifi开启手动代理模式，输入电脑ip和设定的某个端口。然后通过charles就可以抓住手机访问各种网页的各种请求了(比如如果没加密的话也可以抓cookie哦，并且可以把cookie种到浏览器里面，这样浏览器还能模拟移动端的登录状态2333），但是手机上不能输入localhost来直接访问，因为手机上的localhost是手机本身。。。。。解决方案如下～

```shell
sudo vim /etc/hosts
```

加入一行：
127.0.0.1             想要的测试地址名

生效后就可以了