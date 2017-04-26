---
layout:		post
title:			"Where There Is A Shell,There Is A Way!"
subtitle:		"Shell学习笔记"
date:			2017-4-7 23:00:00
author:		"Parsifal"
header-img:	"img/postresources/linux-shell-scripting.png"
catalog:     true
abstract:    "-  `Shell Script`在日常工作中接触地挺多，比如CI中的打包发布等都需要`Shell Script`进行辅助完成。然而一直没有进行系统地学习，都是东凑西凑地完成一个Just Works的脚本。现在打算利用闲余时间，较为系统地去学习下这门技术，并以此文作为学习笔记记录。 
				"
tags:
- 侃侃技术
- Shell
---
## 目录    
{:.no_toc}    
1.    
{:toc}

## 扯扯闲话
`Shell Script`在日常工作中接触地挺多，比如CI中的打包发布等都需要`Shell Script`进行辅助完成。然而一直没有进行系统地学习，都是东凑西凑地完成一个Just Works的脚本。现在打算利用闲余时间，较为系统地去学习下这门技术，并以此文作为学习笔记记录。

## 概念
**Shell**是源于**Linux**系统。**Shell**在**Linux**系统中是作为用户与内核间的桥梁存在，通过**Shell**用户可以方便地操作**Linux**执行一些命令。同时**Shell**又可以理解为一种**命令式语言**和**程序设计式语言**。作为编程语言来说，**Shell**是一个解释型的语言，无编译过程，只需要一个解释器就可以运行。Mac下自带多种**Shell**解释器，如Bourne Shell（sh，Unix标配）、Bourne Again Shell（bash，Linux标配）、Z Shell（zsh，结合Oh My Zsh成为众多程序员在Mac系统里的首选Shell）等。

> Shell 是指一种应用程序，这个应用程序提供了一个界面，用户通过这个界面访问操作系统内核的服务。

**Shell Script**就是写给**Shell**执行的脚本，平常所说的**Shell**也多指代这种**Shell Script**。由于是解释型的语言，所以**Shell Script**只需要一个文本编辑器就可以编写，无需繁重的IDE支持（这里吐槽**Xcode**）。Mac下推荐**TextMate**和**Sublime Text**，都是很好用的编辑器。

## 变量
变量命名规则：
> * 首个字符必须为字母（a-z，A-Z）。
> * 中间不能有空格，可以使用下划线（_）。
> * 不能使用标点符号。
> * 不能使用bash里的关键字（可用help命令查看保留关键字）。

简单语法运用：

```shell
# 只读 删除 字符串拼接
readonly first_name="Parsifal"
readonly last_name="Zhang"
full_name="$first_name $last_name"
echo "I'm ${full_name}"
unset full_name

#数组 字符串截取 字符串长度
chars="abcdefghijklmnopqrstuvwxyz"
count=${#chars}
for (( i = 0; i < count ; i++ )); do
	letter=${chars:$i:1}
	array[i]=${letter}
	echo ${array[i]}
done

echo "array count:${#array[*]} firt item length:${#array[0]}"
```

变量还可以支持被替换，语法如下表：
<table border="1px">
<tbody>
<tr><th>形式</th><th>说明</th></tr>
<tr>
<td>`${var}`</td>
<td>变量本来的值</td>
</tr>
<tr>
<td>`${var:-word}`</td>
<td>如果变量 var 为空或已被删除(unset)，那么返回;word，但不改变;var 的值。</td>
</tr>
<tr>
<td>`${var:=word}`</td>
<td>如果变量 var 为空或已被删除(unset)，那么返回 word，并将 var 的值设置为 word。</td>
</tr>
<tr>
<td>`${var:?message}`</td>
<td>如果变量 var 为空或已被删除(unset)，那么将消息 message 送到标准错误输出，可以用来检测变量 var 是否可以被正常赋值。若此替换出现在Shell脚本中，那么脚本将停止运行。</td>
</tr>
<tr>
<td>`${var:+word}`</td>
<td>如果变量 var 被定义，那么返回 word，但不改变 var 的值。</td>
</tr>
</tbody></table>

