---
title: Linux学习之一Linux Bash Shell命令
date: 2018-02-14 11:11:28
tags: 
    - Linux
    - Shell
---

## 1. 基本的bash shell命令 

###  Linux 文件系统 

#### 常见Linux目录名称 

| 目    录 | 用    途                        |
| ------ | ------------------------------ |
| /      | 虚拟目录的根目录。通常不会在这里存储文件          |
| /bin  | 二进制目录，存放许多用户级的GNU工具            |
| /boot  | 启动目录，存放启动文件                    |
| /dev  | 设备目录，Linux在这里创建设备节点            |
| /etc  | 系统配置文件目录                      |
| /home  | 主目录，Linux在这里创建用户目录            |
| /lib  | 库目录，存放系统和应用程序的库文件              |
| /media | 媒体目录，可移动媒体设备的常用挂载点            |
| /mnt  | 挂载目录，另一个可移动媒体设备的常用挂载点          |
| /opt  | 可选目录，常用于存放第三方软件包和数据文件          |
| /proc  | 进程目录，存放现有硬件及当前进程的相关信息          |
| /root  | root用户的主目录                    |
| /sbin  | 系统二进制目录，存放许多GNU管理员级工具          |
| /run  | 运行目录，存放系统运作时的运行时数据            |
| /srv  | 服务目录，存放本地服务的相关文件              |
| /sys  | 系统目录，存放系统硬件信息的相关文件            |
| /tmp  | 临时目录，可以在该目录中创建和删除临时工作文件        |
| /usr  | 用户二进制目录，大量用户级的GNU工具和数据文件都存储在这里 |
| /var  | 可变目录，用以存放经常变化的文件，比如日志文件        |

#### 查看帮助命令

```bash
man command
e.g. man ls

command --help
e.g. ls --help
```

#### 遍历目录(`cd`)

```bash
cd destination
# 绝对文件路径(以斜线/开始)
christine@server01:~$ cd /usr/bin 
christine@server01:/usr/bin$  

# 当前工作目录
christine@server01:/usr/bin$ pwd 
/usr/bin 

# 相对文件路径
# 单点符（.），表示当前目录
# 双点符（..），表示当前目录的父目录
christine@server01:~/Documents$ cd ..
```

### 文件和目录列表(`ls`)

```bash
ls option
-F 添加指示符,在目录名后加了正斜线（/），在可执行文件的后面加个星号
-a 显示隐藏文件和普通文件及目录
-R 递归选项,列出了当前目录下包含的子目录中的文件
-l 列表格式的输出

$ ls -l 
total 48 
drwxr-xr-x 2 christine christine 4096 Apr 22 20:37 Desktop 
-rwxrw-r-- 1 christine christine  54 May 21 11:26 my_script 
# 参数说明
# 文件类型[目录（ d ）,文件（ - ）,字符型文件（ c ）,块设备（ b ）]
# 文件的权限 
# 文件的硬链接总数； 
# 文件属主的用户名； 
# 文件属组的组名； 
# 文件的大小（以字节为单位）； 
# 文件的上次修改时间； 
# 文件名或目录名。 

# 过滤输出
# 文件扩展匹配（file globbing）
# 问号（ ? ）代表一个字符； 
# 星号（ * ）代表零个或多个字符
$ ls -l my_scr?pt 
$ ls -l my* 
ls -l my_scr[ai]pt 
```

### 处理文件 

#### 创建文件 

```bash
touch new_file
```

#### 复制文件

```bash
cp source destination
-p  same as --preserve 保留时间戳，权限等 mode,ownership,timestamps
-R, -r, --recursive 递归拷贝 copy directories recursively
```

#### 链接文件

