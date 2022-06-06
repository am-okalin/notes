[TOC]
## Linux网络配置
- `/etc/sysconfig/network-scripts/ifcfg-eth0` 网卡信息文件
    + `DEVICE=eth0` 网卡设备名(必须与文件名一致)
    + `BOOTPROTO=none` 获取IP方式(none,static,dhcp)
        * 局域网内搭建了DHCP服务器才可选择自动获取IP地址
        * 若选择dhcp，`USERCTL`以下内容都可自动获取
        * none与static都代表手动指定网卡信息
    + `HWADDR=00:0C:29:17:C4:09` MAC地址
    + `ONBOOT=yes` 是否随网络服务启动
    + `TYPE=Ethernet` 类型为以太网
    + `USERCTL=no` 不允许非root用户控制此网卡
    + `NM_CONTROLLED=no` 是否可由network managet图像管理工具托管
    + `UUID=xxxx-xxx-xxx-xxxx` 唯一识别码，可手动更改
    + `IPADDR=192.168.0.252` IP地址
    + `NETMASK=255.255.255.0` 子网掩码
    + `GATEWAY=192.168.0.1` 网关
    + `DNS1=8.8.8.8` DNS(首选)
    + `IPV6INIT=no` IPV6没有启用
- `/etc/sysconfig/network` 主机名文件
    + `NETWORKING=yes` 是否开启网络服务
    + `HOSTNAME=PCName` 指定主机名
    + 查看主机名命令：`hostname`
    + 命令修改主机名(临时生效) `hostname PCName`  
- `/etc/resolv.conf` DNS配置文件
    + `nameserver 202.106.0.20` 指定名称服务器(即DNS 可配置多个)
    + `search localhost` 没有写全域名时的默认顶级域名
- 复制网卡文件时修改UUID
    + 删除`/etc/sysconfig/network-scripts/ifcfg-eth0`中的mac地址行
    + 删除网卡与mac地址绑定文件`rm -rf /etc/udev/rules.d/70-persistent-net.rules`
    + 重启系统后uuid会被重写

## linux网络命令
### 网络环境查看命令 
- `ifconfig` 查看配置网卡命令
    + 看不到网关与dns，
    + 可被新命令`ip a`代替
- `ifdown eth0` 禁用网卡
- `ifup eth0` 启动网卡
- `netstat [option]` 查看网络状态
    + `-a` 列出全部服务
    + `-t` 列出tcp数据 
    + `-u` 列出udp数据
    + `-l` 列出状态为`listen`的服务
    + `-n` 显示IP地址和端口号，而不是显示域名和服务名
    + `-p` 列出该服务的PID
    + `-r` 列出网关列表，功能和`route`命令一致
- `netstat -an | grep ESTABLISHED | wc -l` 查询已建立的连接有多少个
- `route -n` 查看路由列表(可用看到网关)
- `route add/del default gw 192.168.0.1` 设定/删除临时网关
    + 在一台服务器里，连内网的网卡是不能设置网关的
- `nslookup [domain/ip]` 进行域名与IP解析
    + 不加参数回车进入标准输入格式：>server 可查看到本机的首选DNS

### 网络测试命令
- ICMP(Internet Control Message Protocol)：Internet控制报文协议
    + `ping`与`traceroute`都使用此协议进行网络探测的
- `ping [option] ip/domain` 探测指定ip/域名的网络情况
    + `-c` 次数
- `telnet [ip/domain] [port]` 远程管理与端口探测命令
    + 远程管理用明文传递，很不安全，所以现在只用来进行端口探测
- `traceroute [option] ip/domain` 路由跟踪命令
    + `-n` 使用IP，不使用域名，速度更快
- `tcpdump -i eth0 -nnX port 21` 抓包命令
    + `-i` 指定网卡接口
    + `-nn` 将数据包中的域名与服务转为IP和端口
    + `-X` 以十六进制和ASCII码显示数据包内容
    + `port` 指定监听端口
 
#### curl
- `curl url`发送一个请求
    + `-X 'METHOD'` 指定请求的`METHOD`包括GET, POST, PUT, DELETE等
    + `-H 'headinfo'` 指定请求头信息
    + `-d 'info'` 指定json信息 

#### nc/ncat
- `nc/ncat ip port` 连接远程系统（客户端）
    + `GET / HTTP/1.1` 连接后通过发送指令获取response
    + `HEAD  / HTTP/1.1` 连接后通过发送指令获取操作系统指纹标识（这会告诉我们使用的是什么软件来运行这个 web 服务器的）
- `nc/ncat [option]` option如下
     + `-l port` 监听port_number端口（服务端）
- 未来`socat`可能是更好的选择，可替代`nc/ncat`


## NAT穿透/内网穿透
### frp
`frp`是一个go开发的，专注于内网穿透的高性能的反向代理应用，支持 TCP、UDP、HTTP、HTTPS 等多种协议。可以将内网服务以安全、便捷的方式通过具有公网 IP 节点的中转暴露到公网。