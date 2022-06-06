## 远程登录工具
### SSH协议原理
#### 对称加密算法
- 采用单钥密码系统的加密算法，对同一个密钥可用同时用作信息的加密和解密，这种加密方法称为对称加密，也称为单密钥加密
- 例如：压缩包的密码，加密解密使用同一个密码

#### 非对称加密算法
- 又名`公开密钥加密算法`，非对称加密算法需要两个密钥：公钥和私钥
- 例：A要传给B一个只能AB两人能查看的文件，需要将AB的公钥加入文件，只有对应私钥能解密文件。公钥类似锁用于加密，私钥类似钥匙用于解密

#### SSH安全外壳协议
- 用于保护密码传递时安全。使用接收者公钥对文件加密，接收者接收后，使用自己的私钥进行解密
- 用于远程连接时，将远程机器的公钥加入本机中。

#### ./ssh/known_hosts文件
A通过ssh首次连接到B，B会将公钥1（host key）传递给A，A将公钥1存入known_hosts文件中，以后A再连接B时，B依然会传递给A一个公钥2，OpenSSH会核对公钥，通过对比公钥1与公钥2 是否相同来进行简单的验证，如果公钥不同，OpenSSH会发出警告， 避免你受到DNS Hijack之类的攻击。
- 例 A通过ssh登陆B时提示 Host key verification failed.
- 原因 A的known_hosts文件中记录的B的公钥1 与 连接时B传过来的公钥2不匹配
- 解决办法
    1. 删除A的known_hosts文件中记录的B的公钥（手动进行，不适用于自动化部署情形）
    2. 修改配置文件，在ssh登陆时不通过known_hosts文件进行验证（安全性有所降低），修改完需重启机器
```shell
# 编辑配置文件
vi ~/.ssh/config
# 添加以下两行代码
StrictHostKeyChecking no 
UserKnownHostsFile /dev/null
```

#### SSH命令
- `ssh username@ip` 远程管理指定Linux服务器
    + 连接后要下载服务器公钥，然后进行连接
- `scp [-r] username@ip:server_path local_path` 下载文件
    + `-r` 表示递归，下载目录时加此参数
- `scp [-r] local_path username@ip:server_path` 上传文件
    + `-r` 表示递归，上传目录时加此参数

#### ssh-keygen生成公私钥
- `ssh-keygen`用于生成、管理和转换认证密钥，常用参数如下
    + `-t` 指定密钥类型，包括rsa和dsa，默认rsa，可省略
    + `-b` 指定密钥长度 b=bits,最长4096
    + `-C` 设置注释文字，通常写邮箱
    + `-f` 指定密钥文件存储文件名，若省略生成`id_rsa`与`id_dsa.pub`默认公私钥文件
- 设置个人用户在服务器用公私钥登录
    + 创建`~/.ssh/authorized_keys`(注意权限所属用户，且600权限)


```shell
# 提示输入2次密码，此密码是push时要输入的密码，直接回车push时就可以不用密码了
ssh-keygen -t rsa -C "your_email@xxxx.com"
ssh-keygen -t rsa -C "tony@centos8" -f "tony_rsa"

# 进入服务器 把公钥复制到个人目录~/.ssh 
cat key_rsa.pub >> authorized_keys
systemctl restart sshd
```

### ssh-keyscan 收集指定SSH服务器的公钥
`ssh-keyscan  [-46cHv] [-f file] [-p port] [-T timeout] [-t type] [host | addrlist namelist]`

- `46cHv`
    + `-4` 只使用IPv4
    + `-6` 只使用IPv6
    + `-c` 请求证书，而非文本公钥
    + `-H` 将返回结果中的明文都Hash，避免被偷窥
    + `-v` Debug模式
    + `-p` 指定连接远程主机的端口
- `-f` 包含多个hosts列表的文件
- `-t` 指定收集的密钥类型，可指定逗号分隔的多个。支持`dsa`, `rsa`, `ecdsa`, `ed25519`

### WinSCP
- 在win系统下与服务器交换文件。使用`sftp`协议进行传输。直接下载即可

### 使用
```shell
ssh-keygen -t rsa -C "weitangning@ifreegroup.com" -f "wtn_rsa"
ssh-agent bash
ssh-add ./wtn_rsa
# 将公钥复制到服务器或github上
ssh -T git@github.com
```
