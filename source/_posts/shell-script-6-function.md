---
title: shell脚本(6)创建函数
date: 2018-02-24 23:33:50
toc: true
tags: 
    - Linux
    - Shell
---
## 基本的脚本函数

函数是一个脚本代码块，你可以为其命名并在代码中任何位置重用。

### 创建函数

```bash
第一种格式采用关键字 function：
function name { 
    commands 
} 

第二种格式更接近于其他编程语言:
# 函数名后的空括号表明正在定义的是一个函数
name() { 
commands 
} 
```

### 使用函数

```bash
function func1 { 
   echo "This is an example of a function" 
} 
 
func1 # 使用函数名调用函数 
# 函数需要先定义后使用
# 同名函数，使用最后定义的函数

```

## 返回值

bash  shell会把函数当作一个小型脚本，运行结束时会返回一个退出状态码

### 默认退出状态码 

默认情况下，函数的退出状态码是函数中最后一条命令返回的退出状态码。在函数执行结束后，可以用标准变量 `$? `来确定函数的退出状态码。

```bash
$ cat test4 
#!/bin/bash 
# testing the exit status of a function 

func1() { 
   echo "trying to display a non-existent file" 
   ls -l badfile 
} 
 
echo "testing the function: " 
func1 
echo "The exit status is: $?" 
$  
$ ./test4 
testing the function: 
trying to display a non-existent file 
ls: badfile: No such file or directory 
The exit status is: 1 
$ 
```

### 使用 return 命令 

bash shell使用 return 命令来退出函数并返回特定的退出状态码。 return 命令允许指定一个整数值来定义函数的退出状态码，从而提供了一种简单的途径来编程设定函数退出状态码。

```bash
$ cat test5 
#!/bin/bash 
# using the return command in a function 
 
function dbl { 
   read -p "Enter a value: " value 
   echo "doubling the value" 
   return $[ $value * 2 ] 
} 
 
dbl 
echo "The new value is $?" 
$ 
# 函数一结束就取返回值
# 退出状态码必须是0~255
```

### 使用函数输出

可以将函数的输出保存到变量中

```bash
$ cat test5b 
#!/bin/bash 
# using the echo to return a value 
 
function dbl { 
   read -p "Enter a value: " value 
   echo $[ $value * 2 ] 
} 
 
result=$(dbl) 
echo "The new value is $result" 
$  
$ ./test5b 
Enter a value: 200 
The new value is 400 
$  
```

## 在函数中使用变量 

### 向函数传递参数

bash shell会将函数当作小型脚本来对待,这意味着你可以像普通脚本那样向函数传递参数。函数可以使用标准的参数环境变量来表示命令行上传给函数的参数。例如，函数名会在 `$0`变量中定义，函数命令行上的任何参数都会通过 `$1` 、 `$2` 等定义。也可以用特殊变量` $#` 来判断传给函数的参数数目

在脚本中指定函数时，必须将参数和函数放在同一行，像这样： 

```bash
func1 $value1 10
```

```bash
#!/bin/bash 
# passing parameters to a function 
 
function addem { 
   if [ $# -eq 0 ] || [ $# -gt 2 ] 
   then 
      echo -1 
   elif [ $# -eq 1 ] 
   then 
      echo $[ $1 + $1 ] 
   else 
      echo $[ $1 + $2 ] 
   fi 
   } 
echo -n "Adding 10 and 15: " 
value=$(addem 10 15) 
echo $value 

```

```bash
# 获取命令行中的参数 传递给函数
$ cat test7 
#!/bin/bash 
# trying to access script parameters inside a function 
 
function func7 { 
   echo $[ $1 * $2 ] 
} 
 
if [ $# -eq 2 ] 
then 
   value=$(func7 $1 $2) 
   echo "The result is $value" 
else 
   echo "Usage: badtest1 a b" 
fi 
$  
$ ./test7 
Usage: badtest1 a b 
$ ./test7 10 15 
The result is 150 
$ 
```

### 在函数中处理变量

#### 1.全局变量

全局变量是在shell脚本中任何地方都有效的变量。默认情况下，你在脚本中定义的任何变量都是全局变量。在函数外定义的变量可在函数内正常访问。 

```bash
$ cat test8 
#!/bin/bash 
# using a global variable to pass a value 
function dbl { 
   value=$[ $value * 2 ] 
} 
 
read -p "Enter a value: " value 
dbl 
echo "The new value is: $value" 
$  
$ ./test8 
Enter a value: 450 
The new value is: 900 
$ 
```

#### 2.局部变量 

函数内部使用的任何变量都可以被声明成局部变量。定义局部变量时，需要在变量名前加上local关键字

```bash
$ cat test9 
#!/bin/bash 
# demonstrating the local keyword 
 
function func1 { 
   local temp=$[ $value + 5 ] 
   result=$[ $temp * 2 ] 
} 
 
temp=4 
value=6 
 
func1 
echo "The result is $result" 
if [ $temp -gt $value ] 
then 
   echo "temp is larger" 
else 
   echo "temp is smaller" 
fi 
$  
$ ./test9 
The result is 22 
temp is smaller 
$ 
```

## 数组变量和函数

### 向函数传数组参数 

需要将数组变量的值分解成单个的值，然后将这些值作为函数参数使用。在函数内部，可以将所有的参数重新组合成一个新的变量。

