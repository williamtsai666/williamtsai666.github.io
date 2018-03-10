---
title: shell脚本(4)呈现数据
date: 2018-02-22 23:33:50
toc: true
tags: 
    - Linux
    - Shell
---
## 理解输入和输出

目前有两种显示脚本输出的方法： 1.在显示器屏幕上显示输出 2.将输出重定向到文件中 

### 标准文件描述符 

Linux系统将每个对象当作文件处理。这包括输入和输出进程。Linux用文件描述符(file  descriptor)来标识每个文件对象。文件描述符是一个非负整数，可以唯一标识会话中打开的文件。每个进程一次最多可以有九个文件描述符。出于特殊目的，bash shell保留了前三个文件描述符（ 0 、 1 和 2 ）

 Linux的标准文件描述符

| 文件描述符 |缩 写|描述|
| ---------- | ---- | ---- |
| 0 | STDIN | 标准输入 |
| 1 | STDOUT | 标准输出 |
| 2 | STDERR | 标准错误 |

#### 1.STDIN

`STDIN`文件描述符代表shell的标准输入。对终端界面来说，标准输入是键盘。shell从` STDIN`文件描述符对应的键盘获得输入，在用户输入时处理每个字符。

在使用输入重定向符号（ < ）时，Linux会用重定向指定的文件来替换标准输入文件描述符,许多bash命令能接受 STDIN 的输入，尤其是没有在命令行上指定文件的话

```bash
# 当在命令行上只输入cat命令时，它会从STDIN接受输入。输入一行，cat命令就会显示出一行。
$ cat 
this is a test 
this is a test
# 重定向 用testfile文件中的行作为输入
$ cat < testfile 
```

#### 2.STDOUT

`STDOUT` 文件描述符代表shell的标准输出。在终端界面上，标准输出就是终端显示器。shell的所有输出（包括shell中运行的程序和脚本）会被定向到标准输出中，也就是显示器。默认情况下，大多数bash命令会将输出导向 `STDOUT `文件描述符

```bash
# 输出重定向
$ ls -l > test2 
# 输出追加
$ who >> test2 
# 输出错误时
$ ls -al badfile > test3 
ls: cannot access badfile: No such file or directory 
$ cat test3 
$ 

当命令生成错误消息时，shell并未将错误消息重定向到输出重定向文件。shell创建了输出重定向文件，但错误消息却显示在了显示器屏幕上。注意，在显示test3文件的内容时并没有任何错误。test3文件创建成功了，只是里面是空的。 
shell对于错误消息的处理是跟普通输出分开的。如果你创建了在后台模式下运行的shell脚本，通常你必须依赖发送到日志文件的输出消息。用这种方法的话，如果出现了错误信息，这些信息是不会出现在日志文件中的。
```

#### 3.STDERR 

shell通过特殊的 `STDERR` 文件描述符来处理错误消息。` STDERR `文件描述符代表shell的标准错误输出。shell或shell中运行的程序和脚本出错时生成的错误消息都会发送到这个位置。 默认情况下， `STDERR `文件描述符会和 `STDOUT `文件描述符指向同样的地方。也就是说，默认情况下，错误消息也会输出到显示器输出中。

### 重定向错误 

#### 1.只重定向错误 

`STDERR`文件描述符被设成 2 。可以选择只重定向错误消息，将该文件描述符值放在重定向符号前。该值必须紧紧地放在重定向符号前，否则不会工作。

```bash
$ ls -al badfile 2> test4 
$ cat test4 
ls: cannot access badfile: No such file or directory 
$ 
# 只有错误消息被重定向到文件
$ ls -al test badtest test2 2> test5 
-rw-rw-r-- 1 rich rich 158 2014-10-16 11:32 test2 
$ cat test5 
ls: cannot access test: No such file or directory 
ls: cannot access badtest: No such file or directory 
$ 
```

#### 2. 重定向错误和数据

如果想重定向错误和正常输出，必须用两个重定向符号。需要在符号前面放上待重定向数据所对应的文件描述符，然后指向用于保存数据的输出文件

