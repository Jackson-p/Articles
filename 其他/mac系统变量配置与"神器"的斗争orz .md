# 好说一下对mac系统变量的理解

## 起因

我接触到这个是因为有个烂大街的安装，我们在mac上安装好了mysql之后往往都有一个

```bash
vim ~/.bash_profile
PATH=$PATH:/usr/local/mysql/bin
source ~/.bash_profile
```

的过程，我们在终端里输入了这些命令，然后生效环境变量发现就可以用mysql进行操作了，但是当我们把终端关掉，再重新打开我们输入mysql命令无效，刚刚设置的系统变量可以说。。。。不见了。。。哦对了 补充一下我的命令行还是oh my zsh

## 经过

先，给自己科普一下shell和bash：百度知道上的三句话

* bash 是一个为GNU项目编写的Unix shell，也就是linux用的shell
* Shell俗称壳（用来区别于内核），是指“提供使用者使用界面”的软件，就是一个命令行解释器。
* BASH是SHELL的一种，是大多数LINUX发行版默认的SHELL，除BASH SHELL外还有C SHELL等其它类型的SHELL

再，科普一下自己弄过程中一些小点

* echo $PATH // 打印当前情况下可用的环境变量
* bad assignment的警告是因为bash中 = 的前后加了空格

我探索了下原因

一开始我以为是因为他是个用户级别的系统变量所以随着终端结束就被kill掉了，后来我发现好像是因为zsh的缘故导致其环境变量无法起作用

总结：

Mac配置环境变量的地方
 

1,  /etc/profile   （建议不修改这个文件  全局（公有）配置，不管是哪个用户，登录时都会读取该文件。

2,  /etc/bashrc    （一般在这个文件中添加系统级环境变量） 全局（公有）配置，bash shell执行时，不管是何种方式，都会读取此文件。

注意：前两个我等人类还是干脆不要动的好。。。

3,  ~/.bash_profile  （一般在这个文件中添加用户级环境变量） 每个用户都可使用该文件输入专用于自己使用的shell信息,当用户登录时,该文件仅仅执行一次!

4, 但是当mac机器上安装了zsh后   .bash_profile 文件中的环境变量就无法起到作用。
    接下来的解决方案：

* cd ~
* open .zshrc 
* 在.zshrc文件末尾增加.bash_profile的引用：

* export PATH=${PATH}:/usr/local/mysql/bin

* source ~/.bash_profile

5，以下是环境变量的配置Demo

```bash
vi ~/.bash_profile:

    export ANDROID_HOME=/Users/xxxx/software/sdk
    export NDK=/Users/xxxx/software/android-ndk-r10d
    export GRADLE_HOME=/Users/xxxx/software/gradle-2.4
    export SUBLIME=/Users/xxxx/home/subin
    export PATH=$ANDROID_HOME/tools:$ANDROID_HOME/platform-tools:$GRADLE_HOME/bin:$PATH
    export PATH=$PATH:$SUBLIME:/usr/local/mysql/bin
    export PATH=$PATH:/Users/xxxx/software/apache-tomcat-7.0.70/bin


    错误：-bash: ./startup.sh: Permission denied
```

解决办法：

用命令chmod 修改一下Tomcat的bin目录下的.sh权限就可以了
如chmod u+x *.sh
在此执行，OK了。



## 结果

我现在已经完成了python3（而且设了默认版本,是用brew install直接安装的）、mysql的配置
bash_profile如下

```bash
# set path for mysql
export PATH=$PATH:/usr/local/mysql/bin
export PATH=$PATH:/usr/local/bin // 这句后来echo的时候发现不加也可以。。
alias python=/usr/local/bin/python3

```

每每我运行他的时候，都可以为我创建一个临时的环境，当终端关闭时环境失效，于是我在zsh.rc中运行了上面的这些bash脚本。发现mysql已经在系统变量里不会临时消失

so 我先记录一下我的$PATH的初始状态，方便我为所欲为（也就是删除某一系统级变量）

> /usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin

注：这个是安装了brew之后才出现的sbin

嗯，删除系统变量果然直接到bash_profile里删（临时有效，可以先试试水），真的删的话就去zsh里面直接改就可以额。

所以到这里可能就明白了其实整个过程是zsh给mac加了层试水机制，用户可以在bash_profile里先改改试试，但是核心的永久修改path的权力是到了zsh的手里，zsh变成主臣，bash_profile只是个实验执行者。。
最后自己在.zshrc中配置的python和mysql，可以默认指向python为python3的

```bash
# My settings
export PATH=$PATH:/usr/local/mysql/bin
alias python=/usr/local/bin/python3
alias python2=/usr/bin/python
```

## 升华

我。思考一下为什么有了zsh就会使bash_profile失效了呢？还是说本来他就会失效呢？这个可以问问常皮orz
然后可以试着搞一下zsh

## 画外音

用homebrew的brew install python3之后，我发现了在usr/local/bin底下也有一个python3，当时懵了，以为我以前安过，后来才知道这是因为brew在自己的目录下安装好python后，在相应的目录下（’/usr/local/bin’）创建了python的软连接（即快捷方式）。有大佬说自己用homebrew安装新的python2，但是这样的话pip2的优先级也是需要改的，，，这样我倒是觉得有点儿麻烦。当下觉得还是主默认python3 额外只多安一个包的好，毕竟homebrew也只是个大包管理器，不用太神化