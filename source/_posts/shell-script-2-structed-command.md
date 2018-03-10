---
title: shell脚本(2)结构化命令
date: 2018-02-20 23:33:50
toc: true
tags: 
    - Linux
    - Shell
---
逻辑流程控制通常称为结构化命令(structured command)

## 使用 if-then 语句

```bash
if command
then
    commands
fi
```


bash shell的 if 语句会运行 if 后面的那个命令。如果该命令的退出状态码是0（该命令成功运行），位于 then 部分的命令就会被执行,如果该命令的退出状态码是其他值， then部分的命令就不会被执行，bash shell会继续执行脚本中的下一个命令
```bash
$ cat test1.sh
# !/bin/bash
# testing the if statement
if pwd
then
    echo "It worked"
fi
$
```

if-then 语句的另一种形式：

```bash
if command; then
commands 
fi
```

## if-then-else 语句

```bash
if command
then
  commands
else
  commands
fi
```

## 嵌套 if

```bash
if command1 
then 
    command set 1 
elif command2 
then 
   command set 2 
elif command3 
then 
   command set 3 
elif command4 
then 
   command set 4 
fi 
```

## test 命令

test 命令提供了在 if-then 语句中测试不同条件的途径,如果 test 命令中列出的条件成立,test 命令就会退出并返回退出状态码0。
test 命令的格式非常简单。

```bash
test condition
--------------------------------
if test condition
then
  commands
fi
```

 判断内容是否为空

```bash
$ cat test6.sh 
#!/bin/bash 
# Testing the test command 
# 
my_variable="" 
# 
if test $my_variable
then 
   echo "The $my_variable expression returns a True" 
# 
else 
   echo "The $my_variable expression returns a False" 
fi 
$ 
$ ./test6.sh 
The  expression returns a False 
$ 
```

bash shell提供了另一种条件测试方法，无需在 if-then 语句中声明 test 命令。

```bash
if [ condition ]
then
  commands
fi
# 方括号定义了测试条件。注意，第一个方括号之后和第二个方括号之前必须加上一个空格，否则就会报错。
```

test 命令可以判断三类条件：

```
数值比较
字符串比较
文件比较
```

### 数值比较

test命令的数值比较功能

```bash
n1 -eq n2  检查n1是否与n2相等
n1 -ge n2  检查n1是否大于或等于n2 
n1 -gt n2  检查n1是否大于n2 
n1 -le n2  检查n1是否小于或等于n2 
n1 -lt n2  检查n1是否小于n2 
n1 -ne n2  检查n1是否不等于n2 
```

```bash
$ cat numeric_test.sh
# !/bin/bash
# Using numeric test evaluations
# 
value1=10
value2=11
# 
if [ $value1 -gt 5 ] # 测试变量 value1 的值是否大于 5
then
    echo "The test value $value1 is greater than 5"
fi
# 
if [ $value1 -eq $value2 ] # 测试变量 value1 的值是否和变量 value2 的值相等
then
    echo "The values are equal"
else
    echo "The values are different"
fi
# 
$
```

### 字符串比较

字符串比较测试 

```bash
str1 = str2   检查str1是否和str2相同
str1 != str2  检查str1是否和str2不同
str1 < str2   检查str1是否比str2小
str1 > str2   检查str1是否比str2大
-n str1       检查str1的长度是否非0
-z str1       检查str1的长度是否为0

```

#### 1.字符串相等性 

```bash
$ cat test7.sh 
#!/bin/bash 
# testing string equality 
testuser=rich 
# 
if [ $USER = $testuser ] 
then 
   echo "Welcome $testuser" 
fi 
$  
$ ./test7.sh 
Welcome rich 
$ 
```

记住，在比较字符串的相等性时，比较测试会将所有的标点和大小写情况都考虑在内。

#### 2.字符串顺序

大于号和小于号必须转义，否则shell会把它们当作重定向符号，把字符串值当作文件名；
大于和小于顺序和 sort 命令所采用的不同。

比较测试中使用的是标准的ASCII顺序，根据每个字符的ASCII数值来决定排序结果。 sort命令使用的是系统的本地化语言设置中定义的排序顺序

```bash
$ cat test9.sh
# !/bin/bash
# mis-using string comparisons
# 
val1=baseball
val2=hockey
# 
if [ $val1 \> $val2 ]
  then
  echo "$val1 is greater than $val2"
else
  echo "$val1 is less than $val2"
fi
```

注意,test 命令和测试表达式使用标准的数学比较符号来表示字符串比较，而用文本代码来表示数值比较

#### 3.字符串大小

