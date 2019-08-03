---
title: 趣说shell脚本的pwd、source、$0的连环坑（iOS持续集成续）
date: 2016-07-24 16:00:00
tags: 
     - shell
categories: Tech
keywords: shell
description: 主要讲用shell脚本过程中遇到的一些pwd、source、$0的连环坑
---

# 目录

- 前言
- 踩坑跳坑出坑
  - pwd是获取当前的路径？
  - source惹的祸／$0 也会骗人
  - source/$0/pwd相互影响
  - BASH_SOURCE[0]这又是什么东西？
  - 鲁棒性？用回pwd啦！
  - 又是source的坑！变量命名！
- 小结


# 前言

今天重新写了之前的iOS自动化脚本，并加入自动上传符号表文件到Bugly的功能。结果遇到Shell的几个有趣的“坑”。



简化成以下例子：

> /Users/zack/Desktop/a.sh
> /Users/zack/Desktop/B/b.sh
> /Users/zack/Desktop/B/C/c.sh

有如上面路径结构的三个脚本文件a.sh、b.sh、c.sh。

B目录内的文件是写给别人调用的，b.sh调用c.sh。假设在桌面有个a.sh调用了b.sh。

实际过程写了很多脚本文件比较复杂，其实排查起坑来不像下面讲的那么简单，囧囧囧！



# 踩坑跳坑出坑

接着就开始小小的有趣的踩坑历程啦。




## pwd是获取当前的路径？

首先是a.sh调用b.sh

```
# a.sh
/Users/zack/Desktop/B/b.sh
```

b.sh调用c.sh

```
# b.sh
C/c.sh
```

c.sh随便干点什么

```
# c.sh
echo "I'm c.sh."
```

看起来挺正常的是吧？

> /Users/zack/Desktop/B/b.sh: line 1: C/c.sh: No such file or directory

找不到c.sh？为什么？b.sh不是和C目录同一级吗？

嗯想想......应该和被a.sh调用有关系。我是给别人用的工具，那么我并不知道a.sh会在哪调用我的b.sh。好吧，那我加个绝对路径。

```
# b.sh
｀pwd｀/C/c.sh
```

又有问题？对，你猜的没错。

> /Users/zack/Desktop/B/b.sh: line 1: /Users/zack/Desktop/C/c.sh: No such file or directory

好吧，为什么是`/Users/zack/Desktop/C/c.sh`这个路径呢？打印一下`｀pwd｀`，是`/Users/zack/Desktop`。为虾米？这个不是获取当前工作目录吗？

噢我一定是撞鬼了！不，我是唯物主义者！再想想......这个是a.sh的路径，也就是说这个还是跟调用者有关（后文会解释shell进程相关的原因）。

网上度娘说用这个`dirname`取`$0`

```
# b.sh
`dirname $0`/C/c.sh
```

看结果：

> I'm c.sh.

哇真的可以，感谢度娘！

`dirname`用于取给定路径的目录部分,`$0`是Shell本身的文件名。

继续写代码。



## source惹的祸／$0 也会骗人

发现用source也可以调用其他shell脚本文件。试试。

```
# b.sh
source `dirname $0`/C/c.sh
```

执行一下./a.sh。嗯也是可以的。然后改改c.sh。

```
# c.sh
echo "I'm c.sh."
echo `dirname $0`
```

这时候第二行打印应该是c.sh所在的路径，对吧？

不，你错了！！！哈哈哈！

> I'm c.sh.
> /Users/zack/Desktop/B

为什么是B目录？不是应该`/Users/zack/Desktop/C`吗？我书读得少你不要骗我！！！

打印下`$0`。

```
# c.sh
echo "I'm c.sh."
echo `dirname $0`
echo $0
```

下面是b.sh用source之后的：

> I'm c.sh.
> /Users/zack/Desktop/B
> /Users/zack/Desktop/B/b.sh

下面是b.sh直接用路径之后的：

> I'm c.sh.
> /Users/zack/Desktop/B/C
> /Users/zack/Desktop/B/C/c.sh

哦哦哦，原来是你，source！终于把你揪出来了！用了source之后会导致`$0`不同。

下面是考试重点，记住了！！

> source命令，不再产生新的shell，而是在当前shell下执行一切命令。
>
> source FileName，作用：在当前bash环境下读取并执行FileName中的命令。
>
> source在本shell中执行的，所以能够看到结果
>
> 调用绝对路径执行shell是在一个子shell里运行的，所以执行后，结果并没有反应到父shell里。

嗯明白了！source使用的脚本是在当前shell进程下进行的，所以`$0`依旧是当前的b.sh。那么之前的`pwd`应该是当前shell进程的工作目录，这个可以理解了。在a.sh调用b.sh，a.sh没有执行任何`cd dir`操作，b.sh也没有，所以`pwd`仍然是a.sh所在目录。



## source/$0/pwd相互影响

做个实验：

