---
title: Lua学习笔记
date: 2018-05-24 14:54:25
tags:
categories: LUA随笔
---

## 基础函数
### assert
assert(v[,message])
如果其参数v的值为假（nil或false），它就调用error；否则，返回所有的参数。在错误情况时，message指那个错误对象；如果不提供这个参数，参数默认为"assertionfailed!"
### getmetatable
getmetatable(object)
如果object不包含元表，返回nil。否则，如果在该对象的元表中有**__metatable**域时返回其关联值，没有时返回该对象的元表
### ipairs
ipairs(t)
返回三个值（迭代函数、表t以及0），如此，以下代码
	for i,v in ipairs(t) do body end
### load
加载一个代码块。
如果chunk是一个字符串，代码块指这个字符串。如果chunk是一个函数，load不断地调用它获取代码块的片断。每次对chunk的调用都必须返回一个字符串紧紧连接在上次调用的返回串之后。当返回空串、nil、或是不返回值时，都表示代码块结束。
如果没有语法错误，则以函数形式返回编译好的代码块；否则，返回nil加上错误消息。
如果结果函数有上值，env被设为第一个上值。若不提供此参数，将全局环境替代它。所有其它上值初始化为nil。（当你加载主代码块时候，结果函数一定有且仅有一个上值_ENV。然而，如果你加载一个用函数（参见string.dump，结果函数可以有任意数量的上值）创建出来的二进制代码块时，所有的上值都是新创建出来的。也就是说它们不会和别的任何函数共享。
chunkname在错误消息和调试消息中，用于代码块的名字。如果不提供此参数，它默认为字符串chunk。chunk不是字符串时，则为"=(load)"。
字符串mode用于控制代码块是文本还是二进制（即预编译代码块）。它可以是字符串"b"（只能是二进制代码块），"t"（只能是文本代码块），或"bt"（可以是二进制也可以是文本）。默认值为"bt"。
**Lua不会对二进制代码块做健壮性检查。恶意构造一个二进制块有可能把解释器弄崩溃。**
### loadfile
loadfile([filename[,mode[,env]]])
和load类似，不过是从文件filename或标准输入（如果文件名未提供）中获取代码块。
### next
next(table[,index])
运行程序来遍历表中的所有域。第一个参数是要遍历的表，第二个参数是表中的某个键。next返回该键的下一个键及其关联的值。如果用nil作为第二个参数调用next将返回初始键及其关联值。当以最后一个键去调用，或是以nil调用一张空表时，next返回nil。如果不提供第二个参数，将认为它就是nil。特别指出，你可以用next(t)来判断一张表是否是空的。
索引在遍历过程中的次序无定义，即使是数字索引也是这样。（如果想按数字次序遍历表，可以使用数字形式的for。）
当在遍历过程中你给表中并不存在的域赋值，next的行为是未定义的。然而你可以去修改那些已存在的域。特别指出，你可以清除一些已存在的域。
### pairs
pairs(t)
如果t有元方法**__pairs**，以t为参数调用它，并返回其返回的前三个值。
否则，返回三个值：next函数，表t，以及nil
	for k,v in pairs(t) do body end
