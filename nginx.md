[TOC]
## 帮助文档
- [官方文档](https://www.nginx.cn/doc/)
- [nginx-掘金](https://juejin.cn/post/6942607113118023710#heading-23)

## 基础&概念
- `nginx`匹配规则优先级(大->小)如下，用于`server_name`和`locate`
    + `=`  精确匹配
    + `^~` 匹配到即停止搜索
    + `~`  正则匹配，区分大小写
    + `~*` 正则匹配，不区分大小写
    + ` `  普通模糊匹配，不带任何字符
- `正向代理`: 客户端(VPN或浏览器)，发送请求时代理
- `反向代理`: 服务端，接受请求时代理
- `动静分离`: 分离动态资源(高并发特性+反向代理)与静态资源(静态资源缓存特性)。
- `负载均衡`: 访问量，数据量，并发量的增大导致服务器遇到性能瓶颈，需要将请求转发给(服务器)集群分担压力。负载均衡策略有多种
    + `轮询策略`: 默认。将请求轮询分配。但某台服务器压力大会影响该服务器下所有用户
    + `最小链接数策略`: 将请求优先分配给压力较小的服务器，它可以平衡每个队列的长度
    + `最快响应策略`: 优先分配给响应最快的服务器
    + `客户端IP绑定策略`: 同一IP永远只分配一台服务器。解决了动态网页存在的`session`共享问题

## nginx cli
- `-c </path/to/config>` 为`Nginx`指定一个配置文件，代替缺省的。
- `-t` 运行测试 `nginx`将检查配置文件的语法,尝试打开配置文件中的引用文件。
- `-T` 查看nginx最终配置文件
- `-V` 显示 nginx 的版本，编译器版本和配置参数。
- `-s SIGN` 信号`SIGN`:强制停止`stop` 正常退出`quit` 重启`reopen` 重载配置`reload`

## 信号量控制nginx
| n  |  signal |                     signal description                     |
|----|---------|------------------------------------------------------------|
|  1 | SIGHUP  | 重载配置，用新的配置开始新的工作进程，从容关闭旧的工作进程 |
|  2 | SIGINT  | 快速关闭                                                   |
|  3 | SIGQUIT | 从容关闭                                                   |
|  9 | SIGKILL | 强制中止信号。不能被阻塞/处理/忽略                         |
| 10 | SIGUSR1 | 重新打开日志文件                                           |
| 12 | SIGUSR2 | 平滑升级可执行程序                                         |
| 15 | SIGTERM | 快速关闭                                                   |

## configure arguments
- `--prefix=/etc/nginx` nginx安装目录
- `--sbin-path=/usr/sbin/nginx` nginx执行脚本目录
- `--modules-path=/usr/lib/nginx/modules` nginx模块目录
- `--conf-path=/etc/nginx/nginx.conf` 配置文件
- `--error-log-path=/var/log/nginx/error.log` 错误日志
- `--http-log-path=/var/log/nginx/access.log` http日志
- `--pid-path=/var/run/nginx.pid` 存储`进程ID`即`pid`的文件
- `--lock-path=/var/run/nginx.lock` 锁定安装文件，防止被篡改及误操作
- `--http-client-body-temp-path=/var/cache/nginx/client_temp` 客户端请求时的目录
- `--http-proxy-temp-path=/var/cache/nginx/proxy_temp` http代理的临时目录
- `--http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp` 设定fastcgi临时目录
- `--http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp` 设定uwsgi临时目录
- `--http-scgi-temp-path=/var/cache/nginx/scgi_temp` 设定scgi临时目录

## `nginx.conf`配置结构
- `main` 全局配置
- `main>events` 配置工作模式与连接数
- `main>http` 配置代理、缓存、日志定义等大多数功能和第三方模块的配置
    + `upstream NAME {...}` 配置后端服务器具体地址，负载均衡必备
    + `server {...}` 虚拟主机配置，可有多个
    + `location [=|~|~*|^~] URI {...}` 路由规则

## 内置变量
- `remote_addr` 客户端 IP 地址
- `remote_port` 客户端端口
- `server_addr` 服务端 IP 地址
- `server_port` 服务端端口
- `server_protocol` 服务端协议
- `binary_remote_addr` 二进制格式的客户端 IP 地址
- `connection` TCP 连接的序号，递增
- `connection_request` TCP 连接当前的请求数量
- `uri` 请求的URL，不包含参数
- `request_uri` 请求的URL，包含参数
- `scheme` 协议名， http 或 https 
- `request_method` 请求方法
- `request_length` 全部请求的长度，包含请求行、请求头、请求体
- `args`or`query_string` 全部参数字符串arg_参数名 获取特定参数值
- `is_args` URL 中是否有参数，有的话返回 ? ，否则返回空
- `host` 请求信息中的`Host`，不存在则在请求头中找，最后使用`nginx`中设置的`server_name`
- `http_via` 每经过一层代理服务器，都会添加相应的信息
- `http_cookie` 获取用户 cookie 
- `request_time` 处理请求已消耗的时间
- `https` 是否开启了 https ，是则返回 on ，否则返回空
- `request_filename` 磁盘文件系统待访问文件的完整路径
- `document_root` 由 URI 和 root/alias 规则生成的文件夹路径
- `limit_rate` 返回响应时的速度上限值
- `remote_user` 远程客户端用户名，一般为`-`
- `time_local` 时间和时区
- `request` 请求url及method
- `status` 响应状态码
- `body_bytes_send` 响应客户端内容字节数
- `http_referer` 记录用户从哪个链接跳转过来的
- `http_user_agent` 用户所使用的代理，一般来时都是浏览器
- `http_x_forwarded_for` 通过代理服务器来记录客户端的ip

## 核心模块
### main主模块
- `pid FILE` 存储`进程ID`即`pid`的文件
- `user USER` 设置`worker`进程的用户，默认为`nobody`
- `daemon on/off` `on`用于调试，`off`用于生产
- `error_log FILE debug|info|notice|warn|error|crit|alert|emerg` 日志级别(小->大)
- `worker_processes N` worker进程数,通常为cpu个数`auto`
- `working_directory PATH`  `worker`子进程异常终止后的`core`文件存放目录
- `worker_rlimit_core SIZE`  `worker`子进程异常终止后的`core`文件大小如`50M`
- `worker_rlimit_nofile_number N` `worker`子进程可打开的最大文件句柄数如`20480`

### events事件模块
- [事件模型优化](https://www.nginx.cn/doc/general/optimizations.html)
- `use [kqueue|rtsig|epoll|/dev/poll|select|poll|eventport]` 事件模型，不同操作系统有不同的事件模型，使用默认即可
- `worker_connections N` 单个`worker`的最大并发链接数如`1024`
- `accept_mutex on/off` 是否打开负载均衡互斥锁，默认为off
- `multi_accept on/off` 设置一个进程是否同时接受多个网络连接，默认为off


### 公共配置
- 配置代理,缓存,日志定义等绝大多数功能和第三方模块的配置
- 注：子模块设置共有字段会覆盖
- `include FILE` 引入外部配置，提高可读性，避免单个配置文件过大
- `keepalive_timeout SEC` 保证客户端多次请求的时候不会重复建立新的连接，节约资源损耗。
- `default_type MIME-type` `MIME-type`有`text/plain`(default) `application/x-ns-proxy-autoconfig` `application/octet-stream`等
- `root PATH` 指定静态资源目录位置,如访问`test.com/img/1.png`实际访问的是`PATH/image/1.png`
- `sendfile on/off` 使用高效文件传输，提升传输性能。
- `sendfile_max_chunk SIZE`每个进程每次调用传输数量不能大于`SIZE`如`100k`，默认为0，即不设上限。
- `tcp_nopush on/off` 当数据表累积一定大小后才发送，提高了效率。要求`sendfile on`
- `tcp_nodelay on/off` 为`on`时立即发送，针对与小块数据，与`tcp_nopush`互斥

#### server配置字段
- `server_name [=|~|~*|^~] DOMAIN {...}` 先配置`DNS`解析,
    + 本地的话编辑`/etc/hosts`
    + 第三方买的就在第三方那里配置

#### location配置字段
- `location [=|~|~*|^~] uri {...}` 访问`DOMAIN/uri`时根据location进行匹配
    + `/uri/`:`uri目录`存在则找`uri/index.html`
    + `/uri`:`uri目录`存在则找`uri/index.html`；没有`uri目录`会找`uri文件`
- `alias PATH` 指定静态资源目录位置,使用`alias`末尾一定要添加`/`
- `index FILE` 默认首页文件
- `deny IP` 拒绝访问的IP
- `allow IP` 允许访问的IP
- `return` 停止处理请求，直接返回响应码或重定向到其他URL
    + `return 404` 直接返回状态码
    + `return 404 "pages not found"` 返回状态码 + 一段文本
    + `return 302 /bbs` 返回状态码 + 重定向地址
    + `return https://www.baidu.com` 返回重定向地址

#### log
- `access_log LOGNAME/off` 指定`LOGNAME`使用的日志格式，设为off则取消服务日志
- `log_format LOGNAME FORMAT` `LOGNAME`是自定义的日志名称，`FORMAT`通常使用内置变量拼接。如`log_format myFormat '$remote_addr–$remote_user [$time_local] $request $status $body_bytes_sent $http_referer $http_user_agent $http_x_forwarded_for'`

#### gzip
- `gzip on/off` 传输数据压缩，开启能够节省宽带，但响应的会增加CPU负载
- `gzip_min_length N` 限制最小压缩，小于`N`字节的不被压缩
- `gzip_comp_level N` 定义压缩比，该值越大压缩越多，但是cpu使用会越多
- `gzip_types TYPE` 定义需要被压缩文件的类型:`text/plain` `application/x-javascript` `text/css` `application/xml`

#### autoindex
- 用于快速搭建静态资源下载网站，通常是个单独的`autoindex.conf`文件
- `autoindex on/off` 激活/关闭自动索引
- `autoindex_localtime on/off` 以本地时间来显示文件(GMT)时间。默认为`off`
- `autoindex_exact_size on/off` 设定索引时文件大小的单位(B,KB, MB 或 GB)
- `autoindex_format type` 格式化类型: `html` `json` `xml`

#### rewrite
- `rewrite 正则表达式 要替换的内容 [flag]`
- `flag` 如下
- `redirect` 返回302临时重定向；
- `permanent` 返回301永久重定向；
- `break` 直接使用重写后的`URL`, 不再匹配其它`location`中语句；
- `last` 重写后的`URL`发起新请求，再次进入`server`段，重试`location`的中的匹配

#### if 控制指令
- if(condition){...} 判断条件如下
- `$variable` 仅为变量时，值为空或以0开头字符串都会被当做 false 处理；
- `= ` 或 `! =` 相等或不等；
- `~` 或 `! ~` (非)正则匹配；
- `~*` 或 `! ~*` (非)正则匹配，不区分大小写；
- `-f` 或 `! -f` 检测文件存在或不存在；
- `-d` 或 `! -d` 检测目录存在或不存在；
- `-e` 或 `! -e` 检测文件、目录、符号链接等存在或不存在；
- `-x` 或 `! -x` 检测文件可以执行或不可执行；

### 负载均衡
#### `upstream NAME {...}`
- `上游服务器`:就是后台提供的应用服务器
- `upstream`: 定义`上游服务器`的相关信息,负载均衡配置不可或缺的部分。指令如下
- `zone` 定义共享内存，用于跨 worker 子进程；
- `keepalive` 限制每个`worker`与上游服务器空闲长连接的最大数量
- `keepalive_requests` 一个长连接最多请求 HTTP 的个数
- `keepalive_timeout` 空闲长连接的最长保持时间
- `hash` 哈希负载均衡算法；
- `ip_hash` 依据 IP 进行哈希计算的负载均衡算法；
- `least_conn` 最少连接数负载均衡算法；
- `least_time` 最短响应时间负载均衡算法；
- `random` 随机负载均衡算法；
- `server IP [parameters]` 定义上游服务器地址. `parameters`选项如下
    + `weight=number` 权重值，默认为1；
    + `max_conns=number` 上游服务器的最大并发连接数；
    + `fail_timeout=time` 服务器不可用的判定时间；
    + `max_fails=numer` 服务器不可用的检查次数；
    + `backup` 备份服务器，仅当其他服务器都不可用时才会启用；
    + `down` 标记服务器长期不可用，离线维护；

#### `proxy_pass` 配置代理服务器
- 上下文：`location` `if` `limit_except`
- 语法：`proxy_pass URL` `URL`规则如下
    + 可携带变量
    + 必须`http`or`https`开头
    + `URL`不带`URI`:传递给代理服务的`URL`不变
    + `URL`带`URI`:传递给代理服务的是去除`location_uri`后的`URL`

```shell
location /bbs/{
    # /bbs/abc/test.html 
    # proxy_pass http://127.0.0.1:8080;
    # /abc/test.html 
    proxy_pass http://127.0.0.1:8080/;
}
```
