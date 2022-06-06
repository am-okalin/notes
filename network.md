[TOC]
## OSI七层模型 & TCP/IP五层模型
- OSI七层模型是理论模型，不是实际应用模型，发送时数据从上到下，接收时从下到上。上三层对用户提供服务；下四层对数据传输提供服务，进行实际的数据传递。
- TCP/IP四层模型是实际应用模型，与iso/osi七层协议又对应关系

| TCP/IP五层模型 | OSI七层模型 |        作用        | 
|----------------|-------------|--------------------|
| 应用层         | 应用层      | 给用户提供操作接口 |     
| 应用层         | 表示层      | 数据表示/加密/压缩 |     
| 应用层         | 会话层      | 管理应用会话       |     
| 传输层         | 传输层      | 确定协议:端口      |     
| 网络层         | 网络层      | 添加IP地址         |     
| 数据链路层     | 数据链路层  | 添加mac地址        |     
| 物理层         | 物理层      | 网络跳线           |     

## 网络层
### CIDR 无类域间路由
- IP地址由4x8=32字构成，通常按位划分以`点分十进制`表示。如`192.168.0.255`。
- 早期将IP地址划分为五类过于浪费，现在用的是`CIDR`方式进行划分，将IP地址一分为二，前面是网络号，后面是主机号。同时引入子网掩码概念兼容早期IP划分方法。
- `CIDR`斜线记法：`IP地址/网络ID的位数`。如`10.100.122.2/24`
- `CIDR`广播地址：向主机号全为1地址发送包，则该网络号下所有的主机都可以收到。如`10.100.122.255/24`
- `CIDR`子网掩码：将`子网掩码`和`IP`按位计算`and`，则可得到网络号。如`/24`的子网掩码是`255.255.255.0`与`10.100.122.2`按位计算`and`得`10.100.122.0`即网络号。
- 划分子网：将一个大网络划分多个小的网络，网络ID向主机ID借位，网络ID变多，主机ID变少  
- 划分超网：将多个小网合并一个大网，主机ID向网络ID借位

## 数据链路层
### MAC地址
- MAC地址由6x8byte=48bit构成。每块网卡的MAC地址是唯一的，从出厂的那一刻就确定了。
- 子网内通过arp协议获取ip与mac的映射，直接通过mac地址进行传输包。一旦跨子网mac地址就不能识别地址了，只能通过ip寻址。

## net-tools & iproute2
linux社区已对net-tools停止维护，部分发行版已不支持。iproute旨在取代net-tools，并提供了一些新功能。net-tools中工具的名字比较杂乱，而iproute2则相对整齐和直观，基本是ip命令加后面的子命令。

### `ip addr`内容解析
```shell
root@test:~# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether fa:16:3e:c7:79:75 brd ff:ff:ff:ff:ff:ff
    inet 10.100.122.2/24 brd 10.100.122.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fec7:7975/64 scope link 
       valid_lft forever preferred_lft forever
```
- `ip a`显示了所有网卡，inet表示ipv4，inet6表示ipv6。在ip后面有个`scope` `global`表示这网卡对外，`host`表示网卡仅供内部通信。
- `lo` 全称是 `loopback`，又称环回接口，往往会被分配到`127.0.0.1`这个地址。这个地址用于本机通信，经过内核处理后直接返回，不会在任何网络中出现。
- `link/ether fa:16:3e:c7:79:75 brd ff:ff:ff:ff:ff:ff` 是48bit的`MAC`地址，即网卡的物理地址。8x6`byte`=48`bit`
- `net_device flags`网络设备的状态标识。指的是`<BROADCAST,MULTICAST,UP,LOWER_UP>`
    + `UP`表示网卡储与启动状态
    + `BROADCAST`表示这个网卡由广播地址，可以发送广播包
    + `MULTICAST`表示网卡可以发送多播包
    + `LOWER_UP`表示LAN1是启动的，也即网线插着的
- `mut 1500`最大传输单元为1500byte。仅表示正文部分不允许超过1500byte(不包括头部14byte和尾部校验4byte)，超过则分片传输。
- `qdisc pfifo_fast`表示排队规则`queueing discipline`。内核如果需要通过某个网络接口发送数据包，它都需要按照为这个接口配置的 qdisc把数据包加入队列。
    + `pfifo` 不对进入的数据包做任何的处理，采用先入先出的方式通过队列
    + `pfifo_fast` 包括三个波段`band`，每个`band`里采用先进先出的规则。其中`band0`优先级最高，`band2`优先级最低。数据包按服务类型(Type of Service简称`TOS`)被分配到三个band里。`TOS`是`IP`头里面的一个字段表示包的优先级。

## 协议
- `ARP` 负责监视数据在主机和网路之间的交换
    + `地址解析协议(ARP)`工作在OSI的数据链路层，对应的就是此层
    + 在`win`系统中输入`arp -a`可以查看局域网内ip与mac对应关系








