[TOC]
## 目录与命令行(cli)
|    /boot     |   /dev   |   /etc   |    /lib    |     /tmp     |    /usr/src    | /usr/local/src |                  /opt                  |     /var     |
|--------------|----------|----------|------------|--------------|----------------|----------------|----------------------------------------|--------------|
| 启动相关文件 | 设备文件 | 配置文件 | 系统函数库 | 临时资源目录 | 系统及源码目录 | 用户级源码目录 | 可选程序，所有执行文件在都在应用目录下 | 系统文档内容 |

- 挂载
    + `/mnt` `/misc` `/media`都是空目录，可用于设备挂载
    + `/proc` `/sys`是内存挂载点 其中数据是直接写入内存的，不可操作
- 命令存放
    + `/bin` 系统自带可执行命令
    + `/sbin` 系统自带root可执行命令
    + `/usr/bin` 扩展可执行命令
    + `/usr/sbin` 扩展root可执行命令
- cli解释
```
[user@hostname dir]#/$ ls -lhai
50760790 drwx------. 2 user group size datetime filename
```
    + `#`表示root用户 `$`表示普通用户
    + `50760790`代表inode(i节点)地址
    + 文件权限最后那个`.`代表ACL权限
    + 数字`2`是引用计数，代表文件被引用了几次


## 关机重启
- `shutdown [option] time`
    + 例：`shutdown -r 05:30 &`在05:30的时候重启，`&`表示在后台执行不占用终端
    + `-c` 取消前一个关机命令
    + `-h` 关机(服务器不建议关机，因为不好开)
    + `-r` 重启
- 其他关机命令
    + `halt`
    + `poweroff`
    + `init 0`
- 其他重启命令
    + `reboot`
    + `init 6`
- `logout` 退出当前用户


## 用户登录查看命令
|   user   |   tty    |  from  |  login@  |     idle     | jcpu | pcpu |     what     |
|----------|----------|--------|----------|--------------|------|------|--------------|
| 登录用户 | 登录终端 | 登录IP | 登录时间 | 用户闲置时间 | -    | -    | 运行中的命令 |

- `w [username]`查看用户登录信息，执行后列表说明如上
    + `load average x y z`分别表示1,5,15分钟的平均负载
    + `tty`的值：`tty1`表示本机终端； `pts/0`代表第一个远程终端
    + `jcpu`:该终端连接的所有进程占用的时间，包括当前运行的后台作业占用时间，不包括过去后台作业时间
    + `pcpu`:该终端连接的当前进程占用的时间，值越大越耗费cpu资源
- `who [username]` 仅输出`user` `tty` `login@` `from`的内容
- `last`查询所有登录用户信息与重启信息，可通过本文件查询服务器是否被外人登入
    + `last`默认读取`/var/log/wtmp`文件数据 此文件必须用last命令查看
- `lastlog`查看所有用户最后一次登录时间


## 分配盘符
### 挂载命令`mount`
- `mount [-t 文件系统] [-o 特殊选项] 设备文件名 挂载点`
- `mount` 不加任何参数表示查询系统中已挂载的设备
- `mount -a` 将`/etc/fstab`中的所有挂载设置都挂载一遍
- `-t 文件系统` 加入文件系统类型来指定挂载的类型，可以为ext3,ext4,iso9660(光盘)等文件系统
- `-o 特殊选项` 可以指定挂载的额外选项。这里不建议修改
- 例：`mount -t iso9660 /dev/sr0 /mnt/cdrom/`将光盘`sr0`挂载到`/mnt/cdrom/`目录下，系统知道`sr0`是光盘，可省略`-t iso9660`。

### 卸载命令`umount`
- `umount 设备名或挂载点` 如`umount /mnt/cdrom`
- 若处于挂载目录下，就会卸载失败，报挂载目录正在使用的错误

### 挂载硬盘
- 不确定原来有几块硬盘，所以不知道U盘设备名。使用`fdisk -l`获取U盘设备名
- `fdisk`命令用于查看系统中已经识别的硬盘
- `mount -t vfat /dev/sdb1 /mnt/usb/` 挂载U盘 `vfat`指的是win中的`FAT32`
- 注意：Linux默认不支持NTFS文件系统