```bash
$ cat test11 
#!/bin/bash 
# adding values in an array 
function addarray { 
   local sum=0 
   local newarray 
   newarray=($(echo "$@")) 
   for value in ${newarray[*]} 
   do 
      sum=$[ $sum + $value ] 
   done 
   echo $sum 
} 
 
myarray=(1 2 3 4 5) 
echo "The original array is: ${myarray[*]}" 
arg1=$(echo ${myarray[*]}) 
result=$(addarray $arg1) 
echo "The result is $result" 
$  
$ ./test11 
The original array is: 1 2 3 4 5 
The result is 15 
$ 
```

### 从函数返回数组 

从函数里向shell脚本传回数组变量也用类似的方法。函数用 echo 语句来按正确顺序输出单个数组值，然后脚本再将它们重新放进一个新的数组变量中。

```bash
$ cat test12 
#!/bin/bash 
# returning an array value 
 
function arraydblr { 
   local origarray 
   local newarray 
   local elements 
   local i 
   origarray=($(echo "$@")) 
   newarray=($(echo "$@")) 
   elements=$[ $# - 1 ] 
   for (( i = 0; i <= $elements; i++ )) 
   { 
      newarray[$i]=$[ ${origarray[$i]} * 2 ] 
   } 
   echo ${newarray[*]} 
} 
 
myarray=(1 2 3 4 5) 
echo "The original array is: ${myarray[*]}" 
arg1=$(echo ${myarray[*]}) 
result=($(arraydblr $arg1)) 
echo "The new array is: ${result[*]}" 
$  
$ ./test12 
The original array is: 1 2 3 4 5 
The new array is: 2 4 6 8 10 
```

## 函数递归 

```bash
# 阶乘
$ cat test13 
#!/bin/bash 
# using recursion 
 
function factorial { 
   if [ $1 -eq 1 ] 
   then 
      echo 1 
   else 
      local temp=$[ $1 - 1 ] 
      local result=$(factorial $temp) 
      echo $[ $result * $1 ] 
   fi 
}   
read -p "Enter value: " value 
result=$(factorial $value) 
echo "The factorial of $value is: $result" 
$  
$ ./test13 
Enter value: 5 
The factorial of 5 is: 120 
$ 
```

## 创建库

bash shell允许创建函数库文件，然后在多个脚本中引用该库文件。

第一步是创建一个包含脚本中所需函数的公用库文件

```bash
$ cat myfuncs 
# my script functions 
 
function addem { 
   echo $[ $1 + $2 ] 
} 
 
function multem { 
   echo $[ $1 * $2 ] 
} 
 
function divem { 
   if [ $2 -ne 0 ] 
   then 
      echo $[ $1 / $2 ] 
   else 
      echo -1 
   fi 
} 
```

下一步是在用到这些函数的脚本文件中包含myfuncs库文件。

和环境变量一样，shell函数仅在定义它的shell会话内有效。如果你在shell命令行界面的提示符下运行myfuncs  shell脚本，shell会创建一个新的shell并在其中运行这个脚本。它会为那个新shell定义这三个函数，但当你运行另外一个要用到这些函数的脚本时，它们是无法使用的。

使用函数库的关键在于 source 命令。 source 命令会在当前shell上下文中执行命令，而不是创建一个新shell。可以用 source 命令来在shell脚本中运行库文件脚本。这样脚本就可以使用库中的函数了。

source 命令有个快捷的别名，称作点操作符（dot  operator）。要在shell脚本中运行myfuncs库文件，只需添加下面这行： 

```bash
. ./myfuncs  # 第1个点表示点操作符， 第2个点表示当前目录
```

```bash
$ cat test14 
#!/bin/bash 
# using functions defined in a library file 
. ./myfuncs 
 
value1=10 
value2=5 
result1=$(addem $value1 $value2) 
result2=$(multem $value1 $value2) 
result3=$(divem $value1 $value2) 
echo "The result of adding them is: $result1" 
echo "The result of multiplying them is: $result2" 
echo "The result of dividing them is: $result3" 
$  
$ ./test14 
The result of adding them is: 15 
The result of multiplying them is: 50 
The result of dividing them is: 2 
$ 
```

## 在命令行上使用函数

### 在.bashrc 文件中定义函数 

bash shell在每次启动时都会在主目录下查找.bashrc文件并运行

#### 1.直接定义函数 

可以直接在主目录下的.bashrc文件中定义函数。把你写的函数放在文件末尾就行了

```bash
$ cat .bashrc 
# .bashrc 
 
# Source global definitions 
if [ -r /etc/bashrc ]; then 
        . /etc/bashrc 
fi 

# 该函数会在下次启动新bash shell时生效。
function addem { 
   echo $[ $1 + $2 ] 
} 
$ 
```

#### 2.读取函数文件 

只要是在shell脚本中，都可以用 source 命令（点操作符）将库文件中的函数添加到你的.bashrc脚本中。shell还会将定义好的函数传给子shell进程

```bash
$ cat .bashrc 
# .bashrc 
 
# Source global definitions 
if [ -r /etc/bashrc ]; then 
        . /etc/bashrc 
fi 
 
. /home/rich/libraries/myfuncs 
$ 
```

## 参考书目

《Linux命令行与shell脚本编程大全》第3版