---
title: 调试小帮手GDB
authors: ["Misaki"]
date: 2024-08-08
categories: ["GDB"]
---

## 简单的介绍

c++工程很多时候因为涉及到大量的三方库和框架，调试起来并不那么容易。这时候就需要GDB来扛起这个调试大旗了。

它可以设置断点、单步执行代码、查看调用栈，分析程序的崩溃原因。在一些意料之外的情况发生的时候，避免直接修改源码而又可以快速排查问题。

<!-- more -->

## 前置

*这里假设我们使用cmake作为c++工程的构建工具*

* 设置构建类型为debug
  如果是可执行文件，可以直接加在命令里：
  ~~~cmake
  cmake -DCMAKE_BUILD_TYPE=Debug ..
  ~~~

  当然，更推荐以参数的形式加在cmake文件里：
  ~~~cmake
  set(CMAKE_BUILD_TYPE Debug)
  ~~~

* 或者手动设置调试标记
  ~~~cmake
  set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -g")
  ~~~

* *有时你不会想编译器帮你优化了些什么的, 加上标记关闭它吧*
  ~~~cmake
  set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -O0")
  ~~~

如果你要调试的是第三方库(例如xxx.so)，那就需要它在编译的时候是带了调试标记的。

## 调试

### 断点型

这种方案适合那些你有源码的情况下，不管问题是不是最终会导致中断，都可以排查异常出现的具体位置。

1. 启动

    ~~~sh
    gdb ./program
    ~~~

    使用`run`或者`r`启动可执行程序

    如果要加参数，和正常启动一样设置即可，例如
    ~~~
    r -p 1234
    ~~~

2. 断点的设置和删除

    使用`list`或者`l`查看代码
    ~~~
    l test.cpp:10
    ~~~

    使用`break`或者`b`设置断点
    ~~~
    b test.cpp:11
    ~~~

    使用`info b`查看设置的断点
    ~~~
    info b
    ~~~
    
    使用`delete <number>`或者`d <number>`删除断点(编号)
    ~~~
    d 1
    ~~~

    或者使用`disable`和`enable`来禁用/启用断点
    ~~~
    disable 1

    enable 1
    ~~~

3. 打印和修改变量

    使用`print`或者`p`打印变量
    ~~~
    p input
    ~~~

    使用`set`为变量设置新的值
    ~~~
    set int v = 10
    ~~~

4. 查看堆栈

    使用`backtrace`或者`bt`打印堆栈信息(通常是在中断发生时)
    ~~~
    bt
    ~~~

5. 逐步运行

    - 使用`next`或者`n`单步执行下一行代码

    - 使用`step`或者`s`进入函数

    - 使用`continue`或者`c`执行直到结束或下个断点


### 进程型

这种方案适合你没有源码，或者你的产品并不是个一次性运行的东西(例如它是个http服务)，同时需要**问题**能够触发可执行程序运行时中断。

例如用`ps -ef`配合`grep`找到你的进程id，然后像这样将gdb附加到进程上

~~~sh
gdb attach 12345
~~~

然后你就能像调试可执行程序那样调试进程了，记得退出时用`detach`分离。

## 更多功能

### 条件断点
GDB 支持条件断点，即只有在满足特定条件时，程序才会在断点处暂停。例如：

```bash
(gdb) break my_function if x == 5
```

这将在 `x == 5` 时触发 `my_function` 的断点。

### 监视变量
GDB 可以监视某个变量或内存位置的变化，当其值发生改变时，程序会暂停执行。这对于调试内存问题非常有用：

```bash
(gdb) watch my_variable
```

或监视内存地址：

```bash
(gdb) watch *0x601040
```

### 逆向调试
GDB 支持在某些平台上进行逆向调试，即你可以倒退程序的执行，查看之前的状态。这在调试复杂问题时非常有帮助：

```bash
(gdb) reverse-continue
(gdb) reverse-step
```

### 调试多线程程序
GDB 支持调试多线程程序，你可以查看和控制各个线程：

- **列出所有线程**：

  ```bash
  (gdb) info threads
  ```

- **切换到特定线程**：

  ```bash
  (gdb) thread [thread-number]
  ```

- **设置线程特定的断点**：

  ```bash
  (gdb) break my_function thread 2
  ```

### 调试共享库
GDB 支持调试加载的共享库（动态链接库）。你可以在共享库中设置断点，并查看共享库的加载情况：

- **查看加载的共享库**：

  ```bash
  (gdb) info sharedlibrary
  ```

- **在共享库中设置断点**：

  ```bash
  (gdb) break my_shared_library.c:42
  ```

### 脚本化调试
GDB 支持使用 Python 脚本来扩展和自动化调试任务。你可以编写 Python 脚本来自动化调试步骤或解析调试数据：

```python
# example.py
import gdb
class HelloWorld(gdb.Command):
    def __init__(self):
        super(HelloWorld, self).__init__("hello", gdb.COMMAND_USER)

    def invoke(self, arg, from_tty):
        print("Hello, GDB World!")
HelloWorld()
```

在 GDB 中加载并运行脚本：

```bash
(gdb) source example.py
(gdb) hello
```

### 内存检查
GDB 可以直接检查和操作内存内容：

- **查看内存内容**：

  ```bash
  (gdb) x/16xw 0x601000
  ```

  这里的 `x/16xw` 表示以 16 个字的宽度显示内存内容。

- **修改内存内容**：

  ```bash
  (gdb) set {int}0x601000 = 42
  ```

### 记录和重放调试会话
GDB 允许你记录调试会话，稍后可以重放该会话以重复分析问题：

```bash
(gdb) record
(gdb) replay
```

### 符号表操作
GDB 可以加载、卸载和操作符号表，这在调试多模块程序时非常有用：

- **加载符号表**：

  ```bash
  (gdb) symbol-file my_program
  ```

- **卸载符号表**：

  ```bash
  (gdb) remove-symbol-file my_program
  ```

### 核心转储文件调试
GDB 可以加载核心转储文件（core dump）来分析程序崩溃的原因：

```bash
gdb my_program core
```

加载核心文件后，你可以检查崩溃时的状态、堆栈等信息。

### 会话脚本
可以将一系列 GDB 命令写入一个脚本文件，然后在 GDB 启动时自动执行：

```bash
gdb -x commands.gdb my_program
```