```bash
在Linux中有两种不同类型的文件链接： 1.符号链接  2.硬链接 
# 符号链接,一个文件指向存放在虚拟目录结构中某个地方的另一个文件,这两个通过符号链接在一起的文件，彼此的内容并不相同
# 创建符号链接
$ ls -l data_file 
-rw-rw-r-- 1 christine christine 1092 May 21 17:27 data_file 
$ ln -s data_file  sl_data_file 
$ ls -l *data_file 
-rw-rw-r-- 1 christine christine 1092 May 21 17:27 data_file 
lrwxrwxrwx 1 christine christine    9 May 21 17:29 sl_data_file -> data_file 

# 证明链接文件是独立文件,需要查看inode编号。文件或目录的inode编号是一个用于标识的唯一数字，这个数字由内核分配给文件系统中的每一个对象 
$ ls -i *data_file 
296890 data_file  296891 sl_data_file 

# 硬链接,会创建独立的虚拟文件，其中包含了原始文件的信息及位置。但是它们从根本上而言是同一个文件。引用硬链接文件等同于引用了源文件
$ ls -l code_file 
-rw-rw-r-- 1 christine christine 189 May 21 17:56 code_file 
$ ln code_file  hl_code_file 
$ ls -li *code_file 
296892 -rw-rw-r-- 2 christine christine 189 May 21 17:56  code_file 
296892 -rw-rw-r-- 2 christine christine 189 May 21 17:56  hl_code_file 
# 注意: 链接计数（列表中第三项）显示这两个文件都有两个链接
# 只能对处于同一存储媒体的文件创建硬链接。要想在不同存储媒体的文件之间创建链接，只能使用符号链接。

```

#### 重命名文件(移动/moving)

```bash
# mv 来移动文件的位置
$ mv src dest

# mv 命令移动文件位置并修改文件名称
$ mv /home/christine/Pictures/fzll  /home/christine/fall 
```

#### 删除文件

```bash
rm option file
-f  --force 强制删除(ignore nonexistent files and arguments, never prompt)
-i  删前提示(prompt before every removal)
-r, -R, --recursive 递归删除(remove directories and their contents recursively)

$ rm -rf file_or_dir
```

### 处理目录

#### 创建目录

```bash
mkdir New_Dir 
-p, --parents 递归创建目录 no error if existing, make parent directories as needed
$ mkdir -p New_Dir/Sub_Dir/Under_Dir 
```

#### 删除目录 

```bash
rmdir New_Dir 
# 默认情况下， rmdir命令只删除空目录
# 可以使用rm递归删除
rm -rf New_Dir
```

#### 查看目录(tree)

```bash
# tree 工具 :美观的方式展示目录、子目录及其中的文件
tree dir 
```

### 查看文件内容

#### 查看文件类型

```bash
file file_name

$ file my_file 
my_file: ASCII text # 文本文件
$ file New_Dir 
New_Dir: directory # 目录
$ file sl_data_file 
sl_data_file: symbolic link to 'data_file'  # 符号链接的文件
$ file my_script 
my_script: Bourne-Again shell script, ASCII text executable  # 脚本文件
$ file /bin/ls 
/bin/ls: ELF 64-bit LSB  executable, x86-64, version 1 (SYSV),  # 二进制可执行程序
dynamically linked (uses shared libs), for GNU/Linux 2.6.24,  
```
#### 查看整个文件 

#####  1.cat命令 

```bash
cat file #显示文本文件中所有数据
-n 加上行号
-b 给有文本的行加上行号
```

#### 2.more命令

```bash
more file # 分页工具
空格:换页
回车:逐行
q:退出
```

##### 3.less命令 

```bash
less file
# less is more, more命令的升级版
```

#### 查看部分文件 

##### 1.tail命令 

```bash
# tai命令会显示文件最后几行的内容。默认情况下，它会显示文件的末尾10行
tail file
-n, --lines=[+]NUM output  the last NUM lines, instead of the last 10; or use -n +NUM to output starting with line NUM 控制显示行数
-f, --follow[={name|descriptor}]
              output appended data as the file grows;
              an absent option argument means 'descriptor'
              允许你在其他进程使用该文件时查看文件的内容。tail 命令会保持活动状态，并不断显示添加到文件              中的内容。这是实时监测系统日志的绝妙方式。 
$ tail -n 2 log_file # 显示最后两行
line19 
Last line - line20 
```

##### 2.head命令

```bash
# head命令，显示文件开头那些行的内容。默认情况下，它会显示文件前10行的文本.\
head file

head -5 log_file #显示5行
```

## 2.更多的bash shell命令 

### 监测程序

#### 探查进程(`ps`)

Linux系统中使用的GNU `ps` 命令支持3种不同类型的命令行参数： 

1. Unix风格的参数，前面加单破折线, Unix风格的参数是从贝尔实验室开发的AT&T Unix系统上原有的 ps 命令继承下来的
2. BSD风格的参数，前面不加破折线； 
3. GNU风格的长参数，前面加双破折线。 

