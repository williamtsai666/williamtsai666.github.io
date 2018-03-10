---
title: Linux命令行(2)Linux环境变量
date: 2018-02-15 23:33:50
toc: true
tags: 
    - Linux
    - Shell
---
## 全局环境变量

全局环境变量对于shell会话和所有生成的子shell都是可见的，局部变量则只对创建它们的shell可见。

### 查看全局变量

```bash
env 或 printenv
```

### 显示个别环境变量的值

```bash
printenv HOME
```


也可以使用 echo 显示变量的值。在这种情况下引用某个环境变量的时候，必须在变量前面加上一个美元符($)。

```bash
echo $HOME
```


在 echo 命令中，在变量名前加上 $ 可不仅仅是要显示变量当前的值。它能够让变量作为命令行参数

```bash
ls $HOME
```

## 局部环境变量

局部环境变量只能在定义它们的进程中可见。在Linux系统并没有一个只显示局部环境变量的命令,`set `命令 显示为某个特定进程设置的所有环境变量，包括局部变量、全局变量以及用户定义变量

```bash
$ set 
BASH=/bin/bash 
[...] 
BASH_ALIASES=() 
BASH_ARGC=() 
BASH_ARGV=() 
BASH_CMDS=() 
BASH_LINENO=() 
BASH_SOURCE=() 
[...] 
colors=/etc/DIR_COLORS 
my_variable='Hello World' 
[...] 
$ 
#可以看到，所有通过printenv 命令能看到的全局环境变量都出现在了 set 命令的输出中。但在 set 命令的输出中还有其他一些环境变量，即局部环境变量和用户定义变量。
```

### 设置用户定义变量

#### 设置局部用户定义变量

```bash
$ echo $my_variable

$ my_variable="Hello World"  # 值可以是数值或字符串
$ echo $my_variable
Hello World

# 子shell无法访问父shell定义的局部变量
$ bash 
$ 
$ echo $my_variable 
 
$ exit 
exit 
$ 
$ echo $my_variable 
Hello World 
$ 

# 注意: 局部环境变量使用小写字母，系统环境变量使用大写字母
# 变量名、等号和值之间没有空格,。如果在赋值表达式中加上了空格，bash  shell就会把值当成一个单独的命令
$ my_variable = "Hello World" 
-bash: my_variable: command not found 
$ 


```
#### 设置全局环境变量

在设定全局环境变量的进程所创建的子进程中，该变量都是可见的。创建全局环境变量的方法是先创建一个局部环境变量，然后再把它导出到全局环境中。通过`export` 命令使其变成了全局环境变量，子shell甚至无法使用 export 命令改变父shell中全局环境变量的值

```bash
$ my_variable="I am Global now"
$
$ export my_variable
$
$ echo $my_variable
I am Global now
$
$ bash 
$ 
$ echo $my_variable 
I am Global now 
$ 
$ exit 
exit 
$ 
$ echo $my_variable 
I am Global now 
$ 
```

### 删除环境变量

```bash
$ echo $my_variable
I am Global now
$
$ unset my_variable
$
$ echo $my_variable

# 如果你是在子进程中删除了一个全局环境变量，这只对子进程有效。该全局环境变量在父进程中依然可用。
$ my_variable="I am Global now" 
$ 
$ export my_variable 
$ 
$ echo $my_variable 
I am Global now 
$ 
$ bash 
$ 
$ echo $my_variable 
I am Global now 
$ 
$ unset my_variable 
$ 
$ echo $my_variable 
 
$ exit 
exit 
$ 
$ echo $my_variable 
I am Global now 
$ 
```
如果要用到变量，使用`$`如果要操作变量，不使用`$`。这条规则的一个例外就是使用 printenv 显示某个变量的值。

## 默认的 shell 环境变量 

默认情况下，bash  shell会用一些特定的环境变量来定义系统环境。bash shell源自当初的Unix Bourne shell，因此也保留了Unix Bourne shell里定义的那些环境变量

