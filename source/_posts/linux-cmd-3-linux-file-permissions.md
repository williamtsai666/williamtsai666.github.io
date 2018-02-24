---
title: Linux命令行(3)理解Linux文件权限
date: 2018-02-16 23:33:50
tags: 
    - Linux
    - Shell
---
## Linux 的安全性 

Linux安全系统的核心是用户账户。每个能进入Linux系统的用户都会被分配唯一的用户账户。用户对系统中各种对象的访问权限取决于他们登录系统时用的账户。用户权限是通过创建用户时分配的用户ID（User ID，通常缩写为UID）来跟踪的

### `/etc/passwd`文件

Linux系统使用一个专门的文件来将用户的登录名匹配到对应的UID值。为普通用户创建账户时，大多数Linux系统会从500开始，将第一个可用UID分配给这个账户。root用户账户是Linux系统的管理员，固定分配给它的UID是 0。/etc/passwd是一个标准的文本文件。

```bash
root:x:0:0:root:/root:/bin/bash
mysql:x:501:501::/home/mysql:/sbin/nologin
-----------------------------------
/etc/passwd文件的字段包含了如下信息
登录用户名
用户密码
用户账户的UID(数字形式)
用户账户的组ID(GID)(数字形式)
用户账户的文本描述(称为备注字段)
用户HOME目录的位置
用户的默认shell
 
```

`/etc/passwd`文件中的密码字段都被设置成了x,，绝大多数Linux系统都将用户密码保存在另一个单独的文件中叫作shadow文件，位置在`/etc/shadow`）。只有特定的程序（比如登录程序）才能访问这个文件。

### `/etc/shadow` 文件

`/etc/shadow`文件对Linux系统密码管理提供了更多的控制。只有root用户才能访问`/etc/shadow`文件，这让它比起`/etc/passwd`安全许多。
`/etc/shadow`文件为系统上的每个用户账户都保存了一条记录。记录就像下面这样：

```bash
rich:$1$.FfcK0ns$f1UgiyHQ25wrB/hykCn020:11627:0:99999:7:::
-------------------------------------
在/etc/shadow文件的每条记录中都有9个字段：
与/etc/passwd文件中的登录名字段对应的登录名
加密后的密码
自上次修改密码后过去的天数密码（自1970年1月1日开始计算）
多少天后才能更改密码
多少天后必须更改密码
密码过期前提前多少天提醒用户更改密码 
密码过期后多少天禁用用户账户
用户账户被禁用的日期（用自1970年1月1日到当天的天数表示）
预留字段给将来使用
```

### 添加新用户

useradd用来向Linux系统添加新用户。

`useradd` 命令使用系统的默认值以及命令行参数来设置用户账户。系统默认值被设置在`/etc/default/useradd`文件中。可以使用`useradd -D`命令查看所用Linux系统中的这些默认值

```bash
# /usr/sbin/useradd -D 
GROUP=100 
HOME=/home 
INACTIVE=-1 
EXPIRE= 
SHELL=/bin/bash 
SKEL=/etc/skel 
CREATE_MAIL_SPOOL=yes 
# 
新用户会被添加到GID为 100 的公共组；
新用户的HOME目录将会位于/home/loginname；
新用户账户密码在过期后不会被禁用；
新用户账户未被设置过期日期；
新用户账户将bash shell作为默认shell；
系统会将/etc/skel目录下的内容复制到用户的HOME目录下；
系统为该用户账户在mail目录下创建一个用于接收邮件的文件。
$ ls -al /etc/skel
total 32 
drwxr-xr-x   2 root root  4096 2010-04-29 08:26 . 
drwxr-xr-x 135 root root 12288 2010-09-23 18:49 .. 
-rw-r--r--   1 root root   220 2010-04-18 21:51 .bash_logout 
-rw-r--r--   1 root root  3103 2010-04-18 21:51 .bashrc 
-rw-r--r--   1 root root   179 2010-03-26 08:31 examples.desktop 
-rw-r--r--   1 root root   675 2010-04-18 21:51 .profile 

```

