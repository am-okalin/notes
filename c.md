[TOC]
## 帮助链接
-[cygwin](http://www.cygwin.com/install.html)

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

## 数据类型
- 整形：short,int,long
- 浮点型：float,double
- 字符型：char
- 枚举：enum
- 结构体：struct
```c
// 结构体名 和 结构体变量 至少存在一个
struct 结构体名 {
    结构体所包含的变量或数组
} 结构体变量;

// 声明结构体变量stu1, stu2
struct 结构体名 stu1, stu2;

// 或者直接用typedef创建新类型(常见)
typedef struct {
    结构体所包含的变量或数组
} 结构体名;
```
- 共用体：union
- 空：void
- 数组：特殊的指针
- 指针