```bash
ps option
# Unix风格的ps命令参数 
-A  显示所有进程 
-N  显示与指定参数不符的所有进程 
-a  显示除控制进程(session leader)和无终端进程外的所有进程 
-d  显示除控制进程外的所有进程 
-e  显示所有进程 
-C cmdlist  显示包含在cmdlist列表中的进程 
-G grplist  显示组ID在grplist列表中的进程 
-U userlist  显示属主的用户ID在userlist列表中的进程 
-g grplist  显示会话或组ID在grplist列表中的进程
-p pidlist  显示PID在pidlist列表中的进程 
-s sesslist  显示会话ID在sesslist列表中的进程 
-t ttylist  显示终端ID在ttylist列表中的进程 
-u userlist  显示有效用户ID在userlist列表中的进程 
-F  显示更多额外输出（相对-f参数而言） 
-O format  显示默认的输出列以及format列表指定的特定列 
-M  显示进程的安全信息 
-c  显示进程的额外调度器信息 
-f  显示完整格式的输出 
-j  显示任务信息 
-l  显示长列表 
-o format  仅显示由format指定的列 
-y  不要显示进程标记（process flag，表明进程状态的标记） 
-Z  显示安全标签（security context） ① 信息 
-H  用层级格式来显示进程（树状，用来显示父进程） 
-n namelist  定义了WCHAN列显示的值 
-w  采用宽输出模式，不限宽度显示 
-L  显示进程中的线程 
-V  显示ps命令的版本号 
# GNU长参数 
--deselect  显示所有进程，命令行中列出的进程 
--Group grplist  显示组ID在grplist列表中的进程 
--User userlist  显示用户ID在userlist列表中的进程 
--group grplist  显示有效组ID在grplist列表中的进程 
--pid pidlist  显示PID在pidlist列表中的进程 
--ppid pidlist  显示父PID在pidlist列表中的进程 
--sid sidlist  显示会话ID在sidlist列表中的进程 
--tty ttylist  显示终端设备号在ttylist列表中的进程 
--user userlist  显示有效用户ID在userlist列表中的进程 
--format format  仅显示由format指定的列 
--context  显示额外的安全信息 
--cols n  将屏幕宽度设置为n列 
--columns n  将屏幕宽度设置为n列 
--cumulative  包含已停止的子进程的信息 
--forest  用层级结构显示出进程和父进程之间的关系 
--headers  在每页输出中都显示列的头 
--no-headers  不显示列的头 
--lines n  将屏幕高度设为n行 
--rows n  将屏幕高度设为n排 
--sort order  指定将输出按哪列排序 
--width n  将屏幕宽度设为n列 
--help  显示帮助信息 
--info  显示调试信息 
--version  显示ps命令的版本号 
# 默认情况下， ps命令只会显示运行在当前控制台下的属于当前用户的进程
$ ps 
PID TTY          TIME CMD 
3081 pts/0    00:00:00 bash 
3209 pts/0    00:00:00 ps 
$
# -ef选项 -e 参数指定显示所有运行在系统上的进程； -f 参数则扩展了输出
$ ps -ef 
UID        PID  PPID  C STIME TTY          TIME CMD 
root        1    0  0 11:29 ?        00:00:01 init [5] 
root        2    0  0 11:29 ?        00:00:00 [kthreadd] 
root      3078  1981  0 12:00 ?        00:00:00 sshd: rich [priv] 
rich      3080  3078  0 12:00 ?        00:00:00 sshd: rich@pts/0 
rich      3081  3080  0 12:00 pts/0    00:00:00 -bash 
rich      4445  3081  3 13:48 pts/0    00:00:00 ps -ef 
# 参数说明
UID：启动这些进程的用户。 
PID：进程的进程ID。 
PPID：父进程的进程号（如果该进程是由另一个进程启动的）。 
C：进程生命周期中的CPU利用率。 
STIME：进程启动时的系统时间。 
TTY：进程启动时的终端设备。 
TIME：运行进程需要的累计CPU时间。 
CMD：启动的程序名称。 

# -l 参数，它会产生一个长格式输出。
$ ps -l 
F S  UID PID  PPID  C PRI  NI ADDR SZ WCHAN  TTY      TIME  CMD 
0 S  500 3081  3080  0  80  0 -  1173 wait pts/0  00:00:00 bash 
0 R  500 4463  3081  1  80  0 -  1116 -    pts/0  00:00:00 ps 
# 参数说明
F ：内核分配给进程的系统标记。 
S ：进程的状态（O代表正在运行；S代表在休眠；R代表可运行，正等待运行；Z代表僵化，进程已结束但父进程已不存在；T代表停止）。 
PRI ：进程的优先级（越大的数字代表越低的优先级）。 
NI ：谦让度值用来参与决定优先级。 
ADDR ：进程的内存地址。 
SZ ：假如进程被换出，所需交换空间的大致大小。 
WCHAN ：进程休眠的内核函数的地址。

# 常用ps命令选项
$ ps -elf

# 显示进程树
$ ps --forest
```