```bash
$ cat test10.sh 
#!/bin/bash 
# testing string length 
val1=testing 
val2='' 
# 
if [ -n $val1 ] 
then 
   echo "The string '$val1' is not empty" 
else 
   echo "The string '$val1' is empty" 
fi 
# 
if [ -z $val2 ] 
then 
   echo "The string '$val2' is empty" 
else 
   echo "The string '$val2' is not empty" 
fi 

if [ -z $val3 ] # 未被定义过，长度为0
then 
   echo "The string '$val3' is empty" 
else 
   echo "The string '$val3' is not empty" 
fi 
$  
$ ./test10.sh 
The string 'testing' is not empty 
The string '' is empty 
The string '' is empty 
$ 
```

### 文件比较

文件比较是shell编程中最为强大、也是用得最多的比较形式。它允许你测试Linux文件系统上文件和目录的状态。

test命令的文件比较功能

```bash
-d file  检查file是否存在并是一个目录
-e file  检查file是否存在
-f file  检查file是否存在并是一个文件
-r file  检查file是否存在并可读
-s file  检查file是否存在并非空
-w file  检查file是否存在并可写
-x file  检查file是否存在并可执行
-O file  检查file是否存在并属当前用户所有
-G file  检查file是否存在并且默认组与当前用户相同
file1 -nt file2  检查file1是否比file2新
file1 -ot file2  检查file1是否比file2旧
```

#### 1.检查目录

```bash
$ cat test11.sh
# !/bin/bash
# Look before you leap
# 
jump_directory=/home/arthur
# 
if [ -d $jump_directory ]
then
  echo "The $jump_directory directory exists"
  cd $jump_directory
  ls
else
  echo "The $jump_directory directory does not exist"
fi
# $
$ ./test11.sh
The /home/arthur directory does not exist
$
```

## 复合条件测试

if-then 语句允许你使用布尔逻辑来组合测试。有两种布尔运算符可用：

```bash
[ condition1 ] && [ condition2 ]
[ condition1 ] || [ condition2 ] 
```

```bash
$ cat test22.sh 
#!/bin/bash 
if [ -d $HOME ] && [ -w $HOME/testing ] 
then 
   echo "The file exists and you can write to it" 
else 
   echo "I cannot write to the file" 
fi 
```

## if-then 的高级特性

bash shell提供了两项可在 if-then 语句中使用的高级特性：

> 用于数学表达式的双括号

>  用于高级字符串处理功能的双方括号

### 使用双括号

双括号命令允许你在比较过程中使用高级数学表达式。 test 命令只能在比较中使用简单的
算术操作,双括号命令的格式如下：

```bash
(( expression ))

```

双括号命令符号
```bash
val++  后增
val--  后减
++val  先增
--val  先减
!      逻辑求反
~      位求反
**     幂运算
<<     左位移
>>     右位移
&      位布尔和
|      位布尔或
&&     逻辑和
||     逻辑或
```

```bash
$ cat test23.sh
# !/bin/bash
# using double parenthesis
#
val1=10
# 
if (( $val1 ** 2 > 90 )) # 不需要将双括号中表达式里的大于号转义
then
  (( val2 = $val1 ** 2 ))
  echo "The square of $val1 is $val2"
fi
$ 
$ ./test23.sh
The square of 10 is 100
$
```

### 使用双方括号

双方括号命令提供了针对字符串比较的高级特性。双方括号命令的格式如下：

```bash
[[ expression ]]
```

双方括号里的 expression 使用了 test 命令中采用的标准字符串比较。但它提供了 test 命令未提供的另一个特性——模式匹配（pattern matching）。

在模式匹配中，可以定义一个正则表达式来匹配字符串值。
```bash
$ cat test24.sh
# !/bin/bash
# using pattern matching
# 
if [[ $USER == r* ]]
then
  echo "Hello $USER"
else
  echo "Sorry, I do not know you"
fi
$ 
$ ./test24.sh
Hello rich
$
```

## case 命令

case 命令会采用列表格式来检查单个变量的多个值

```bash
case variable in
pattern1 | pattern2) commands1;;
pattern3) commands2;;
*) default commands;;
esac
```

case 命令会将指定的变量与不同模式进行比较。如果变量和模式是匹配的，那么shell会执行为该模式指定的命令。可以通过竖线操作符在一行中分隔出多个模式模式。星号会捕获所有与已知模式不匹配的值

```bash
$ cat test26.sh
# !/bin/bash
# using the case command
# 
case $USER in
rich | barbara)
  echo "Welcome, $USER"
  echo "Please enjoy your visit";;
testing)
  echo "Special testing account";;
jessica)
  echo "Do not forget to log off when you're done";;
*)
  echo "Sorry, you are not allowed here";;
esac
$ 
$ ./test26.sh
Welcome, rich
Please enjoy your visit
$
```

