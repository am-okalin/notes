# refferrance
- [nvm安装](https://github.com/nvm-sh/nvm#installing-and-updating)
- [protoc安装](https://github.com/protocolbuffers/protobuf/releases)
- [graphviz](https://www.graphviz.org/download/)
- [cygwin](http://www.cygwin.com/install.html)

# 安装nvm npm yarn
- [nvm安装](https://github.com/nvm-sh/nvm#installing-and-updating)
```shell
# 查看npm版本
nvm ls-remote
nvm install 14.14.0
npm version
# 全局安装yarn
npm install -g yarn
yarn --version
# 查看registry字段是否为国内源
yarn config list
yarn config set registry https://registry.npm.taobao.org
```

# python
```shell
python -V
# 安装虚拟环境支持多版本python
pip install virtualenv
# 查看已安装的第三方包
pip list
# 环境变量中的 Python3/Scripts 在前则使用 Python3/Scripts 作为虚拟环境的解释器
# 创建虚拟环境
virtualenv [虚拟环境名字]
# 进入虚拟环境后包的安装卸载都是在这个虚拟环境中，不会影响到外部
# win进入虚拟环境：进入环境变量中的 Scripts 文件夹中，执行 activate
# *nix进入虚拟环境：source /path/to/virtualenv/bin/activate
# 退出虚拟环境： deactivate
```

# C
## gcc
- `-o filename.o` 输出文件为filename.o
- `-g filename.c` 将filename.c编译为可以被gdb调试的文本
- eg. `gcc max.o hello.c -o hello.out` 将hello.c编译为hello.out, max.o为hello.c使用的已经编译好的执行文件

## make
- `make`用于模块化管理c语言项目文件，执行`make`命令会自动解析当前目录下的`Makefile`文件执行编译
- `Makefile`文件语法如下：
```Makefile
hello.out:max.o hello.c
    gcc max.o hello.c -o hello.out
max.o:max.c
    gcc -c max.c
```

## gdb
- `gdb filename.o`用于调试C语言编译文件，前提是要使用`gcc -g`进行编译

### gdb命令
|         命令         | 缩写 |                      说明                     |
|----------------------|------|-----------------------------------------------|
| file file.o          |      | 加载需要调试的程序                            |
| list [n/-/+]         | l    | (第n行为中心)显示多行, [-/+]表示之前/后的代码 |
| break $line          | b    | 设置断点                                      |
| info [option]        | i    | 描述程序状态                                  |
| delete [breakpoints] | d    | 删除[指定]断点                                |
| run [...argc]        | r    | 开始运行程序                                  |
| start                | st   | 开始执行程序，在main函数的第一条语句前断点    |
| step                 | s    | 执行下条语句，是函数则进入                    |
| next                 | n    | 执行下条语句，是函数则跳过                    |
| continue             | c    | 继续程序的运行，知道遇到下个断点              |
| kill                 | k    | 终止正在调试的程序                            |
| set var key=val      |      | 运行中动态改变变量的值                        |
| print                | p    | 打印内部变量值                                |
| display              | disp | 跟踪查看某个变量的值，每次停下都显示其值      |
| watch                |      | 监视变量值的变化                              |
| backtrace            | bt   | 查看函数调用信息(堆栈)                        |
| frame                | f    | 查看栈帧                                      |
| quit                 | q    | 退出GDB                                       |

# php
## 安装
### 源码安装PHP
- 安装依赖`gcc autoconfig`等
- 解压`tar -jxvf php-7.4.4.tar.bz2`
- `configure`是用于指定配置的shell脚本
    + `./configure --help` 查看帮助
    + `--prifix=path`指定安装目录
- make
- make install
- `php -i | grep php.ini`查看`php.ini`加载路径，`cp php.ini-development /usr/local/lib/php.ini`拷贝源码包中的ini文件至加载路径。

### dnf/yum安装php
- `dnf install php php-cli php-common php-devel php-pear php-gd php-bcmath php-pdo php-mysqlnd`
- `php-devel`是php开发包，包含phpize
- `php-pear`PHP的官方开源类库，安装后可用`pecl`命令进行扩展管理

### php多版本安装
```shell
dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
# 查看版本
dnf module list php
# 切换版本前要重置版本
dnf module reset php


# 安装必要扩展
dnf install -y php-pecl-zip php-pecl-igbinary php-sodium php-pecl-redis php-pecl-swoole
```

## 扩展
### 配置
- 通过`php -i | grep extension_dir`找到扩展默认加载路径{$extension_dir}
- 在{$extension_dir}中包含的扩展，都可在`php.ini`中使用相对路径加载，如`extension=redis.so`

### 使用phpize添加扩展
- phpize编译的扩展库可以随时启用停用比较灵活
- 下载扩展源码包，进入包目录，执行phpize，
- `./configure --with-php-config=php_path/bin/php-config`
- make
- make install
- 编辑`php.ini`开启扩展

### 通过pecl方式安装扩展
- 安装`php-pear`后就可以使用`pecl`进行扩展管理，但要使用pecl要想安装phpize
- 可以在[pecl官网](https://pecl.php.net/packages.php)搜索可安装的扩展包(也可以用命令搜索)
- `php -i | grep extension_dir`找到php加载扩展的路径，`extension_dir`下的`*.so`文件就是php扩展文件，新加入的扩展要`chmod 755 extension.so`赋予权限
- `php.ini`中载入了`php.d/*.ini`,在此目录下创建文件加载扩展。执行`php -m`就能看到扩展已经被加载了
- 常用扩展：`zip swoole igbinary redis`

#### pecl COMMAND
- `search [package]` 搜索扩展包，会自动模糊匹配不用加正则
- `download  [package]` 下载扩展包
- `build  [package]` 从C的源码中构建扩展扩展包
- `run-tests  [package]`  运行测试(make test)
- `install [package]` 安装扩展包
- `uninstall [package]` 卸载扩展包
- `upgrade [package]` 升级扩展包

### phpcs&phpmd
- https://my.oschina.net/u/3349174/blog/857515
- vendor/bin/php-cs-fixer --config=.php_cs --verbose fix app
- vendor/bin/phpmd app text phpmd.xml
- phpmd ./application/index/controller/CountryProvider.php text ./phpmd.xml

## cli
- `php -m`查看扩展
- `php -v`查看版本
- `php -i`查看php相关信息
- php -S localhost:8001 -t path

## composer
- [入门指南](https://docs.phpcomposer.com/00-intro.html)  
- [中文文档](https://docs.phpcomposer.com/)  
- [命令行](https://docs.phpcomposer.com/03-cli.html)  
- 地址：[win_composer_global_path](C:/Users/tony/AppData/Roaming/Composer)
- 全局配置中国全量镜像`composer config -g repo.packagist composer https://packagist.phpcomposer.com`


