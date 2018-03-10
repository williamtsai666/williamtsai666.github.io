---
title: Linux命令行(4)软件包管理系统(PMS)
date: 2018-02-17 23:33:50
toc: true
tags: 
    - Linux
    - Shell
---

## 包管理基础

各种主流Linux发行版都采用了某种形式的包管理系统(package management system,PMS)来控制软件和库的安装。PMS利用一个数据库来记录各种相关内容：
Linux系统上已安装了什么软件包；
每个包安装了什么文件；
每个已安装软件包的版本。

软件包存储在服务器上，可以利用本地Linux系统上的PMS工具通过互联网访问。这些服务器称为仓库（repository）。可以用PMS工具来搜索新的软件包，或者是更新系统上已安装软件包。 

软件包通常会依赖其他的包，为了前者能够正常运行，被依赖的包必须提前安装在系统中。PMS工具将会检测这些依赖关系，并在安装需要的包之前先安装好所有额外的软件包。

PMS的不足之处在于目前还没有统一的标准工具。PMS工具及相关命令在不同的Linux发行版上有很大的不同。Linux中广泛使用的两种主要的PMS基础工具是dpkg和rpm。

基于Debian的发行版（如Ubuntu和Linux Mint）使用的是 dpkg 命令，这些发行版的PMS工具也是以该命令为基础的。 dpkg 会直接和Linux系统上的PMS交互，用来安装、管理和删除软件包。

基于Red Hat的发行版（如Fedora、openSUSE及Mandriva）使用的是 rpm 命令，该命令是其PMS的底层基础。类似于 dpkg 命令， rmp 命令能够列出已安装包、安装新包和删除已有软件。

## 基于 Debian 的系统

`dpkg` 命令是基于Debian系PMS工具的核心。包含在这个PMS中的其他工具有：

```bash
apt-get
apt-cache
aptitude
```

到目前为止，最常用的命令行工具是aptitude,`aptitude`工具本质上是`apt`工具和 `dpkg` 的前端。 `dpkg`是软件包管理系统工具，而aptitude则是完整的软件包管理系统。
命令行下使用 `aptitude` 命令有助于避免常见的软件安装问题，如软件依赖关系缺失、系统环境不稳定及其他一些不必要的麻烦。本节将会介绍如何在命令行下使用 `aptitude `命令工具。

### 用 aptitude 管理软件包

aptitude 命令直接进入软件包管理,q退出

```bash
aptitude show package_name
------------------------------
ptitude show mysql-client
Package: mysql-client             
State: not installed
Version: 5.5.38-0ubuntu0.14.04.1
...
```

无法通过 aptitude 看到的一个细节是所有跟某个特定软件包相关的所有文件的列表。要得到这个列表，就必须用 dpkg 命令。

```bash
dpkg -L package_name
------------------------------
$ dpkg -L vim-common
/.
/usr/lib/mime/packages
/usr/lib/mime/packages/vim-common
...
```

同样可以进行反向操作，查找某个特定文件属于哪个软件包

```bash
dpkg --search absolute_file_name
-------------------------------------
$ dpkg --search /usr/bin/xxd
vim-common: /usr/bin/xxd
```

### 用 aptitude 安装软件包

#### 搜索软件包

```bash
aptitude search package_name
--------------------------------------
$ aptitude search wine
p  gnome-wine-icon-theme          - red variation of the GNOME- ...
v  libkwineffects1-api            - 
p  libkwineffects1a                - library used by effects...
p  q4wine                          - Qt4 GUI for wine (W.I.N.E)
```

在每个包名字之前都有一个 p 或 i 。如果看到一个 i ，说明这个包现在已经安装到了你的系统上了。如果看到一个 p 或 v ，说明这个包可用，但还没安装

#### 从软件仓库中安装软件包。

```bash
aptitude install package_name
-----------------------------
sudo aptitude install wine
```

要检查安装过程是否正常，只要再次使用 search 选项就可以了。软件包出现了 i u  ，这说明它已经安装好了。

#### 用 aptitude 更新软件

尽管 aptitude 可以帮忙解决安装软件时遇到的问题，但解决有依赖关系的多个包的更新会比较烦琐。要用软件仓库中的新版本妥善地更新系统上所有的软件包，可用 safe-upgrade 选项。

```bash
aptitude safe-upgrade
```

还有一些不那么保守的软件升级选项：

```bash
aptitude full-upgrade
aptitude dist-upgrade
```


这些选项执行相同的任务，将所有软件包升级到最新版本。它们同 safe-upgrade 的区别在于，它们不会检查包与包之间的依赖关系。整个包依赖关系问题非常麻烦。如果不是很确定各种包的依赖关系，那还是坚持用 safe-upgrade 选项吧

#### 用 aptitude 卸载软件

```bash
#要想只删除软件包而不删除数据和配置文件
aptitude remove

#要删除软件包和相关的数据和配置文件
aptitude purge
---------------------------------------------------
sudo aptitude purge wine
```

要看软件包是否已删除，可以再用 aptitude 的 search 选项。如果在软件包名称的前面看到一个 c ，意味着软件已删除，但配置文件尚未从系统中清除；如果前面是个 p 的话，说明配置文件也已删除。

#### aptitude 仓库

aptitude 默认的软件仓库位置是在安装Linux发行版时设置的。具体位置存储在文件`/etc/apt/sources.lis`t中。

