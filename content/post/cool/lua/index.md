---
title: "Lua"
date: 2022-06-17T15:11:35+08:00
draft: false
categories: cool
tags : 
    - lua
image: "lua_official_logo_icon_168116.svg"
lastmod: 2022-06-17T23:48:35+08:00
description: "简单了解一下lua"
---

# Lua简述
Lua 是一门小巧的解释型编程语言

## 数据类型
Lua是动态类型语言，变量无需类型定义。他有8个基本类型分为：

### 基本类型

| 数据类型   | 描述                                               |
| ---------- | -------------------------------------------------- |
| `nil`      | 仅nil值属于该类，条件表达式中和false效果一样       |
| `boolean`  | 包含true和false                                    |
| `number`   | 双精度的实浮点数                                   |
| `string`   | 单双引号包含的字符串                               |
| `function` | Lua和C的函数                                       |
| `userdate` | 表示任意存储在变量中C数据结构                      |
| `thread`   | 协同线程                                           |
| `table`    | 数组的索引可以是任意类型，通过构造表达式`{}`来创建 |

---
可以通过type函数得到变量或值的类型

```lua
print(type("Hello world"))      --> string
print(type(10.4*3))             --> number
print(type(type))               --> function
print(type(true))               --> boolean
print(type(nil))                --> nil
print(type(type(X)))            --> string
```
---

### nil

- `nil` 类型表示一种没有任何有效值，它只有一个值*nil*，例如打印一个没有赋值的变量，便会输出一个 *nil* 值
- 对于全局变量和 `table`，`nil` 还有一个"删除"作用，给全局变量或者 `table` 表里的变量赋一个 *nil* 值，等同于把它们删掉
- `nil` 作比较时应该加上双引号`"nil"`

### boolean

- `boolean` 类型只有两个可选值：*true*（真） 和 *false*（假），Lua 把 *false* 和 *nil* 看作是 *false*，其他的都为 *true*，数字 0 也是 *true*

### number

- Lua 默认只有一种 `number` 类型 -- `double`（双精度）类型（默认类型可以修改 **luaconf.h** 里的定义）

### string

- 字符串由一对双引号或单引号来表示
- 也可以用 2 个方括号 `[[ ]]` 来表示"一块"字符串
- 在对一个数字字符串上进行算术操作时，Lua 会尝试将这个数字字符串转成一个数字
- 使用 `#` 来计算字符串的长度，放在字符串前面

### table

- 在 Lua 里，`table` 的创建是通过"构造表达式"来完成，最简单构造表达式是{}，用来创建一个空表。也可以在表里添加一些数据，直接初始化表
  ```lua
  -- 创建一个空的 table
  local tbl1 = {}

  -- 直接初始表
  local tbl2 = {"apple", "pear", "orange", "grape"}
  ```
- Lua 中的表（`table`）其实是一个"关联数组"（associative arrays），数组的索引可以是数字或者是字符串
  ```lua
  -- table_test.lua 脚本文件
  a = {}
  a["key"] = "value"
  key = 10
  a[key] = 22
  a[key] = a[key] + 11
  for k, v in pairs(a) do
      print(k .. " : " .. v)
  end
  ```
- 不同于其他语言的数组把 0 作为数组的初始索引，在 Lua 里表的默认初始索引一般以 1 开始
- `table` 不会固定长度大小，有新数据添加时 `table` 长度会自动增长，没初始的 `table` 都是 *nil*
- Lua 中table的索引可以用`[]`，也可以用`.`

### function

- 在 Lua 中，函数是被看作是"第一类值（First-Class Value）"，函数可以存在变量里
  ```lua
  -- function_test.lua 脚本文件
  function factorial1(n)
      if n == 0 then
          return 1
      else
          return n * factorial1(n - 1)
      end
  end
  print(factorial1(5))
  factorial2 = factorial1
  print(factorial2(5))
  ```
- `function` 可以以匿名函数（anonymous function）的方式通过参数传递
    ```lua
    -- function_test2.lua 脚本文件
    function testFun(tab,fun)
        for k ,v in pairs(tab) do
            print(fun(k,v));
        end
    end


    tab={key1="val1",key2="val2"};
    testFun(tab,
    function(key,val)--匿名函数
        return key.."="..val;
    end
    );
    ```
