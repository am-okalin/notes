[TOC]
## 虚拟机
- [VBOX下载](https://www.virtualbox.org/wiki/Downloads)
- 若无法创建64位虚拟机，则要进入boot开启`Intel 虚拟化技术`或`Intel VT-x`或`virtualization`(差不多和virtual沾点边就是了)
- 虚拟系统克隆: 可将已有的虚拟系统进行复制
    + 链接克隆：删除源镜像后不可使用
    + 完整克隆：删除源镜像后可以使用
- virtualBox网络配置
    + [virtualbox四种网络连接方式及其设置方法](https://www.jianshu.com/p/bdf5d72e6ba4)
    + 设置网卡`NAT转换`模式即可上网
    + 设置网卡`仅主机`模式可分配固定IP，并让虚拟机与主机通信

## linux初始化
- linux有多个发行版本，目前服务器主流是centos，个人PC主流是ubuntu(debian衍生)。
- 硬盘概念，win与linux的文件系统不同，它们的初始化也不同
    + win： 分区-格式化-分配挂载点(即盘符)
    + linux： 分区-格式化-设备文件名-分配挂载点
    + 格式化：写入文件系统
    + 挂载：将盘符与分区链接在一起的过程
- 系统分区：在安装linux系统时要进行系统分区：
    + `/`(根分区) ext4格式
    + `/boot`(启动分区400M。若不分区且根分区写满，就没有空间用于系统启动) ext4格式
    + `swap`(内存交换分区，内存小于4G分配内存的2倍，内存大于4G分配内存的1倍) swap格式
- 安装好后的基础配置

```shell
# 用root用户创建个人用户，并添加sudoers
useradd tony
passwd tony
sed -i '/^root.*ALL=(ALL).*ALL/a\tony\tALL=(ALL) \tALL' /etc/sudoers

# 配置$HOME/.bashrc文件
su tony
vi $HOME/.bashrc
mkdir ~/bin ~/workspace

# 需要安装的基础包，不同linux发行版安装命令不同，但包名一样
sudo yum -y install make autoconf automake cmake perl-CPAN libcurl-devel libtool gcc gcc-c++ glibc-headers zlib-devel git-lfs telnet ctags lrzsz jq expat-devel openssl-devel vim mlocate git psmisc
```
```shell
# $HOME/.bashrc
 
# User specific aliases and functions
alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'
 
# Source global definitions(载入全局环境变量配置)
if [ -f /etc/bashrc ]; then
        . /etc/bashrc
fi
 
# User specific environment
# Basic envs
export LANG="en_US.UTF-8" # 设置系统语言为 en_US.UTF-8，避免终端出现中文乱码
export PS1='[\u@dev \W]\$ ' # 默认的 PS1 设置会展示全部的路径，为了防止过长，这里只展示："用户名@dev 最后的目录名"
export WORKSPACE="$HOME/workspace" # 设置工作目录
export PATH=$HOME/bin:$PATH # 将 $HOME/bin 目录加入到 PATH 变量中
 
# Default entry folder
cd $WORKSPACE # 登录系统，默认进入 workspace 目录
```

- 各linux发行版都可以访问 https://pkgs.org/ 获取最新的包

### debian
- [debian下载/手册](https://www.debian.org/)
- [ubuntu下载/手册](https://ubuntu.com/)
- [ubuntu网易镜像](http://mirrors.163.com/ubuntu-releases/)

#### dpkg(Debian Packager)
- 是基于debian的发行系统(ubuntu,knoppix)用于管理软件的工具
- `dpkg [option] [package ...]`
- 已安装的软件信息 选项如下
    + `-I` 显示一个deb包说明，
    + `-l` 显示所有已安装的deb包及简短说明
    + `-s` 报告指定包的状态信息
    + `-L` 显示一个包安装到系统里面的文件目录信息
    + `-S` 查看系统中某个文件属于哪个包
    + `-p` 显示包具体信息
- 软件包文件中的信息 选项如下
    + `-c` 显示一个deb文件的目录
    + `-i` 安装软件包
    + `-r` 删除软件包
    + `-P` 删除软件包和全部配置信息

#### apt(advanced packaging tools)
- 是debian及其衍生发行版(ubuntu)的软件包管理器，apt可用自动下载，配置，安装二进制/源代码格式的软件包，因此简化了unix系统上的软件管理过程。
- `apt[-action] [command] [package]`
- `apt list --[option]` 列出包
    + `installed` 已安装的包
    + `upgradeable` 可升级的包
    + `all-versions [package]` 包的所有版本
- `apt search` 搜索包
- `apt install` 安装包
- `apt upgrade` 重新获取软件包列表
- `apt update` 更新包
- `apt remove` 移除软件包
- `apt sorce` 下载源码档案
- `apt purge` 移除软件包和配置文件

```shell
lsb_release -a
# 更新软件包
apt update
apt upgrade
# 安装必要的工具
apt install mlocate git 
```

### centos
- [centos换阿里源](https://developer.aliyun.com/mirror/centos?spm=a2c6h.13651102.0.0.86c01b11ZXPaAb)
- 系统下载访问 [centos下载](https://www.centos.org/download/)
- rpm是centos最原始的软件包管理工具，现在主流是yum进行包管理，dnf是新版本centos默认包管理工具，在yum上进行了优化扩展启用了软件组的形式进行管理。三者都要掌握，推荐使用主流先

#### rpm(redhat package manage)
- 是文件格式也是包管理工具。由于其开源设计理念，现包括`openLinxu` `suse` `turbo linux`等发行版本都有在用。可以说是行内标准
- `rpm [option] [package ...]`
- 查询、认证 已安装的软件信息 选项如下
    + `-qa` 查看所有已安装软件
    + `-qi` 查看已安装软件包描述信息
    + `-ql` 查看已安装软件的文件信息/安装路径
    + `-qf` 查询某个文件是哪个rpm包产生的
    + `-qc` 查看已安装软件的配置文件
    + `-qd` 查看已安装软件的文档所在位置
- 查询、认证 软件包文件中的信息 选项如下
    + `-qpi` 列出rpm包描述信息
    + `-qpl` 列出rmp包文件信息
    + `-qpc` 查看包配置文件
    + `-qpd` 查看包文档所在位置
- 安装、升级、卸载 选项如下
    + `-ivh` 安装，显示安装进度
    + `-e` 卸载
    + `-Uvh` 升级
- 其他
    + `--rebuliddb` 维护RPM数据库信息

#### yum
- 配置文件分为`main`和`repo`
    + `main`全局配置， 位于`/etc/yum.conf`
    + `repo`各个源配置，位于`/etc/yum.repos.d/*.repo`，修改`*.repo`可启用/禁用源
- yum下载软件包的服务器地址被称为`源`。`epel`和`remi`是最常用的源，源的安装实际上也是安装一个rpm包。https://centos.pkgs.org 收录了centos上使用的最新rpm包，可通过此网站安装`epel`和`remi`源
```shell
# 先搜索epel 和 remi
yum search epel 
yum search remi
# 存在则安装
yum install epel-release.noarch
# 获取url地址进行安装
yum install https://rpms.remirepo.net/enterprise/8/remi/x86_64/remi-release-8.4-1.el8.remi.noarch.rpm
yum update -y #更新包
yum install -y yum-utils

# 安装必用软件
yum install -y wget openssh-server openssh-clients bison gcc make vim git
```

##### yum COMMAND
- `yum [option] [command] [package ...]` command如下
- `check` 检查 RPM 数据库问题
- `check-update` 检查是否有可用的软件包更新
- `list updates [package ...]` 列出可更新的包
- `list extras [package ...]` 列出已安装但不在资源库的包
- `deplist [package ...]` 列出软件包的依赖关系
- `search str` 在软件包详细信息中搜索指定字符串
- `provides command` 查找提供指定command内容的软件包
- `info [package ...]` 显示关于软件包或组的详细信息
- `install [package ...]` 向系统中安装一个或多个软件包,`package`可为url包地址
- `reinstall [package ...]` 覆盖安装软件包
- `update [package ...]` 更新系统中的一个或多个软件包，同时升级系统内核
- `upgrade [package ...]` 更新软件包同时考虑软件包取代关系
- `downgrade [package ...]` 降级软件包
- `makecache` 创建元数据缓存
- `clean all` 清除全部缓存
- `clean headers` 清除rpm头文件  
- `clean oldheaders` 清除旧的rpm头文件  
- `clean packages` 清除临时包文件（/var/cache/yum 下文件）  
- `erase` 从系统中移除一个或多个软件包
- `groups` 显示或使用、组信息
- `history` 显示或使用事务历史
- `repo-pkgs` 将一个源当作一个软件包组，这样我们就可以一次性安装/移除全部软件包。
- `shell` 运行交互式的 yum shell
- `repolist all/enabled` 显示全部/可用源
- `config-manager --enable/--disable [repo ...]` 开启、关闭某源
    + `--add-repo [repo ...]` 添加开启源,repo可为url地址


#### dnf
- RPM软件包管理器，已经在`Fedora 22`取代yum，未来将取代redhat家族的其他系统的包管理器
- 以上yum绝大多数命令，dnf都可以使用，列举一些没有的command
- `dnf config-manager --set-enabled epel-testing`启用epel-testing源(通常这个源是禁用的)

##### dnf module
- `list [module]` 列出可用modules，后面可用加包名称进行筛选。其中的 [d] 表示预设，[e] 表示已启用，[x] 表示已停用，[i] 表示已安装
- `info [module]` 查询模块信息
- `remove [module]` 卸载模块信息
- `provide [module]` 查询模块的提供软件库信息
- `update [module]` 更新
- `disable [module]` 禁用
- `install [module:stream]` 安装modules的指定版本，切换已安装的版本时加上参数(等你安装失败的时候会提示你安装啥)
- `enable [module:stream]` 启用指定模块流而不安装软件
- `reset [module]` 重置module版本(重置版本后就可以install/enable此module的其他版本)
