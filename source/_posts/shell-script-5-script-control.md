---
title: shell脚本(5)控制脚本
date: 2018-02-23 23:33:50
toc: true
tags: 
    - Linux
    - Shell
---
## 处理信号 

Linux利用信号与运行在系统中的进程进行通信。

###  Linux 信号

Linux系统和应用程序可以生成超过30个信号，一下是常用的Linux系统信号。

```
1  SIGHUP  挂起进程 
2  SIGINT  终止进程
3  SIGQUIT  停止进程 
9  SIGKILL  无条件终止进程 
15  SIGTERM  尽可能终止进程 
17  SIGSTOP  无条件停止进程，但不是终止进程 
18  SIGTSTP  停止或暂停进程，但不终止进程 
19  SIGCONT  继续运行停止的进程 
```

默认情况下，bash shell会忽略收到的任何 `SIGQUIT (3)` 和 `SIGTERM (5) `信号（正因为这样，交互式shell才不会被意外终止）。但是bash shell会处理收到的 `SIGHUP (1)` 和` SIGINT (2)` 信号。

如果bash  shell收到了 SIGHUP 信号，比如当你要离开一个交互式shell，它就会退出。但在退出之前，它会将 SIGHUP 信号传给所有由该shell所启动的进程（包括正在运行的shell脚本）。 

通过 SIGINT 信号，可以中断shell。Linux内核会停止为shell分配CPU处理时间。这种情况发生时，shell会将 SIGINT 信号传给所有由它所启动的进程，以此告知出现的状况。 

### 生成信号

bash  shell允许用键盘上的组合键生成两种基本的Linux信号。

#### 中断进程 

`Ctrl`+`C`组合键会生成 `SIGINT `信号，并将其发送给当前在shell中运行的所有进程。

#### 暂停进程 

`Ctrl`+`Z`组合键会生成一个 `SIGTSTP` 信号，停止shell中运行的任何进程。停止（stopping）进程跟终止（terminating）进程不同：停止进程会让程序继续保留在内存中，并能从上次停止的位置继续运行。

### 捕获信号

`trap`命令允许你来指定shell脚本要监看并从shell中拦截的Linux信号。如果脚本收到了 trap 命令中列出的信号，该信号不再由shell处理，而是交由本地处理。

```bash
# trap 命令的格式是： 
trap commands signals 
```

```bash
$ cat test1.sh 
#!/bin/bash 
# Testing signal trapping 
# 
trap "echo ' Sorry! I have trapped Ctrl-C'" SIGINT 
# 
echo This is a test script 
# 
count=1 
while [ $count -le 10 ] 
do 
   echo "Loop #$count" 
   sleep 1 
   count=$[ $count + 1 ] 
done 
# 
echo "This is the end of the test script" 
# 

$ ./test1.sh 
This is a test script 
Loop #1 
Loop #2 
^C Sorry! I have trapped Ctrl-C 
```

###  捕获脚本退出

要捕获shell脚本的退出，只要在 trap 命令后加上 EXIT 信号。

```bash
$ cat test2.sh 
#!/bin/bash 
# Trapping the script exit 
# 
trap "echo Goodbye..." EXIT 
# 
count=1 
while [ $count -le 5 ] 
do 
   echo "Loop #$count" 
   sleep 1 
   count=$[ $count + 1 ] 
done 
# 
$ 
$ ./test2.sh 
Loop #1 
Loop #2 
Loop #3 
Loop #4 
Loop #5 
Goodbye... 
$ 
# 提前退出脚本，同样能够捕获到 EXIT
$ ./test2.sh 
Loop #1 
Loop #2 
Loop #3 
^CGoodbye... 
 
```

### 修改或移除捕获

要想在脚本中的不同位置进行不同的捕获处理，只需重新使用带有新选项的 trap 命令。 