#### 实时监测进程 (top)

```bash
top - 09:42:15 up 10 days,  9:55,  1 user,  load average: 0.00, 0.00, 0.00
Tasks:  22 total,  1 running,  21 sleeping,  0 stopped,  0 zombie
Cpu(s):  0.3%us,  0.0%sy,  0.0%ni, 99.7%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
Mem:    524288k total,  110564k used,  413724k free,        0k buffers
Swap:    65536k total,    65536k used,        0k free,    28868k cached
  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND
    1 root      20  0 19232  116  24 S  0.0  0.0  0:00.01 init
    3 root      20  0    0    0    0 S  0.0  0.0  0:00.00 khelper/512917
  866 mysql    20  0  373m 1336  128 S  0.0  0.3  1:36.24 mysqld
  943 root      20  0 1194m  28m 3664 S  0.0  5.6  0:42.29 node
11213 root      20  0 11440 1788 1400 S  0.0  0.3  0:00.00 bash

# 说明
## 第一部分 :系统的概况

# 第一行显示了当前时间、系统的运行时间、登录的用户数以及系统的平均负载。
# 平均负载有3个值：最近1分钟的、最近5分钟的和最近15分钟的平均负载。值越大说明系统的负载越高。由于进程短期的突发性活动，出现最近1分钟的高负载值也很常见，但如果近15分钟内的平均负载都很高，就说明系统可能有问题

# 第二行显示了进程概要信息—— top命令的输出中将进程叫作任务（task）：有多少进程处在运行、休眠、停止或是僵化状态（僵化状态是指进程完成了，但父进程没有响应）。 

# 第三行显示了CPU的概要信息。 top 根据进程的属主（用户还是系统）和进程的状态（运行、空闲还是等待）将CPU利用率分成几类输出

# 第四行显示了系统的物理内存：总共有多少内存，当前用了多少，还有多少空闲

# 第五行显示了系统交换空间的状态

## 第二部分：当前运行中的进程的详细列表

PID：进程的ID。 
USER：进程属主的名字。 
PR：进程的优先级。 
NI：进程的谦让度值。 
VIRT：进程占用的虚拟内存总量。 
RES：进程占用的物理内存总量。 
SHR：进程和其他进程共享的内存总量。 
S：进程的状态（D代表可中断的休眠状态，R代表在运行状态，S代表休眠状态，T代表跟踪状态或停止状态，Z代表僵化状态）。 
%CPU：进程使用的CPU时间比例。 
%MEM：进程使用的内存占可用内存的比例。 
TIME+：自进程启动到目前为止的CPU时间总量。 
COMMAND：进程所对应的命令行名称，也就是启动的程序名。

# 默认情况下,top 命令在启动时会按照 %CPU 值对进程排序。可以在 top 运行时使用多种交互命令重新排序。每个交互式命令都是单字符。
f :选择对输出进行排序的字段，
d :修改轮询间隔。
q :退出 
```

#### 结束进程

##### Linux进程信号

在Linux中，进程之间通过信号来通信。进程的信号就是预定义好的一个消息，进程能识别它并决定忽略还是作出反应。进程如何处理信号是由开发人员通过编程来决定的。

```bash
# Linux进程信号
信号 名称  描述 
1    HUP  挂起 
2    INT  中断 
3    QUIT  结束运行 
9    KILL  无条件终止 
11  SEGV  段错误 
15  TERM  尽可能终止 
17  STOP  无条件停止运行，但不终止 
18  TSTP  停止或暂停，但继续在后台运行 
19  CONT  在STOP或TSTP之后恢复执行 
```

