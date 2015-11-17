---
layout: post
title: "lua中require到底做了什么事情？"
date: 2015-11-15 20:47:27 +0800
comments: true
categories: lua
---
require到底做了什么呢？起初我认为他和其他的语言的require是一样的，即将另一个模块导入进来。比如有一个如下的A.lua文件
<!--more-->

```
DEBUG = false
USER_NAME = "newtomao"
USER_IMAGE = "cat.png"
VERSION = ""

function printBool(boolValue)
	print("[printBook]:" .. tostring(boolValue))
end

function getRandomNumber()
	return  math.random(100)
end
```

当我在另一个文件B.lua里面`require("A")`的时候，我们可以直接调用`getRandomNumber`和上面定义的常量。因此我们可以任务require就是将另一个文件注册到全局表_G中。这样我们就可以在全局范围内的使用A文件中定义的函数和变量。

我们讲require"A"的结果打印出来如下：

```
retA = require("A")
dump(retA)
```
结果如下：

```
"require之后的结果" = true
```

但是如果A文件不是上面的样子，而是如下的样子：

```
local A = {}
A.userName = "hello"
function A.doAction()
	print("doAction")
end
return A
```

此时我们require(A.lua)之后，是否可以也直接调用doAction呢？且慢。仔细看下面有一个`return`, 这表示这个lua文件如果被require进来，返回的是A，就是一个完整的模块table。

```
retA = require("A")
dump(retA,"require之后的结果")
```
 打印的结果如下：
```
"require之后的结果" = {
    "doAction" = function: 0x7fcc024dddb0
    "userName" = "hello"
}
```

你看她返回的结果是一个table，打印了A的结构。此时我们就可以通过`retA.doAction()`的形式去调用。
retA就是类似一个对象，但是又不完全是对象，它更像是一个命名空间。规定了函数和变量的上下文环境是在A这个空间内。如果外面想要访问userName，需要在前面加上命名空间的限定。
如果A被我们require多次结果会不会不同呢？比如如下：

```
a = require("A")
a2 = require("A")
print(a == a2)
```
结果是怎么样的呢？答案就是true，是的，a和a2是一个地址。

```
print(a)
table: 0x7fac50500ad0
print(a2)
table: 0x7fac50500ad0
```

那么这样看来你require多次和require一次的结果是一样的。那么require到底做了什么？
我们来看lua中对require函数的实现：

```
static int ll_require (lua_State *L) {
  const char *name = luaL_checkstring(L, 1);
  int i;
  lua_settop(L, 1);  /* _LOADED table will be at index 2 */
  lua_getfield(L, LUA_REGISTRYINDEX, "_LOADED");
  lua_getfield(L, 2, name);
  if (lua_toboolean(L, -1)) {  /* is it there? */
    if (lua_touserdata(L, -1) == sentinel)  /* check loops */
      luaL_error(L, "loop or previous error loading module " LUA_QS, name);
    return 1;  /* package is already loaded */
  }
  /* else must load it; iterate over available loaders */
  lua_getfield(L, LUA_ENVIRONINDEX, "loaders");
  if (!lua_istable(L, -1))
    luaL_error(L, LUA_QL("package.loaders") " must be a table");
  lua_pushliteral(L, "");  /* error message accumulator */
  for (i=1; ; i++) {
    lua_rawgeti(L, -2, i);  /* get a loader */
    if (lua_isnil(L, -1))
      luaL_error(L, "module " LUA_QS " not found:%s",
                    name, lua_tostring(L, -2));
    lua_pushstring(L, name);
    lua_call(L, 1, 1);  /* call it */
    if (lua_isfunction(L, -1))  /* did it find module? */
      break;  /* module loaded successfully */
    else if (lua_isstring(L, -1))  /* loader returned error message? */
      lua_concat(L, 2);  /* accumulate it */
    else
      lua_pop(L, 1);
  }
  lua_pushlightuserdata(L, sentinel);
  lua_setfield(L, 2, name);  /* _LOADED[name] = sentinel */
  lua_pushstring(L, name);  /* pass name as argument to module */
  lua_call(L, 1, 1);  /* run loaded module */
  if (!lua_isnil(L, -1))  /* non-nil return? */
    lua_setfield(L, 2, name);  /* _LOADED[name] = returned value */
  lua_getfield(L, 2, name);
  if (lua_touserdata(L, -1) == sentinel) {   /* module did not set a value? */
    lua_pushboolean(L, 1);  /* use true as result */
    lua_pushvalue(L, -1);  /* extra copy to be returned */
    lua_setfield(L, 2, name);  /* _LOADED[name] = true */
  }
  return 1;
}
```

这个函数并不复杂，但是对于初学者来说可能看到之后会被吓到，大部分都是出栈入栈的操作。我们先看前几行

```
lua_settop(L, 1);  /* _LOADED table will be at index 2 */
  lua_getfield(L, LUA_REGISTRYINDEX, "_LOADED");
  lua_getfield(L, 2, name);
  if (lua_toboolean(L, -1)) {  /* is it there? */
    if (lua_touserdata(L, -1) == sentinel)  /* check loops */
      luaL_error(L, "loop or previous error loading module " LUA_QS, name);
    return 1;  /* package is already loaded */
  }
```
不用看函数，且看注释，他的意思就是先将栈指针移到第一个位置，由于lua所以从1开始，移到1即意味着要讲栈里面的元素清空，然后将LUA_REGISTRYINDEX执行的表数据里面名字是“_LOADED” 这个数据放到第2个位置，接下来讲在第2位的数据里面，名字叫做name的数据放到第三位。那么在第二位的不正是loaded的数据嘛！
接下来，lua检查栈顶元素是否是bool类型，那么栈顶元素是哪一个呢?不正是第三步放进来的数据嘛！
检查这个数据是否是boolean类型，只要存在，也就是加载过这个name模块，那么就任务已经被加载过了。就直接返回无需再次加载。

而如果没有数据，那么就需要检查是否能够正确地加载。

这就解释了为什么多次require其实并没有什么意义。

接下来代码会寻找一个有效地loader加载器加载lua模块然后讲模块加载的结果放到了_LOADED中。因此我们在require之后，可以通过打印package.loaded来检查。

```
 "package.loaded" = {
     "A" = {
         "doAction" = function: 0x7f88fec5c2d0
         "userName" = "hello"
     }
     "B" = true
    }
```


看到这里，各位同学应该已经了解这个reuqire之后，其实并未创建新的对象，而是将lua文件运行之后的结果返回来了。那么这个已经被放到了loaded的“A”更像是一个类文件的模板。而不是对象。

那么这对于我们习惯面向对象开发的同学来说，如何用lua来实现面向对象的开发呢？
别急，请看下一篇文章。