### thread

> 在 Lua 里，最主要的线程是协同程序（coroutine）。它跟线程（thread）差不多，拥有自己独立的栈、局部变量和指令指针，可以跟其他协同程序共享全局变量和其他大部分东西。

> 线程跟协程的区别：线程可以同时多个运行，而协程任意时刻只能运行一个，并且处于运行状态的协程只有被挂起（suspend）时才会暂停。

### userdata

> userdata 是一种用户自定义数据，用于表示一种由应用程序或 C/C++ 语言库所创建的类型，可以将任意 C/C++ 的任意数据类型的数据（通常是 struct 和 指针）存储到 Lua 变量中调用。

## 变量

Lua 有三种变量：全局变量、**局部变量**、表中的域。

局部变量需要用local显式声明`local b = 5`，作用域为声明位置到所在语句块结束。

变量默认值为*nil*。

像golang一样，lua可以为多个变量同时赋值，并且通过赋值顺序交换变量的值：
```lua
x,y=2,10
x,y=y,x
```
但是lua会对变量和值个数不匹配进行截取或补*nil*的操作。

## 控制流

Lua有4种循环处理方式：
```lua
-- 1
while( condition )
do 
    statements
end
-- 2 exp3默认步长为1，exp会在循环前求值
for var=exp1,exp2,exp3 do
    statements
end
-- 3
repeat
    statements
end ( condition )
-- 4
-- 不同类型循环，相同类型循环可嵌套

```

2种循环控制语句：
```lua
-- 1
break
-- 2
::Label:: statement
if condition then
    goto Label
end
```

3中控制语句
```lua
-- 1
if condition
then
    statements
end
-- 2
if condition
then
    statements
else
    statements
end
```

## 函数

Lua 函数定义格式如下：
```lua
[可选参数] function 函数名 ( 参数1, 参数2..., 参数n )
    函数体
    return [多]返回值
end
```

- 可选参数规定了函数是全局的还是局部的，默认是全局函数，加上`local`参数则为局部函数
- 函数可以作为参数传递给函数
- 使用`...`表示可变参数，`{...}`用于函数体中，表示变长参数构成的数组，固定参数必须放在变长参数之前
- 通过`select('#', ...)`返回可变参数长度，通过`select(n, ...)`用于返回从起点**n**开始到结束位置的所有参数列表

## 运算符

介绍几个稍微特殊的
- `/` 双精度浮点数除法
- `//` 向下取整除法
- `~=` 表示不等于
- `and`,`or`和`not`为逻辑运算

## 字符串

Lua 语言中字符串可以使用以下三种方式来表示：

- 单引号间的一串字符。
- 双引号间的一串字符。
- [[ 与 ]] 间的一串字符。

> 转义字符用于表示不能直接显示的字符，比如后退键，回车键，等。如在字符串转换双引号可以使用 `"\""`

### 字符串操作
| 方法                                          | 用途                                                  |
| --------------------------------------------- | ----------------------------------------------------- |
| `string.upper(str)`                           | 全部字母变大写                                        |
| `string.lower(str)`                           | 全部字母变小写                                        |
| `string.gsub(mainstr,findstr,replacestr,num)` | 将*mainstr*中*findstr*替换为*replacestr*，替换*num*次 |
| `string.find(str,substr,[init,[end]])`        | *str*中搜索*substr*，返回起始和结束索引               |
| `string.reverse(str)`                         | 反转字符串                                            |
| `string.format(...)`                          | 格式化，c语言风格                                     |
| `string.byte(str[,int]])`,`string,char(str)`  | byte转换某处字符为整数，char将整数转为字符串          |
| `string.len(str)`                             | 返回字符串长度                                        |
| `string.rep(str,n)`                           | 重复字符串*n*次                                       |
| `..`                                          | 连接字符串                                            |
| `string.gmatch(str,pattern)`                  | 返回迭代器，返回*str*中符合*pattern*的子串            |
| `string.sub(str,i[,j])`                       | 截取子串，i位置到j位置（包含该位置）                  |

