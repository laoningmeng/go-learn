# 字典操作和约束

## 1. Map 类型

Map 类型就是键值对，也可以跟python 一样称之为字典



## 2.Map的原理

这里的原理不仅仅适用于golang, map的底层实现方式多为hash， 实现过程为，我们把key当作参数传入哈希函数中，得到一个哈希值，然后将其存入到hashTable 中，



下面是几个概念

* 哈希表
* 哈希桶
* 负载因子







对于key-value 键值对， 我们先将key使用哈希函数把键处理一下， 得到一个哈希值， 然后将这个key-value 放置到哈希桶中，有两种比较常用的方式

* 取模法, 区间为[0,m-1] m为hash bucket数量。hash%m
* 与运算法, hash & m

要想不出现空桶，就必须保证m的数量为2的整数次幂。



如果有数据进入相同的桶，这时就发生了`哈希冲突`， 解决冲突的方式，常见的有两种：

* 开放地址法： 根据哈希值找到桶后，比对一些key是不是有值/是不是相同， 如果是插入时，则找一下个桶，查找时，根据哈希值找到对应桶，然后比对key， 如果不同就找下一个桶，直到出现空桶
* 拉链法，后面追加一个桶



通常把存储键值数量和桶的数量的比值，叫做负载因子，作为是否扩容的参考标准。





## 3. 哈希表扩容

哈希表中记录桶的数量， 旧桶的地址，扩容的进度， 将扩容时间分摊到多次哈希表操作上，这种方式叫渐进式扩容， 避免扩容瞬间抖动。



 

## 4. map键的数据类型

这个类型不许可以进行比较，所以函数等不能是键的类型。因为定位到bucket 桶以后定位具体的位置时，需要对key进行判断。















____

## 关于如何阅读golang 的源代码







## 关于调试

* 使用goland 的dubug 进行处理， 这种一般是标准库的内容

* 更底层一些的结构，如map的实现原理，就需要借助一下其他的工具来看了

  * 分析汇编代码， 这里使用`go tool compile --help ` 可以获取到汇编指令,如

    ```shell
    go tool compile -S -N main.go >>aa.txt
    ```

    

  * dlv调试

    dlv类似g d b， 不过go 推荐dlv, 另外如果是远程项目，无法像本地一样使用ide进行断点调试，可以使用goland remote+dlv,下面是dlv的使用

    首先是dlv的可用命令

    ```shell
    Available Commands:
      attach      Attach to running process and begin debugging.
      connect     Connect to a headless debug server with a terminal client.
      core        Examine a core dump.
      dap         Starts a headless TCP server communicating via Debug Adaptor Protocol (DAP).
      debug       Compile and begin debugging main package in current directory, or the package specified.
      exec        Execute a precompiled binary, and begin a debug session.
      help        Help about any command
      run         Deprecated command. Use 'debug' instead.
      test        Compile test binary and begin debugging program.
      trace       Compile and begin tracing program.
      version     Prints version.
    
    ```

    

    进入dlv后可以使用的命令

    ```go
    args ------------------------ 打印函数参数.
    break (alias: b) ------------ 设置断点.
    breakpoints (alias: bp) ----- 输出活动断点的信息.
    call ------------------------ 恢复进程，注入一个函数调用(还在实验阶段!!)
    clear ----------------------- 删除断点.
    clearall -------------------- 删除多个断点.
    condition (alias: cond) ----- 设置断点条件.
    config ---------------------- 修改配置参数.
    continue (alias: c) --------- 运行到断点或程序终止.
    deferred -------------------- 在延迟调用的上下文中执行命令.
    disassemble (alias: disass) - 反汇编程序.
    down ------------------------ 将当前帧向下移动.
    edit (alias: ed) ------------ 在$DELVE_EDITOR或$EDITOR中打开你所在的位置
    exit (alias: quit | q) ------ 退出调试器.
    frame ----------------------- 设置当前帧，或在不同的帧上执行命令.
    funcs ----------------------- 打印函数列表.
    goroutine ------------------- 显示或更改当前goroutine
    goroutines ------------------ 列举程序goroutines.
    help (alias: h) ------------- 打印帮助信息.
    list (alias: ls | l) -------- 显示源代码.
    locals ---------------------- 打印局部变量.
    next (alias: n) ------------- 转到下一个源行.
    on -------------------------- 在命中断点时执行命令.
    print (alias: p) ------------ 计算一个表达式.
    regs ------------------------ 打印CPU寄存器的内容.
    restart (alias: r) ---------- 重启进程.
    set ------------------------- 更改变量的值.
    source ---------------------- 执行包含delve命令列表的文件
    sources --------------------- 打印源文件列表.
    stack (alias: bt) ----------- 打印堆栈跟踪信息.
    step (alias: s) ------------- 单步执行程序.
    step-instruction (alias: si)  单步执行一条cpu指令.
    stepout --------------------- 跳出当前函数.
    thread (alias: tr) ---------- 切换到指定的线程.
    threads --------------------- 打印每个跟踪线程的信息.
    trace (alias: t) ------------ 设置跟踪点.
    types ----------------------- 打印类型列表
    up -------------------------- 向上移动当前帧.
    vars ------------------------ 打印包变量.
    whatis ---------------------- 打印表达式的类型.
    ```

    下面是使用实例

    ```go
    // 打断点
    dlv debug main.go
    b main.go:6 // 第六行设置断点
    c // c continue 继续
    goroutine // 显示当前的gorutine
    。。。。
    ```

    

​	