##### kill命令 

```bash
# kill 命令可通过进程ID（PID）给进程发信号。默认情况下， kill 命令会向命令行中列出的全部PID发送一个 TERM 信号
# 要发送进程信号，你必须是进程的属主或登录为root用户

# -s 参数支持指定信号名或信号值
$ kill -s HUP 3940 

# 常用kill命令,强制终止进程
$ kill -9 3940
```

##### killall命令 

```bash
# killall 它支持通过进程名而不是PID来结束进程。 killall 命令也支持通配符
$ killall http* 
```

### 监测磁盘空间 

####  挂载存储媒体

Linux文件系统将所有的磁盘都并入一个虚拟目录下。在使用新的存储媒体之前，需要把它放到虚拟目录下。这项工作称为挂载（mounting）

#####  1.mount命令 

```bash
# Linux上用来挂载媒体的命令叫作mount 。默认情况下， mount命令会输出当前系统上挂载的设备列表
$ mount 
proc on /proc type proc (rw) 
sysfs on /sys type sysfs (rw)
/dev/sda1 on /boot type ext3 (rw) 
sunrpc on /var/lib/nfs/rpc_pipefs type rpc_pipefs (rw) /dev/sdb1 on /media/disk type vfat (rw,nosuid,nodev,uhelper=hal,shortname=lower,uid=503) 
$ 
# mount 命令提供如下四部分信息 
# 媒体的设备文件名 
# 媒体挂载到虚拟目录的挂载点 
# 文件系统类型 
# 已挂载媒体的访问状态 

# 手动在虚拟目录中挂载设备命令
mount -t type device directory 
# type:指定了磁盘被格式化的文件系统类型
## vfat：Windows长文件系统。 
## ntfs：Windows NT、XP、Vista以及Windows 7中广泛使用的高级文件系统。 
## iso9660：标准CD-ROM文件系统。 
# device:存储设备的设备文件的位置
# directory:挂载点在虚拟目录中的位置

# 手动将U盘/dev/sdb1挂载到/media/disk
mount -t vfat /dev/sdb1 /media/disk 

# mount命令的参数 
-a  挂载/etc/fstab文件中指定的所有文件系统 
-f  使mount命令模拟挂载设备，但并不真的挂载 
-F  和-a参数一起使用时，会同时挂载所有文件系统 
-v  详细模式，将会说明挂载设备的每一步 
-I  不启用任何/sbin/mount.filesystem下的文件系统帮助文件 
-l  给ext2、ext3或XFS文件系统自动添加文件系统标签 
-n  挂载设备，但不注册到/etc/mtab已挂载设备文件中 
-p num  进行加密挂载时，从文件描述符num中获得密码短语 
-s  忽略该文件系统不支持的挂载选项 
-r  将设备挂载为只读的 
-w  将设备挂载为可读写的（默认参数） 
-L label  将设备按指定的label挂载 
-U uuid  将设备按指定的uuid挂载 
-O  和-a参数一起使用，限制命令只作用到特定的一组文件系统上 
-o  给文件系统添加特定的选项 

-o 参数允许在挂载文件系统时添加一些以逗号分隔的额外选项。
# 常用的选项。 
ro ：以只读形式挂载。 
rw ：以读写形式挂载。 
user ：允许普通用户挂载文件系统。 
check=none ：挂载文件系统时不进行完整性校验。 
loop ：挂载一个文件。 
```

##### 2.umount命令

```bash
# 从Linux系统上移除一个可移动设备时，不能直接从系统上移除，而应该先卸载。
# umount 命令支持通过设备文件或者是挂载点来指定要卸载的设备
umount [directory | device ] 

# 如果有任何程序正在使用设备上的文件，系统就不会允许你卸载它
[root@testbox mnt]# umount /home/rich/mnt 
umount: /home/rich/mnt: device is busy  
[root@testbox mnt]# cd /home/rich 
[root@testbox rich]# umount /home/rich/mnt 
[root@testbox rich]# ls -l mnt 
total 0 
#  如果在卸载设备时，系统提示设备繁忙，无法卸载设备，通常是有进程还在访问该设备或使用该设备上的文件。
#  这时可用 lsof 命令获得使用它的进程信息，然后在应用中停止使用该设备或停止该进程。 lsof 命令的用法很简
#  单： lsof /path/to/device/node ，或者 lsof /path/to/mount/point 。
```

