---
layout: post
title: "Serpent：lua序列化和pretty的输出"
date: 2015-11-15 21:32:34 +0800
comments: true
categories: Serpent
---
![Alt text](/images/1447594435836.png)

翻译自http://notebook.kulchenko.com/programming/serpent-lua-serializer-pretty-printer
本人翻译能力有限，如有不妥之处还望指正。

为ZeroBrane Studio我一直开发Lua debugger。我意识到我需要一个table的序列化打印浮躁的结构，例如堆栈和一些在调试器组件使用的表数据。Serpent就是我开发出来的用于lua序列化和pretty的输出。
<!--more-->

因为动态变量的要求lua本身几乎不提供内置的方法去序列化table。那样需要被一个函数去处理（我们在this thread讨论过）。我第一步停在了TableSerializaiton 页面上，在处理table序列化方面它提供了几个不错的信息并且带有不少的例子。

看了已有的模块，我发现没有个哪一个满足我的需求
1. 需要是单纯的lua（这条需求排除了类似PlutoLibrary的库）
2. 需要既能够漂亮的输出，又能够强大的序列化。（这条排除了serialize.lua）
3. 需要能够打印共享的handler和自我引用（这条排除了许多的实现）
4. 序列化不同类型的key，包括以table做key（这条排除了pretty.lua）
5. 需要剪短并且不需要太多的依赖。（这条排除了 tserialise/tpretty from lua-nucleo）

metalua的serialize.lua很接近我的需要，但是他不能够处理全局的函数或者math.huge数。而且也不能够漂亮的输出（这个也要求很多不同的方法）；table.tostring和serialize结合起来几乎是我想要的一大半代码了。


Penlight的pretty.lua 能够做到很漂亮的输出，但是依赖两个其他的模块，并且不能够处理boolean值和将table作为key的数据，共享表格以及自身引用。

lua-nucleo的tserialize.lua在序列化不同类型方面做的很出色（能够很好的打印math.huge数字，而另外两个模块就不支持），但是不能够处理作为key或者value的函数。

我更希望结果是可读性很好的，而且我能够使用loadstring得到的依然是一个有效的片段。例如，我想keys能够以数字的方式被列出，比如`{'a','b'}` 打印之后就是`{[1] = 'a', [2] = 'b'}`,`{1, nil, 3}`打印之后就是 `{1, [3]=3}`,`{foo = 'foo'}`打印之后就是`{['foo'] = 'foo'}.`



Serpent并不处理upvalues和元表;对于弱引用或者transient 对象（io.stdin和其他userdata的对象是能够通过名字序列化的），函数尽可能序列化，全队函数被他们的名字替代。

```
local b = {text="ha'ns", ['co\nl or']='bl"ue', str="\"\n'\\\000"}
local c = function() return 1 end
local a = {
  x=1, [true] = {b}, [not true]=2, -- boolean as key
  ['true'] = 'some value', -- keyword as a key
  z = c, -- function as value
  list={'a',nil,nil, -- embedded nils
        [9]='i','f',[5]='g',[7]={}, ['3'] = 33}, -- empty table
  [c] = print, -- function as key, global as value
  [io.stdin] = 3, -- global userdata as key
  ['label 2'] = b, -- shared reference
  [b] = 0/0, -- table as key, undefined value as value
  [math.huge] = -math.huge, -- huge as number value
}
a.c = a -- self-reference
a[a] = a -- self-reference with table as key
```

The library provides three functions -- dump, line, and block -- with the last two being shortcuts for the main dump function.

这个库提供三个函数，dump,line和block，后面两个是dump函数的简易版本。

####漂亮的多行打印

-------

调试的时候如果你像看到一个表套表，这个就非常有用了。

```
local serpent = require("serpent")
print(serpent.block(a))
```

输出结构：

```
{
  [1/0 --[[math.huge]]] = -1/0 --[[-math.huge]],
  c = nil --[[ref]],
  ["label 2"] = {
    ["co\nl or"] = "bl\"ue",
    str = "\"\n'\\\000",
    text = "ha'ns"
  } --[[table: 001752B0]],
  list = {
    "a",
    nil,
    nil,
    "f",
    "g",
    nil,
    {} --[[table: 00175350]],
    [9] = "i",
    ["3"] = 33
  } --[[table: 00175328]],
  ["true"] = "some value",
  x = 1,
  z = loadstring("LuaQ...",'@serialized') --[[function: 00171020]],
  [false] = 2,
  [io.stdin --[[file (767D0958)]]] = 3,
  [true] = {
    nil --[[ref]]
  } --[[table: 00175300]]
} --[[table: 001752D8]]
```


####单行打印

-------
这是一个更加紧凑一点的展示。一行就展示出序列化的字符串。
不是一个完全的展示，不包括自我引用，共享数据，循环引用。

local serpent = require("serpent")
print(serpent.dump(a, {name = 'a'}))

输出结果：

```
do local a={[false]=2,[true]={[1]={str="\"\n'\\\000",text="ha'ns",["co\nl or"]
="bl\"ue"}},c=nil --[[ref]],["label 2"]=nil --[[ref]],[1/0 --[[math.huge]]]=-1/0
--[[-math.huge]],x=1,z=loadstring("LuaQ...",'@serialized'),[io.stdin]=3,list=
{[1]="a",[4]="f",[5]="g",[7]={},["3"]=33,[9]="i"},["true"]="some value"};
a.c=a;a[a]=a;a["label 2"]=a[true][1];a[a[true][1]]=0/0;a[a.z]=print;return a;end
```