## for 命令

bash shell提供了 for 命令，允许你创建一个遍历一系列值的循环。每次迭代都使用其中一个值来执行已定义好的一组命令。下面是bash shell中 for 命令的基本格式。

```bash
for var in list
do
  commands
done
---------------------------
for var in list; do
  commands
done
```

### 读取列表中的值

```bash
$ cat test1
# !/bin/bash
# basic for command
for test in Alabama Alaska 
do
   echo The next state is $test
done
$ ./test1
The next state is Alabama
The next state is Alaska
$
```

### 读取列表中的复杂值

使用转义字符（反斜线）来将单引号转义；
使用双引号来定义用到单引号的值。

```bash
$ cat test2
# !/bin/bash
# another example of how not to use the for command
for test in I don\'t know if "this'll" work
do
   echo "word:$test"
done
$ ./test2
word:I
word:don't
word:know
word:if
word:this'll
word:work
$
```

for 命令用`空格`来划分列表中的每个值。如果在单独的数据值中有空格，就必须用双引号将这些值圈起来
```bash
$ cat test3
# !/bin/bash
# an example of how to properly define values
for test in Nevada "New Hampshire" "New Mexico" "New York"
do
  echo "Now going to $test"
done
$ ./test3
Now going to Nevada
Now going to New Hampshire
Now going to New Mexico
Now going to New York
$
```

### 从变量读取列表

```bash
$ cat test4
# !/bin/bash
# using a variable to hold the list
list="Alabama Alaska Arizona"
list=$list" Connecticut" # 添加文本
for state in $list
do
   echo "Have you ever visited $state?"
done
$ ./test4
Have you ever visited Alabama?
Have you ever visited Alaska?
Have you ever visited Arizona?
Have you ever visited Connecticut?
$
```

添加/拼接文本

```bash
$var='hello'
$var=$var" world!"
```

### 更改字段分隔符

IFS(internal field separator,内部字段分隔符) 环境变量定义了bash shell用作字段分隔符的一系列字符。默认情况下，bash shell会将下列字符当作字段分隔符：
```
空格
制表符
换行符
```

```bash
$ cat test5b
# !/bin/bash
# reading values from a file
file="states"
# 可以在shell脚本中临时更改 IFS 环境变量的值来限制被bash shell当作字段分隔符的字符
#告诉bash shell在数据值中忽略空格和制表符
IFS=$'\n'
 # 命令替换$() 来执行任何能产生输出的命令，然后在 for 命令中使用该命令的输出
for state in $(cat $file)
do
    echo "Visit beautiful $state"
done
$ ./test5b
Visit beautiful Alabama
Visit beautiful North Carolina
$
```
```bash
# 还原IFS方法
 在处理代码量较大的脚本时，可能在一个地方需要修改 IFS 的值，然后忽略这次修改，在脚本的其他地方继续沿用 IFS 的默认值。一个可参考的安全实践是在改变 IFS 之前保存原来的 IFS 值，之后再恢复它。 
这种技术可以这样实现： 
IFS.OLD=$IFS 
IFS=$'\n' 
<在代码中使用新的IFS值> 
IFS=$IFS.OLD 
这就保证了在脚本的后续操作中使用的是 IFS 的默认值。 
```

还有其他一些 IFS 环境变量的绝妙用法。假定你要遍历一个文件中用冒号分隔的值（比如在/etc/passwd文件中）。你要做的就是将 IFS 的值设为冒号。

```bash
IFS=:
```

如果要指定多个 IFS 字符，只要将它们在赋值行串起来就行。

```bash
IFS=$'\n':;"
```

这个赋值会将换行符、冒号、分号和双引号作为字段分隔符。如何使用 IFS 字符解析数据没有任何限制

### 用通配符读取目录

可以用 for 命令来自动遍历目录中的文件。进行此操作时，必须在文件名或路径名中使用通配符。它会强制shell使用文件扩展匹配。文件扩展匹配是生成匹配指定通配符的文件名或路径名的过程。

```bash
$ cat test6
# !/bin/bash
# iterate through all the files in a directory
for file in /home/rich/test/*
do
    if [ -d "$file" ] #路径可能包含空格，使用双引号括起来
    then
      echo "$file is a directory"
    elif [ -f "$file" ]
    then
      echo "$file is a file"
    fi
done
$ ./test6
/home/rich/test/dir1 is a directory
/home/rich/test/myprog.c is a file
/home/rich/test/myprog is a file
```

## C 语言风格的 for 命令

bash中C语言风格的 for 循环的基本格式如下:

```bash
for (( variable assignment ; condition ; iteration process ))
```