```bash
$ cat /etc/apt/sources.list
#deb cdrom:[Ubuntu 14.04 LTS _Trusty Tahr_ - Release i386 (20140417)]/
trusty main restricted
 
# See http://help.ubuntu.com/community/UpgradeNotes for how to upgrade to
# newer versions of the distribution.
deb http://us.archive.ubuntu.com/ubuntu/ trusty main restricted
deb-src http://us.archive.ubuntu.com/ubuntu/ trusty main restricted
---------------------------------------
deb (or deb-src) address  distribution_name  package_type_list
deb 或 deb-src 的值表明了软件包的类型。
deb值说明这是一个已编译程序源
deb-src值则说明这是一个源代码的源
address 条目是软件仓库的Web地址
distribution_name 条目是这个特定软件仓库的发行版版本的名称,在这个例子中，发行版名称是trusty
package_type_list 条目可能并不止一个词，它还表明仓库里面有什么类型的包。
你可以看到诸如main、restricted、universe和partner这样的值
```
## 基于 Red Hat 的系统

和基于Debian的发行版类似，基于Red  Hat的系统也有几种不同的可用前端工具。常见的有以下3种。

```bash
yum ：在Red Hat和Fedora中使用。
urpm ：在Mandriva中使用。
zypper ：在openSUSE中使用。
```

这些前端都是基于 rpm 命令行工具的

### 列出已安装包

```bash
yum list installed
```

输出的信息可能会在屏幕上一闪而过，所以最好是将已安装包的列表重定向到一个文件中。可以用 more 或 less 命令（或一个GUI编辑器）按照需要查看这个列表。

```bash
yum list installed > installed_software
```

yum 擅长找出某个特定软件包的详细信息。它能给出关于包的非常详尽的描述，另外你还可以通过一条简单的命令查看包是否已安装

```bash
yum list package_name

# yum list xterm  
Loaded plugins: langpacks, presto, refresh-packagekit  
Adding en_US to language list  
Available Packages  
xterm.i686 253-1.el6  

# yum list installed xterm  
Loaded plugins: refresh-packagekit 
Error: No matching Packages to list  
# 
```
找出系统上的某个特定文件属于哪个软件包
```bash
yum provides file_name
```

### 用 yum 安装软件

```bash
yum install package_name
```

也可以手动下载 rpm 安装文件并用 yum 安装，这叫作本地安装。基本的命令是：

```bash
yum localinstall package_name.rpm
```

### 用 yum 更新软件

列出所有已安装包的可用更新

```bash
yum list updates
```

更新某个特定软件包

```bash
yum update package_name
```

对更新列表中的所有包进行更新

```bash
yum update
```

### 用 yum 卸载软件

和 aptitude 一样，你需要决定是否保留软件包的数据和配置文件

只删除软件包而保留配置文件和数据文件,使用remove

```bash
yum remove package_name
```

要删除软件和它所有的文件,使用erase

```bash
yum erase package_name
```

### 处理损坏的包依赖关系

有时在安装多个软件包时，某个包的软件依赖关系可能会被另一个包的安装覆盖掉。这叫作损坏的包依赖关系（broken dependency）如果系统出现了这个问题，先试试下面的命令：

```bash
yum clean all
```


然后试着用 yum update。有时，只要清理了放错位置的文件就可以了。如果这还解决不了问题，试试下面的命令：

```bash
yum deplist package_name
```


这个命令显示了所有包的库依赖关系以及什么软件可以提供这些库依赖关系,一旦知道某个包需要的库，你就能安装它们了
如果这样仍未解决问题，还有最后一招：

```bash
yum update --skip-broken
--skip-broken 选项允许你忽略依赖关系损坏的那个包，继续去更新其他软件包。
```

### yum 软件仓库

要想知道你现在正从哪些仓库中获取软件，输入如下命令：

```bash
yum repolist
```


yum 的仓库定义文件位于`/etc/yum.repos.d`

### 包信息

```bash
yum info package_name
```

## 从源码安装

在好用的` rpm` 和 `dpkg`工具出现之前，管理员必须知道如何从`tarball`来解包和安装软件。将文件下载到你的Linux系统上，然后解包。
要解包一个软件的`tarball`，用标准的 tar 命令。

```bash
tar -zxvf sysstat-11.1.1.tar.gz
 
$ cd sysstat-11.1.1
$ ls
```
在这个目录的列表中，应该能看到README或AAAREADME文件。读这个文件非常重要。该文件中包含了软件安装所需要的操作。
按照README文件中的建议，下一步是系统配置。它会检查你的Linux系统，确保它拥有合适的编译器能够编译源代码，另外还要具备正确的库依赖关系

```bash
./configure
```

如果哪里有错了，在 configure 步骤中会显示一条错误消息说明缺失了什么东西

用 make 命令来构建各种二进制文件
make 命令会编译源码，然后链接器会为这个包创建最终的可执行文件。
和configure 命令一样,make 命令会在编译和链接所有的源码文件的过程中产生大量的输出
```bash
make
```

make 步骤结束时，可运行的软件程序就会出现在目录下！但是从那个目录下运行程序有些不便。你会想将它安装到Linux系统中常用的位置上。要这样的话，就必须以root用户身份登录（或者用 sudo 命令，如果你的Linux发行版偏好这个的话），然后用 make 命令的 install 选项。

```bash
make install
```
## 参考书目

《Linux命令行与shell脚本编程大全》第3版