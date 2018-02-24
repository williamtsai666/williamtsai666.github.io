---
title: shell脚本(3)处理用户输入
date: 2018-02-21 23:33:50
tags: 
    - Linux
    - Shell
---

bash shell提供了一些不同的方法来从用户处获得数据，包括命令行参数(添加在命令后的数据)、命令行选项(可修改命令行为的单个字母)以及键盘输入

## 命令行参数

### 读取参数

bash shell会将一些称为位置参数(positional parameter)的特殊变量分配给输入到命令行中的所有参数位置参数变量是标准的数字:`$0`是程序名,`$1`是第一个参数,`$2` 是第二个参数,依次类推,直到第九个参数`$9`

```bash
$ cat test2.sh
# !/bin/bash
# testing two command line parameters
# 
total=$[ $1 * $2 ]
echo The first parameter is $1.
echo The second parameter is $2.
echo The total value is $total.
$ 
$ ./test2.sh 2 5
The first parameter is 2.
The second parameter is 5.
The total value is 10.
```
每个参数都是用空格分隔的，要在参数值中包含空格，必须要用引号(单引号或双引号均可)
如果脚本需要的命令行参数不止9个，你仍然可以处理，但是需要稍微修改一下变量名。在第9个变量之后，你必须在变量数字周围加上花括号，比如`${10}`

```bash
$ cat test4.sh 
#!/bin/bash 
# handling lots of parameters 
# 
total=$[ ${10} * ${11} ] 
echo The tenth parameter is ${10} 
echo The eleventh parameter is ${11} 
echo The total is $total 
$  
$ ./test4.sh 1 2 3 4 5 6 7 8 9 10 11 12 
The tenth parameter is 10 
The eleventh parameter is 11 
The total is 110 
$ 
```

### 读取脚本名

用`$0`参数获取shell在命令行启动的脚本名

```bash
$ cat test5b.sh
# !/bin/bash
# Using basename with the $0 parameter
# 
name=$(basename $0) # basename 命令会返回不包含路径的脚本名
echo
echo The script name is: $name
# 

$ bash /home/Christine/test5b.sh 
The script name is: test5b.sh 
$ ./test5b.sh 
The script name is: test5b.sh 
```

### 测试参数

在使用参数前一定要检查其中是否存在数据。

```bash
$ cat test7.sh 
#!/bin/bash 
# testing parameters before use 
# 
if [ -n "$1" ]  # -n测试来检查命令行参数 $1 中是否有数据
then 
   echo Hello $1, glad to meet you. 
else 
   echo "Sorry, you did not identify yourself. " 
fi 
$  
```

## 特殊参数变量

### 参数统计

特殊变量`$#`含有脚本运行时携带的命令行参数的个数

```bash
$ cat test8.sh 
#!/bin/bash 
# getting the number of parameters 
# 
echo There were $# parameters supplied. 
$  
$ ./test8.sh 
There were 0 parameters supplied. 
$  
$ ./test8.sh 1 2 3 4 5 
There were 5 parameters supplied. 
```

```bash
# 获取最后一个参数 ${!#}

$ cat test10.sh
# !/bin/bash
# Grabbing the last parameter
# 
params=$#
echo
echo The last parameter is $params
echo The last parameter is ${!#}
echo
# 
$ bash test10.sh 1 2 3 4 5 
The last parameter is 5 
The last parameter is 5 

$ bash test10.sh 
The last parameter is 0 
The last parameter is test10.sh 

# 注意，当命令行上没有任何参数时，$#的值为 0 ，params 变量的值也一样，但${!#}变量会返回命令行用到的脚本名
```

### 抓取所有的数据

`$*`和 `$@`变量可以用来轻松访问所有的参数。这两个变量都能够在单个变量中存储所有的命令行参数

`$*` 变量会将命令行上提供的所有参数当作一个单词保存。这个单词包含了命令行中出现的一个参数值。基本上` $*` 变量会将这些参数视为一个整体，而不是多个个体。

`$@` 变量会将命令行上提供的所有参数当作同一字符串中的多个独立的单词。这样你就能够遍历所有的参数值，得到每个参数。这通常通过 for 命令完成。
```bash
$ cat test12.sh
#!/bin/bash
# testing $* and $@
#
echo
count=1
#
for param in "$*"
do
  echo "\$* Parameter #$count = $param"
  count=$[ $count + 1 ]
done
#
echo
count=1
#
for param in "$@"
do
  echo "\$@ Parameter #$count = $param"
  count=$[ $count + 1 ]
done
$ 
$ ./test12.sh rich barbara katie jessica
 
$* Parameter #1 = rich barbara katie jessica
 
$@ Parameter #1 = rich
$@ Parameter #2 = barbara
$@ Parameter #3 = katie
$@ Parameter #4 = jessica
$
```