## 传参
传参用到的字符：
<table border="1.0">
<tr>
<th>参数处理</th>
<th>说明</th>
</tr>
<tr><td>`$n`</td><td>n 代表一个数字，1 为执行脚本的第一个参数，2 为执行脚本的第二个参数，以此类推……</td></tr>
<tr><td>`$#`</td><td>传递到脚本的参数个数</td></tr>
<tr><td>`$#`</td><td>以一个单字符串显示所有向脚本传递的参数。
如"$*"用「"」括起来的情况、以"$1 $2 … $n"的形式输出所有参数。</td></tr>
<tr><td>`$$`</td><td>脚本运行的当前进程ID号</td></tr>
<tr><td>`$!`</td><td>后台运行的最后一个进程的ID号</td></tr>
<tr><td>`$@`</td><td>与$*相同，但是使用时加引号，并在引号中返回每个参数。
如"$@"用「"」括起来的情况、以"$1" "$2" … "$n" 的形式输出所有参数。</td></tr>
<tr><td>`$-`</td><td> 显示Shell使用的当前选项，与set命令功能相同。</td></tr>
<tr><td>`$?`</td><td>显示最后命令的退出状态。0表示没有错误，其他任何值表明有错误。同样也可以用于获取函数返回值</td></tr>
</table>

```shell
#!/bin/bash
echo "这是个shell script:$0"
inputCount=$#
echo "你输入的参数个数有：${inputCount}"
echo "它们是：$*"

for i in "$@"; do
	echo "${i}"
done
```

## 数组
**Bash Shell**只支持一维数组，并不支持多维数组。

```shell
#!/bin/bash
array=(a b c d e f g)

echo "item count:${#array[*]}"
echo "items:${array[*]}"

for item in ${array[@]}; do
	echo "item:${item}"
done
```

## 命令替换
可使用反引号进行命令替换：

```shell
#!/bin/bash
#通过单引号 可运行命令
DATE=`date`
echo "Current date is $DATE"

USERS=`who`
echo "Current users are $USERS"

UPDATETIME=`uptime`
echo "Uptime is $UPDATETIME"
```

## 运算符
原生Bash不支持简单的算数运算符，需要借助其他命令实现，如`awk`和`expr`，`expr`较为常用。`expr`是一款表达式计算工具，使用它能完成表达式的求值操作。
<table border="1px"><caption>算术运算符列表</caption>
<tbody>
<tr><th>运算符</th><th>说明</th><th>举例</th></tr>
<tr>
<td>`+`</td>
<td>加法</td>
<td>`expr $a + $b` 结果为;30。</td>
</tr>
<tr>
<td>`-`</td>
<td>减法</td>
<td>`expr $a - $b` 结果为 10。</td>
</tr>
<tr>
<td>`*`</td>
<td>乘法</td>
<td>`expr $a \* $b` 结果为 ;200。</td>
</tr>
<tr>
<td>`/`</td>
<td>除法</td>
<td>`expr $b / $a` 结果为;2。</td>
</tr>
<tr>
<td>`%`</td>
<td>取余</td>
<td>`expr $b % $a` 结果为;0。</td>
</tr>
<tr>
<td>`=`</td>
<td>赋值</td>
<td>`a=$b` 将把变量 b 的值赋给 a。</td>
</tr>
<tr>
<td>`==`</td>
<td>相等。用于比较两个数字，相同则返回 true。</td>
<td>[ $a == $b ] 返回false。</td>
</tr>
<tr>
<td>`!=`</td>
<td>不相等。用于比较两个数字，不相同则返回 true。</td>
<td>`[ $a != $b ]` 返回 true。</td>
</tr>
</tbody>
</table>` 
`<table border="1px"><caption>关系运算符列表</caption>
<tbody>
<tr><th>运算符</th><th>说明</th><th>举例</th></tr>
<tr>
<td>`-eq`</td>
<td>检测两个数是否相等，相等返回 true。</td>
<td>`[ $a -eq $b ]` 返回;true。</td>
</tr>
<tr>
<td>`-ne`</td>
<td>检测两个数是否相等，不相等返回 true。</td>
<td>`[ $a -ne $b ]` 返回 true。</td>
</tr>
<tr>
<td>`-gt`</td>
<td>检测左边的数是否大于右边的，如果是，则返回 true。</td>
<td>`[ $a -gt $b ]` 返回 false。</td>
</tr>
<tr>
<td>`-lt`</td>
<td>检测左边的数是否小于右边的，如果是，则返回 true。</td>
<td>`[ $a -lt $b ]` 返回 true。</td>
</tr>
<tr>
<td>`-ge`</td>
<td>检测左边的数是否大等于右边的，如果是，则返回 true。</td>
<td>`[ $a -ge $b ]` 返回 false。</td>
</tr>
<tr>
<td>`-le`</td>
<td>检测左边的数是否小于等于右边的，如果是，则返回 true。</td>
<td>`[ $a -le $b ]` 返回 true。</td>
</tr>
</tbody>
</table>