```bash
# -m  创建用户的HOME目录
useradd -m test
-c comment  给新用户添加备注 
-d home_dir  为主目录指定一个名字（如果不想用登录名作为主目录名的话） 
-e expire_date  用YYYY-MM-DD格式指定一个账户过期的日期 
-f inactive_days  指定这个账户密码过期后多少天这个账户被禁用；0表示密码一过期就立即禁用，1表示
禁用这个功能 
-g initial_group  指定用户登录组的GID或组名 
-G group ...  指定用户除登录组之外所属的一个或多个附加组 
-k  必须和-m一起使用，将/etc/skel目录的内容复制到用户的HOME目录 
-m  创建用户的HOME目录 
-M  不创建用户的HOME目录（当默认设置里要求创建时才使用这个选项） 
-n  创建一个与用户登录名同名的新组 
-r  创建系统账户 
-p passwd  为用户账户指定默认密码 
-s shell  指定默认的登录shell 
-u uid  为账户指定唯一的UID 
```

```bash
# useradd -D -s /bin/tsch 
useradd更改默认值的参数 
-b default_home  更改默认的创建用户HOME目录的位置 
-e expiration_date  更改默认的新账户的过期日期 
-f inactive  更改默认的新用户从密码过期到账户被禁用的天数 
-g group  更改默认的组名称或GID 
-s shell  更改默认的登录shell 
```

### 删除用户

默认情况下， `userdel `命令会只删除`/etc/passwd`文件中的用户信息，而不会删除系统中属于该账户的任何文件。

```bash
# 删除用户的HOME目录以及邮件目录
userdel -r test
```

### 修改用户

```bash
usermod # 修改用户账户的字段，还可以指定主要组以及附加组的所属关系
passwd # 修改已有用户的密码
chpasswd # 从文件中读取登录名密码对，并更新密码
chage # 修改密码的过期日期
chfn # 修改用户账户的备注信息
chsh # 修改用户账户的默认登录shell
```

##### 1.`usermod` 

`usermod` 命令是用户账户修改工具中最强大的一个。参数大部分跟 useradd 命令的参数一样.

-g 修改默认的登录组
-l  修改用户账户的登录名。
-L 锁定账户，使用户无法登录。
-p 修改账户的密码。
-U 解除锁定，使用户能够登录。

##### 2.`passwd`和`chpasswd`

```bash
# 更改密码
# passwd test
Changing password for user test.
New UNIX password:
Retype new UNIX password:
passwd: all authentication tokens updated successfully.
```


如果只用` passwd `命令，它会改你自己的密码。系统上的任何用户都能改自己的密码，但只有root用户才有权限改别人的密码。
-e 选项能强制用户下次登录时修改密码。你可以先给用户设置一个简单的密码，之后再强制在下次登录时改成他们能记住的更复杂的密码。

如果需要为系统中的大量用户修改密码，` chpasswd `命令能从标准输入自动读取登录名和密码对（由冒号分割）列表，给密码加密，然后为用户账户设置。
你也可以用重定向命令来将含有` userid:passwd` 对的文件重定向给该命令。
```bash
chpasswd < users.txt
```
##### 3.`chsh`、`chfn`和`chage`

`chsh `、 `chfn` 和 `chage` 工具专门用来修改特定的账户信息
`chsh` 命令用来快速修改默认的用户登录shell。使用时必须用shell的全路径名作为参数，不能只用shell名。

```bash
chsh -s /bin/csh test
Changing shell for test.
Shell changed.
```
`chfn` 命令提供了在`/etc/passwd`文件的备注字段中存储信息的标准方法。 `chfn` 命令会将用于
Unix的` finger` 命令的信息存进备注字段，而不是简单地存入一些随机文本（比如名字或昵称之类的），或是将备注字段留空。