```bash
# 正常输出STDOUT 重定向到test7文件
# 错误消息STDERR 重定向到test6文件
$ ls -al test test2 test3 badtest 2> test6 1> test7 
$ cat test6 
ls: cannot access test: No such file or directory 
ls: cannot access badtest: No such file or directory 
$ cat test7 
-rw-rw-r-- 1 rich rich 158 2014-10-16 11:32 test2 
-rw-rw-r-- 1 rich rich   0 2014-10-16 11:33 test3 
$ 

# 将STDERR和STDOUT的输出重定向到同一个输出文件,使用特殊的重定向符号 &>
$ ls -al test test2 test3 badtest &> test7 
$ cat test7 
ls: cannot access test: No such file or directory 
ls: cannot access badtest: No such file or directory 
-rw-rw-r-- 1 rich rich 158 2014-10-16 11:32 test2 
-rw-rw-r-- 1 rich rich   0 2014-10-16 11:33 test3 
$ 
# 当使用 &> 符时，命令生成的所有输出都会发送到同一位置，包括数据和错误。
# 为了避免错误信息散落在输出文件中，相较于标准输出，bash shell自动赋予了错误消息更高的优先级。

```

## 在脚本中重定向输出

### 临时重定向

如果想在脚本中生成错误消息，可以将单独的一行输出重定向到 STDERR 。使用输出重定向符来将输出信息重定向到 STDERR 文件描述符。在重定向到文件描述符时，你必须在文件描述符数字之前加一个 &

```bash
$ cat test8 
#!/bin/bash 
# testing STDERR messages 
 
echo "This is an error" >&2 
echo "This is normal output" 
$ 
$ ./test8 
This is an error 
This is normal output 
# 默认情况下，Linux会将 STDERR 导向 STDOUT 。所以上面的结果输出正常，但是，如果你在运行脚本时重定向了STDERR ，脚本中所有导向 STDERR 的文本都会被重定向。

$ ./test8 2> test9 
This is normal output 
$ cat test9 
This is an error 
$ 
```

### 永久重定向

如果脚本中有大量数据需要重定向，那重定向每个 echo 语句就会很烦琐。可以用 exec 命令告诉shell在脚本执行期间重定向某个特定文件描述符

```bash
$ cat test10 
#!/bin/bash 
# redirecting all output to a file 
exec 1>testout 
 
echo "This is a test of redirecting all output" 
echo "from a script to another file." 
echo "without having to redirect every individual line" 
$ ./test10 
$ cat testout 
This is a test of redirecting all output 
from a script to another file. 
without having to redirect every individual line 
$ 
# exec 命令会启动一个新shell并将 STDOUT 文件描述符重定向到文件。脚本中发给 STDOUT 的所有输出会被重定向到文件。 
```

```bash
# 可以在脚本执行过程中重定向 STDOUT
$ cat test11 
#!/bin/bash 
# redirecting output to different locations 
 
exec 2>testerror 
 
echo "This is the start of the script" 
echo "now redirecting all output to another location" 
 
exec 1>testout 
 
echo "This output should go to the testout file" 
echo "but this should go to the testerror file" >&2 
$ 
$ ./test11 
This is the start of the script 
now redirecting all output to another location 
$ cat testout 
This output should go to the testout file 
$ cat testerror 
but this should go to the testerror file 
$ 

```

## 在脚本中重定向输入 

可以使用与脚本中重定向 STDOUT 和 STDERR 相同的方法来将 STDIN 从键盘重定向到其他位置。 exec 命令允许你将 STDIN 重定向到Linux系统上的文件中.

```bash
$ cat test12 
#!/bin/bash 
# redirecting file input 
# 重定向输入，从文件testfile中获得输入，而不是STDIN
exec 0< testfile 
count=1 
 
while read line 
do 
   echo "Line #$count: $line" 
   count=$[ $count + 1 ] 
done 
$ ./test12 
Line #1: This is the first line. 
Line #2: This is the second line. 
Line #3: This is the third line. 
```

## 创建自己的重定向

在脚本中重定向输入和输出时，并不局限于这3个默认的文件描述符。在shell中最多可以有9个打开的文件描述符。其他6个从 3 ~ 8 的文件描述符均可用作输入或输出重定向。可以将这些文件描述符中的任意一个分配给文件，然后在脚本中使用它们。

### 创建输出文件描述符 