`
`
<table border="1px"><caption>布尔运算符列表</caption>
<tbody>
<tr><th>运算符</th><th>说明</th><th>举例</th></tr>
<tr>
<td>`!`</td>
<td>非运算，表达式为 true 则返回 false，否则返回 true。</td>
<td>`[ ! false ]` 返回 true。</td>
</tr>
<tr>
<td>`-o`</td>
<td>或运算，有一个表达式为 true 则返回 true。</td>
<td>`[ $a -lt 20 -o $b -gt 100 ]` 返回true。</td>
</tr>
<tr>
<td>`-a`</td>
<td>与运算，两个表达式都为 true 才返回 true。</td>
<td>`[ $a -lt 20 -a $b -gt 100 ]` 返回false。</td>
</tr>
</tbody>
</table>`
`<table border="1px"><caption>字符串运算符列表</caption>
<tbody>
<tr><th>运算符</th><th>说明</th><th>举例</th></tr>
<tr>
<td>`=`</td>
<td>检测两个字符串是否相等，相等返回 true。</td>
<td>`[ $a = $b ]` 返回 false。</td>
</tr>
<tr>
<td>`!=`</td>
<td>检测两个字符串是否相等，不相等返回 true。</td>
<td>`[ $a != $b ]` 返回;true。</td>
</tr>
<tr>
<td>`-z`</td>
<td>检测字符串长度是否为0，为0返回 true。</td>
<td>`[ -z $a ]` 返回 false。</td>
</tr>
<tr>
<td>`-n`</td>
<td>检测字符串长度是否为0，不为0返回 true。</td>
<td>`[ -z $a ]` 返回 true。</td>
</tr>
<tr>
<td>`str`</td>
<td>检测字符串是否为空，不为空返回 true。</td>
<td>`[ $a ]` 返回;true。</td>
</tr>
</tbody>
</table>

`
`
<table border="1px"><caption>文件测试运算符列表</caption>
<tbody>
<tr><th>操作符</th><th>说明</th><th>举例</th></tr>
<tr>
<td>`-b file`</td>
<td>检测文件是否是块设备文件，如果是，则返回 true。</td>
<td>`[ -b $file ]` 返回 false。</td>
</tr>
<tr>
<td>`-c file`</td>
<td>检测文件是否是字符设备文件，如果是，则返回 true。</td>
<td>`[ -b $file ]` 返回;false。</td>
</tr>
<tr>
<td>`-d file`</td>
<td>检测文件是否是目录，如果是，则返回 true。</td>
<td>`[ -d $file ]` 返回 false。</td>
</tr>
<tr>
<td>`-f file`</td>
<td>检测文件是否是普通文件（既不是目录，也不是设备文件），如果是，则返回 true。</td>
<td>`[ -f $file ]` 返回;true。</td>
</tr>
<tr>
<td>`-g file`</td>
<td>检测文件是否设置了 SGID 位，如果是，则返回 true。</td>
<td>`[ -g $file ]` 返回;false。</td>
</tr>
<tr>
<td>`-k file`</td>
<td>检测文件是否设置了粘着位(Sticky Bit)，如果是，则返回 true。</td>
<td>`[ -k $file ]` 返回;false。</td>
</tr>
<tr>
<td>`-p file`</td>
<td>检测文件是否是具名管道，如果是，则返回 true。</td>
<td>`[ -p $file ]` 返回;false。</td>
</tr>
<tr>
<td>`-u file`</td>
<td>检测文件是否设置了 SUID 位，如果是，则返回 true。</td>
<td>`[ -u $file ]` 返回;false。</td>
</tr>
<tr>
<td>`-r file`</td>
<td>检测文件是否可读，如果是，则返回 true。</td>
<td>[ -r $file ] 返回;true。</td>
</tr>
<tr>
<td>`-w file`</td>
<td>检测文件是否可写，如果是，则返回 true。</td>
<td>`[ -w $file ]` 返回;true。</td>
</tr>
<tr>
<td>`-x file`</td>
<td>检测文件是否可执行，如果是，则返回 true。</td>
<td>`[ -x $file ]` 返回;true。</td>
</tr>
<tr>
<td>`-s file`</td>
<td>检测文件是否为空（文件大小是否大于0），不为空返回 true。</td>
<td>`[ -s $file ]` 返回;true。</td>
</tr>
<tr>
<td>`-e file`</td>
<td>检测文件（包括目录）是否存在，如果是，则返回 true。</td>
<td>`[ -e $file ]` 返回;true。</td>
</tr>
</tbody>
</table>