#### 使用 df 命令 

```bash
# 查看所有已挂载磁盘的使用情况, df命令会显示每个有数据的已挂载文件系统
$ df 
Filesystem          1K-blocks      Used Available Use% Mounted on 
/dev/sda2            18251068  7703964  9605024  45% / 
/dev/sda1              101086    18680    77187  20% /boot 
# 参数说明
## 设备的设备文件位置； 
## 能容纳多少个1024字节大小的块； 
## 已用了多少个1024字节大小的块； 
## 还有多少个1024字节大小的块可用； 
## 已用空间所占的比例； 
## 设备挂载到了哪个挂载点上。 

# 常用选项 -h(human reading)，查看可用空间
$ df -h 
Filesystem            Size  Used Avail Use% Mounted on 
/dev/sdb2              18G  7.4G  9.2G  45% / 
/dev/sda1              99M  19M  76M  20% /boot 
```

#### 使用 du 命令 

```bash
# du命令可以显示某个特定目录（默认情况下是当前目录）的磁盘使用情况
# 默认情况下， du 命令会显示当前目录下所有的文件、目录和子目录的磁盘使用情况，它会以磁盘块为单位来表明每个文件或目录占用了多大存储空间。对标准大小的目录来说，这个输出会是一个比较长的列表
$ du 
484    ./.gstreamer-0.10 
8      ./Templates 
8      ./Download 
8      ./.ccache/7/0 
# 每行输出左边的数值是每个文件或目录占用的磁盘块数。注意，这个列表是从目录层级的最底部开始，然后按文件、子目录、目录逐级向上

# 常见参数
-c ：显示所有已列出文件总的大小。 
-h ：按用户易读的格式输出大小，即用K替代千字节，用M替代兆字节，用G替代吉字节。 
-s ：显示每个输出参数的总计。 

# 常见用法 查看当前目录所有的文件，目录和子目录的磁盘使用情况
du -h
```

### 处理数据文件 

#### 排序数据

```bash
# sort命令按照会话指定的默认语言的排序规则对文本文件中的数据行排序
sort [options] file
# sort命令参数 
-b  --ignore-leading-blanks  排序时忽略起始的空白 
-C  --check=quiet  不排序，如果数据无序也不要报告 
-c  --check  不排序，但检查输入数据是不是已排序；未排序的话，报告 
-d  --dictionary-order  仅考虑空白和字母，不考虑特殊字符 
-f  --ignore-case  默认情况下，会将大写字母排在前面；这个参数会忽略大小写 
-g  --general-number-sort  按通用数值来排序（跟-n不同，把值当浮点数来排序，支持科学计数法表示的值） 
-i  --ignore-nonprinting  在排序时忽略不可打印字符 
-k  --key=POS1[,POS2]  排序从POS1位置开始；如果指定了POS2的话，到POS2位置结束 
-M  --month-sort  用三字符月份名按月份排序 
-m  --merge  将两个已排序数据文件合并 
-n  --numeric-sort  按字符串数值来排序（并不转换为浮点数） 
-o  --output=file  将排序结果写出到指定的文件中 
-R  --random-sort  按随机生成的散列表的键值排序 
  --random-source=FILE  指定-R参数用到的随机字节的源文件 
-r  --reverse  反序排序（升序变成降序） 
-S  --buffer-size=SIZE  指定使用的内存大小 
-s  --stable  禁用最后重排序比较 
-T  --temporary-directory=DIR  指定一个位置来存储临时工作文件 
-t  --field-separator=SEP  指定一个用来区分键位置的字符 
-u  --unique  和-c参数一起使用时，检查严格排序；不和-c参数一起用时，仅输出第一例相似的两行 
-z  --zero-terminated  用NULL字符作为行尾，而不是用换行符 

# 常见用法
# /etc/passwd根据用户ID进行数值排序
$ sort -t ':' -k 3 -n /etc/passwd 
root:x:0:0:root:/root:/bin/bash 
bin:x:1:1:bin:/bin:/sbin/nologin 

# 查看占用空间
$ du -sh * | sort -nr 
1008k  mrtg-2.9.29.tar.gz 
972k    bldg1 
888k    fbs2.pdf 

```

