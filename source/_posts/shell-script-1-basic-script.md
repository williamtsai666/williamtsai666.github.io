---
title: shell学习(1)构建基本脚本
date: 2018-02-19 23:33:50
toc: true
tags: 
    - Linux
    - Shell
---
## 使用多个命令 

```bash
# 使用分号分隔多个命令
$ date ; who 
```

## 基本shell脚本文件

```bash
# !/bin/bash
# This script displays the date and who's logged on
echo -n "The time and date are: " #把文本字符串和命令输出显示在同一行
date
echo "Let's see who's logged into the system:"
who
$
# 运行脚本文件
$ test1 
bash: test1: command not found 
PATH 环境变量中没有找到命令，需要通过一下两种方式运行脚本:
1.将shell脚本文件所处的目录添加到 PATH 环境变量中； 
2.在提示符中用绝对或相对文件路径来引用shell脚本文件。
$ ./test1 
```

## 显示消息

```bash
echo Hello World
# 默认情况不需要引号，单双引号混用时才需要引号
-n 不换行
```

## 使用变量

### 环境变量

shell维护着一组环境变量，用来记录特定的系统信息。比如系统的名称、登录到系统上的用户名、用户的系统ID（也称为UID）、用户的默认主目录以及shell查找程序的搜索路径。通过set命令查看当前环境变量列表。

在环境变量名称之前加上美元符($) 来使用这些环境变量,并且可以再双引号中使用

```bash
echo "User info for userid: $USER"
echo UID: $UID
echo HOME: $HOME
echo ${variable} #变量名两侧额外的花括号通常用来帮助识别美元符后的变量名
```

反斜线允许shell脚本将美元符解读为实际的美元符，而不是变量

```bash
$ echo "The cost of the item is \$15"
The cost of the item is $15
```

### 用户变量

shell脚本还允许在脚本中定义和使用自己的变量，用户变量可以是任何由字母、数字或下划线组成的文本字符串，长度不超过20个，区分大小写。

使用等号将值赋给用户变量。在变量、等号和值之间不能出现空格。

变量每次被引用时，都会输出当前赋给它的值。重要的是要记住，引用一个变量值时需要使用美元符，而引用变量来对其进行赋值时则不要使用美元符,

```bash
value1=10
value2=$value1
 
echo The resulting value is $value2
--------
The resulting value is 10
--------
value2=value1
--------
The resulting value is value1
# 没有美元符，shell会将变量名解释成普通的文本字符串
```
### 命令替换

shell脚本中最有用的特性之一就是可以从命令输出中提取信息，并将其赋给变量。把输出赋给变量之后，就可以随意在脚本中使用了。

有两种方法可以将命令输出赋给变量:

```
反引号字符(`)
$() 格式
```

命令替换允许你将shell命令的输出赋给变量，注意，赋值等号和命令替换字符之间没有空格

```bash
testing=`date`
testing=$(date)

$ cat test5 
#!/bin/bash 
testing=$(date) 
echo "The date and time are: " $testing 
$ 
$ chmod u+x test5 
$ ./test5 
The date and time are:  Mon Jan 31 20:23:25 EDT 2014 
$ 
```

 命令替换会创建一个子shell来运行对应的命令。子shell（subshell）是由运行该脚本的shell所创建出来的一个独立的子shell（child shell）。正因如此，由该子shell所执行命令是无法使用脚本中所创建的变量的。 
  在命令行提示符下使用路径 ./ 运行命令的话，也会创建出子shell；要是运行命令的时候不加入路径，就不会创建子shell。如果你使用的是内建的shell命令，并不会涉及子shell。

## 重定向输入和输出

### 输出重定向

最基本的重定向将命令的输出发送到一个文件中。bash shell用大于号(>)来输出重定向,

双大于号（>>）来追加数据

```bash
command > outputfile
command >> outputfile #追加数据
```

### 输入重定向

输入重定向将文件的内容重定向到命令,输入重定向符号是小于号（<）：

```bash
command < inputfile
```


记忆：在命令行上，命令总是在左侧，而重定向符号“指向”数据流动的方向。
``` bash
$ wc < test6
      2      11      60
  -------------------------------------------------   
