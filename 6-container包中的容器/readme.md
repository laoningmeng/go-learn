# container包中的容器



## 1. 双向链表

前面介绍了数组和切片这种连续内存的数据结构， 下面介绍一下链表的数据结构。go 语言中链表的实现在

`container/list`中。

其中list 包中，开放出来两种结构

```go
// Element is an element of a linked list.
type Element struct {
	// Next and previous pointers in the doubly-linked list of elements.
	// To simplify the implementation, internally a list l is implemented
	// as a ring, such that &l.root is both the next element of the last
	// list element (l.Back()) and the previous element of the first list
	// element (l.Front()).
	next, prev *Element

	// The list to which this element belongs.
	list *List

	// The value stored with this element.
	Value any
}

```

元素E lement 的数据结构，对外开放访问Value， 通过包内属性`next`, `prev`我们看出来这个是一个双向链表结构



下面是List 的数据结构 

```go
// List represents a doubly linked list.
// The zero value for List is an empty list ready to use.
type List struct {
	root Element // sentinel list element, only &root, root.prev, and root.next are used
	len  int     // current list length excluding (this) sentinel element
}
```



链表这种结构，记录长度还是非常有必要的，因为可以根据长度取判断位置。



链表的缺点：

* 我们可以看到链表的数据结构比较复杂，意味着需要消耗更多的空间。
* 查询比较麻烦，时间复杂度是O（n）, 但是对于插入，删除的时间复杂度又为O(1)



**延迟初始化：** 定一个一个变量以后，在需要时才会进行初始化操作。



## 2. 环表

位置`container/ring`包下的`Ring` .