使用 exec 命令来给输出分配文件描述符。和标准的文件描述符一样，一旦将另一个文件描述符分配给一个文件，这个重定向就会一直有效，直到你重新分配。

```bash
$ cat test13 
#!/bin/bash 
# using an alternative file descriptor 
# 将文件描述符3 重定向到另一个文件
exec 3>test13out 
 
echo "This should display on the monitor" 
echo "and this should be stored in the file" >&3 
echo "Then this should be back on the monitor" 
$ ./test13 
This should display on the monitor 
Then this should be back on the monitor 
$ cat test13out 
and this should be stored in the file 
$ 
```

### 重定向文件描述符 

如果要恢复已重定向的文件描述符，你可以分配另外一个文件描述符给标准文件描述符。这意味着你可以将 STDOUT 的原来位置重定向到另一个文件描述符，然后再利用该文件描述符重定向回 STDOUT。

```bash
$ cat test14 
#!/bin/bash 
# storing STDOUT, then coming back to it 
# 将文件描述符 3 重定向到文件描述符1的当前位置，也就是STDOUT。这意味着任何发送给文件描述符3的输出都将出现在显示器上。
exec 3>&1 
# 将STDOUT重定向到文件，shell现在会将发送给STDOUT 的输出直接重定向到输出文件中。但是，文件描述符 3仍然指向STDOUT原来的位置，也就是显示器。如果此时将输出数据发送给文件描述符3，它仍然会出现在显示器上
exec 1>test14out 
echo "This should store in the output file" 
echo "along with this line." 
# 在向 STDOUT （现在指向一个文件）发送一些输出之后，脚本将 STDOUT 重定向到文件描述符3的当前位置（现在仍然是显示器）。这意味着现在 STDOUT 又指向了它原来的位置：显示器。 
exec 1>&3 
echo "Now things should be back to normal" 
$ 
$ ./test14 
Now things should be back to normal 
$ cat test14out 
This should store in the output file 
along with this line. 
$ 
```

### 创建输入文件描述符 

可以用和重定向输出文件描述符同样的办法重定向输入文件描述符。在重定向到文件之前，先将 STDIN 文件描述符保存到另外一个文件描述符，然后在读取完文件之后再将 STDIN 恢复到它原来的位置。 

```bash
$ cat test15 
#!/bin/bash 
# redirecting input file descriptors 

# 文件描述符6用来保存 STDIN 的位置
exec 6<&0 
# 将 STDIN 重定向到一个文件,read 命令的所有输入都来自重定向后的 STDIN （也就是输入文件）。
exec 0< testfile 
 
count=1 
while read line 
do 
   echo "Line #$count: $line" 
   count=$[ $count + 1 ] 
done 
# 将 STDIN 重定向到文件描述符 6 ，从而将STDIN恢复到原先的位置。
exec 0<&6 
read -p "Are you done now? " answer 
case $answer in 
Y|y) echo "Goodbye";; 
N|n) echo "Sorry, this is the end.";; 
esac 
$ ./test15 
Line #1: This is the first line. 
Line #2: This is the second line. 
Line #3: This is the third line. 
Are you done now? y 
Goodbye 
```

### 关闭文件描述符 

如果你创建了新的输入或输出文件描述符，shell会在脚本退出时自动关闭它们。然而在有些情况下，你需要在脚本结束前手动关闭文件描述符.

要关闭文件描述符，将它重定向到特殊符号 `&-`

```bash
# 关闭文件描述符 3 ，不再在脚本中使用它
exec 3>&- 
```

```bash
$ cat badtest 
#!/bin/bash 
# testing closing file descriptors 
exec 3> test17file 
echo "This is a test line of data" >&3 
exec 3>&- 
echo "This won't work" >&3 
$ ./badtest 
./badtest: 3: Bad file descriptor 
$ 
```

##  列出打开的文件描述符

lsof 命令会列出整个Linux系统打开的所有文件描述符。

