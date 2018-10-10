---
title: SHELL常遇到的问题总结以及基本操作
date: 2018-05-12 17:22:55
tags:
categories: SHELL
---
## 基本建议
### set -e
Every script you write should include set -e at the top. This tells bash that it should exit the script if any statement returns a non-true return value. The benefit of using -e is that it prevents errors snowballing into serious issues when they could have been caught earlier. Again, for readability you may want to use set -o errexit.
Using -e gives you error checking for free. If you forget to check something, bash will do it for you. Unfortunately it means you can't check $? as bash will never get to the checking code if it isn't zero.
### set -x
会在执行每一行 shell 脚本时，把执行的内容输出来。它可以让你看到当前执行的情况，里面涉及的变量也会被替换成实际的值
### shellcheck
主要作用
[1]. 它指出并解释了典型的初学者的语法问题，导致shell给出隐含的错误消息。
[2]. 它指出并解释了典型的中级语义问题，导致shell的行为奇怪和反直觉。
[3]. 它还指出了可能导致高级用户的其他工作脚本在未来情况下失败的微妙注意事项，角落和陷阱。
### 注意local
在 bash，如果不加 local 限定词，变量默认都是全局的，在顶级作用域里，是否是全局变量并不重要。但是在函数里面，声明一个全局变量可能会污染到其他作用域（尤其在你根本没有注意到这一点的情况下）。所以，对于在函数内声明的变量，请务必记得加上 local 限定词。
### trap 信号
trap 的主要应用场景并不是捕获哪个信号。trap 命令支持“捕获”许多不同的流程——准确来说，允许用户给特定的流程注入函数调用，其中最为常用的是trap func EXIT和trap func ERR。
**trap func EXIT**允许在脚本结束时调用函数。由于无论正常退出抑或异常退出，所注册的函数都能得以调用，在需要调用一个清理函数的场景下，我都是用它注册清理函数，而不是简单地在脚本结尾调用清理函数。
**trap func ERR**允许在运行出错时调用函数。一个常用的技法是，使用全局变量ERROR存储错误信息，然后在注册的函数中根据存储的值完成对应的错误报告。把原本四分五裂的错误处理逻辑集中到一处，有时候会起奇效。不过要记住，程序异常退出时，既会调用EXIT注册的函数，也会调用ERR注册的函数
其他常用操作
**trap 'command_list'  signals**
其中，command_list 是一个命令清单，可以包含一个函数，在接收到信号列表中包含的某个信号后运行。而 signals 是将要捕捉的信号的列表
**trap ''  signals**	忽略信号
**trap - signals**	重置trap


## SHELL中对文件的操作
    Conditional Logic on Files
    -a file exists.
    -b file exists and is a block special file.
    -c file exists and is a character special file.
    -d file exists and is a directory.
    -e file exists (just the same as -a).
    -f file exists and is a regular file.
    -g file exists and has its setgid(2) bit set.
    -G file exists and has the same group ID as this process.
    -k file exists and has its sticky bit set.
    -L file exists and is a symbolic link.
    -n string length is not zero.
    -o Named option is set on.
    -O file exists and is owned by the user ID of this process.
    -p file exists and is a first in, first out (FIFO) special file or
    named pipe.
    -r file exists and is readable by the current process.
    -s file exists and has a size greater than zero.
    -S file exists and is a socket.
    -t file descriptor number fildes is open and associated with a
    terminal device.
    -u file exists and has its setuid(2) bit set.
    -w file exists and is writable by the current process.
    -x file exists and is executable by the current process.
    -z string length is zero.

## SHELL中对字符串的操作
### 字符串连接
字符串的连接虽然没有lua中那么方便，也是相当简单的。如果想连接两个字符串，可以用一下方法，例如
```bash
--------------------------
$value1=home
$value2=${value1}"="
--------------------------
var1=/etc/
var2=yum.repos.d/
var3=${var1}${var2}
--------------------------
```
### 字符串长度
    ${#string}	$string的长度
### 字符串的读取以及替换
    说明："* $substring”可以是一个正则表达式.

    ${string:position}	在$string中, 从位置$position开始提取子串
    ${string:position:length}	在$string中, 从位置$position开始提取长度为$length的子串
    ${string#substring}	从变量$string的开头, 删除最短匹配$substring的子串
    ${string##substring}	从变量$string的开头, 删除最长匹配$substring的子串
    ${string%substring}	从变量$string的结尾, 删除最短匹配$substring的子串
    ${string%%substring}	从变量$string的结尾, 删除最长匹配$substring的子串
    ${string/substring/replacement}	使用$replacement, 来代替第一个匹配的$substring
    ${string//substring/replacement}	使用$replacement, 代替所有匹配的$substring
    ${string/#substring/replacement}	如果$string的前缀匹配$substring, 那么就用$replacement来代替匹配到的$substring
    ${string/%substring/replacement}	如果$string的后缀匹配$substring, 那么就用$replacement来代替匹配到的$substring
### 数字大小比较
```shell
-eq		等于
-ne		不等于
-gt		大于
-lt		小于
-ge		大于等于
-le		小于等于
```