`finger`命令可以非常方便地查看Linux系统上的用户信息。
```bash
# finger rich
Login: rich                            Name: Rich Blum
Directory: /home/rich                  Shell: /bin/bash
On since Thu Sep 20 18:03 (EDT) on pts/0 from 192.168.1.2
No mail.
No Plan.
```

如果在使用 `chfn`命令时没有参数，它会向你询问要将哪些适合的内容加进备注字段。
```bash
chfn test
Changing finger information for test.
Name []: Ima Test
Office []: Director of Technology
Office Phone []: (123)555-1234
Home Phone []: (123)555-9876
```

`chage` 命令用来帮助管理用户账户的有效期

```bash
-d  设置上次修改密码到现在的天数 
-E  设置密码过期的日期 
-I  设置密码过期到锁定账户的天数 
-m  设置修改密码之间最少要多少天 
-W  设置密码过期前多久开始出现提醒信息
```





## Linux 组

组权限允许多个用户对系统中的对象（比如文件、目录或设备等）共享一组共用的权限。Ubuntu就会为每个用户创建一个单独的与用户账户同名的组。在添加用户前后可用 `grep` 命令或 `tail` 命令
查看/etc/group文件的内容比较（` grep USERNAME  /etc/group 或 tail /etc/group` ）。

#### `/etc/group`文件

/etc/group文件包含系统上用到的每个组的信息

```bash
root:x:0:root
bin:x:1:root,bin,daemon
daemon:x:2:root,bin,daemon
sys:x:3:root,bin,adm
jessica:x:503:
mysql:x:27:
test:x:504:
----------------------
系统账户用的组通常会分配低于500的GID值，而用户组的GID则会从500开始分配。
/etc/group文件有4个字段：
组名
组密码
GID
属于该组的用户列表
```

#### 创建新组(groupadd)

```bash
# /usr/sbin/groupadd shared 
# tail /etc/group 
shared:x:505:
```


在创建新组时，默认没有用户被分配到该组。` groupadd `命令没有提供将用户添加到组中的
选项，但可以用 `usermod` 命令来弥补这一点
```bash
# usermod -G shared rich
# usermod -G shared test
-g 指定的组名会替换掉该账户的默认组。
-G 将该组添加到用户的属组的列表里，不会影响默认组。
# tail /etc/group 
shared:x:505:rich, test 
# 
# 如果更改了已登录系统账户所属的用户组，该用户必须登出系统后再登录，组关系的更改才能生效。 
```

#### 修改组(groupmod)

```bash
# /usr/sbin/groupmod -n sharing shared  
-g 修改已有组的GID
-n 修改组名
# tail /etc/group 
sharing:x:505:test,rich
```

## 理解文件权限

### 使用文件权限符 

```bash
$ ls –l 
-rw-rw-r-- 1 rich rich  50 2010-09-13 07:49 file1.gz
drwxrwxr-x 2 rich rich 4096 2010-09-03 15:12 test1
$
-----------------------------------------------------
输出结果的第一个字段就是描述文件和目录权限的编码。这个字段的第一个字符代表了对象的类型：
- 代表文件
d 代表目录
l 代表链接
c 代表字符型设备
b 代表块设备
n 代表网络设备
之后有3组三字符的编码。每一组定义了3种访问权限：
r 代表对象是可读的
w 代表对象是可写的
x 代表对象是可执行的
若没有某种权限，在该权限位会出现单破折线。这3组权限分别对应对象的3个安全级别：
对象的属主
对象的属组
系统其他用户
 
-rwxrwxr-x 1 rich rich 4882 2010-09-18 13:58 myprog
文件myprog有下面3组权限。
rwx ：文件的属主（设为登录名rich）。
rwx ：文件的属组（设为组名rich）。
r-x ：系统上其他人。
```

### 默认文件权限

`umask` 命令用来设置所创建文件和目录的默认权限

```bash
# touch 命令用分配给我的用户账户的默认权限创建了这个文件
$ touch newfile 
$ ls -al newfile 
-rw-r--r--    1 rich     rich            0 Sep 20 19:16 newfile 
$ 

# umask
0022
```

