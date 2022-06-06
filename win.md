[TOC]
## 帮助文档
- [win10安装介质](https://www.microsoft.com/zh-cn/software-download/windows10)
- [win命令](https://docs.microsoft.com/zh-cn/windows-server/administration/windows-commands/windows-commands)
- [WSL帮助文档](https://docs.microsoft.com/zh-cn/windows/wsl/)

## win10装机配置
- 下载官方介质生成启动盘 60min
- 重启两次，格式化后安装系统会新建4个分区，在主分区离线安装
- 卸载软件&安装软件
- 设置开机启动：`win+r`输入`shell:startup`，将想要启动的应用快捷方式放入

### 安装软件
- [clash_win](https://github.com/Fndroid/clash_for_windows_pkg)
- [nezuko.cc](https://nezuko.cc/auth/login)
- [Google浏览器](https://www.google.cn/chrome/)

- [迅雷](https://www.xunlei.com/)
- [GIT](https://git-scm.com/downloads)
- [sublime](./sublime)

- [微信](https://weixin.qq.com/)
- [TIM](https://office.qq.com/)
- [7zip](https://www.7-zip.org/)
- [xshell](https://www.netsarang.com/zh/free-for-home-school/)
- [Jetbrain](https://www.jetbrains.com/zh-cn/)
- [WSL](https://docs.microsoft.com/zh-cn/windows/wsl/)
- [Docker](https://www.docker.com/get-started/)


## Windows终端
### Terminal快捷键
- `shift`+`ctrl`+`p` 打开命令面板
- `shift`+`alt`+`-/+` 拆分新命令窗格
- `shift`+`ctrl`+`w` 关闭命令窗格
- `alt`+`方向`切换命令窗格

### 常用命令
- `tracert [path]` 跟踪path路径
- `netstat -anop tcp` 显示端口情况
    + `a` 显示所有连接和监听端口
    + `n` 数字形式显示地址和端口
    + `o` 显示进程ID
    + `r` 显示路由表
    + `p TCP/UDP` 显示指定协议
- `tasklist | findstr "15272"` 列出进程并搜索。
- `taskkill -PID 15272 -F` 强制杀死进程


## WSL
### 简介
- 适用于 Linux 的 Windows 子系统可让开发人员按原样运行 GNU/Linux 环境(包括大多数命令行工具、实用工具和应用程序,且不会产生传统虚拟机或双启动设置开销)
- 安装WLS见[WSL帮助文档](https://docs.microsoft.com/zh-cn/windows/wsl/)
- `适用于 Linux 的 Windows 子系统`/`WSL的分发版本`，用`Distro`表示

### wsl命令行
#### `wsl [CommandLine] [Options...]` 用于运行`Linux`命令的参数
- 例 `wsl -d docker-desktop -u root ls -l`
- `-d/--distribution <Distro>` 指定分发版本 
- `-u/--user <uname>` 指定用户身份 
- `ls -l` 用子系统的`shell`(命令行解释器)，在win目录下运行`ls -l`命令

#### `wsl [Argument] [Options...]` 用于管理`Distro`的参数
- `--export <Distro> <FileName>` 将`Distro`导出`tar`包。 `<FileName`为`-`则输出到标准输出中
- `--import <Distro> <InstallLocation> <FileName>` `tar`包导入为`Distro`。`<FileName`为`-`则从标准输入中导入
- `--list --verbose` 列出所有`Distro`详细信息
- `--set-default <Distro>` 将`Distro`设置为默认值。
- `--set-default-version <Version>` 更改安装`Distro`的默认安装版本。
- `--set-version <Distro> <Version>` 更改`Distro`的wsl版本。
- `--shutdown` 终止所有`Distro`
- `--terminate, -t <Distro>` 终止`Distro`
- `--unregister, -t <Distro>` 注销`Distro`

#### `bash [Options...]`
- `不指定选项`表示在当前目录执行启用bash
- `~` 进入默认子系统``
- `-c "<command>"` 运行命令，列显输出，并返回到 Windows 命令提示符

### WSL分发版本`Distro`
#### docker-desktop
- 再设置中开启WSL集成环境，可在多个`distro`中共用`docker`
- 安装docker-desktop后执行`wsl -l`可看到多了`docker-desktop`与`docker-desktop-data`两个`Distro`。 
- `$home\AppData\Local\Docker\wsl\`下包含`data`和`distro`两个目录下分别对应两个`Distro`的`vhdx`(虚拟磁盘)
- 进入`\\wsl$`通过网络访问`Distro`的`vhdx`内部文件系统。也可以通过进入`Distro`的`/mnt/wsl`目录找到对应文件系统，但这样访问不到部分数据