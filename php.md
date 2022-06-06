[TOC]
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
- `php`
    + `-m` 查看扩展
    + `-v` 查看版本
    + `-i` 查看php相关信息
- php -S localhost:8001 -t path

## composer
[入门指南](https://docs.phpcomposer.com/00-intro.html)  
[中文文档](https://docs.phpcomposer.com/)  
[命令行](https://docs.phpcomposer.com/03-cli.html)  
地址：[win_composer_global_path](C:/Users/tony/AppData/Roaming/Composer)
全局配置中国全量镜像`composer config -g repo.packagist composer https://packagist.phpcomposer.com`