```
# b.sh
echo `pwd`
source `dirname $0`/C/c.sh
echo `pwd`
```

```
# c.sh
cd ..
echo "I'm c.sh."
```

猜猜这时候b.sh打印什么？

> /Users/zack/Desktop
> I'm c.sh.
> /Users/zack

如果b.sh再改成这样呢？

```
# b.sh
echo `pwd`
`dirname $0`/C/c.sh
echo `pwd`
```

> /Users/zack/Desktop
> I'm c.sh.
> /Users/zack/Desktop

哈哈很有趣，这样可以很容易很真切理解这几个东西的作用和影响了！



## BASH_SOURCE[0]这又是什么东西？

 上面的问题还是没有解决。基友丢来一行代码。

```
echo "$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"
```

用BASH_SOURCE[0]来获取当前脚本的路径。

> 与FUNCNAME相似的另外一个比较有用的常量是BASH_SOURCE，同样是一个数组，不过它的第一个元素是当前脚本的名称。
> 这在source的时候非常有用，因为在被source的脚本中，$0是父脚本的名称，而不是被source的脚本名称。而BASH_SOURCE就可以派上用场了。
> 唯一遗憾的是，这种做法会让脚本失去一些可移植性，因为不是所有的shell都支持这些常量。

试下试下：

```
# b.sh
`dirname $0`/C/c.sh
```

```
# c.sh
echo "I'm c.sh."
echo `dirname ${BASH_SOURCE[0]}`
```

结果如下，哐哐哐！

> I'm c.sh.
> /Users/zack/Desktop/B/C

b.sh再改成source c.sh的。

```
# b.sh
source `dirname $0`/C/c.sh
```

> I'm c.sh.
> /Users/zack/Desktop/B/C

Good的！哈哈哈！好高兴解决了！



## 鲁棒性？用回pwd啦！

从**source/$0/pwd相互影响**这节的实验可以看到，b.sh使用source c.sh时，在c.sh里面cd dir，会影响外面的pwd；而直接调用c.sh的话，即使cd dir，回到b.sh依旧不影响pwd。

我怎么尽可能保证鲁棒性呢？因为我不知道别人是怎么调用我的。设计得好的话，作为被调用者，应该只产生脚本本身的作用，而不影响到其他东西，比如调用者的工作目录路径。

从另一个角度，我作为调用者的话，怎么防止被调用者影响到我的工作目录路径呢？因为我们经常用pwd来做些操作的。

解决大概如下：

1. 作为调用者，尽可能使用绝对路径或相对路径调用其他脚本文件，不用source；或者在使用其他脚本文件前保存路径`pwd=｀pwd｀`，调用完后再`cd pwd`回去。
2. 作为被调用者，同上，在开始前保存pwd文件，执行完任务后再cd回去。



## 又是source的坑！变量命名！

呼呼差不多了，再加点东西。

```
# b.sh
myPath=`dirname ${BASH_SOURCE[0]}`
echo $myPath
source ${myPath}/C/c.sh
echo $myPath
```

```
# c.sh
myPath=`pwd`
echo "I'm c.sh."
cd $myPath
```

这里然后c.sh保存了当前路径，最后返回当前路径。b.sh保存了一个`myPath`，调用b.sh前后各打印一次。你猜打印结果一样吗？？

> /Users/zack/Desktop/B
> I'm c.sh.
> /Users/zack/Desktop

没道理啊，为什么呢？c.sh应该是不会影响到路径的啊！这个做过试验了的。

猜不到吧，因为c.sh的myPath覆盖了b.sh......

因为source c.sh之后，c.sh和b.sh在同一个shell进程下运行，变量也公用，所以......

怎么解决呢？只能是通过命名空间或者命名规则了吧！



# 小结

呼呼至此结束了！

shell脚本挺有趣的，虽然有时处理些东西麻烦或者不够规范，比如获取脚本参数、脚本只能返回整型等。晚了，关于pwd、source、$0等等相关更细的知识就不介绍了，度娘知道。



自动化编译脚本也重写得很优雅很美了！类似下面这样：

```
function archive() {
}
function exportArchive() {
}
...

archive
exportArchive
publishToFirIfNeed
submitAppStoreIfNeed
sendEmail
```

所有变量什么的都放在函数里面，每个函数再分别调用其他脚本文件。打包、包导出、上传fir、上传appStore、发邮件都分到单独的脚本文件，每个可以单独用；整个根调用文件看起来整洁清晰，步骤也很明显，做的事情就是最下面那5个调用。

后面会补到之前写的文章中：
简书：[详解Shell脚本实现iOS自动化编译打包提交](http://www.jianshu.com/p/bd4c22952e01) 
个人博客：[详解Shell脚本实现iOS自动化编译打包提交](http://zackzheng.info/2015/12/27/2015-12-27-an-automated-script-for-building-archiving-submission-sending-emails)



让编程一直成为一种有趣的爱好！



-END-
欢迎到我的博客交流：https://zackzheng.info