---
layout: post
title: "lua一份模板创建多个对象"
date: 2015-11-15 20:48:08 +0800
comments: true
categories: 
---
上一章，我们说过reuqire其实就是将一份已经加载过得数据返回，因此多次加载一份文件，其实就是一个数据，而不是多个。
那么如何实现多个呢？

<!--more-->
只要我们脑洞大开，就可以发现其实只要通过setmetatable就能够巧妙地实现这个问题。
只要我们将require进来的数据作为另一个表数据的元数据，是不是就算是已经创建了一个新的对象呢？比如

```
a = require("A")
b = {}
setmetatable(b,{__index =  a})
```
 此时我们require了A，创建了模块A的数据并把它作为b的元表，而B是一个非常干净的table，它是否就相当于a的一个对象呢？
 想想这样和其他语言的创建对象是否如出一辙呢？如此一来，我们就可以多次创建。比如

```
c = {}
c.x = 10
c.y = 100
setmetatable(c,{__index =  a})
d = {}
setmetatable(d,{__index =  a})
```

看，这样b,c,d都是以a为元表。每个表都可以有自己的数据并不影响彼此（只要元表不改变）。
A.lua 的结构如下：
```
local A = {}
A.number1 = 1
A.number2 = 1
function A.doAction()
	print(string.format("doAction:%d",(A.number1+A.number2)))
end 
function A.moveToPosition(x,y)
	print(string.format("we want to move to %d,%d",x,y))
end 
return A
```

但是这个A里面怎么操作我们自己的数据呢？比如我们想在`doAction`中调用b自己的数据,但是你看这个函数里面定义的是将A的number1和number2进行了加法。如果我们的b也有这两个变量，会不会调用的就是b的变量呢？

```
a = require("A")
b = {}
b.number1 = 10
b.number2 = 10
setmetatable(b,{__index =  a})
b.doAction()
```
但是打印的却是

```
doAction:2
```
这说明最终使用的还是a的变量，怎么才能让它调用b自身的变量呢？

简单地方法就是将b作为参数传给doAction。

```
function A.doAction(tableData)
	print(string.format("doAction:%d",(tableData.number1+tableData.number2)))
end 
```
如此当b调用doAction的时候，就需要`b.doAction(b)`这样子！这非常奇怪，但也能够容忍。
有没有别的方法呢？
是的，有。lua提供另外一种语法来解决这个问题。可以讲调用者自己传到函数中做第一个参数。

```
function A:doAction()
	print(string.format("doAction:%d",(self.number1+self.number2)))
end 
```
仔细看，这个函数和上面的定义最大的区别就是用了冒号。如此一来，在函数里面就可以用self来指示函数的调用者。
因此，我们的调用方式就变成了`b:doAction()`即可！

讲到此处，你是否对lua有一个整体的感受了。如果不懂也没有关系，报错也没有关系，请下载我上一篇文章推荐的zerobrain studio编辑器。通过断点调试一点点的尝试，你就能够很快明白的！



