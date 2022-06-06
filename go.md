[TOC]

## 入门
### linux安装
- 安装golang请访问[go_download](https://go.dev/dl/)
```shell
tar -C /usr/local -xzf go1.16.linux-amd64.tar.gz

# 配置环境变量，更改全局goenv
sudo vim /etc/profile.d/go.sh
export GOROOT=/usr/local/go
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin

# 设置代理
go env -w GOPROXY=https://proxy.golang.com.cn,direct
# 不走 proxy 的私有仓库或组，多个逗号相隔(可选)
go env -w GOPRIVATE=*.corp.example.com,github.com/org_private
```

### 源码文件
go程序生命周期中的三种文件，分为`可执行文件`、`归档文件`、`源码文件`(`命令源码文件`、`库源码文件`、`测试源码文件`)

- `归档文件` 实际上就是`静态链接库文件`，以`.a`结尾
- `命令源码文件` 作为可执行的程序的入口，拥有`main`函数
- `库源码文件` 用于集中放置各种待被使用的程序实体(全局常量、全局变量、接口、结构体、函数等等)
- `测试源码文件` 用于对前两种源码文件中的程序实体的功能和性能进行测试`xx_test.go`

### 代码包
- GO源码以`代码包`为基本组织单位。`代码包`与系统目录一一对应。由于目录可以有子目录，所以`代码包`也可以有`子包`。
- 若包A依赖包B 则 B 是 A 的`依赖包`，A 是 B 的`触发包`
- `internal`内部引用目录:  `internal`目录下的包仅父级包可访问 ，祖先不可访问
- `$GOROOT/src/` 存放`官方包`的路径
- `$GOPATH/src/包名` 存放`第三方包`的路径

### GPM 并发编程模型
- `machine` 系统级线程
- `goroutine` 用户级线程
- `processor` 调度器，用于调度G与M对接。通常P的数量等于cpu核心数
    + 在等待I/O或者锁解除时P会分离对应的G与M
    + 在G需要恢复时，P会寻找M将两者对接
    + M不够用时P会向操作系统申请新的M
- 执行go语句时会先从`某个存放空闲的G队列`中获取一个G，找不到G时才会创建G；然后用G去包装go语言的函数把它追加到`某个存放可运行的G队列`中。因此
    + 已存在的G总会被优先复用
    + go函数的执行时间总是会明显滞后于它所属的go语句的执行时间

### todo 垃圾回收机制


## 命令行
- `go help <command>`查看`<command>`基础命令的详细帮助文档
- `go help <topic>`查看`<topic>`扩展命令的详细帮助文档
- `govendor`目前已弃用，建议`go mod`。

### environment
- `CGO_ENABLED` 指明cgo工具是否可用的标识
- `GOHOSTARCH` 程序运行环境的编译体系结构(用于交叉编译，默认与 GOARCH 相同)
- `GOHOSTOS` 程序运行环境的操作系统(用于交叉编译，默认与 GOOS 相同)
- `GOARCH` 程序构建环境的目标计算架构
- `GOOS` 程序构建环境的目标操作系统
- `GOMAXPROCS` 用于设置应用程序可使用的处理器个数与核数，即`processor`的数量
- `GOBIN` 表示编译器和链接器的安装位置，默认是 $GOROOT/bin
- `GOEXE` 可执行文件的后缀。
- `GOENV` go环境配置文件的位置。
- `GOROOT` go编译器源码目录
- `GOPATH` go工作空间 默认为`$HOME/go` 该路径下有三个子目录
    + `src` 存放源码，如`.go` `.c` `.h` `.s`
    + `pkg` 编译时生成的中间文件(包对象)如`.a`,`install`会将编译好的包直接从这里引用,不必再次构建
    + `bin` 编译后生成的可执行文件，`PATH=$PATH:/usr/local/go/bin`
- `GOTOOLDIR` Go工具目录的绝对路径。
- `GOPROXY` 例`https://goproxy.cn,direct,EOF`，表示代理地址列表，当前代理地址40X时自动访问下一个，`direct`表示源地址抓取(如`github`),遇到`EOF`时终止并抛错`invalid version: unknown revision...`
- `GO111MODULE` 
    + `on` 使用Go Modules,go 会忽略 $GOPATH和 vendor文件夹,只根据go.mod下载依赖。
    + `off` 会查找 vendor目录和 $GOPATH来查找依赖关系，也就是继续使用`GOPATH模式`
    + `auto` (默认模式)根据当前项目目录下是否存在 go.mod文件或 $GOPATH/src之外并且其本身包含go.mod文件时才会使用新特性 Go Modules模式

### go env
- `go env [-json] [-u] [-w] [VAR ...]` 设置/查看环境变量，指定了`[VAR ...]`则只打印/设置选中的环境变量的值
- `-json` 以`json`格式打印环境变量
- `-u` 重置变量`VAR [...]`为初始值，以空格分隔。撤销`GOENV`中对应变量的修改
- `-w` 修改变量`VAR=VALUE [...]`，以空格分隔。修改内容会写入`GOENV`中，重启后也会生效
- 系统环境变量的值会覆盖`go env`的配置

### build/install/run/test flag公共参数
- `-n` 打印编译时会用到的所有命令，但不运行
- `-x` 打印编译时会用到的所有命令
- `-a` 强制对所有`包`重新构建
- `-v` 编译时显示包名
- `-p n` 开启并发编译，默认为cpu的逻辑核数
- `-race` 开启竞态条件检测
- `-work` 打印出编译时生成的`临时文件`的路径，并在编译结束时保留它。
- `-gcflags` 给go编译器传入参数, 使用`go tool compile`的参数
- `-ldflags` 给go链接器传入参数, 使用`go tool link `的参数

### go build
- 编译`命令源码文件`会在当前目录下生成`可执行文件`，不可编译多个`命令源码文件`，因为多个`main`函数在编译时会抛出重复定义错误 
- 编译`库源码文件`/`测试源码文件`时，日做检查性编译，不输出任何结果文件。
- 编译A包时会先检测A的`依赖包`，若其编译状态不是最新，则先编译`依赖包`然后编译A
- 不加参数代表编译当前目录对应代码包
- `-o` 指定输出文件名称
- `-i` 安装`依赖包`。即产生`依赖包`的归档文件至`$GOPATH/pkg/...`下

### go install
- `install`只比`build`多做了一件事: 安装编译后的`结果文件`到`指定目录`
- `命令源码文件`的编译生成的`可执行文件`保存在`$GOROOT/bin`下
- `库源码文件`的编译生成的`归档文件`保存在`$GOROOT/pkg`下

### go run
- `go run`在`临时目录`中编译并运行`命令源码文件`
- 若命令源码文件有参数如`go run showds.go -p ~/golang/goc2p`等于编译后执行`./showds -p=~/golang/goc2p`

### go get
- `go get` 获取`vcs`的(远程)包至`$GOPATH`，然后执行`go install`构建并安装它
- `-d` 只执行下载，不执行安装(`install`)。
- `-t` 下载并安装指定的代码包中的测试源码文件中依赖的代码包。
- `-u` 更新已有代码包及其依赖包。
- `-f` 与`-u`一起用时有效。该标记会忽略对`已下载代码包的导入路径`的检查
- `-fix` 下载代码包后先执行`修正`动作，而后再进行编译和安装。
- `-insecure` 允许命令程序使用非安全的scheme（如HTTP）去下载指定的代码包。

### go clean
- go clean删除执行其他命令时产生的一些文件或目录
- 删除`go build`生成的可执行文件
- 删除`go test -c`生成的`xxx.test`文件
- 删除编译时生成的`临时文件`
- `-i` 删除当前包在`$GOROOT/bin`或`$GOROOT/pkg`下生成的文件
- `-r` 删除`当前包`及其所有`依赖包`的上述文件
- `-cache` 删除所有`go build`命令的缓存
- `-testcache` 删除当前包所有的测试结果

### go doc
- 打印附于Go语言程序实体上的文档。可把`程序实体的标识符`作为该命令的参数，按`源码包查找方式`找到包并查看其文档
- `go doc`后不加参数表示查看当前包的所有文档，也可打印标准包如`go doc http.Request`
- `-cmd` 输出`main`包中`可导出的程序实体`的文档
- `-u` 输出`不可导出的程序实体`的文档
- `-c` 区分参数中的大小写
- `-src` 查看`程序实体`的源码
- `-ex`查看`程序实体`示例代码
- `-html`查看HTML格式的文档, 如`godoc -http=:6060`

### go test
- `go test basic pkgtool` 测试多个代码包，执行其中`测试源码文件`
- `go test envir_test.go envir.go` 参数为文件名会生成`虚拟代码包`并执行测试
- `-c` 生成用于运行测试的可执行文件`pkg.test`，但不执行它。
    + `pkg`为被测试代码包的导入路径的最后一个元素的名称
- `-i` 安装/重新安装运行测试所需的依赖包，但不编译和运行测试代码
- `-o` 指定用于运行测试的可执行文件的名称。追加该标记不会影响测试代码的运行。
- `go test`还有很多其他标记，应单独出个`go测试`模块的记录

```shell
# 默认执行每个测试文件的第一个测试函数
go test dir/package
# 清理测试缓存(缓存不会影响测试准确性)
go clean -testcache
# -bench 执行任意名称的性能测试函数
# -run 执行名为空的功能测试函数(即不执行功能测试)
go test -bench=. -run=^$ dir/packages
```

### go list
- `go list [-f format]/[-json] [optoin] [packages]` 展示包相关信息
- `[packages]`不填表示当前目录包，`all`表示所有包
- `-f format`以模板格式打印。模板格式默认为`-f '{{.ImportPath}}'`
- `-json` 以`json`格式打印。不设置此选项则默认为模板格式打印
- `-deps` 将`packages`的`依赖包`也打印出来
- `-e` 忽略错误信息打印
- `go list -m [-u] [-retracted] [-versions] [build flags] [modules]`
    + `-m` 使`go list`列出模块而不是包，仅应用与支持`mod`的包目录下
    + `-u` 添加&列出可用的升级信息。使用`go get -u need-upgrade-package`升级后会将新的依赖版本更新到`go.mod`也可以使用`go get -u`升级所有依赖
- [go list的打印字段的含义](https://www.kancloud.cn/cattong/go_command_tutorial/261354)

### go fmt
- `go fmt [path...]`等同于`gofmt -l -w [path...]`
- `gofmt [flags] [path...]` 格式化包或源文件，`flags`如下
    + `-d` 打印差异输出
    + `-e` 打印所有错误
    + `-l` 打印被格式化的文件名
    + `-r 'pattern -> replacement'` 格式化前载入规则
    + `-s` 尝试简化代码,如`s[a:len(s)]`简化为`s[a:]`
    + `-w` 格式化后的源不输出到`stdout`, 格式化文件时如果失败则回滚至源文件

### go tool fix
- `go fix`是`go tool fix`的简单封装
- `go fix`把指定代码包的所有Go语言源码文件中的旧版本代码修正为新版本的代码

### go tool vet
- `go vet`是`go tool vet`的简单封装
- `go vet`用于检查Go语言源码中静态错误的工具，可使用`-n`/`-x`标记
- `vet`属于Go自带的特殊工具，也是比较底层的命令之一。Go语言自带的特殊工具的存放路径是`$GOROOT/pkg/tool/$GOOS_$GOARCH/`，我们暂且称之为Go工具目录。
- `all` 检查全部。如果有其他检查标记被设置，则命令程序会将此值变为false。默认值为true。
- `asmdecl` 对汇编语言的源码文件进行检查。默认值为false。
- `assign` 检查赋值语句。默认值为false。
- `atomic` 检查代码中对代码包sync/atomic的使用是否正确。默认值为false。
- `buildtags` 检查编译标签的有效性。默认值为false。
- `composites` 检查复合结构实例的初始化代码。默认值为false。
- `compositeWhiteList` 是否使用复合结构检查的白名单。仅供测试使用。默认值为true。
- `methods` 检查那些拥有标准命名的方法的签名。默认值为false。
- `printf` 检查代码中对打印函数的使用是否正确。默认值为false。
- `printfuncs` 需要检查的代码中使用的打印函数的名称的列表，多个函数名称之间用英文半角逗号分隔。默认值为空字符串。
- `rangeloops` 检查代码中对在```range```语句块中迭代赋值的变量的使用是否正确。默认值为false。
- `structtags` 检查结构体类型的字段的标签的格式是否标准。默认值为false。
- `unreachable` 查找并报告不可到达的代码。默认值为false

## Go Modules
- `GO111MODULE` 可配置GoModules开关, `off`关闭，`on`开启，`auto`表示在`$GOPATH/src`下，且没有包含`go.mod`时则关闭`Go Modules`，其他情况下都开启`Go Modules`
- 使用模块时`GOPATH`仅用于存储源码`GOPATH/pkg/mod/`与命令`GOPATH/bin/`
- `go module`根据`go.mod`安装`package`的原則是先拉最新的`release tag`，若无`tag`则拉最新的`commit`。 `golang`会自动生成一个`go.sum`文件来记录`dependency tree`

### `go mod <command> [arguments]` 仅在开启后可用
- `go mod <command> [arguments]` `<command>`如下
    + `download` 下载go.mod 文件中的依赖包
    + `edit` 编辑`go.mod`
    + `graph` 打印模块依赖图
    + `init` 当前目录内初始化`mod`(创建`go.mod`)
    + `tidy` 拉去缺少的模块，移除不用的模块
    + `vendor` 将依赖复制到vendor下(是不是要兼容旧版本的go？)
    + `verify` 验证依赖是否正确
    + `why` 解释为什么需要依赖
- `go.mod`文件语法中提供了以下四个命令
    + `module` 指定包的名字(路径)
    + `exclude` 忽略依赖项模块
    + `require` 指定的依赖项模块 `<导入包路径> <版本> [// indirect]`
    + `replace` 替换依赖项模块 `$module => $newmodule`。`$newmodule`可以是本地磁盘的相对/绝对/网络路径
- `go.mod`被创建后，它的内容将会被`go toolchain`全面掌控。`go toolchain`会在各类命令执行时维护、修改`go.mod`文件。命令如`get`, `build`, `mod`

## go error 错误处理
- 错误应该只处理一次，做一个决定
- 降级过的错误不应该继续上抛
- 应当优雅减少`Error Handling`。类似`bufio.Scanner`的`Scan()`和`Err()`的联合处理，大大减少了`client`的调用

### 错误的几种定义方式
- `Sentinel Error`特定的错误，类似于`io.Eof`，更底层的有`syscall.ENOENT`。通常是用于开发时的调试，而不是业务开发时的返回
- `Error Type` 实现了`error接口`的自定义类型。如`os.PathError`。可以用`类型断言`或`类型switch`的方式通过`自定义类型`的变量获取更多的上下文
- `Opaque errors` 不透明的错误处理： 只返回错误而不假设其内容，通过行为获取更详细的信息。
    + 如`net.Error`在外部判断时想要更详细的信息可通过`Timeout`等方法获取更详细的信息

### `github.com/pkg/errors` 堆栈错误包
- 可用`errors.Wrap`包装错误，并记录错误堆栈
- 可用`errors.Cause`获取根因，进行断言。
- 在错误产生处返回包装错误，在程序顶部用`%+v`记录堆栈详情
- 仅在业务、项目中使用`pkg/errors`，库中不应该使用。

### go error新特性
- `errors.Is`和`errors.As`参数都可以对实现了`uwrap()`接口的错误进行递归判断

## concurrent programming 并发编程
- 共享资源的并发访问使用传统并发原语；
- 复杂的任务编排和消息传递使用 Channel；
- 消息通知机制使用 Channel，除非只想 signal 一个 goroutine，才使用 Cond；
- 简单等待所有任务的完成用 WaitGroup，也有 Channel 的推崇者用 Channel，都可以；
- 需要和 Select 语句结合，使用 Channel；
- 需要和超时配合时，使用 Channel 和 Context。

### 名词解释
- `临界区` 共享资源，可以是IO操作，数据结构，变量
- `data race` 数据竞争/竞态条件，多线程对`临界区`的并发读写
- `Mutex` 排他/互斥锁，通过阻塞的方式解决`data race`的问题。
- `重入锁` 同个线程可对`临界区`多次加锁解锁,可递归调用，`mutex`是`不可重入锁`
- `死锁` 多个线程对多个`临界区`的相互持有与等待。
    + 如go1锁住v1请求v2,go2锁住v2请求v1就会导致死锁

### 工具
- [race-detector](https://go.dev/blog/race-detector) 可做`data race`检测，只要在`test/run/build/install`时加上`-race`参数就可以检测了
- [go vet](#go-tool-vet) 可检查`死锁`([chekdead方法](https://go.dev/src/runtime/proc.go?h=checkdead#L4935))，它是通过[copylock分析器](https://github.com/golang/tools/blob/master/go/analysis/passes/copylock/copylock.go)静态分析实现的。

### channel
- `Communicating Sequential Process`简称`CSP`，中文直译`通信顺序进程`。CSP 允许使用进程组件来描述系统，它们独立运行，并且只通过消息传递的方式通信。
- go通过引入`channel`实现`CSP`思想，其主要应用场景有
    + `数据交流`: 当作并发的`buffer`或`queue`。将`goroutine`当作生产者`Producer`和消费者`Consumer`
    + `数据传递`: `goroutine`间的数据传递，相当于把数据的拥有权 (引用) 托付出去。
    + `信号通知`: 一个`goroutine`可以将信号`closing、closed、data ready等` 传递给另一个或者另一组`goroutine`
    + `任务编排`: 让一组`goroutine`按顺序并发或者串行的执行
    + `锁`: 利用`channel`实现锁机制
- 使用反射操作channel

### channel下的任务编排
- Or-Done 任意一个`<-inchan`完成后，就关闭`<-outchan`
- 扇入模式 每个`inchan`都写入数据，一个`outchan`输出数据
- 扇出模式
- Stream
- map-reduce


### context
#### 基本使用方法
```golang
type Context interface {
    Deadline() (deadline time.Time, ok bool)
    Done() <-chan struct{}
    Err() error
    Value(key interface{}) interface{}
}
```

- `Deadline`: `deadline`指`ctx`被取消的截止日期。没设置截至日期则`ok=false`。后续调用返回结果相同。
- `cancel`、`timeout`、`deadline`都会导致`ctx`被`cancel`
- `Done` 根据`ctx`的类不同，有不同的值或处理方式
    + ctx不为`cancelCtx`: `Done()`返回`nil`
    + ctx为`cancelCtx`且未被`cancel()`: `Done()`返回阻塞的`chan`
    + ctx为`cancelCtx`且被`cancel()`: `Done()`返回被关闭的`chan`。`Err`方法返回`close`的原因。
- `Value` 返回此 `ctx` 中和指定的`key`相关联的`value`。
- `context.Background()`返回一个非`nil`的、空的`Context`，没有任何值，不会被 cancel，不会超时，没有截止日期。一般用在主函数、初始化、测试以及创建根`Context`的时候。
- `context.TODO()`底层实现与`context.Background()`一样。当你不清楚是否该用 Context，或者目前还不知道要传递一些什么上下文信息的时候，就可以使用这个方法。

#### context使用规范
- 把`Context`作为方法的第一个参数
- 不使用`nil`作为`Context`的参数值
- `Context`只用来临时做函数之间的上下文透传，不能持久化`Context`或者把`Context`长久保存。把`Context`持久化到数据库、本地文件或者全局变量、缓存中都是错误的用法。
- `key`的类型不应该是`字符串类型`或者其它`内建类型`，否则容易在包之间使用`Context`时候产生冲突。使用`WithValue()`时，`key`的类型应该是`自定义类型`(非必须)
- 使用`struct{}`作为底层类型定义`key`的类型。使用`接口`或者`指针`作为底层类型定义`exported key`的静态类型。这样可以尽量减少内存分配

#### context使用场景
- 上下文传递`request-scoped`: 如处理http请求，处理链路上的数据传递
- 控制子goroutine运行
- 超时控制的方法调用
- 可取消的方法调用

#### WithValue
```golang
type valueCtx struct {
    Context
    key, val interface{}
}
```

- `WithValue`基于`Context intesrface`创建了`valueCtx`的实例。它持有一个KV键值对(用于传递上下文)。
- `valueCtx`覆盖了`Value`方法使用`链式查找`,它优先从自己的存储中检查这个`key`，不存在则从`Context intesrface`中继续检查，若`Context intesrface`也是`valueCtx`，则重复此过程(装饰器模式)

#### WithCancel
- `WithCancel(parent Context) (ctx Context, cancel CancelFunc)` 返回的`ctx`为`cancelCtx`类型，会新建`ctx.Done`对象，用于取消长时间的任务。触发如下情况时`ctx.Done`就会被`close`
    + `parent.Done`被`close`时
    + 执行`WithCancel`返回的`cancel()`方法时
- `WithCancel`执行时会调用`propagateCancel`方法，此方法会顺着`parent`向上查找到一个`cancelCtx`或`nil`
    + 找到`nil`(根ctx)，就会新起一个`goroutine`，用于监听`parent.Done`是否已关闭。
    + 找到`cancelCtx`就把`当前ctx`加入到这个`cancelCtx`的`child`，以便这个`cancelCtx`被取消的时候通知`当前ctx`
- `cancel()`是向下传递的，`子孙ctx`会被`cancel()`，但`祖先ctx`不会被`cancel()`
- 注: 只要任务完成(正常或异常结束)，就要调用`cancel`。这样才可以释放`ctx`资源(通知` child`处理`cancel`；从`parent`的`child`中移除自己；甚至释放相关的`goroutine`)

#### WithDeadline
- `WithDeadline(parent Context, d time.Time) (ctx Context, cancel CancelFunc)` d为截止时间，ctx可能为`cancelCtx`或`timerCtx`。
    + `d`晚于`parent的截止时间`则以后者为准，返回`cancelCtx`类型实例
    + 若`当前时间`超过了`截止时间`, 则返回已`cancel`的`timerCtx`。否则启动一个`timer`,到`截止时间`取消这个`timerCtx`
- `timerCtx.Done()`被`close`，有以下时间触发
    + 截止时间到了
    + `cancel` 函数被调用
    + `parent.Done`被`close`
- `timerCtx`也实现了`canceler`接口。不可依赖截止时间被动取消，`cancel`一定要尽早调用，这样才能尽早释放资源。

#### WithTimeout
```golang
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc) {
    // 当前时间+timeout就是deadline
    return WithDeadline(parent, time.Now().Add(timeout))
}
```
WithTimeout 其实是和 WithDeadline 一样，只不过一个参数是超时时间，一个参数是截止时间。超时时间加上当前时间，其实就是截止时间

## go project 工程化
### 规范设计
- 非编码类规范： 开源规范 文档规范 版本规范 Commit规范 发布规范
- 编码类规范： 目录规范 代码规范 接口规范 日志规范 错误码规范

#### 开源规范
- 发布可用的版本：确保每次发布都经过充分测试(详细的单元测试)，每个发布版本都是可用的。
- 合适的git工作流。遵循`Angular commit message`规范。提交记录中不出现内部IP、内部域名、密码、密钥等信息
- `LICENSE`开源软件协议： `GPL`,`MPL`,`LGPL`,`Apache`,`BSD`,`MIT`
- `CONTERIBUTING.md`： 用于说明如何给本项目贡献代码，包含详细贡献流程
- `CHANGELOG`目录： 用于存放版本变更历史
- 详细的文档说明
    + `Maikefile`： 对项目进行构建、测试、安装等操作
    + `README.md`： 包含`项目描述`,`依赖项`,`安装方法`,`使用方法`,`贡献方法`,`作者`,`遵循软件协议`
    + `docs`目录： 存放项目所有文档如 `安装文档`,`使用文档`,`开发文档`
    + `examples`目录： 存放示例代码

## 日志包设计
### 日志包应具备的功能
- 日志包要具备的功能可分为`基础`、`高级`、`可选`
- 设计日志包要关注的点 `高性能`、`并发安全`、`插件化能力`、`日志参数控制`、

#### 基础功能
- 支持基本日志信息 `时间戳`、`文件名`+`行号`(定位问题)、`级别`(过滤error)、`日志信息`
- 支持不同日志`级别`(由低至高) `trace`(可选的), `debug`, `info`, `warn`, `error`, `panic`(可选的), `fatal`
    + `输出级别` 如`glog.Info("xxx")`指输出级别为`info`
    + `开关级别` 启动应用程序时期望那些`输出级别`的日志被打印，如`开关级别 >= error`则仅记录`error`, `panic`, `fatal`
- 支持自定义配置 如根据不同环境设置不同日志级别，不同情况下输出不同的日志格式
- 支持输出到标准输出和文件

#### 高级功能
- 支持多种日志格式 `text` `json`
- 能按级别分类输出 如错误级别日志单独输出到文件
- 支持`structured logging`(结构化日志)
- 支持`日志轮转` 建议第三方工具实现，如在`Linux`中使用`Logrotate`来轮转日志
- 具备`hook`能力可对日志内容进行自定义处理，如`panic`级别出现时用邮件进行告警
    + 在一个大型系统中，日志告警是非常重要的功能，但更好的实现方式是将告警能力做成旁路功能。通过旁路功能，可以保证日志包功能聚焦、简洁。例如：可以将日志收集到 Elasticsearch，并通过 ElastAlert 进行日志告警。
    
#### 可选功能
- 支持颜色输出
- 兼容标准库log包
- 支持输出到不同位置 如`elasticsearch` `kafka` `fluentd` `logstash`
    + 也可以通过`filebeat`采集硬盘日志再进行投递
    

### 日志包的使用(如何记录日志)
#### 使用时机，哪种代码时嵌入
- 在分支语句处打印日志
- 写操作必须打印日志
- 循环中打印日志要慎重 循环打印会降低性能，建议循环中记录要点，循环外总结打印
- 错误产生的最原始位置打印日志，如error一层层上报时仅在最原始层打印错误日志

#### 设置`开关级别`
- `Debug`级别仅在开发调试阶段需要
- `Info`以满足需求为主要目标，不可频度过高(高频度可记录在Debug级别)
- `Warn`通常为`异常但不影响程序运行`的日志，常用于业务级别警告日志
- `Error`错误日志
- `Panic`很少使用，通常在错误堆栈或defer处理错误时使用
- `Fatal`严重错误，通常为系统级别错误，会导致程序无法运行

#### 日志内容
- 不要输出敏感信息 密码、密钥
- 应用Debug日志调试时可用`log.Debugf("XXXXXX-1: input key was: %s", setKeyName)`，调试完成后在找到这些临时日志并删掉。
- 日志内容应`小写字母开头 英文点号结尾`
- 输出时应用明确类型如`%s`而不是`%v`
- 日志应包含`requestID`,`uid`,`行为`

#### 最佳实践
- 支持requestID
- 根据故障排查过程优化日志打印
- 结构化日志记录，添加通用字段在每行日志中
- 支持动态日志输出，如在请求中通过debug=true开启debug日志

#### 分布式日志解决方案(EFK/ELK)
- ELK: Elasticsearch + Logstash + Kibana
- EFK: Elasticsearch + Filebeat + Kibana
- `Filebeat`比`Logstash`更轻量级



## references
- [goproxy](https://goproxy.io/zh/)

### 官方文档
- [go_download](https://go.dev/dl/)
- [go_cmd](https://golang.google.cn/cmd/)
- [go_cmd_zh](https://www.kancloud.cn/cattong/go_command_tutorial/261347)
- [go_env](https://golang.google.cn/cmd/go/#hdr-Environment_variables)
- [go_mod](https://golang.org/ref/mod)
- [go_effective](https://go-zh.org/doc/effective_go.html)

### 比较好的入门教程
- [go语言圣经](https://books.studygolang.com/gopl-zh/)
- [go语言高级编程](https://chai2010.cn/advanced-go-programming-book/)

### 知名的开源项目
- https://github.com/avelino/awesome-go
- https://github.com/golang-standards/project-layout