wc 命令可以对对数据中的文本进行计数。默认情况下，它会输出3个值：
文本的行数
文本的词数
文本的字节数
```
另外一种输入重定向的方法，称为内联输入重定向（inline input redirection）。这种方法无需使用文件进行重定向，只需要在命令行中指定用于输入重定向的数据就可以了

内联输入重定向符号是远小于号（<<）。除了这个符号，你必须指定一个文本标记来划分输入数据的开始和结尾。任何字符串都可作为文本标记，但在数据的开始和结尾文本标记必须一致。

```bash
command << marker
data
marker
```


在命令行上使用内联输入重定向时，shell会用 PS2 环境变量中定义的次提示符来提示输入数据。
``` bash
$ wc << EOF
> test string 1
> test string 2
> test string 3
> EOF
      3       9      42 
$ 
# 次提示符会持续提示，以获取更多的输入数据，直到你输入了作为文本标记的那个字符串。
# wc 命令会对内联输入重定向提供的数据进行行、词和字节计数。
```
## 管道

将一个命令的输出作为另一个命令的输入,这个过程叫作管道连接（piping）,管道被放在命令之间，将一个命令的输出重定向到另一个命令中：

```bash
command1 | command2
```


可以利用管道将 rpm 命令的输出送入 sort 命令来产生结果

```bash
$ rpm -qa | sort
abrt-1.1.14-1.fc14.i686 
abrt-addon-ccpp-1.1.14-1.fc14.i686 
$ rpm -qa | sort > rpm.list
# 可以在一条命令中使用任意多条管道。可以持续地将命令的输出通过管道传给其他命令来细化操作。 
$ rpm -qa | sort | more 
```

## 执行数学运算

### expr 命令（复杂）

```bash
$ expr 1 + 5
6
$ expr 5 \* 2
10
 
var1=10
var2=20
var3=$(expr $var2 / $var1)
```

### 使用方括号

在bash中，在将一个数学运算结果赋给某个变量时，可以用美元符和方括号`$[ operation ] `将数学表达式围起来

```bash
$ var1=$[1 + 5]
$ echo $var1
6
$ var2=$[$var1 * 2]
$ echo $var2
12
$
```
bash shell数学运算符只支持整数运算。需要使用bc命令计算， z shell（zsh）提供了完整的浮点数算术操作。

## 退出脚本

shell中运行的每个命令都使用退出状态码（exit status）告诉shell它已经运行完毕。退出状态码是一个0～255的整数值，在命令结束运行时由命令传给shell。可以捕获这个值并在脚本中使用。

### 查看退出状态码

Linux提供了一个专门的变量 $? 来保存上个已执行命令的退出状态码

```bash
$ date
Sat Jan 15 10:01:30 EDT 2014
$ echo $?
0
```

按照惯例，一个成功结束的命令的退出状态码是 0 。如果一个命令结束时有错误，退出状态码就是一个正数值。

Linux退出状态码 

```
0  命令成功结束 
1  一般性未知错误 
2  不适合的shell命令 
126  命令不可执行 
127  没找到命令 
128  无效的退出参数 
128+x  与Linux信号x相关的严重错误 
130  通过Ctrl+C终止的命令 
255  正常范围之外的退出状态码 
```

### exit 命令

默认情况下，shell脚本会以脚本中的最后一个命令的退出状态码退出,exit 命令允许你在脚本结束时指定一个退出状态码。

```bash
$ cat test13
# !/bin/bash
# testing the exit status
var1=10
var2=30
var3=$[$var1 + $var2]
echo The answer is $var3
exit 5
$
```

也可以在 exit 命令的参数中使用变量。

```bash
exit $var3
```


退出状态码超过255时。shell通过模运算计算退出码,最终的结果是指定的数值除以256后得到的余数

## 参考书目

《Linux命令行与shell脚本编程大全》第3版