#### 搜索数据

```bash
# grep 命令会在输入或指定的文件中查找包含匹配指定模式的字符的行。 grep 的输出就是包含了匹配模式的行
# 默认情况下， grep 命令用基本的Unix风格正则表达式来匹配模式
grep [options] pattern [file]
# 常见参数 
-v 反向搜索（输出不匹配该模式的行）
-n 显示匹配模式的行所在的行号
-c 含有匹配的模式行数
-e 指定多个匹配模式
$ grep -e t -e f file1 

# egrep 命令是 grep 的一个衍生，支持POSIX扩展正则表达式。POSIX扩展正则表达式含有更多的可以用来指定匹配模式的字符
#  fgrep 则是另外一个版本，支持将匹配模式指定为用换行符分隔的一列固定长度的字符串

```

#### 压缩数据 

#####1. 压缩文件

```bash
gzip file_name
```

#####2.解压文件

```bash
gunzip file_name
```

#### 归档数据 

```bash
tar function [options] object1 object2 ... 
# tar命令的功能 
-A  --concatenate  将一个已有tar归档文件追加到另一个已有tar归档文件 
-c  --create  创建一个新的tar归档文件 
-d  --diff  检查归档文件和文件系统的不同之处 
  --delete  从已有tar归档文件中删除 
-r  --append  追加文件到已有tar归档文件末尾 
-t  --list  列出已有tar归档文件的内容 
-u  --update  将比tar归档文件中已有的同名文件新的文件追加到该tar归档文件中 
-x  --extract  从已有tar归档文件中提取文件 
# tar命令选项 
-C dir  切换到指定目录 
-f file  输出结果到文件或设备file  
-j  将输出重定向给bzip2命令来压缩内容 
-p  保留所有文件权限 
-v  在处理文件时显示文件 
-z  将输出重定向给gzip命令来压缩内容 
```

##### 1.归档文件

```bash
# 文件名以.tgz/.tar.gz结尾 代表是gzip压缩过的tar文件
tar -zcvf dest.tar.gz file1 dir1
```

#####2.列出文件

```bash
tar -tf test.tar.gz
```

###### 3.提取文件

```bash
tar -zxvf test.tar.gz
```

## 3.理解shell 

### shell 的父子关系

用于登录某个虚拟控制器终端或在GUI中运行终端仿真器时所启动的默认的交互shell，是一个父shell

在CLI提示符后输入 /bin/bash 命令或其他等效的 bash 命令时，会创建一个新的shell程序。这个shell程序被称为子shell（child shell）。子shell也拥有CLI提示符，同样会等待命令输入.

```bash
$ ps -f 
UID        PID  PPID  C STIME TTY          TIME CMD 
501      1841  1840  0 11:50 pts/0    00:00:00 -bash 
501      2429  1841  4 13:44 pts/0    00:00:00 ps -f 
$ 
$ bash 
$ 
$ ps -f 
UID        PID  PPID  C STIME TTY          TIME CMD 
501      1841  1840  0 11:50 pts/0    00:00:00 -bash 
501      2430  1841  0 13:44 pts/0    00:00:00 bash 
501      2444  2430  1 13:44 pts/0    00:00:00 ps -f 
$ 
# 第一个bash shell程序，也就是父shell进程，其原始进程ID是 1814 。第二个bash shell程序，即子shell进程，其PID是 2430 。注意，子shell的父进程ID（PPID）是 1841 ，指明了这个父shell进程就是该子shell的父进程

#  bash命令行参数 
-c string  从string中读取命令并进行处理 
-i  启动一个能够接收用户输入的交互shell 
-l  以登录shell的形式启动 
-r  启动一个受限shell，用户会被限制在默认目录中 
-s  从标准输入中读取命令 
```

#### 查看父子进程之间的关系

```bash
ps -elf --forest
```

#### 退出shell

```bash
exit
```

### 进程列表 

进程列表 生成一个子shell来执行对应的命令

