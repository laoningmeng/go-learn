# 通道操作



## 1. 什么是通道

这是go内置的一种数据结构，一种类似数组， 字符串一样的数据类型。



## 2. 通道有什么用

进程间的通信， 一个通道相当于一个队列， 并且是并发安全的， 通道中的各个元素都按照发送顺序排列，先被发送的，一定先被接收。

使用场景：

* 停止信号监听
* 定时任务
* 生产和消费解藕
* 控制并发数量







## 3. 定义通道



使用make 创建

```go
var myChan chan int = make(chan int)
var myChan chan int = make(chan int, 2) // 第二个参数为缓冲区的大小
```

* 非缓冲通道
* 缓冲通道

## 4. 通道的使用

因为通道可以看做一个队列，那么队列就有两个操作，进队列和出队列，也可以说成接受和发送你消息



接收表达式:

* ```go
  message:=<-ch
  ```

发送表达式：

* ```go
  ch<-message
  ```



这个箭头就是我们要发送信息的走向，ch为通道变量



## 5. chan的结构及原理

```go
// This file contains the implementation of Go channels.

// Invariants:
//  At least one of c.sendq and c.recvq is empty,
//  except for the case of an unbuffered channel with a single goroutine
//  blocked on it for both sending and receiving using a select statement,
//  in which case the length of c.sendq and c.recvq is limited only by the
//  size of the select statement.
//
// For buffered channels, also:
//  c.qcount > 0 implies that c.recvq is empty.
//  c.qcount < c.dataqsiz implies that c.sendq is empty.

type hchan struct {
	qcount   uint           // total data in the queue
	dataqsiz uint           // size of the circular queue
	buf      unsafe.Pointer // points to an array of dataqsiz elements 有缓冲时存储数据
	elemsize uint16
	closed   uint32 
	elemtype *_type // element type
	sendx    uint   // send index  发送时缓冲区当前数据的索引
	recvx    uint   // receive index 接收时缓冲时当前数据的索引
	recvq    waitq  // list of recv waiters 也是一个队列，用于等待goroutine的接收数据
	sendq    waitq  // list of send waiters

	// lock protects all fields in hchan, as well as several
	// fields in sudogs blocked on this channel.
	//
	// Do not change another G's status while holding this lock
	// (in particular, do not ready a G), as this can deadlock
	// with stack shrinking.
	lock mutex // 锁定channel的读写，每个读写操作发送和接受必须时互斥的
}


type waitq struct {
	first *sudog // sudog 代表goroutine
	last  *sudog
}



// sudog represents a g in a wait list, such as for sending/receiving
// on a channel.
//
// sudog is necessary because the g ↔ synchronization object relation
// is many-to-many. A g can be on many wait lists, so there may be
// many sudogs for one g; and many gs may be waiting on the same
// synchronization object, so there may be many sudogs for one object.
//
// sudogs are allocated from a special pool. Use acquireSudog and
// releaseSudog to allocate and free them.
type sudog struct {
	// The following fields are protected by the hchan.lock of the
	// channel this sudog is blocking on. shrinkstack depends on
	// this for sudogs involved in channel ops.

	g *g

	next *sudog
	prev *sudog
	elem unsafe.Pointer // data element (may point to stack)

	// The following fields are never accessed concurrently.
	// For channels, waitlink is only accessed by g.
	// For semaphores, all fields (including the ones above)
	// are only accessed when holding a semaRoot lock.

	acquiretime int64
	releasetime int64
	ticket      uint32

	// isSelect indicates g is participating in a select, so
	// g.selectDone must be CAS'd to win the wake-up race.
	isSelect bool

	// success indicates whether communication over channel c
	// succeeded. It is true if the goroutine was awoken because a
	// value was delivered over channel c, and false if awoken
	// because c was closed.
	success bool

	parent   *sudog // semaRoot binary tree
	waitlink *sudog // g.waiting list or semaRoot
	waittail *sudog // semaRoot
	c        *hchan // channel
}
```



hchan 就是channel 的具体实现



下面简述一下channel的原理：

```go
```





## 6. 发送和接受的一些特性



* 同一个通道，发送操作之间时互斥的，接收消息也是

* 发送操作完成前下一次的发送动作会被阻塞，接收消息也是
* 发送和接收中对元素值的处理时不可分隔的。也就是接受和发送的数据数据被处理时，数据时完整的。
* 元素值进入通道时会被复制，也就是值传递。







## 7. 深拷贝和浅拷贝

**数据类型**

数据类型分为基础数据类型和对象数据类型

基本数据类型存储在栈空间

引用数据类型： 真是数据存放在堆，栈上存的时该对象的引用， 也就是引用数据类型在栈上存储了指针。当解释器在寻找引用值时，会先从栈上找指针，进而获取到堆上实体。



**浅拷贝和深拷贝**



* 浅拷贝时复制指向某个对象的指针，而不复制对象本身。新旧对象还是共享同一块内存。
* 深拷贝会复制一个一摸一样的对象，新对象和原对象不共享内存，修改新对象不会改变旧对象。

* 值类型的数据默认时深拷贝, 如int float string bool array struct
* 引用类型的数据默认时浅拷贝，如slice, map,function







## 8. 双向管道



上面我们说了通道有发送的，和接受的，通过箭头·`<-`的方向来判断，其实通道还有双向的，就是省略箭头



* 单向的

  ```go
  make(chan <-int, 1)
  ```

* 双向的

  ```go
  make(chan int, 1)
  ```

  函数中：

```go
func test(channel chan int){
  
}
```

单项通道的意义时一种约束，保证职能的单一性



## 9. 通道取值方式



```go
// for...range 取值
for ele:=range mychan{
  fmt.Println(ele)
}
// for 取值

for{
  ele:=<-mychan
  fmt.Println(ele)
}

select{
  case <-mychan[0]:
  default:
  //... 这里需要注意一些，如果直接用通道变量的话，有default 可能不会再阻塞了
}


```







## 10. 关闭通道



通道使用完以后要手动关闭

```go
var channel=make(chan int,2)

close(channel)
```