```bash
$ cat test3.sh 
#!/bin/bash 
# Modifying a set trap 
# 
trap "echo ' Sorry... Ctrl-C is trapped.'" SIGINT 
# 
count=1 
while [ $count -le 5 ] 
do 
   echo "Loop #$count" 
   sleep 1 
   count=$[ $count + 1 ] 
done 
# 
trap "echo ' I modified the trap!'" SIGINT 
# 
count=1 
while [ $count -le 5 ] 
do 
   echo "Second Loop #$count" 
   sleep 1 
   count=$[ $count + 1 ] 
done 
# 
$ 
$ ./test3.sh 
Loop #1 
Loop #2 
Loop #3 
^C Sorry... Ctrl-C is trapped. 
Loop #4 
Loop #5 
Second Loop #1 
Second Loop #2 
^C I modified the trap! 
Second Loop #3 
Second Loop #4 
Second Loop #5 
$ 

# 删除已设置好的捕获。在trap 命令与希望恢复默认行为的信号列表之间加上两个破折号
# 也可以在 trap 命令后使用单破折号来恢复信号的默认行为,单破折号和双破折号都可以正常发挥作用。
# Remove the trap 
trap -- SIGINT 

```

## 以后台模式运行脚本 

在后台模式中，进程运行时不会和终端会话上的 `STDIN` 、 `STDOUT`以及`STDERR`关联

### 后台运行脚本

以后台模式运行shell脚本非常简单。只要在命令后加个`&`符就行了。

```bash
$ command &
```

```bash
$ cat test4.sh 
#!/bin/bash 
# Test running in the background 
# 
count=1 
while [ $count -le 10 ] 
do 
   sleep 1 
   count=$[ $count + 1 ] 
done 
# 
$ 
$ ./test4.sh &  # 将命令作为系统中的一个独立的后台进程运行
[1] 3231  # [shell分配给后台进程的作业号] Linux系统分配给进程的进程ID（PID）

# 当后台进程结束时，它会在终端上显示出一条消息
[1]   Done                    ./test4.sh 

# 当后台进程运行时，它仍然会使用终端显示器来显示STDOUT和STDERR消息
```

## 在非控制台下运行脚本 

`nohup`命令运行了另外一个命令来阻断所有发送给该进程的 SIGHUP 信号。这会在退出终端会话时阻止进程退出。

```bash
$ nohup ./test1.sh & 
[1] 3856 
$ nohup: ignoring input and appending output to 'nohup.out' 
 
# 由于 nohup命令会解除终端与进程的关联，进程也就不再同STDOUT和STDERR联系在一起。为了保存该命令产生的输出，nohup命令会自动将STDOUT和STDERR的消息重定向到一个名为nohup.out的文件中。
```

## 作业控制 

启动、停止、终止以及恢复作业的这些功能统称为作业控制。

### 查看作业 

作业控制中的关键命令是 jobs 命令。 jobs 命令允许查看shell当前正在处理的作业。 

```bash
$ jobs -l 
[1]+  1897 Stopped                 ./test10.sh 
[2]-  1917 Running                 ./test10.sh > test10.out & 

# jobs命令参数
-l  列出进程的PID以及作业号 
-n  只列出上次shell发出的通知后改变了状态的作业 
-p  只列出作业的PID 
-r  只列出运行中的作业 
-s  只列出已停止的作业 
# +-号解释
带加号的作业会被当做默认作业。在使用作业控制命令时，如果未在命令行指定任何作业号，该作业会被当成作业控制命令的操作对象。
当前的默认作业完成处理后，带减号的作业成为下一个默认作业。
任何时候都只有一个带加号的作业和一个带减号的作业，不管shell中有多少个正在运行的作业。
```

### 重启停止的作业 

在bash作业控制中，可以将已停止的作业作为后台进程或前台进程重启。要以后台模式重启一个作业，可用` bg `命令加上作业号。 

```bash
$ ./test11.sh 
^Z 
[1]+  Stopped                 ./test11.sh 
$ 
# 因为该作业是默认作业（带+号），只需要使用 bg 命令就可以将其以后台模式重启
$ bg 
[1]+ ./test11.sh & 
$ 
$ jobs 
[1]+  Running                 ./test11.sh & 
$ 
```