```bash
# bash shell支持的Bourne变量(部分)
HOME  当前用户的主目录 
IFS  shell用来将文本字符串分割成字段的一系列字符 
OPTARG  getopts命令处理的最后一个选项参数值 
OPTIND  getopts命令处理的最后一个选项参数的索引号 
PATH  shell查找命令的目录列表，由冒号分隔 
PS1  shell命令行界面的主提示符 
PS2  shell命令行界面的次提示符 

# bash shell环境变量(部分)
BASH  当前shell实例的全路径名 
BASH_COMMAND  shell正在执行的命令或马上就执行的命令 
BASH_VERSION  当前运行的bash shell的版本号 
BASHPID  当前bash进程的PID 
ENV  如果设置了该环境变量，在bash  shell脚本运行之前会先执行已定义的启动文件（仅用于当bash shell以POSIX模式被调用时） 
FUNCNAME  当前执行的shell函数的名称
GROUPS  含有当前用户属组列表的数组变量 
HISTSIZE  最多在历史文件中存多少条命令 
HOSTFILE  shell在补全主机名时读取的文件名称 
HOSTNAME  当前主机的名称 
HOSTTYPE  当前运行bash shell的机器 
LANG  shell的语言环境类别
PPID  bash shell父进程的PID 
PWD  当前工作目录 
RANDOM  返回一个0～32767的随机数（对其的赋值可作为随机数生成器的种子） 
SHELL  bash shell的全路径名 
TMPDIR  目录名，保存bash shell创建的临时文件 
UID  当前用户的真实用户ID（数字形式） 
```



## 设置 PATH 环境变量

当你在shell命令行界面中输入一个外部命令时，shell必须搜索系统来找到对应的程序。 PATH 环境变量定义了用于进行命令和程序查找的目录,PATH 中的目录使用冒号分隔

```bash
# ubuntu PATH
$ echo $PATH 
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin: 
/sbin:/bin:/usr/games:/usr/local/games 
$ 

# 如果命令或者程序的位置没有包括在 PATH 变量中，那么如果不使用绝对路径的话，shell是没法找到的。如果shell找不到指定的命令或程序，它会产生一个错误信息
$ myprog 
-bash: myprog: command not found 

# 引用原来的 PATH 值，然后再给这个字符串添加新目录
PATH=$PATH:/home/christine/Scripts
# 如果希望子shell也能找到你的程序的位置，一定要记得把修改后的 PATH 环境变量导出
```

## 定位系统环境变量 

在你登入Linux系统启动一个bash  shell时，默认情况下bash会在几个文件中查找命令。这些文件叫作启动文件或环境文件。bash检查的启动文件取决于你启动bash  shell的方式。启动bash shell有3种方式： 

1. 登录时作为默认登录shell
2. 作为非登录shell的交互式shell 
3. 作为运行脚本的非交互shell 

### 1.登录时作为默认登录shell

当你登录Linux系统时，bash shell会作为登录shell启动。登录shell会从5个不同的启动文件里读取命令： 
```bash
/etc/profile 
$HOME/.bash_profile 
$HOME/.bashrc 
$HOME/.bash_login 
$HOME/.profile 
```

`/etc/profile`文件是系统上默认的bash  shell的主启动文件。系统上的每个用户登录时都会执行这个启动文件,另外4个启动文件是针对用户的，可根据个人需求定制。

#### /etc/profile文件 

/etc/profile文件是bash  shell默认的的主启动文件。只要你登录了Linux系统，bash就会执行/etc/profile启动文件中的命令。不同的Linux发行版在这个文件里放了不同的命令。

```bash
# centos
# /etc/profile

# System wide environment and startup programs, for login setup
# Functions and aliases go in /etc/bashrc

# It's NOT a good idea to change this file unless you know what you
# are doing. It's much better to create a custom.sh shell script in
# /etc/profile.d/ to make custom changes to your environment, as this
# will prevent the need for merging in future updates.

pathmunge () {
    case ":${PATH}:" in
        *:"$1":*)
            ;;
        *)
            if [ "$2" = "after" ] ; then
                PATH=$PATH:$1
            else
                PATH=$1:$PATH
            fi
    esac
}


if [ -x /usr/bin/id ]; then
    if [ -z "$EUID" ]; then
        # ksh workaround
        EUID=`id -u`
        UID=`id -ru`
    fi
    USER="`id -un`"
    LOGNAME=$USER
    MAIL="/var/spool/mail/$USER"
fi

# Path manipulation
if [ "$EUID" = "0" ]; then
    pathmunge /sbin
    pathmunge /usr/sbin
    pathmunge /usr/local/sbin
else
    pathmunge /usr/local/sbin after
    pathmunge /usr/sbin after
    pathmunge /sbin after
fi

HOSTNAME=`/bin/hostname 2>/dev/null`
HISTSIZE=1000
if [ "$HISTCONTROL" = "ignorespace" ] ; then
    export HISTCONTROL=ignoreboth
else
    export HISTCONTROL=ignoredups
fi

export PATH USER LOGNAME MAIL HOSTNAME HISTSIZE HISTCONTROL

# By default, we want umask to get set. This sets it for login shell
# Current threshold for system reserved uid/gids is 200
# You could check uidgid reservation validity in
# /usr/share/doc/setup-*/uidgid file
if [ $UID -gt 199 ] && [ "`id -gn`" = "`id -un`" ]; then
    umask 002
else
    umask 022
fi

for i in /etc/profile.d/*.sh ; do
    if [ -r "$i" ]; then
        if [ "${-#*i}" != "$-" ]; then
            . "$i"
        else
            . "$i" >/dev/null 2>&1
        fi
    fi
done

unset i
unset -f pathmunge


# 说明 ubuntu和centos两个发行版的/etc/profile文件都有： for 语句。它用来迭代/etc/profile.d目录下的所有文件。）这为Linux系统提供了一个放置特定应用程序启动文件的地方，当用户登录时，shell会执行这些文件
$ ls -l /etc/profile.d 
total 80 
-rw-r--r--. 1 root root 1127 Mar  5 07:17 colorls.csh 
-rw-r--r--. 1 root root 1143 Mar  5 07:17 colorls.sh 
-rw-r--r--. 1 root root   92 Nov 22  2013 cvs.csh 

不难发现，有些文件与系统中的特定应用有关。大部分应用都会创建两个启动文件：一个供bash shell使用（使用.sh扩展名），一个供c shell使用（使用.csh扩展名）。 lang.csh和lang.sh文件会尝试去判定系统上所采用的默认语言字符集，然后设置对应的 LANG环境变量。 
```