### 端口作用
- 端口共有65536个(2<sup>16</sup>)，10000以内系统使用，10000以外供用户使用
- TCP与UDP协议都有 2<sup>16</sup> 个端口。不过通常不分协议，如tpc:80端口被httpd服务占用，则udp:80端口也禁止使用
- 服务与端口是对应，且可以修改
- 协议对应常见端口号
    + `FTP(文件传输协议)`: 20(数据传递) 21(传输命令)
    + `SSH(安全shell协议)`: 22
    + `telnet(远程登录协议)`: 23(telnet是明文传输，win/linux都是禁用此端口的) 
    + `DNS(域名系统)`: 53
    + `http(超文本传输协议)`: 80 
    + `SMTP(简单邮件传输协议)`: 25(发信) 
    + `POP(邮局协议3代)`: 110(收信)
- `netstat -an` 查看本机启用端口

### DNS(Domain Name System)作用
- DNS也被称为域名解析，由于IP地址记忆困难，所以用DNS表示IP与domain之间做映射
- `hosts`文件：做静态IP与域名对应
    + 优先级高于DNS解析
    + linux: `/etc/hosts`
    + win: `C:\Windows\System32\drivers\etc\hosts`
- 将域名解析为IP地址
    + 客户机向DNS服务器发送域名查询请求
    + DNS服务器告知客户机WEB服务器的IP地址
    + 客户机与WEB服务器通信
- 域名空间结构(完全合格域名)
    + 根域： `.`表示根域名，在全球只有13台。负责管理一级域
    + 顶/一级域：由ISO(域名分配组织)决定
        * 组织域： `gov`政府 `com`商业 `edu`教育 `org`民间团体组织 `net`网络服务机构 `mil`军事部门
        * 国家/地区域：`cn`中国 `hk`香港 `jp`日本 `uk`英国等
    + 二级域： 向ISO申请并缴费 `imooc` `baidu`
    + 主机名： 申请二级域后自己规定的 `www` `NEWS`
- DNS查询过程(www.imooc.com.cn)
    + 客户机将域名发给本地DNS，本地DNS若没有缓存则请求根DNS解析cn
    + 本地DNS请求cn解析com.cn
    + 本地DNS请求com.cn解析imooc.com.cn
    + 将imooc.com.cn地址映射缓存并返回给客户机
- DNS查询类型，按查询方式分
    + 递归查询：客户机向dns发送请求，dns请求其他dns直到获取查询结果或返回查询失败
    + 迭代查询：服务器收到一次迭代查询回复一次结果，这个结果不一定是目标IP与域名的映射关系，也可以是其他DNS服务器的地址
- DNS查询类型，按查询内容分
    + 正向查询：根据域名查找IP
    + 反向查询：根据IP查找域名

### 网关(gateway)
- 网关(gateway)又称为网间连接器、协议转换器
- 网关在网络层以上实现网络互联，是复杂的网络互连设备，仅用于两个高层协议不同的网络互联
- 网关既可以用于广域网互联，也可以用于局域网互联
- 网关是一种充当转换任务的服务器或路由器
- 网关作用
    + 网关在所有内网计算机访问的不是本网段的数据报时使用
    + 网关负责将内网IP转换为公网IP，公网IP转换为内网IP



### 常用协议

| 分层   | 协议                                 |
| ------ | ------------------------------------ |
| 应用层 | DHCP HTTP HTTPS RTMP P2P DNS GTP RPC |
| 传输层 | UDP TCP                              |
| 网络层 | ICMP IP OSPF BGP IPSec GRE           |
| 链路层 | ARP VLAN STP                         |
| 物理层 | 网络跳线                             |

### 应用层协议

- DNS/HTTPDNS:  通过域名查找IP地址

- HTTP: 端口通常80，超文本传输协议，无状态的

- HTTPS: 端口通常443，HTTP+SSL 加密的HTTP协议
- DCHP: 连接网络时据DCHP协议配置动态IP地址

### 传输层协议

- UDP: 无连接(发送数据包后不管是否到达目的地)
- TCP: 面向连接(保证数据包能到达目的地，若不能到达则重新发送，直到到达)

### 网络层协议

#### 路由协议

- 内部网关协议-OSPF: 开放最短路径优先(Open Shortest Path First)， 最主要的特征就是使用分布式的链路状态协议。 主要特点为：
  - 向本自治系统中所有路由器发送信息。
  - 发送的信息就是与本路由器相邻的所有的路由器的链路状态，但这只是路由器所知道的部分信息。
  - 只有当链路层状态发生变化时，路由器才向所有的路由器用 洪泛法 发送此信息。 
- 外部网关协议-BGP: 边界网关协议，只能是力求寻找一条能够到达目的网络且比较好的路由，而非要寻找一条最佳路由