能迭代表t中的所有键值对。
### pcall
pcall(f[,arg1,···])
传入参数，以保护模式调用函数f。这意味着f中的任何错误不会抛出；取而代之的是，pcall会将错误捕获到，并返回一个状态码。第一个返回值是状态码（一个布尔量），当没有错误时，其为真。此时，pcall同样会在状态码后返回所有调用的结果。在有错误时，pcall返回false加错误消息。
### print
print(···)
接收任意数量的参数，并将它们的值打印到stdout。它用tostring函数将每个参数都转换为字符串。print不用于做格式化输出。仅作为看一下某个值的快捷方式。多用于调试。完整的对输出的控制，请使用string.format以及io.write。
### rawequal
rawequal(v1,v2)
在不触发任何元方法的情况下检查v1是否和v2相等。返回一个布尔量。
### rawget
rawget(table,index)
在不触发任何元方法的情况下获取table[index]的值。table必须是一张表；index可以是任何值。
### rawlen
rawlen(v)
在不触发任何元方法的情况下返回对象v的长度。v可以是表或字符串。它返回一个整数。
### rawset
rawset(table,index,value)
在不触发任何元方法的情况下将table[index]设为value。table必须是一张表，index可以是nil与NaN之外的任何值。value可以是任何Lua值。
这个函数返回table。
### select
select(index,···)
如果index是个数字，那么返回参数中第index个之后的部分；负的数字会从后向前索引（-1指最后一个参数）。否则，index必须是字符串"#"，此时select返回参数的个数。
### setmetatable
setmetatable(table,metatable)
给指定表设置元表。（你不能在Lua中改变其它类型值的元表，那些只能在C里做。）如果metatable是nil，将指定表的元表移除。如果原来那张元表有"__metatable"域，抛出一个错误。
这个函数返回table。
### tonumber
tonumber(e[,base])
如果调用的时候没有base，tonumber尝试把参数转换为一个数字。如果参数已经是一个数字，或是一个可以转换为数字的字符串，tonumber就返回这个数字；否则返回nil。
字符串的转换结果可能是整数也可能是浮点数，这取决于Lua的转换文法（参见§3.1）。（字符串可以有前置和后置的空格，可以带符号。）
当传入base调用它时，e必须是一个以该进制表示的整数字符串。进制可以是2到36（包含2和36）之间的任何整数。大于10进制时，字母'A'（大小写均可）表示10，'B'表示11，依次到'Z'表示35。如果字符串e不是该进制下的合法数字，函数返回nil。
### tostring
tostring(v)
可以接收任何类型，它将其转换为人可阅读的字符串形式。浮点数总被转换为浮点数的表现形式（小数点形式或是指数形式）。（如果想完全控制数字如何被转换，可以使用string.format。）
如果v有"__tostring"域的元表，tostring会以v为参数调用它。并用它的结果作为返回值。
### type
type(v)
将参数的类型编码为一个字符串返回。函数可能的返回值有"nil"（一个字符串，而不是nil值），"number"，"string"，"boolean"，"table"，"function"，"thread"，"userdata"。
### _VERSION
_VERSION
一个包含有当前解释器版本号的全局变量（并非函数）。当前这个变量的值为"Lua5.3"。
xpcall(f,msgh[,arg1,···])
这个函数和pcall类似。不过它可以额外设置一个消息处理器msgh
## 元表
Lua中的每个值都可以有一个元表。这个元表就是一个普通的Lua表，它用于定义原始值在特定操作下的行为。如果你想改变一个值在特定操作下的行为，你可以在它的元表中设置对应域。例如，当你对非数字值做加操作时，Lua会检查该值的元表中的**__add**域下的函数。如果能找到，Lua则调用这个函数来完成加这个操作。
Lua中的每个值都有一套预定义的操作集合，如数字相加等。但无法将两个table相加，此时可通过元表修改一个值的行为，使其在面对一个非预定义的操作时执行一个指定操作。``
访问机制：
一般的元方法都只针对Lua的核心，也就是一个虚拟机。它会检测一个操作中的值是否有元表，这些元表是否定义了关于次操作的元方法。例如两个table相加，先检查两者之一是否有元表，之后检查是否有一个叫**__add**的字段，若找到，则调用对应的值。**__add**等即时字段，其对应的值（往往是一个函数或是table）就是“元方法”
## 元方法
元表中的键对应着不同的事件名，键关联的那些值被称为元方法。在上面那个例子中引用的事件为"add"，完成加操作的那个函数就是元方法。
## 元表的作用
[1].修改表的操作符功能或为操作符添加新功能
[2].使用元表中的**__index**方法，我们可以实现在表中查找键不存在时转而在元表中查找键值的功能
## 对元表的处理
Lua提供了两个十分重要的用来处理元表的方法，如下：
setmetatable(table,metatable):此方法用于为一个表设置元表，setmetatable函数会返回第一个参数
getmetatable(table)：此方法用于获取表的元表对象
## 实例
```lua
将其中的一个表设为另一个表的元表
table={}
metatable={}
setmetatable(table,metatable)
简写如下
table=setmetatable(table,metatable)
**__index**元方法,如下，完成了在原表中查找键不存在时转而去元表中查找该键的功能
ytable=setmetatable({key1="value"},{
	__index=function(ytable,key)
		ifkey=="key2"then
			return"metaTableValue"
		else
			returnytable[key]
		end
	end
})
print(ytable.key1,ytable.key2)==>>valuemetaTableValue
实例1运行过程分析：
1.表ytable为{key="value"}
2.为ytable设置了一个元表，该元表的键**__index**存储了一个函数，我们称这个函数为元方法。
3.这个元方法的工作也十分简单，它仅查找索引“key2”,如果找到该索引值，则返回"metaTableValue",否则返回ytable中索引对应的值

简化上述表：
ytable=setmetatable({key1="value"},{
	__index={
		key2="metaTableValue"
	}
})
print(ytable.key1,ytable.key2)
```
##参考书籍
[1].[Lua5.3参考手册](https://cloudwu.github.io/lua53doc/manual.html)