```bash
$ /usr/sbin/lsof -a -p $$ -d 0,1,2 
COMMAND  PID USER   FD   TYPE DEVICE SIZE NODE NAME 
bash    3344 rich    0u   CHR  136,0         2 /dev/pts/0 
bash    3344 rich    1u   CHR  136,0         2 /dev/pts/0 
bash    3344 rich    2u   CHR  136,0         2 /dev/pts/0 
------------------------------------------------------------------
-p 指定进程ID,PID
-d 指定要显示的文件描述符编号
-a 对两个选项结果进行AND运算
$$ 当前pid

# lsof的默认输出 
COMMAND  正在运行的命令名的前9个字符 
PID  进程的PID 
USER  进程属主的登录名 
FD  文件描述符号以及访问类型（r代表读，w代表写，u代表读写） 
TYPE  文件的类型（CHR代表字符型，BLK代表块型，DIR代表目录，REG代表常规文件） 
DEVICE  设备的设备号（主设备号和从设备号）
SIZE  如果有的话，表示文件的大小 
NODE  本地文件的节点号 
NAME  文件名 

与STDIN、STDOUT和STDERR关联的文件类型是字符型。因为STDIN、STDOUT和STDERR文件描述符都指向终端，所以输出文件的名称就是终端的设备名。
```

## 阻止命令输出 

可以将 `STDERR` 重定向到一个叫作null文件的特殊文件。null文件里什么都没有。shell输出到null文件的任何数据都不会保存，全部都被丢掉了。 
在Linux系统上null文件的标准位置是`/dev/null`。你重定向到该位置的任何数据都会被丢掉，不会显示。 

```bash
$ ls -al > /dev/null 
$ cat /dev/null 
$ 
# 避免错误消息
$ ls -al badfile test16 2> /dev/null 
-rwxr--r--    1 rich     rich          135 Oct 29 19:57 test16* 
$ 
# 快速清除现有文件的数据,清除日志文件的常用方法
$ cat /dev/null > testfile 
$ cat testfile 
$ 清除日志文件的一个常用方法
```

## 创建临时文件

Linux系统有特殊的目录，专供临时文件使用。Linux使用`/tmp`目录来存放不需要永久保留的文件。大多数Linux发行版配置了系统在启动时自动删除`/tmp`目录的所有文件。系统上的任何用户账户都有权限在读写`/tmp`目录中的文件。有个特殊命令可以用来创建临时文件。 `mktemp `命令可以在`/tmp`目录中创建一个唯一的临时文件。shell会创建这个文件，但不用默认的` umask` 值。它会将文件的读和写权限分配给文件的属主，并将你设成文件的属主。

### 创建本地临时文件 

默认情况下， `mktemp` 会在本地目录中创建一个文件。要用 `mktemp` 命令在本地目录中创建一个临时文件，你需要指定一个文件名模板。模板可以包含任意文本文件名，在文件名末尾加上6个 X 

```bash
$ mktemp testing.XXXXXX 
testing.1DRLuV 

# shell脚本中保存文件名称
tempfile=$(mktemp test.XXXXXX) 
```

### 在/tmp 目录创建临时文件 

 `mktemp` 命令使用`-t` 选项在系统的临时目录来创建临时文件。创建好后会返回用来创建临时文件的全路径，而不是只有文件名。

```bash
$ mktemp -t test.XXXXXX 
/tmp/test.xG3374 
$ ls -al /tmp/test* 
-rw------- 1 rich rich 0 2014-10-29 18:41 /tmp/test.xG3374 
$ 
```

### 创建临时目录 

`mktemp` 命令-d 选项来创建一个临时目录而不是临时文件

```bash
tempdir=$(mktemp -d dir.XXXXXX) 
cd $tempdir 
tempfile1=$(mktemp temp.XXXXXX) 
tempfile2=$(mktemp temp.XXXXXX) 
```

## 记录消息 (tee)

如果需要将输出同时发送到显示器和日志文件，只要用特殊的 tee 命令就行。 
tee 命令相当于管道的一个T型接头。它将从 STDIN 过来的数据同时发往两处。一处是STDOUT ，另一处是 tee 命令行所指定的文件名。

```bash
tee filename
-a 将数据追加到文件 
```

```bash
# 由于tee会重定向来自STDIN的数据，可以用它配合管道命令来重定向命令输出。 
$ date | tee testfile 
Sun Oct 19 18:56:21 EDT 2014 
$ cat testfile 
Sun Oct 19 18:56:21 EDT 2014 
$ 
```



## 参考书目

《Linux命令行与shell脚本编程大全》第3版