```bash
$ cat test8
# !/bin/bash
# testing the C-style for loop
for (( i=1; i <= 10; i++ ))
do
   echo "The next number is $i"
done
```

使用多个变量
```bash
# !/bin/bash
# multiple variables
for (( a=1, b=10; a <= 10; a++, b-- ))
do
   echo "$a - $b"
done
```

## while命令

while 命令允许定义一个要测试的命令，然后循环执行一组命令，只要定义的测试命令返回的是退出状态码 0 。它会在每次迭代的一开始测试 test 命令。在 test 命令返回非零退出状态码时， while 命令会停止执行那组命令。while 命令的格式是：

```bash
while test command
do
  other  commands
done
```

```bash
$ cat test10
# !/bin/bash
# while command test
var1=10
while [ $var1 -gt 0 ]
do
  echo $var1
  var1=$[ $var1 - 1 ]
done
```


while 命令允许你在 while 语句行定义多个测试命令。只有最后一个测试命令的退出状态码会被用来决定什么时候结束循环

## until 命令

until 命令和 while 命令工作的方式完全相反。until 命令要求你指定一个通常返回非零退出状态码的测试命令。只有测试命令的退出状态码不为 0 ，bash shell才会执行循环中列出的命令。一旦测试命令返回了退出状态码 0 ，循环就结束了。until 命令的格式如下:

```bash
until test commands
do
  other commands
done
```

```bash
$ cat test12
# !/bin/bash
# using the until command
var1=100
until [ $var1 -eq 0 ]
do
  echo $var1
  var1=$[ $var1 - 25 ]
done
```

## 循环处理文件数据

通常必须遍历存储在文件中的数据。这要求结合已经讲过的两种技术：

```
使用嵌套循环
修改 IFS 环境变量
```

这个脚本使用了两个不同的 IFS 值来解析数据。第一个 IFS 值解析出`/etc/passwd`文件中的单独的行。内部 for 循环接着将 IFS 的值修改为冒号，允许你从`/etc/passwd`的行中解析出单独的值

```bash
# !/bin/bash
# changing the IFS value
IFS.OLD=$IFS
IFS=$'\n'
for entry in $(cat /etc/passwd)
do
    echo "Values in $entry –"
    IFS=:
    for value in $entry
    do
      echo "$value"
    done
done
$
```

## 控制循环

有两个命令能帮我们控制循环内部的情况,`break`命令和`continue`命令。

### break命令

用 break 命令来退出任意类型的循环，包括while 和 until 循环。

有时你在内部循环，但需要停止外部循环。 break 命令接受单个命令行参数值：

```bash
break n
```

其中 n 指定了要跳出的循环层级。默认情况下， n 为 1 ，表明跳出的是当前的循环。如果你将n 设为 2 ， break 命令就会停止下一级的外部循环。

### continue 命令 

continue 命令可以提前中止某次循环中的命令，但并不会完全终止整个循环。可以在循环内部设置shell不执行命令的条件。

continue 命令也允许通过命令行参数指定要继续执行哪一级循环：

```bash
continue n
```

其中 n 定义了要继续的循环层级。

## 处理循环的输出

在shell脚本中，你可以对循环的输出使用管道或进行重定向,可以通过在 done 命令之后添加一个处理命令来实现。

```bash
for file in /home/rich/*
do
  if [ -d "$file" ]
  then
      echo "$file is a directory"
  elif
      echo "$file is a file"
  fi
done > output.txt
# shell会将 for 命令的结果重定向到文件output.txt中，而不是显示在屏幕上
# 这种方法同样适用于将循环的结果管接给另一个命令
for state in "North Dakota" Connecticut Illinois Alabama Tennessee 
do 
   echo "$state is the next place to go" 
done | sort 
# 。 for 命令的输出传给了 sort 命令，该命令会改变 for 命令输出结果的顺序
```

## 实例

### 查找可执行文件

```bash
$ cat test25
# !/bin/bash
# finding files in the PATH
IFS=:
for folder in $PATH # 对环境变量 PATH 中的目录进行迭代
do
  echo "$folder:"
  for file in $folder/* # 迭代特定目录中的所有文件
  do
    if [ -x $file ] # 检查各个文件是否具有可执行权限
    then
      echo "  $file"
    fi
  done
done
$
```

### 创建多个用户账户

```bash
$ cat test26
# !/bin/bash
# process new user accounts
input="users.csv"
while IFS=',' read -r userid name
do
  echo "adding $userid"
  useradd -c "$name" -m $userid
done < "$input"
$
$input 变量指向数据文件，并且该变量被作为 while 命令的重定向数据。
```
## 参考书目

《Linux命令行与shell脚本编程大全》第3版