```bash
$ pwd ; ls ; cd /etc ; pwd ; cd ; pwd ; ls ; echo $BASH_SUBSHELL 
Documents  junk.dat  Pictures  Templates 
0 
# 在命令输出的最后，显示的是数字 0 。这就表明这些命令不是在子shell中运行的
$ (pwd ; ls ; cd /etc ; pwd ; cd ; pwd ; ls ; echo $BASH_SUBSHELL) 
Documents  junk.dat  Pictures  Templates 
1 
# 这次在命令输入的最后显示出了数字 1 。这表明的确创建了子shell，并用于执行这些命令。 所以说，命令列表就是使用括号包围起来的一组命令，它能够创建出子shell来执行这些命令

# 进程列表是一种命令分组（command grouping）。另一种命令分组是将命令放入花括号中，并在命令列表尾部加上分号（;）。语法为 { command; } 。使用花括号进行命令分组并不会像进程列表那样创建出子shel

# echo $BASH_SUBSHELL 。如果该命令返回 0 ，就表明没有子shell。如果返回1 或者其他更大的数字，就表明存在子shell
```

### 别出心裁的子 shell 用法

#### 后台作业(background job)

```bash
command &
[1] 2396 # 1:后台作业号 2396:后台作业进程id
[1]+ Done command #作业完成后的提示
```


显示当前运行在后台模式中的所有用户的进程(作业)

```bash
jobs
[1]+ Running command
```

### 理解 shell 的内建命令

#### 外部命令

也被称为文件系统命令，是存在于bash shell之外的程序。它们并不是shell程序的一部分。外部命令程序通常位于`/bin`、`/usr/bin`、`/sbin`或`/usr/sbin`中。

```bash
# ps就是一个外部命令。你可以使用 which 和 type 命令找到它。
$ which ps
/bin/ps
$
$ type -a ps
ps is /bin/ps
$
$ ls -l /bin/ps
-rwxr-xr-x 1 root root 93232 Jan  6 18:32 /bin/ps
# 当外部命令执行时，会创建出一个子进程。这种操作被称为衍生（forking）。外部命令 ps 很方便显示出它的父进程以及自己所对应的衍生子进程。 
$ ps -f 
UID        PID  PPID  C STIME TTY          TIME CMD 
christi+  2743  2742  0 17:09 pts/9    00:00:00 -bash 
christi+  2801  2743  0 17:16 pts/9    00:00:00 ps -f 

# 就算衍生出子进程或是创建了子shell，你仍然可以通过发送信号与其沟通，发送信号（signaling）使得进程间可以通过信号进行通信
```

#### 内建命令

内建命令和外部命令的区别在于前者不需要使用子进程来执行。它们已经和shell编译成了一体，作为shell工具的组成部分存在。不需要借助外部程序文件来运行。

```bash
# cd 和 exit 命令都内建于bash shell。可以利用 type 命令来了解某个命令是否是内建的。
$ type cd
cd is a shell builtin
$
$ type exit
exit is a shell builtin
# 因为既不需要通过衍生出子进程来执行，也不需要打开程序文件，内建命令的执行速度要更快，效率也更高
```


要注意，有些命令有多种实现。例如 echo 和 pwd 既有内建命令也有外部命令。两种实现略有不同。要查看命令的不同实现，使用 type 命令的 -a 选项。

```bash
$ type -a echo
echo is a shell builtin
echo is /bin/echo
$
$ which echo
/bin/echo
$
$ type -a pwd
pwd is a shell builtin
pwd is /bin/pwd
$
$ which pwd
/bin/pwd
$
```


命令 type -a 显示出了每个命令的两种实现。注意， which 命令只显示出了外部命令文件。

##### 1.history命令

```bash
# 查看最近用过的命令列表
history
!! #重新执行上一条命令
!20 #唤回历史列表中任意一条命令。只需输入惊叹号和命令在历史列表中的编号即可

# 命令历史记录被保存在隐藏文件.bash_history中，它位于用户的主目录中。这里要注意的是，bash命令的历史记录是先存放在内存中，当shell退出时才被写入到历史文件中。
$ history -a #强制将命令历史记录提前写入写入
$ history -n #更新终端会话的历史记录
```

#####2. 命令别名

alias 命令是另一个shell的内建命令。命令别名允许你为常用的命令（及其参数）创建另一个名称，从而将输入量减少到最低。 

```bash
#查看当前可用的别名
alias -p

#创建别名 一个别名仅在它所被定义的shell进程中才有效。
alias li='ls -li'
```

## 参考书目

《Linux命令行与shell脚本编程大全》第3版