### 匹配模式
> Lua 中的匹配模式直接用常规的字符串来描述。 它用于模式匹配函数 `string.find`, `string.gmatch`, `string.gsub`, `string.match`。

> 你还可以在模式串中使用字符类。字符类指可以匹配一个特定字符集合内任何字符的模式项。比如，字符类 %d 匹配任意数字。所以你可以使用模式串 %d%d/%d%d/%d%d%d%d 搜索 dd/mm/yyyy 格式的日期：

[官方文档-字符串](http://www.lua.org/manual/5.4/manual.html#6.4 "String Manipulation")

## 迭代器

迭代器（iterator）是一种对象，它能够用来遍历标准模板库容器中的部分或全部元素，每个迭代器对象代表容器中的确定的地址。在 Lua 中迭代器是一种支持指针类型的结构，它可以遍历集合的每一个元素。

### 泛型for迭代器

```lua
for k,v in pairs(t) do
    print(k,v)
end
```

### 无状态的迭代器
无状态的迭代器是指不保留任何状态的迭代器，因此在循环中我们可以利用无状态迭代器避免创建**闭包**花费额外的代价。

```lua
function square(iteratorMaxCount,currentNumber)
   if currentNumber<iteratorMaxCount
   then
      currentNumber = currentNumber+1
   return currentNumber, currentNumber*currentNumber
   end
end
-- square, [状态常量], [控制变量]，函数返回[控制变量]，其他--直到第一个nil元素
-- 常量保持不变，变量根据函数迭代而改变
for i,n in square,3,0
do
   print(i,n)
end
```

### 多状态的迭代器

很多情况下，迭代器需要保存多个状态信息而不是简单的状态常量和控制变量，最简单的方法是使用闭包，还有一种方法就是将所有的状态信息封装到 table 内，将 table 作为迭代器的状态常量，因为这种情况下可以将所有的信息存放在 table 内，所以迭代函数通常不需要第二个参数。

```lua
array = {"Google", "Runoob"}

function elementIterator (collection)
   local index = 0
   local count = #collection
   -- 闭包函数
   return function ()
      index = index + 1
      if index <= count
      then
         --  返回迭代器的当前元素
         return collection[index]
      end
   end
end

for element in elementIterator(array)
do
   print(element)
end
```

## table（表）

- table 是 Lua 的一种数据结构用来帮助我们创建不同的数据类型，如：数组、字典等。
- Lua table 使用关联型数组，你可以用任意类型的值来作数组的索引，但这个值不能是 nil。
- Lua table 是不固定大小的，你可以根据自己需要进行扩容。

```lua
-- 初始化表
mytable = {}

-- 指定值
mytable[1]= "Lua"

-- 移除引用
mytable = nil
-- lua 垃圾回收会释放内存
```

| 方法                                      | 用途                                                   |
| ----------------------------------------- | ------------------------------------------------------ |
| `table.concat(table[,sep[,start[,end]]])` | 元素间以sep分隔                                        |
| `table.insert(table,[pos,]value)`         | pos位置插入value的一个元素                             |
| `table.remove(table[,pos])`               | 返回table数组部分位于pos位置的元素，其它的元素会被前移 |
| `table.sort(table[,comp])`                | 对给定的table进行升序排序                              |

## 模块与包

### 定义模块
```lua
-- 文件名为 module.lua
-- 定义一个名为 module 的模块
module = {}
 
-- 定义一个常量
module.constant = "这是一个常量"
 
-- 定义一个函数
function module.func1()
    io.write("这是一个公有函数！\n")
end
 
local function func2()
    print("这是一个私有函数！")
end
 
function module.func3()
    func2()
end
 
return module
```

### require函数

```lua
require("模块名")
```
### 模块加载
[模块加载机制](http://www.lua.org/manual/5.4/manual.html#6.3 "Modules")

## 元表
在 Lua table 中我们可以访问对应的 key 来得到 value 值，但是却无法对两个 table 进行操作(比如相加)。

因此 Lua 提供了元表(Metatable)，允许我们改变 table 的行为，每个行为关联了对应的元方法。

例如，使用元表我们可以定义 Lua 如何计算两个 table 的相加操作 a+b。

当 Lua 试图对两个表进行相加时，先检查两者之一是否有元表，之后检查是否有一个叫 __add 的字段，若找到，则调用对应的值。 __add 等即时字段，其对应的值（往往是一个函数或是 table）就是"元方法"。

有两个很重要的函数来处理元表：

- `setmetatable(table,metatable)`: 对指定 table 设置元表(metatable)，如果元表(metatable)中存在 __metatable 键值，setmetatable 会失败。
- `getmetatable(table)`: 返回对象的元表(metatable)。
  
以下实例演示了如何对指定的表设置元表：

```lua
mytable = {}                          -- 普通表
mymetatable = {}                      -- 元表
setmetatable(mytable,mymetatable)     -- 把 mymetatable 设为 mytable 的元表
以上代码也可以直接写成一行：

mytable = setmetatable({},{})
以下为返回对象元表：

getmetatable(mytable)                 -- 这会返回 mymetatable
```

### __index 元方法

这是 metatable 最常用的键。

当通过键来访问 table 的时候，如果这个键没有值，那么Lua就会寻找该table的metatable（假定有metatable）中的__index 键。如果__index包含一个表格，Lua会在表格中查找相应的键。

```lua 
mytable = setmetatable({key1 = "value1"}, { __index = { key2 = "metatablevalue" } })
print(mytable.key1,mytable.key2)
-- output: value1    metatablevalue
```

> Lua 查找一个表元素时的规则，其实就是如下 3 个步骤:
> 1. 在表中查找，如果找到，返回该元素，找不到则继续
> 2. 判断该表是否有元表，如果没有元表，返回 nil，有元表则继续。
> 3. 判断元表有没有 __index 方法，如果 __index 方法为 nil，则返回 nil；如果 __index 方法是一个表，则重复 1、2、3；如果 __index 方法是一个函数，则返回该函数的返回值。

### __newindex元方法

__newindex 元方法用来对表更新，__index则用来对表访问 。当你给表的一个缺少的索引赋值，解释器就会查找__newindex 元方法：如果存在则调用这个函数而不进行赋值操作。

### 为表添加操作符

| 模式       | 描述              |
| ---------- | ----------------- |
| `__add`    | 对应的运算符 '+'  |
| `__sub`    | 对应的运算符 '-'  |
| `__mul`    | 对应的运算符 '*'  |
| `__div`    | 对应的运算符 '/'  |
| `__mod`    | 对应的运算符 '%'  |
| `__unm`    | 对应的运算符 '-'  |
| `__concat` | 对应的运算符 '..' |
| `__eq`     | 对应的运算符 '==' |
| `__lt`     | 对应的运算符 '<'  |
| `__le`     | 对应的运算符 '<=' |

### __call元方法
__call 元方法在 Lua 调用一个值时调用

```lua
-- 计算表中最大值，table.maxn在Lua5.2以上版本中已无法使用
-- 自定义计算表中最大键值函数 table_maxn，即计算表的元素个数
function table_maxn(t)
    local mn = 0
    for k, v in pairs(t) do
        if mn < k then
            mn = k
        end
    end
    return mn
end

-- 定义元方法__call
mytable = setmetatable({10}, {
  __call = function(mytable, newtable)
        sum = 0
        for i = 1, table_maxn(mytable) do
                sum = sum + mytable[i]
        end
    for i = 1, table_maxn(newtable) do
                sum = sum + newtable[i]
        end
        return sum
  end
})
newtable = {10,20,30}
print(mytable(newtable))
-- output: 70
```

### __tostring元方法

```lua
mytable = setmetatable({ 10, 20, 30 }, {
  __tostring = function(mytable)
    sum = 0
    for k, v in pairs(mytable) do
                sum = sum + v
        end
    return "表所有元素的和为 " .. sum
  end
})
print(mytable)
-- output: 表所有元素的和为 60
```

## 协程

[协同程序](https://www.runoob.com/lua/lua-coroutine.html "菜鸟教程lua协程")

[Coroutine](http://www.lua.org/manual/5.4/manual.html#6.2 "Coroutine Manipulation")



## 参考资料

https://www.runoob.com/manual/lua53doc/contents.html

https://www.runoob.com/lua/lua-tutorial.html