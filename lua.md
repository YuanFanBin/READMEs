#（一）Lua 5.1.4 扩展编写

## 1. Lua扩展demo - 实现一个简单的版本比较

```c
#define lutillib_c

#define LUA_LIB

#include "lua.h"
#include "lauxlib.h"
#include "lualib.h"
#include <string.h>
#include <ctype.h>

static int util_version_compare(lua_State *L)
{
    size_t v1l, v2l, optl;
    const char *v1 = luaL_checklstring(L, 1, &v1l);
    const char *v2 = luaL_checklstring(L, 2, &v2l);
    const char *opt = luaL_checklstring(L, 3, &optl);
    int compare = 0;
    int i = 0, j = 0, num1, num2;

#define isdig(x) (isdigit(x) || (x)=='.')
#define lshift(x, c) ((x) * 10 + ((c) - '0'))

    /* version compare */
    while (i < v1l || j < v2l) {
        num1 = 0, num2 = 0;
        while (i < v1l && isdigit(v1[i])) {
            num1 = lshift(num1, v1[i++]);
        }
        while (j < v2l && isdigit(v2[j])) {
            num2 = lshift(num2, v2[j++]);
        }
        if ((i < v1l && !isdig(v1[i])) || (j < v2l && !isdig(v2[j]))) {
            lua_pushnil(L);
            lua_pushfstring(L, "v1 = '%s', v2 = '%s'", v1, v2);
            return 2;
        }
        if (num1 > num2) { compare = 1; break;  } 
        if (num1 < num2) { compare = -1; break;  }
        ++i, ++j;
    }
    /* >, <, =, >=, <= */
    if (!strncmp(opt, ">", optl)) {
        lua_pushboolean(L, compare == 1);
    } else if (!strncmp(opt, ">=", optl)) {
        lua_pushboolean(L, compare != -1);
    } else if (!strncmp(opt, "<", optl)) {
        lua_pushboolean(L, compare == -1);
    } else if (!strncmp(opt, "<=", optl)) {
        lua_pushboolean(L, compare != 1);
    } else if (!strncmp(opt, "=", optl)) {
        lua_pushboolean(L, compare == 0);
    } else {
        lua_pushnil(L);
        lua_pushfstring(L, "opt = '%s'", opt);
        return 2;
    }
    return 1;
}

static const luaL_Reg utillib[] = {
    {"version_compare", util_version_compare},
    {NULL, NULL}
};

LUALIB_API int luaopen_util(lua_State *L) {
    luaL_register(L, "util", utillib);
    return 1;
}
```

## 2. 编译.so

### 2.1 修改lua-5.1.4/src/Makefile文件

```Makefile
LIB_O= lauxlib.o lbaselib.o ldblib.o liolib.o lmathlib.o loslib.o ltablib.o \
       lstrlib.o loadlib.o linit.o lutillib.o

LUA_SO=lutillib.so

ALL_T= \$(LUA_A) \$(LUA_T) \$(LUAC_T) $(LUA_SO)

$(LUA_SO): $(CORE_O) $(LIB_O)
             $(CC) -o $@ -shared $? -ldl -lm

lutillib.o: lutillib.c lua.h luaconf.h lauxlib.h lualib.h
```

### 2.2 在lua-5.1.4目录下make编译代码

```sh
$ make clean && make linux  # centos6.5环境
```

## 3. 测试

```lua
-- test.lua
local util = require('util')      -- 查找当前文件中的util.so文件
local res, msg = util.version_compare('5.0', '5', '=')
print('res = ' .. tostring(res) .. ' msg = ' .. tostring(msg))
```

```lua
$ lua test.lua     # 使用系统lua
ok = true msg = nil
```

.so文件可放入如下目录：

```sh
$ lua test.lua
lua: test.lua:1: module 'util' not found:
    no field package.preload['util']
    no file './util.lua'
    no file '/usr/share/lua/5.1/util.lua'
    no file '/usr/share/lua/5.1/util/init.lua'
    no file '/usr/lib/lua/5.1/util.lua'
    no file '/usr/lib/lua/5.1/util/init.lua'
    no file './util.so'
    no file '/usr/lib/lua/5.1/util.so'
    no file '/usr/lib/lua/5.1/loadall.so'
stack traceback:
    [C]: in function 'require'
    test.lua:1: in main chunk
    [C]: ?
```

#（二）Lua 5.1.4 如何操作table的堆栈

## 1. lua_next

next详解请看 [Lua 5.0 Reference Manual][1] 的`next(table [, index])`部分解释；或者参考 [云风][2] 翻译的 [Lua 5.3 参考手册][3] 的`lua_next`部分。官方文档中有一点需要关注：

> When called with **nil** as its second argument, **next** returns the first index of the table and its associated value.