`umask` 值只是个掩码。它会屏蔽掉不想授予该安全级别的权限,要把 `umask` 值从对象的全权限值中减掉。对文件来说，全权限的值是 666 （所有用户都有读和写的权限）；而对目录来说，则是 777 （所有用户都有读、写、执行权限）

## 改变安全性设置

### 改变权限

`chmod `命令用来改变文件和目录的安全性设置

```bash
chmod options mode file 
-R 让权限的改变递归地作用到文件和子目录。你可以使用通配符指定多个文件，然后利用一条命令将权限更改应用到这些文件上。

# mode 参数可以使用八进制模式或符号模式进行安全性设置。
# 1.八进制模式 设置非常直观，直接用期望赋予文件的标准3位八进制权限码即可。
$ chmod 760 newfile
$ ls -l newfile
-rwxrw----    1 rich    rich            0 Sep 20 19:16 newfile

# 2.符号模式下指定权限的格式:
[ugoa…][[+-=][rwxXstugo…]
第一组字符定义了权限作用的对象：
u 代表用户
g 代表组
o 代表其他
a 代表上述所有
下一步，后面跟着的符号表示你是想在现有权限基础上增加权限(+)，还是在现有权限基础上移除权限(-)，或是将权限设置成后面的值(=)。
第三个符号代表作用到设置上的权限。额外的设置有以下几项:
X ：如果对象是目录或者它已有执行权限，赋予执行权限。
s ：运行时重新设置UID或GID。
t ：保留文件或目录。
u ：将权限设置为跟属主一样。
g ：将权限设置为跟属组一样。
o ：将权限设置为跟其他用户一样。
$ chmod o+r newfile
#ls -F 它能够在具有执行权限的文件名后加一个星号
$ ls -lF newfile
-rwxrw-r--    1 rich    rich            0 Sep 20 19:16 newfile*
```

options 为 chmod 命令提供了另外一些功能。 

### 改变所属关系

`chown `改变文件的属主
`chgrp `改变文件的默认属组

```bash
# chown 命令的格式如下
chown options owner[.group] file
-R 选项配合通配符可以递归地改变子目录和文件的所属关系。 
-h 选项可以改变该文件的所有符号链接文件的所属关系

# 同时改变所在属主和属组
# chown dan.shared newfile 
# ls -l newfile 
-rw-rw-r--    1 dan      shared             0 Sep 20 19:16 newfile 
# 
```

```bash
# chgrp命令可以更改文件或目录的默认属组,用户账户必须是这个文件的属主，除了能够更换属组之外，还得是新组的成员.
$ chgrp shared newfile
$ ls -l newfile
-rw-rw-r--    1 rich    shared          0 Sep 20 19:16 newfile
```

## 共享文件

Linux系统上共享文件的方法是创建组
Linux还为每个文件和目录存储了3个额外的信息位。
设置用户ID（`SUID`）：当文件被用户使用时，程序会以文件属主的权限运行。
设置组ID（`SGID`）：对文件来说，程序会以文件属组的权限运行；对目录来说，目录中创建的新文件会以目录的默认属组作为默认属组。
粘着位：进程结束后文件还驻留（粘着）在内存中。


`SGID`位对文件共享非常重要。启用`SGID`位后，你可以强制在一个共享目录下创建的新文件都属于该目录的属组，这个组也就成为了每个用户的属组。

```bash
$ mkdir testdir
$ ls -l
drwxrwxr-x    2 rich    rich        4096 Sep 20 23:12 testdir/
$ chgrp shared testdir
```

```bash
$ chmod g+s testdir  #将目录的SGID位置位，以保证目录中新建文件都用shared作为默认属组
$ ls -l
drwxrwsr-x    2 rich    shared      4096 Sep 20 23:12 testdir/
```
## 参考书目

《Linux命令行与shell脚本编程大全》第3版