要以前台模式重启作业，可用带有作业号的 `fg` 命令。

```bash
$ fg 2 
./test12.sh 
This is the script's end... 
$ 
```

## 调整谦让度

在多任务操作系统中(Linux)，内核负责将CPU时间分配给系统上运行的每个进程。调度优先级（scheduling  priority）是内核分配给进程的CPU时间（相对于其他进程）。在Linux系统中，由shell启动的所有进程的调度优先级默认都是相同的。 
调度优先级是个整数值，从-20（最高优先级）到+19（最低优先级）。默认情况下，bash shell以优先级0来启动所有进程。

### nice 命令

nice 命令允许你设置命令启动时的调度优先级。要让命令以更低的优先级运行，只要用 nice的 -n 命令行来指定新的优先级级别。 

```bash
$ nice -n 10 ./test4.sh > test4.out & 
[1] 4973 
$ 
$ ps -p 4973 -o pid,ppid,ni,cmd 
  PID  PPID  NI CMD 
 4973  4721  10 /bin/bash ./test4.sh 
$
# nice 命令阻止普通系统用户来提高命令的优先级
# nice命令可以在破折号后面跟上优先级
$ nice -10 ./test4.sh > test4.out & 
```

### renice 命令

renice命令可以修改系统上已运行命令的优先级。renice 命令会自动更新当前运行进程的调度优先级。

```bash
# -n 指定优先级 -p指定进程号
$ renice -n 10 -p 5055 
5055: old priority 0, new priority 10 
$ 
# renice命令有一些限制：
1.只能对属于你的进程执行 renice
2.只能通过 renice 降低进程的优先级
3. root用户可以通过 renice 来任意调整进程的优先级
```

## 定时运行作业 

### 用 at 命令来计划执行作业 

​	at 命令允许指定Linux系统何时运行脚本。 at 命令会将作业提交到队列中，指定shell何时运行该作业。 at 的守护进程 atd 会以后台模式运行，检查作业队列来运行作业。大多数Linux发行版会在启动时运行此守护进程。

​	atd 守护进程会检查系统上的一个特殊目录（通常位于/var/spool/at）来获取用 at 命令提交的作业。默认情况下， atd 守护进程会每60秒检查一下这个目录。有作业时， atd 守护进程会检查作业设置运行的时间。如果时间跟当前时间匹配， atd 守护进程就会运行此作业。 

#### 1.at命令的格式 

```bash
at [-f filename] time 
----------------------------
默认情况下， at 命令会将 STDIN 的输入放到队列中。用-f参数来指定用于读取命令（脚本文件）的文件名
time 参数指定了Linux系统何时运行该作业。如果你指定的时间已经错过， at 命令会在第二天的那个时间运行指定的作业。
时间格式:
标准的小时和分钟格式，比如10:15。 
AM/PM指示符，比如10:15 PM。 
特定可命名时间，比如now、noon、midnight或者teatime（4 PM）。 
除了指定运行作业的时间，也可以通过不同的日期格式指定特定的日期。 
标准日期格式，比如MMDDYY、MM/DD/YY或DD.MM.YY。 
文本日期，比如Jul 4或Dec 25，加不加年份均可。 
你也可以指定时间增量。 
  当前时间+25 min 
  明天10:15 PM 
  10:15+7天 
  
作业会被提交到作业队列（job queue）。作业队列会保存通过at命令提交的待处理的作业。
针对不同优先级，存在26种不同的作业队列。作业队列通常用小写字母a~z和大写字母A~Z来指代。
作业队列的字母排序越高，作业运行的优先级就越低（更高的 nice 值）。默认情况下，at 的作业会被提交到 a作业队列。如果想以更高优先级运行作业，可以用 -q 参数指定不同的队列字母。 
```

#### 2.获取作业的输出 

当作业在Linux系统上运行时，显示器并不会关联到该作业。Linux系统会将提交该作业的用户的电子邮件地址作为 STDOUT 和 STDERR 。任何发到 STDOUT 或 STDERR 的输出都会通过邮件系统发送给该用户。 

