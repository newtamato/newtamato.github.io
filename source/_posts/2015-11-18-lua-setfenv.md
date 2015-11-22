---
layout: post
title: "setfenv有何用？"
date: 2015-11-18 00:39:31 +0800
comments: true
categories: lua
---
上一章，我们提到了模块里面的函数，使用点语法的方式定义函数和变量。点之前的名字类似一个命名空间。比如`function A.function() end`,此处A就相当于一个命名空间，那么我们能不能有更好地方法不用写这个命名空间呢？当然是可以的。lua提供这样的方式。

<!--more-->

这个方法涉及到一个核心函数`setfenv()`, 我们看一下这个函数的API：
setfenv(f, table)：设置一个函数的环境
1. 当第一个参数为一个函数时，表示设置该函数的环境
2. 当第一个参数为一个数字时，为1代表当前函数，2代表调用自己的函数，3代表调用自己的函数的函数，以此类推

看第一条，他说到可以设置函数的环境。如果我们不调用setfenv，那么函数的环境是什么呢？其实就是默认的_G.所有已经加载了函数变量模块都会被注册到_G中，因此我们可以在B.lua文件中直接调用A.lua中定义的USER_NAME就不会报错。

但是如果我们调用了setfenv()，会发生什么事情？我们来试试。
A.lua

```
USER_NAME = "jolie"
ITEM_NAME = "abc"
ITEM_COUNT = 123
```

调用函数：

```
require("A")
setfenv(1,{})
tostring(USER_NAME)
print("i want to print user name")
```

结果会是什么样子的呢？
结果就是tostring，print无法被调用，咦？这是怎么回事呢？其实很简单，原来tostring和print很早就被注册到了_G中，我们把函数环境设成了空表，于是再也找不到了tostring和print了。这就报错了！

因此既然我们用得到print和tostring，我们就把这两个函数保存起来，传递给我们当前的函数环境。

```
setfenv(1,{print = _G["print"],tostring = _G["tostring"]})
```

再次运行，发现了没有，没有报错。正确打印了。
但是这样会不会因此影响到其他的模块呢？比如有一个B.lua，他是否能够正确执行？

A.lua

```
local A = {print = _G["print"],tostring = _G["tostring"]}
setfenv(1,A)
-- setfenv(1,{})
name = "hello,A"
function doA( ... )
	print("i am A, i call doA function")
end

function doA2( ... )
	print("i am doA2, i call doA2 function")
end

return A
```
B.lua

```
local B = {}
function B.doAction()
	local a = {}
	table.insert(a,1)
	table.insert(a,2)
	table.insert(a,3)
	for k,v in pairs(a) do
		print(k,v)
	end
end
return B
```

调用代码：

```
a = require("A")
a.doA()
a.doA2()
print("i want to print user name , "..a.name)
b = require("B")
b.doAction()
```

看见打印了么？没错！我们设置了A的函数环境，但是并没有因此而影响到B和调用的地方。这说明setfenv只是对当前的模块函数环境进行了设置，而并没有影响到其他。这就是setfenv第一个参数的作用，它指定了当前函数，而不是全局函数，因此只对A这个当前的函数模块进行了设置。

setfenv()还有什么特别的用处？敬请等待下回分析！