## 用户(组)操作
- [用户创建相关](https://www.cnblogs.com/mylinux/p/5576316.html)  
- [修改用户账号信息](https://www.jb51.net/LINUXjishu/57943.html)
- 用户的角色分为：root用户，普通用户，虚拟用户  
- 用户与用户组可以是多对多的关系  

### 配置文件
- `/etc/passwd` user的配置文件 
- `/etc/shadow` user影子口令文件  
- `/etc/group` group配置文件 
- `/etc/gshadow` group影子文件
- `/etc/login.defs` 创建用户的规则文件[配置项可看第一个帮助文档]
- `/etc/default/useradd` 通过useradd添加用户的规则文件
- `/etc/sudoers` 记录用户是否有执行`sudo`的权限的文件。编辑文件添加`uname ALL=(ALL) NOPASSWD: ALL`到`root ALL`下面就可以使用`sudo su`不用密码  
- `/etc/skel/` 添加user时此目录下的文件会被复制到`/home/user/`下。如果通过修改`/etc/passwd`文件添加用户，要自己创建`/home/user/`目录并复制`/etc/skel/`下文件。然后用`chown`改变`/home/user/`目录所属  

### 相关命令
- `pwck` 校验passwd与shadow是否合法  
- `grpconv` 同步group到gshadow  
- `pwunconv` 同步shadow到passwd后删除shadow  
- `su -`/`su -l`/`su --login user` 切换同时改变环境变量与工作目录  
- `finger`&`chfn` 查看&修改 用户信息的工具  
- `id` 查看当前用户(组)ID  
- `groups` 查看用户所属组   
- `useradd [option] username` 添加用户命令  
    + `-g group` 指定组（要求gname必须已创建
    + `-d home_path` 指定家目录
    + `-s` 指定所用的shell
- `passwd uname` 设置密码
    + `-l` 锁住用户
    + `-u` 解锁用户
- `usermod [option]` 修改用户信息  
    + `-G` 给已有的用户增加工作组(替换旧工作组)  
- `userdel uname` 删除用户uname
    + `-r` 删除用户及用户家目录数据
- `gpasswd [option] uname gname` 给已有的用户增加工作组  
    + `-a` 添加用户到组(多添加一个组)
    + `-d uname gname` 从组删除用户 或在`/etc/group`找到`gname`那行删除`uname`
    + `-A` 指定管理员
    + `-M` 指定组成员和-A的用途差不多；
    + `-r` 删除密码；
    + `-R` 限制用户登入组，只有组中的成员才可以用newgrp加入该组。
- `groupadd gname` 给用户添加组  
- `groupmod gname` 修改用户组    
- `groupdel gname` 删除组gname
- 临时关闭用户
    + 在`/etc/shadow`文件中找到user第二个字段（密码）前面加上*  
    + 使用`passwd uname -l`命令锁住用户

## 搜索
### locate 文件搜索
- `locate file` 无需遍历文件系统 在数据库按文件名搜索 速度更快
- `/var/lib/mlocate`是locate命令所搜索的后台数据库，每天更新。使用`updatedb`可强制更新数据库
- `/etc/updatedb.conf`是locate的配置文件，其中配置了搜索规则
```shell
# 筛选规则全都生效  yes表示生效
PRUNE_BIND_MOUNTS = "yes"
# 屏蔽的文件系统  以下文件系统不筛选(如linux部分文件系统为ext4，不在其中)
PRUNEFS = "9p afs anon_inodefs auto autofs bdev binfmt_misc cgroup cifs coda configfs cpuset debugfs devpts ecryptfs exofs fuse fuse.sshfs fusectl gfs gfs2 gpfs hugetlbfs inotifyfs iso9660 jffs2 lustre mqueue ncpfs nfs nfs4 nfsd pipefs proc ramfs rootfs rpc_pipefs securityfs selinuxfs sfs sockfs sysfs tmpfs ubifs udf usbfs ceph fuse.ceph"
# 屏蔽文件关键字 包括以下关键字的文件都不搜索
PRUNENAMES = ".git .hg .svn .bzr .arch-ids {arch} CVS"
# 屏蔽目录 以下目录中的文件都不搜索 特别注意/tmp目录
PRUNEPATHS = "/afs /media /mnt /net /sfs /tmp /udev /var/cache/ccache /var/lib/yum/yumdb /var/lib/dnf/yumdb /var/spool/cups /var/spool/squid /var/tmp /var/lib/ceph"
```

### whereis与which 命令搜索
- `whereis`与`which`只能搜索`$PATH`中指定路径的命令，不能搜索shell自带命令如`cd`
- `which` 搜索命令所在路径及别名
- `whereis` 搜索命令所在路径级帮助文档所在位置
- `whatis` 获取命令的简要说明，等同于`man -f`
- `whoami` 获取当前操作用户名称

### find 文件搜索
- `find`命令功能强大，但遍历全部文件，耗费资源。
- `find`是完全匹配，即搜索内容必须与文件名完全一致
- `find [搜索范围] [搜索条件]` 如`find /root/ -name file`
- 可使用通配符进行搜索如`*` `?` `[]`
- `find /var/log -mtime +10` 搜索10天前修改的文件
    + `atime`文件访问时间 `ctime`改变文件属性 `mtime`修改文件内容        
    + `-10`10天内 `10`10天当天 `+10`10天以前
- `-size` 指定查找文件大小 参数如`-25k` `-`表示小于，`+`表示大于
- `-inum 262422` 查找节点是`262422`的文件 
- `-iname` 不区分大小写
- `-user` 按用户搜索
- `-nouser` 搜索没有所有者的文件。在linux中所有的文件都应该有所有者，但有两种情况除外，剩下没有所有者的文件都是垃圾文件
    + `/proc` `/sys`中的文件是由内存产生，有可能没有所有者
    + 外部存储设备如U盘，u盘的文件来自于win拷贝，那么文件是有可能没有所有者的 
- `-a` `-o` 分别表示逻辑与 逻辑或。 用于上面两参数之间
- `-exec/-ok command {} \;` 将找到的结果用`command`进行处理
    + 如`find /root -inum 262422 -exec rm -rf {} \;` 将找到文件删除
- `find / -name filename &` 使用`&`可以使此命令在后台执行

### grep 字符串搜索
- `grep [option] str file` 在`file`文件中搜索包含`str`的行，`option`如下
    + `-i` 忽略大小写
    + `-v` 排除字符串 加了此选项就是搜索不包含`str`的行
- `grep`是包含匹配，即只要包含了`str`就会被搜索到
- `grep`使用正则表达式进行匹配


## 压缩与解压缩
- 常用压缩格式： `.zip` `.gz` `.bz2` `.tar.gz` `.tar.bz2`
- `.zip` 使用`zip`命令
    + `zip file.zip file` 压缩文件
    + `zip -r file.zip dir` 压缩目录
    + `unzip file.zip` 解压缩
- `.gz` 使用`gzip`命令
    + `gzip file` 将文件压缩为`file.gz`，源文件会消失
    + `gzip -c file > file.gz` `-c`是将压缩后的数据输出(可保留源文件)，使用`>`重定向符号输出到`file.gz`
    + `gzip -r dir` (解)压缩dir下的所有子文件(gzip不可以压缩目录)
    + `gzip -d file.gz` 解压缩 
    + `gunzip file.gz` 解压缩
- `.bz2` 使用`bzip2`命令
    + `bzip2`同`gzip`不能压缩目录
    + `bzip2 -k file` 压缩文件，`-k`表示保留源文件
    + `bzip2 -d file.bz2` 解压缩 `-k`保留压缩文件
    + `bunzip2 file.bz2` 解压缩 `-k`保留压缩文件
- `tar`命令：`gzip`与`bzip2`都不能压缩目录，要用`tar`命令先将目录打包，然后再压缩。
    + `-t` 指test 查看`.tar.*`包内容但不解压
    + `-c` 打包
    + `-x` 解包
    + `-v` 显示过程
    + `-f` 指定文件名
    + `-z` 指定`.tar.gz`格式
    + `-j` 指定`.tar.bz2`格式
    + `-C` 指定解压地址
    + `tar -ztvf dir.tar.gz`查看文件内容
    + `tar -zcvf /tmp/dir.tar.gz dir file` 将`dir`和`file`打包后压缩为`.gz`格式
    + `tar -jxvf /tmp/dir.tar.bz2 -C /tmp/` 将`.tar.bz2`的文件解压并解包到/tmp/目录下 


## 链接
- 文件实际存储在block块中，文件系统通过索引表 将inode与block地址关联起来
- 链接分为硬链接与软链接，对文件创建链接后，修改链接文件或源文件都会改变文件内容
- 硬链接：`ln sourcePath hard`
    + 硬链接相当于两文件指向索引表中同一条数据，
    + 可通过inode识别，使用硬链接后会使文件引用计数+1
    + 不能跨分区，不能针对目录使用，且难以区分硬链接，所有不建议使用
- 软链接：`ln -s sourcePath soft`
    + 软链接有自己的inode与block块，但数据块中只保存源文件的文件名和indoe
    + 软链接的权限都为 `lrwxrwxrwx`
    + 删除原文件，软链接不能使用


## 下载
### wget命令
- [wget命令详解](https://blog.csdn.net/T146lLa128XX0x/article/details/79266691)
- wget是linux上的命令行的下载工具。这是一个GPL许可证下的自由软件。支持断点续传；同时支持FTP与HTTP下载方式
- `wget [option] [url]`
- 启动参数
    + `-b` 启动后转入后台执行
- 记录与输入参数
    + `-o` 将下载记录写入file
    + `-a` 将下载记录追加写入file
    + `-d` 打印调试输出
    + `-q` 静默模式
    + `-v` 冗长模式(默认此模式)
- 下载参数
    + `-O` 将要下载的文档写入FILE当中
    + `-c` 接着下载没下载完的文件(断点续传)
    + `-T` 设定响应超时的秒数
- 特殊实例：`wget -qO- https://get.doctor.com/ | sh` `O-`表示标准输出，然后sh通过管道执行这个标准输出的内容。

## 文本处理
### 将文本复制到剪贴板
``` shell
# win:系统自带clip
clip < README.TXT
echo hello world | clip
echo | clip # 清空剪贴板
# unix:安装工具xsel  xsel操作在三个寄存器上:-b系统剪切板;-p默认寄存器;-s
cat file | xsel -i -b
# 命令取出内容
xsel -o -b
```

### vim复制
- `[range]s/source/destination/[option]`
- `range`替换范围
    + `n,m`第n行至第m行, `.`表示当前行, `$`表示最后一行
    + `%` 整个文件，同`1,$`
- `s`表示替换操作
- `source`&`destination`允许使用模式匹配
- `option`，省略option时仅对每行第一个匹配串进行替换
    + `g` 全局替换
    + `c` 表示进行确认
    + `p` 表示替代结果逐行显示(CTRL+L恢复屏幕)
- 如果在源字符串和目的字符串中出现特殊字符，需要用`\`转义

### sed命令
- `sed [-nefr] [function]`
- `option` 如下
    + `-n` 使用安静(silent)模式。在一般 sed 的用法中，所有来自 STDIN 的数据一般都会被列出到终端上。但如果加上 -n 参数后，则只有经过sed 特殊处理的那一行(或者动作)才会被列出来。
    + `-e` 直接在命令列模式上进行 sed 的动作编辑
    + `-f file` 运行 file 内的`sed function`
    + `-r` 使`function`支持的是延伸型正规表示法的语法。(默认是基础正规表示法语法)
    + `-i` 直接修改读取的文件内容，而不是输出到终端。
- `[n1[,n2]]function` 中n1,n2代表`选择进行动作的行数`。指定function执行在哪一行/区间
- `function` 如下
- `a`新增， a 的后面可以接字串，而这些字串会在新的一行出现(目前的下一行)～
- `c`取代， c 的后面可以接字串，这些字串可以取代 n1,n2 之间的行！
- `i`插入， i 的后面可以接字串，而这些字串会在新的一行出现(目前的上一行)；
- `d`删除，因为是删除啊，所以 d 后面通常不接任何咚咚；
- `p`列印，亦即将某个选择的数据印出。通常 p 会与参数 sed -n 一起运行～
- `s`取代，通常这个 s 的动作可以搭配正规表示法！例如 1,20s/old/new/g 就是啦！


### awk 文本分析工具用法
- `awk [options] 'program' FILE...` `program`表示程序，外部必须加上单引号识别
- `awk [options] -f programFile FILE...` `programFile`程序写在文件中用此方法执行
- `awk [options] 'PATTERN{action}' file1,file2....`

#### POSIX option
- `-f` 指定程序脚本
- `-F` 指明输入时用到的字段分隔符
- `-v var=VALUE` 自定义变量。 `在awk中的变量引用不需要加$`

#### 修饰符
- `N`: 显示宽度
- `+`: 显示数值符号
- `-{number}`: 左对齐`number`个字符
- 例 `awk -F: '{printf "username: %-20s shell: %s\n",$1,$NF}' /etc/passwd`

#### 自定义变量
- 变量可在`option`或`program`中定义
- `option`中定义: `-v var=VALUE`
- `program`中定义: `awk 'BEGIN{test="hello";print test}'`

#### 内置变量
- `$number` `$0`代表整行,`$1`代表第1列,`$2`代表第2列
- `FS` : input field seperator,输入的分隔符，默认为空白字符
- `OFS`: output field seperator,输出的分隔符，默认为空白字符
- `RS` : input record seperator,输入的换行符
- `ORS`: output record seperator,输出时的换行符
- `NF` : number of field ,字段个数
    + `awk '{print NF}' /etc/fstab`:打印每行有几个字段
    + `awk '{print $NF}' /etc/fstab`: 打印每行中的最后一个字段
- `NR` : number of record,文件中的行数
    + `awk '{print NR}' /etc/fstab`: 打印行号，其会个行号都显示
    + `awk 'END{print NR}' /etc/fstab`: 显示文本的总行数，其只是在文本处理完成后，只显示一次行号
- `FNR`: 对每个文件进行行数单独编号
    + `awk '{print FNR}' file1 file2`: 会对每个文件的行数进行单独的编号显示
- `FILENAME`: awk命令所处理的文件的名称
    + `awk '{print FILENAME}' file1`: 显示当前文件名，但会每行显示一次
    + `awk 'END{print FILENAME}' file1`: 显示当前文件名，但只会显示一次
- `ARGC`: 命令行中参数的个数，其awk命令也算一个参数
    + `awk 'END{print ARGC}' /etc/fstab`: 显示共有几个参数
- `ARGV`: 其是一个数组，保存的是命令行所给定的各参数
    + `awk 'END{print ARGV[0]}' /etc/fstab`: 显示第一个参数，默认第一个参数是awk命令本身

#### program
- 对于每个输入行，awk都会执行每个脚本代码块一次
- `BEGIN`awk在开始处理输入文件之前会执行`BEGIN`块，因此它是初始化`FS（字段分隔符）`变量、打印页眉或初始化其它在程序中以后会引用的全局变量的极佳位置。
- `END`块用于执行最终计算或打印应该出现在输出流结尾的摘要信息。
- `if...else if...else` 控制语句
- `for(i in tmps){...}` 遍历数组
- `for(i=1;i<len;i++){...}` 循环

### sort
- `sort [-bcdfimMnr] [-o outfile] [-t separator] file`
- `n` 依照数值的大小排序。
- `r` 以相反的顺序来排序。
- `k col` 按指定列排序，col为数值
- `b` 忽略每行前面开始出的空格字符。
- `d` 排序时，处理英文字母、数字及空格字符外，忽略其他的字符。
- `f` 排序时，将小写字母视为大写字母。
- `i` 排序时，除了040至176之间的ASCII字符外，忽略其他的字符。
- `m` 将几个排序好的文件进行合并。
- `M` 将前面3个字母依照月份的缩写进行排序。
- `u` 意味着是唯一的(unique)，输出的结果是去完重了的。
- `t separator` 指定排序时所用的栏位分隔字符。

## openssl
```shell
# rand子命令生成16位随机数
openssl rand -hex 16
```

## chrony服务
- `/etc/chrony.conf` 配置文件，写入要同步时间的服务器可加入多个
```shell
vim /etc/chrony.conf
# 添加时间服务器
server 210.72.145.44 iburst
server ntp.aliyun.com iburst
# 重启服务
systemctl restart chronyd.service
# 时间同步
chronyc sources -v
```