[TOC]
## 帮助链接
- [阿里云控制台](https://cr.console.aliyun.com/)   
- [dockerHub](https://hub.docker.com/)  
- [docker下载](https://download.docker.com)  
- [docker文档](https://docs.docker.com/install/) 
- [docker安装包列表](https://download.docker.com/linux/static/stable/x86_64/)  
- [centeos依赖包列表](https://download.docker.com/linux/centos/7/x86_64/stable/Packages/)  
- [docker的存储库官方源](https://download.docker.com/linux/centos/docker-ce.repo)  
- [docker的存储库阿里源](http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo) 
- [Docker 架构介绍与实战](https://github.com/endlif/GitCode/tree/master/docker)
- [docker目录挂载](https://blog.csdn.net/weixin_34248118/article/details/85757376)
- [dockerGhost项目地址](https://github.com/Albert-W/dockerGhost)
- [centos下防火墙引起的网络问题解决](https://blog.csdn.net/qq_14869093/article/details/102566291?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase)

## 准备知识
### docker简介
- 打包应用及依赖包到一个轻量级、可移植的容器中，然后发布到仍和流行的Linux机器上，也可实现虚拟化。容器性能开销极低
- Docker 从 17.03 版本之后分为 CE（Community Edition: 社区版） 和 EE（Enterprise Edition: 企业版），我们用社区版
- Docker 非常适合于高密度环境以及中小型部署，可以用更少的资源做更多的事情
- Docker 属于 Linux 容器的一种封装，提供简单易用的容器使用接口。
- 对于容器里面的进程来说，它接触到的各种资源都是虚拟的，从而实现与底层系统的隔离。## 
- Docker 将应用程序与该程序的依赖，打包在一个文件里面。运行这个文件，就会生成一个虚拟容器。程序在这个虚拟容器里运行，就好像在真实的物理机上运行一样。有了 Docker ，就不用担心环境问题。
- 通过docker的接口，用户可以创建使用容器，将自己的应用放入容器，容器还可以进行版本管理，复制，分析修改，就像管理普通代码一样

### 容器与虚拟机的区别
- 容器与虚拟机都是实现操作系统虚拟化的一种途径
- 虚拟机 
    + 模拟整台机器包括硬件及操作系统
    + 会占用全部预分配资源
- 容器
    + 与宿主机共享硬件资源及操作系统，实现资源动态分配
    + 容器包含应用和其所有的依赖包，但是与其他容器共享内核
    + 容器在宿主机操作系统中，在用户空间以分离的进程运行

### 镜像&容器&仓库
- 镜像(image)：运行容器的前提
    + 可看作是特殊文件系统，包括程序，库，资源，配置。镜像不包含任何动态数据，其内容在构建之后也不会被改变。
    + 镜像是一堆只读层(read-only layer)的统一视角。是统一只读文件系统(unioned Read-Only File System)
- 容器(container) = 镜像(image) + 读写层
- 仓库(Registry)：存放镜像的场所，可看成一个控制中心，用来保存镜像。
- 镜像(Image)和容器(container)的关系，就像是面向对象程序设计中的类和实例一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。


## 安装
安装可访问[官方安装文档](https://docs.docker.com/engine/install/)，找到适合自己系统的安装方式，下面以centos8安装为例

### yum安装`Docker Engine - Community`
```shell
# 安装docker-ce
yum config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo # 添加docker源(存储库)
yum install docker-ce docker-ce-cli containerd.io # 安装docker
docker verison # 查看docker-ce是否安装成功
usermod -aG docker tony # 可选：将个人用户加入docker用户组

# todo::整理iptable知识点
# 防火墙规则错误导致docker网路包无法访问外部网络。暂用关闭防火墙重启的方式避过这个问题
systemctl status firewalld # 查看firewalld日志可看到iptable错误
systemctl enable docker # 配置docker自启动
systemctl stop/disable firewalld # 关闭防火墙并关闭自启
reboot # 重启
docker run busybox nslookup www.baidu.com # 网络测试

# 安装docker-compose
curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
docker-compose --version
```

## registry 注册仓库
- 仓库全名：`namespace/name:[version]`
- 仓库坐标：`domain/namespace/name:[version]`
- 公共仓库的镜像无需登录 Registry 即可下载的。
- 登录的Registry和操作镜像的Registry必须一致。若登录阿里云Registry，但push不指定阿里云的domain就会报错(不指定domian默认为dockerhub)
- 官方仓库dockerhub 源在国外要使用`镜像加速器`才可pull/push。pull/push时关闭代理
```shell
# -p 创建多级目录
mkdir -p /etc/docker
# `/etc/docker/daemon.json`为默认images源加载位置，推荐使用json文件方式
# 也可以选择用阿里云的个人加速器: https://2ps7i7rv.mirror.aliyuncs.com
tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]
}
EOF
systemctl daemon-reload && systemctl restart docker
```
- 将images提交到远程仓库
```shell
# 若不在后面添加domain则默认为登录官方存储库
# 阿里云控制台中设置固定密码，创建命名空间
docker login registry.cn-hangzhou.aliyuncs.com
# 确认登录信息
cat /root/.docker/config.json
# 从阿里云Registry中拉取镜像
docker pull registry.cn-hangzhou.aliyuncs.com/namespace/name:[version]
# 以image_id为原型创建镜像，若要提交到dockerhub则可省略domain
docker tag image_id registry.cn-hangzhou.aliyuncs.com/namespace/name:[version]
# 将新创建的镜像推送
docker push registry.cn-hangzhou.aliyuncs.com/namespace/name:[version]
```

## Volume 挂载目录
- 提供独立与容器之外的持久化存储(如数据库)
- 提供host与container，container与container之间共享数据
- 挂载目录后可实现host与container数据同步修改，成为很好的开发环境
- docker启动后退出问题
    + 容器运行的命令在执行完后会自动退出，挂起命令会一直执行(top)
    + 如果容器中没有前台进程执行，容器认为空闲就会自行退出
    + nginx这样后台运行的程序要加 -d(后台运行)
    + create/run时添加-it(交互运行) 
- Volume挂载示例：
```shell
# -v hots_path:container_path，挂载卷，若省略host_path会自动生成共享目录
docker run -d -v /usr/share/nginx/html nginx
# 检查容器信息，查看挂载源目录(mounts source)也就是host_path
# 进入挂载源目录，linux可直接进入。cd host_path
docker inspect nginx
# 注：mac要用screen命令登录到docker虚拟机中再进入host_path
echo "info" > index.html
# 进入容器查看/usr/share/nginx/html/index.html改动是否生效
docker exec -it nginx /bin/bash
cat /usr/share/nginx/html/index.html

# create创建但不启动用法同run，-it挂起容器避免启动后立即退出，-v挂在卷
docker create -it -v $PWD/data:/var/mydata --name data_container ubuntu
# --volumes-from old_container new_container
# 挂载new_container目录等同于old_container。执行后两个container与host数据同步
docker run -it --volumes-from data_container ubuntu /bin/bash
# 进入container后执行mount命令,可以看到有/var/mydata目录的挂载信息
```

## docker命令
|               docker命令              |              释义             |
|---------------------------------------|-------------------------------|
| ps -a/container ls -a                 | 查看所有运行过的容器          |
| start/stop/exec/rm container          | 开启/停止/执行/移除指定容器   |
| create/run [--name container] image   | 创建不启动/启动容器           |
| cp file container:/path               | 将文件拷贝到容器中指定目录下  |
| exec -it container /bin/bash          | 进入容器文件系统交互          |
| exec -it container ls path            | 展示容器中path下所有文件      |
| commit -m "info" container image_name | 提交改动过的容器为镜像        |
| inspect container_name                | 检查容器信息                  |
| images                                | 列出所有镜像                  |
| rmi image                             | 移除指定容器镜像              |
| docker build -t image_name path       | 使用path/Dockerfile自定义镜像 |
| tag old_image new_image               | old_image为原型创建new_image  |
| search/pull/push image:version        | 搜索/获取/推送镜像            |
| login/logout [option] domain          | 登录/登出registry，可缺domain |

- `build context` 构建镜像
    + `context` 指定操作空间，不加`-f`就使用`context`中的Dockerfile进行构建
    + `-t` 指定镜像tag
    + `--no-cache` 构建时不依赖从上次构建缓存
    + `-f path/Dockerfile` 使用指定目录下dockerfile构建镜像
- `docker run --name nginx -p 127.0.0.1:8080:80/tpc -d nginx` 运行nginx容器
- `docker run -id --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql`运行mysql容器

### 公共option
- `--interactive -i` Keep STDIN open even if not attached
- `--tty -t` Allocate a pseudo-TTY
- `--detach -d` Run container in background and print container ID
- `--privileged` Give extended privileges to this container(给容器中的root扩展权限)


## Dockerfile
### 概念
- 通过编写Dockerfile文件可自定义docker镜像
- 镜像存储方式：镜像是被分层存储的，可减少存储空间
    + Dockerfile中每一行产生一个新层，每一层都有一个image_id
    + image层都是只读的(R)，iamge创建容器后，container层读写的(RW)
- CMD 相当于启动docker时候后面添加的参数如`docker run alpine1 echo "hello world"`
    + 每个dockerfile中只能有一个 CMD 如果有多个那么只执行最后一个
    + 在Dockerfile中添加CMD命令就不用每次在启动时输入参数
    + 如果`alpine1`的Dockerfile中有CMD命令，则不生效
- ENTRYPOINT 相当于容器启动时执行。
    + 每个dockerfile中只能有一个 ENTRYPOINT 如果有多个那么只执行最后一个
    + 若Dockerfile有ENTRYPOINT，则使用CMD相当于给ENTRYPOINT添加参数。
    + ENTRYPOINT 与 CMD 结合使用，如例1容器启动参数为`tail -f /usr/local/test.txt`

### .dockerignore文件
- 在Docker CLI将上下文发送到Docker-daemon之前，它将在操作目录中查找`.dockerignore`，将其中的文件排除于操作目录中，类似`.gitignore`
- 这有助于避免不必要地将较大或敏感的文件和目录发送到守护程序，并避免使用ADD或COPY将它们添加到映像中。

### Dockerfile 语法
| Dockerfile命令 |                           用途                          |
|----------------|---------------------------------------------------------|
| FROM image     | 以image为base image                                     |
| RUN            | 执行命令，按顺序在file中执行                            |
| CMD            | 执行命令，仅执行最后一个CMD命令，可与ENTRYPOINT配合使用 |
| ADD            | 添加文件，可添加远程文件                                |
| COPY           | 拷贝文件                                                |
| EXPOSE         | 暴露端口                                                |
| WORKDIR        | 指定运行命令的路径                                      |
| MAINTAINER     | 标明维护者(已弃用，被label代替)                                              |
| ARG            | 设定环境变量(及默认值，可被构建命令或者docker-composer.yml中的环境变量赋值) 仅`build image`的过程中有效                |
| ENV            | 设定环境变量 持续生效且允许被覆盖                       |
| USER           | 指定执行命令的用户，通常不会用root在容器中执行          |
| ENTRYPOINT     | 入点，相当于启动执行                                    |
| VOLUME         | mount point 指定容器所挂在的卷                          |

### Dockerfile创建镜像
- 对于过于简单的构建镜像可通过管道符`|`或标准输入符`<<`创建镜像。不浪费资源
- 连字符`-`占据PATH的位置，并指示Docker从stdin而不是目录中读取构建上下文
```shell
# 管道符
echo -e 'FROM busybox\nRUN echo "hello world"' | docker build -
# stdin 标准输入
docker build -<<EOF
FROM busybox
RUN echo "hello world"
EOF
```
- 构建时需要用到文件或复杂构建采用书写Dockerfile构建
```shell
mkdir d1 && cd d1
vim Dockerfile
# `-t`设置image标签名，`.`表示使用当前目操为context
docker build -t hello_docker .
docker images hello_docker
docker run hello_docker
```

### 例1
```shell
# alpline是针对docker做的一个极小的Linux环境，latest表示最新版本
FROM alpine:latest
# 标注开发者
MAINTAINER yourname
# 创建test内容为content
ADD content /usr/local/test
# 拷贝操作空间(宿主机指定目录)下的hello文件
COPY /hello /usr/local/hello
# 运行容器后执行 tail -f /usr/local/test.txt
CMD ["-f","/usr/local/test.txt"]
ENTRYPOINT ["tail"]
```
### 例2
```shell 
FROM ubuntu
MAINTAINER yourname
# 更换国内下載源，将文件中所有`archive.ubuntu.com`替换成`mirrors.ustc.edu.cn`
RUN sed -i 's/archive.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list 
RUN apt-get update
RUN apt-get install -y nginx
# 将执行build目录下的index.html拷贝到容器中/var/www/html目录下
COPY index.html /var/www/html
# 入点的参数是数组，`-g`设置配置文件外的全局指令，`deamon off;`表示前台运行
# 把nginx停留在前台，以便docker可用正确跟踪进程(否则容器会在启动后立刻停止)
ENTRYPOINT [ "/usr/sbin/nginx","-g","daemon off;"]
# 暴露nginx服务端口
EXPOSE 80
```

## 多容器app
- 通过`YML`文件声明式的定义应用程序的各个服务，并由`docker-compose`命令完成应用的创建和启动。
- `docker-compose.yml`文件用于定义应用程序的各个服务。它通过缩进表达层级关系

### 样例
```yaml
version: "3.7"
services:
  web:
    image: busybox
    container_name: imapms
    command: top
    networks:
      - imapms_net

networks:
  imapms_net:
    ipam:
      driver: default
      config:
        - subnet: 192.19.20.0/24
          gateway: 192.19.20.1

```

### `yml`语法
|    命令    |            用途            |
|------------|----------------------------|
| build      | 本地创建镜像               |
| command    | 指定命令覆盖镜像的缺省命令 |
| depends_on | 表达依赖关系               |
| ports      | 暴露端口                   |
| volumes    | 挂载卷                     |
| image      | 从registry中pull镜像       |
| up         | 启动服务                   |
| stop       | 停止服务                   |
| rm         | 删除服务中的各个容器       |
| logs       | 观察各个容器的日志         |
| ps         | 列出服务相关容器           |

## 用docker构建测试容器
```shell
docker run  -itd -p 50022:22 --privileged --name myCentos centos /usr/sbin/init
docker exec -it myCentos /bin/bash
#todo容器中执行命令...
docker commit -m "exec sshd success" myCentos my_centos
docker stop myCentos
docker rm myCentos
docker run -itd -p 50022:22 --privileged --name myCentos1 my_centos /usr/sbin/init
docker exec -it myCentos1 /bin/bash
```

```shell
dnf -y --disablerepo '*' --enablerepo=extras swap centos-linux-repos centos-stream-repos
dnf -y distro-sync
dnf -y install net-tools openssh-server openssh-clients passwd  gcc make vim git wget
systemctl status sshd
systemctl start sshd
passwd root
```