## 条件语句

**if条件语句：**

**1）if ... else语句：**

```shell
if [ expression ]
then
   Statement(s) to be executed if expression is true
fi
```

**2) if ... else ... fi 语句：**

```shell
if [ expression ]
then
   Statement(s) to be executed if expression is true
else
   Statement(s) to be executed if expression is not true
fi
```

**3) if ... elif ... fi 语句：**

```shell
if [ expression 1 ]
then
   Statement(s) to be executed if expression 1 is true
elif [ expression 2 ]
then
   Statement(s) to be executed if expression 2 is true
elif [ expression 3 ]
then
   Statement(s) to be executed if expression 3 is true
else
   Statement(s) to be executed if no expression is true
fi
```

**case esac条件语句：**

```shell
case 值 in
模式1)
    command1
    command2
    command3
    ;;
模式2）
    command1
    command2
    command3
    ;;
*)
    command1
    command2
    command3
    ;;
esac
```

## 循环控制语句

**for in循环语句：**

```shell
for 变量 in 列表
do
    command1
    command2
    ...
    commandN
done
```

**while循环语句：**

```shell
while command
do
   Statement(s) to be executed if command is true
done
```

## Shell函数
**Shell**函数的定义格式如下：

```shell
function_name () {
    list of commands
    [ return value ]
}
```

函数返回值，可以显式增加return语句；如果不加，会将最后一条命令运行结果作为返回值。**Shell**函数返回值只能是整数，一般用来表示函数执行成功与否，0表示成功，其他值表示失败。如果**return**其他数据，比如一个字符串，往往会得到错误提示：“numeric argument required”。如果一定要让函数返回字符串，那么可以先定义一个变量，用来接收函数的计算结果，脚本在需要的时候访问这个变量来获得函数返回值。

在Shell中，调用函数时可以向其传递参数。在函数体内部，通过 n的形式来获取参数的值，例如，n的形式来获取参数的值，例如，1表示第一个参数，$2表示第二个参数...
<table border="1px">
<tbody>
<tr><th>特殊变量</span></th><th>说明</th></tr>
<tr>
<td>`$#`</td>
<td>传递给函数的参数个数。</td>
</tr>
<tr>
<td>`$*`</td>
<td>显示所有传递给函数的参数。</td>
</tr>
<tr>
<td>`$@`</td>
<td>与$*相同，但是略有区别，请查看<a href="http://c.biancheng.net/cpp/view/2739.html" target="_blank">Shell特殊变量</a>。</td>
</tr>
<tr>
<td>`$?`</td>
<td>函数的返回值。</td>
</tr>
</tbody>
</table>


## 参考资料
- [Shell 教程](http://www.runoob.com/linux/linux-shell.html)
- [Shell学习笔记](http://www.cnblogs.com/maybe2030/p/5022595.html)