```bash
$ at -f test13.sh now 
# at 命令利用 sendmail 应用程序来发送邮件。最好在脚本中对 STDOUT 和 STDERR 进行重定向
```

#### 3.列出等待的作业 

atq 命令可以查看系统中有哪些作业在等待。 

```bash
$ atq 
20      2015-07-14 13:03 = Christine 
```

#### 4.删除作业

atrm 命令来删除等待中的作业

```bash
$ atq 
18      2015-07-15 13:03 a Christine 
17      2015-07-14 16:00 a Christine 
19      2015-07-14 13:30 a Christine 
$ 
# 指定想要删除的作业号
# 只能删除你提交的作业
$ atrm 18 
$ 
$ atq 
17      2015-07-14 16:00 a Christine 
19      2015-07-14 13:30 a Christine 
$ 
```

### 安排需要定期执行的脚本 

Linux系统使用cron程序来安排要定期执行的作业。cron程序会在后台运行并检查一个特殊的表（被称作cron时间表），以获知已安排执行的作业。

#### 1.cron时间表

cron时间表采用一种特别的格式来指定作业何时运行。其格式如下： 

```bash
min hour dayofmonth month dayofweek command
----------------------------------------------------------------------------------
文本值（mon、tue、wed、thu、fri、sat、sun）或数值（0为周日，6为周六）来指定dayofweek表项。
*通配符

# 每天的10:15运行一个命令
15 10 * * * command 
# 每周一4:15 PM运行的命令
15 16 * * 1 command
# 命令列表必须指定要运行的命令或脚本的全路径名
15 10 * * * /home/rich/test4.sh > test4out 

```

#### 2.构建cron时间表 

每个系统用户（包括root用户）都可以用自己的cron时间表来运行安排好的任务。Linux提供了 crontab 命令来处理cron时间表。

```bash
$ crontab -l 
no crontab for rich 
$ 
# 默认情况下，用户的cron时间表文件并不存在。
# 要为cron时间表添加条目，可以用 -e 选项。在添加条目时， crontab 命令会启用一个文本编辑器，使用已有的cron时间表作为文件内容（或者是一个空文件，如果时间表不存在的话） 
$crontab -e

```

#### 3.浏览cron目录

如果你创建的脚本对精确的执行时间要求不高，用预配置的cron脚本目录会更方便。有4个基本目录：hourly、daily、monthly和weekly

```bash
$ ls /etc/cron.*ly 
/etc/cron.daily: 
/etc/cron.hourly: 
/etc/cron.monthly:
/etc/cron.weekly: 

# 如果脚本需要每天运行一次，只要将脚本复制到daily目录，cron就会每天执行它
```

#### 4.anacron程序 

cron程序的唯一问题是它假定Linux系统是7×24小时运行的。如果某个作业在cron时间表中安排运行的时间已到，但这时候Linux系统处于关机状态，那么这个作业就不会被运行。当系统开机时，cron程序不会再去运行那些错过的作业。

如果anacron知道某个作业错过了执行时间，它会尽快运行该作业。这意味着如果Linux系统关机了几天，当它再次开机时，原定在关机期间运行的作业会自动运行。

anacron程序只会处理位于cron目录的程序，比如/etc/cron.monthly。它用时间戳来决定作业是否在正确的计划间隔内运行了。每个cron目录都有个时间戳文件，该文件位于/var/spool/ anacron

```bash
$ sudo cat /var/spool/anacron/cron.monthly 
20150626 
$ 
# anacron程序使用自己的时间表（通常位于/etc/anacrontab）来检查作业目录。
$ sudo cat /etc/anacrontab 
#period in days   delay in minutes   job-identifier   command 
1       5       cron.daily              nice run-parts /etc/cron.daily 

anacron时间表的基本格式： 
period delay identifier command 
# run-parts程序负责运行目录中传给它的任何脚本
# anacron不会运行位于/etc/cron.hourly的脚本。程序不会处理执行时间需求小于一天的脚本。 
```


## 参考书目

《Linux命令行与shell脚本编程大全》第3版