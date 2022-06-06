## 帮助链接
- [pprof命令详解](https://www.kancloud.cn/cattong/go_command_tutorial/261357)
- [pprof](https://www.cnblogs.com/qcrao-2018/p/11832732.html)


## 概要文件
- `概要文件`: 通过标准库的`runtime`和`runtime/pprof`中的程序能生成包含`实时性数据`的`概要文件`


### CPU概要文件
- Go语言运行时系统会以`100Hz`的频率对`Goroutine堆栈上`的`程序计数器`进行取样，即每`10ms`取样一次
- `CPU主频`即CPU内核工作的时钟频率`CPU Clock Speed`(单位HZ)，`时钟频率`的倒数即为`时钟周期`，在一个`时钟周期`内，CPU执行一条`运算指令`。
- `pprof.StartCPUProfile(file)`开始记录`cpu使用情况`
- `pprof.stopCPUProfile()`停止记录`cpu使用情况`(取样的频率设置为0)

### 内存概要文件
- Go语言运行时系统会记录`用户程序运行期间`的所有`堆内存分配`。只要`堆内存`被分配`MemProfileRate`，分析器就会对其进行取样
- `pprof.WriteHeapProfile(file)`将`堆内存分配情况`写入文件
- 分析器的取样间隔`runtime.MemProfileRate byte`: 分析器会在每分配指定的`字节数量`后对`内存使用情况`进行取样
    + 默认是`512*1024`即`512K`个字节
    + 我们将`MemProfileRate`赋值为`0`表示取消取样

### 程序阻塞概要文件
- `程序阻塞概要文件`用于保存用户程序中的`Goroutine阻塞事件`的记录
- 设置分析器的取样间隔`runtime.SetBlockProfileRate(*rate)`: 每发生几次Goroutine阻塞事件时对这些事件进行取样。
    + 默认为1，即每次阻塞都会取样
    + 设置为0或负数，表示取消取样


### 其他概要文件
- 通过`pprof.Lookup("block").WriteTo(file, 0)`将保存在运行时内存中的`内存使用情况`记录取出，并在记录的实例上调用`WriteTo`方法将记录写入到文件中。

|     名称     |           说明          |       默认取样频率      |
|--------------|-------------------------|-------------------------|
| goroutine    | 活跃goroutine的信息记录 | 获取时取样一次          |
| threadcreate | 系统线程创建情况的记录  | 获取时取样一次          |
| heap         | 堆内存情况分配的记录    | 默认每分配512KB取样一次 |
| block        | goroutine阻塞时间的记录 | 默认没次阻塞时取样      |