对lua代码来说，可以利用`next(t)`来判断表是否为空；对c代码来说需要增加demo中的第四行代码。

## 2. demo

```c
/* 如何遍历table */
static int util_traverse(lua_State *L)
{                                                                                                              
    luaL_checktype(L, 1, LUA_TTABLE);
    lua_pushnil(L); /* for first key */
    /* lua_next 使用nil调用时，会返回第一个(初始)key/value键值对 */
    while (lua_next(L, 1)) { /* -1， value; -2, key, -3, table */
        /* 数据加工 */
        lua_pop(L, 1); /* 从栈中弹出1个元素 */
        /* 此时栈 -1, key; -2, table */
    }
    /* 此时栈 -1, table */
    
    lua_pushstring(L, "haha");
    return 1;
}
```

```lua
local util = require("util")
local t = {
    ['key1'] = 'value1',
    ['key2'] = 'value2'
}
local key, value = util.traverse(t)
print("key = " .. tostring(key) .. " value = " .. tostring(value))
```

参考资料：
http://www.lua.org/manual/5.0/manual.html#5.1
http://blog.codingnow.com
http://cloudwu.github.io/lua53doc/manual.html#lua_next

[1]: http://www.lua.org/manual/5.0/manual.html#5.1
[2]: http://blog.codingnow.com
[3]: http://cloudwu.github.io/lua53doc/manual.html#lua_next

# （三）Lua 5.1.4 dofile, loadfile, loadstring

## 1. Lua层面理解dofile, loadfile, loadstring