####  $HOME目录下的启动文件 

剩下的启动文件都起着同一个作用：提供一个用户专属的启动文件来定义该用户所用到的环境变量。大多数Linux发行版只用这四个启动文件中的一到两个

```bash
$HOME/.bash_profile
$HOME/.bashrc
$HOME/.bash_login
$HOME/.profile
```

shell会按照按照下列顺序，运行第一个被找到的文件，余下的则被忽略：

```bash
$HOME/.bash_profile
$HOME/.bash_login
$HOME/.profile
```

注意，这个列表中并没有`$HOME/.bashrc`文件。这是因为该文件通常通过其他文件运行的。

```bash
# .bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then
	. ~/.bashrc
fi

# User specific environment and startup programs

PATH=$PATH:$HOME/bin

export PATH

# .bash_profile启动文件会先去检查HOME目录中是不是还有一个叫.bashrc的启动文件。如果有的话，会先执行启动文件里面的命令
```



### 2.作为非登录shell的交互式shell

如果你的bash shell不是登录系统时启动的（比如是在命令行提示符下敲入 bash 时启动），那么你启动的shell叫作交互式shell。交互式shell不会像登录shell一样运行，但它依然提供了命令行提示符来输入命令。 
如果bash是作为交互式shell启动的，它就不会访问/etc/profile文件，只会检查用户HOME目录中的.bashrc文件。 

```bash
# .bashrc

# User specific aliases and functions

alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'

# Source global definitions
if [ -f /etc/bashrc ]; then
	. /etc/bashrc
fi
# .bashrc文件有两个作用：一是查看/etc目录下通用的bashrc文件，二是为用户提供一个定制自己的命令别名和私有脚本函数的地方。
```

### 3.作为运行脚本的非交互shell

当shell启动一个非交互式shell进程时，它会检查 BASH_ENV 环境变量来查看要执行的启动文件。如果有指定的文件，shell会执行该文件里的命令，这通常包括shell脚本变量设置。 

如果父shell是登录shell，在/etc/profile、/etc/profile.d/ * .sh和$HOME/.bashrc文件中设置并导出了变量，用于执行脚本的子shell就能够继承这些变量。 

### 环境变量持久化 

对全局环境变量来说（Linux系统中所有用户都需要使用的变量），最好是在`/etc/profile.d`目录中创建一个以.sh结尾的文件。把所有新的或修改过的全局环境变量设置放在这个文件中。 
对个人用户永久性bash shell变量应该放在`$HOME/.bashrc`文件。

### 数组变量

```bash
# 要给某个环境变量设置多个值，可以把值放在括号里，值与值之间用空格分隔
mytest=(one two three four five)
$ echo $mytest
one
$ echo ${mytest[2]}
three
# 要显示整个数组变量，可用星号作为通配符放在索引值的位置
$ echo ${mytest[*]}
one two three four five
# 修改
mytest[2]=seven
# 删除
$ unset mytest[2] 
$ 
$ echo ${mytest[*]} 
one two four five 
$ 
$ echo ${mytest[2]} 
 
$ echo ${mytest[3]} 
four 
# 删除整个数组
$ unset mytest 
$ 
$ echo ${mytest[*]} 
 
```
## 参考书目

《Linux命令行与shell脚本编程大全》第3版