### 1.1 dofile, loadfile
首先我们来了解一下官方 [Programming in Lua (first edition)](http://www.lua.org/pil/contents.html) 的 [Chapter 8 - Compilation, Execution, and Errors](http://www.lua.org/pil/8.html) 对 `loadfile` 的解释。

> Like `dofile`, `loadfile` also loads a lua chunk from a file, but it does not run the chunk.

阅读完官方文档，可理解为：
`loadfile`： 编译代码生成中间码，以一个函数的方式返回编译后的chunk块，可重复利用该chunk块；此函数并不执行代码，若有任何错误，将返回**nil**及**err**消息。
`dofile`： 读入文件编译并执行，若有错误，将抛出异常；每次执行都会编译代码，运行效率较低。定义可理解为：

```lua
function dofile(filename)
    local f = assert(loadfile(filename))
    return f()
end
```

对于一些简单的任务，使用`dofile`会很方便；但是，`loadfile`会更加灵活。

### 1.2 loadfile, loadstring

`loadstring`与`loadfile`相似，不同之处在于`loadstring`会读入字符串而不是文件，例：

```lua
f = loadstring("i = i + 1")
```

在使用`loadstring`之前请确保没有其他更好的方法来解决问题，`loadstring`函数过于强大，很容易被注入各类代码。
和其他任何函数一样，chunks块能够声明`local`变量并`return`相关数据。

```lua
f = loadstring("local a = 10; return a + 20")
printf(f())     --> 30
```

文档中如下内容：
>  A common mistake is to assume that loadfile (or loadstring) defines functions. In Lua, function definitions are assignments; as such, they are made at runtime, not at compile time.
    
描述在lua中当执行`loadfile`,`loadstring`时，并没有定义函数，定义行为发生在运行时，例：

```lua
-- file foo.lua
function foo(x)
    print(x)
end
```

当运行如下代码时

```lua
f = loadfile("foo.lua")
```

foo被编译成功，但没有定义该函数，为了定义函数，需运行如下代码

```lua
f() -- defines 'foo'
foo("ok")   --> ok
-- or
```

对于`loadstring`，我们可以使用调用链的方式，使用`assert`来避免`str`中的语法错误，`loadstring`中仅使用`global`变量，`local`变量无法被`loadstring`所定义的函数使用，但是函数`g`可以使用`local`变量

```lua
i = 10
local i = 1

str = "i = i + 1; return i"
print(assert(loadstring(s))()) --> 11

g = function() i = i + 1; return i end
print(g()) --> 2
```

官方文档中提供的两个很有意思的例子：

```lua
-- 1
print "enter your expressiong:"
local l = io.read()
local func = assert(loadstring("return " ... l))
print("the value of your expression is " .. func())
-- 2
print "enter function to be plotted (with variable 'x'):"
local l = io.read()
local f = assert(loadstring("return " .. l))
for i = 1, 20 do
    x = i -- global 'x' (to be visiable from the chunk)
    print(string.rep("x", f()))
end
```

如上两个例子可灵活的在客户端使用，或者在服务器中lua作为粘合剂时，提供的一种热更新机制，当然要确保正确使用，防止一些恶意的代码注入。

## 2. 源码层面理解luaB_dofile, luaB_loadfile, luaB_loadstring

使用 [Lua 5.1.4](http://www.lua.org/ftp/lua-5.1.4.tar.gz) 源码， [Lua 5.1 Reference Manual](http://www.lua.org/manual/5.1/) 英文参考手册， [云风](http://blog.codingnow.com/) 翻译的中文参考手册 [Lua 5.3 参考手册](http://cloudwu.github.io/lua53doc/manual.html)。
PS: 英文不好，整份中文的一起对照着看。

### 2.1 dofile(luaB_dofile)

到`src/lbaselib.c`中看`luaB_dofile`源码

```c
static int luaB_dofile (lua_State *L) {
  const char *fname = luaL_optstring(L, 1, NULL);
  int n = lua_gettop(L);
  if (luaL_loadfile(L, fname) != 0) lua_error(L);
  lua_call(L, 0, LUA_MULTRET);
  return lua_gettop(L) - n;
} 
```

`luaL_optstring`获取文件名， `lua_gettop`获取当前栈中元素个数， `luaL_loadfile`加载并编译文件内容（文件内容可为lua源码，或者lua字节码数据），随后调用`lua_call`执行当前栈中的函数，即文件。
从源码可看出`dofile`函数接受0~1个参数，当没有参数或参数为`nil`时`dofile`将从标准输入读入数据，当只有1个参数时从指定参数对于的文件读入数据，并执行该文件。
更详细解释请参考： [Simon Liu](http://www.pagefault.info/?p=462) 的文章

### 2.2 loadfile(luaB_loadfile)

到`src/lbaselib.c`中看`luaB_loadfile`源码

```c
static int luaB_loadfile (lua_State *L) {
    const char *fname = luaL_optstring(L, 1, NULL);
    return load_aux(L, luaL_loadfile(L, fname));
}
```

在`luaB_loadfile`中只是简单的调用`luaL_loadfile`去编译文件代码，`luaL_loadfile`若产生错误则通过`load_aux`调整堆栈，将`nil`及`err`消息放入栈顶。

### 2.3 loadstring(luaB_loadstring)

到`src/lbaselib.c`中看`luaB_loadstring`源码

```c
static int luaB_loadstring (lua_State *L) {
    size_t l;
    const char *s = luaL_checklstring(L, 1, &l);
    const char *chunkname = luaL_optstring(L, 2, s);
    return load_aux(L, luaL_loadbuffer(L, s, l, chunkname));
}
```

在`luaB_loadstring`中使用`luaL_loadbuffer`调用`lua_load`函数编译生成对应的字节码数据，`luaL_loadbuffer`与`luaL_loadfile`的不同之处在与`luaL_loadfile`函数会对文件做相应过滤和处理操作，并且数据由文件中读入，而不是直接从内存中获取。

参数`chunkname`作为错误信息的chunk的名字，用于调试，若无此参数，则调试时以`s`对应内容提示，例：

```lua
--test.lua
i = nil
f = loadstring("i = i + 1; return i", "my_chunkname")
--f = loadstring("i = i + 1; return i")
f()
```

```sh
$ lua test.lua
lua: [string "my_chunkname"]:1: attempt to perform arithmetic on global 'i' (a nil value)
stack traceback:
        [string "my_chunkname"]:1: in function 'f'
        test.lua:3: in main chunk
        [C]: ?
```

```sh
$ lua test.lua
lua: [string "i = i + 1; return i"]:1: attempt to perform arithmetic on global 'i' (a nil value)
stack traceback:
        [string "i = i + 1; return i"]:1: in function 'f'
        test.lua:4: in main chunk
        [C]: ?
```

通常使用如下方法：

```lua
--test.lua
i = nil
f = loadstring("i = i + 1; return i", "=my_chunkname") -- 此处有"="
f()
```

```sh
$ lua test.lua
lua: my_chunkname:1: attempt to perform arithmetic on global 'i' (a nil value)
stack traceback:
        my_chunkname:1: in function 'f'
        test.lua:4: in main chunk
        [C]: ?
```

### 2.4 三个函数的输入源

`dofile`, `loadfile`, `loadstring`均接受lua代码和lua字节码数据，lua代码会做相应的编译操作；lua字节码数据为直接编译成功的结果，无需再次编译，可节省编译开销，lua字节码可由程序`luac`生成，但各个lua版本及机器环境可能会造成lua字节码有所不同。
更详细内容请参考： [lontoken's Blog](http://lontoken.com/analysis-of-lua-bytecode-flie.html) 的文章


参考资料：
http://www.lua.org/pil/contents.html
http://www.lua.org/manual/5.1/
http://cloudwu.github.io/lua53doc/manual.html
http://lontoken.com/analysis-of-lua-bytecode-flie.html
