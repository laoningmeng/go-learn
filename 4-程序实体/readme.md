# 程序实体



## 1. 声明变量的几种方式



```go
var name string = "zhangsan"
var name="zhangsan"
name:= "zhangsan" // 这种用于函数内部使用
var name = *flag.String("name","everyone","提示信息") // flag.String 返回的是*string  使用* 将指针对应的值取出来
```



上面几种声明变量的方式`var name string="xxx"`这个是最完整的，剩下的都利用了go 的类型推断。







## 2. 局部变量和全局变量

在很多其他语言里也有局部变量和全局变量的概念， 不过为什么局部变量可以直接"覆盖掉" 全局的变量的？

全局变量放在一个内存位置， 函数定义时，又单独开辟的内存空间， 所以所谓的覆盖，其实是在那块内存空间重新声明了一个变量。 





## 3. 什么是闭包

闭：里面的元素对外面来说是封闭的，包：包含了需要的外界元素。 其实有些像国中国， 他所依赖的上层函数的生命周期虽然结束了，但是它能独立存在，等待gc回收



## 4. 作用域

可以理解成开辟的内存空间， 全局作用域的生命周期较长的内存块， 如创建一个函数的局部作用域， 是开辟的一块临时存储空间，用完就销毁了。



## 5. 变量的访问权限

* 包级私有的
* 模块级私有的
* 公开的





## 6. 变量类型的判断

类型断言

```go
value, ok := interface{}(container).([]string)
```

类型断言的语法为：

x.(T) 这个x代表要被判断的那个着，这个值当下必须是接口类型，如果container 变量不是接口类型时，需要先将其类型转化一下。



一对不包含任何东西的花括号， 除了可以表示空代码段以外还表示不包含任何内容的数据结构。如`struct{}` 就表示一种数据类型。



## 7. 类型转换

累类型转换的表达式 T(x)

需要注意的几点：

* 精度不同的类型转换时，注意可能有精度的丢失

* 字符串和整数的转换，需要注意一下，因为我们知道所有的字符串其实都是一个个数字的映射，所以我们执行下面的语句的时候

  ```go
  fmt.Println(string(100)) //d
  ```

  如何解决我只想要"100" 的清情况 ，go 内置了一些函数解决了这个问题

* 同理还有[]byte 和字符串的转换， string 这个类型不是基本的数据类型，底层是[]byte实现的。

* []byte 跟string 的转化有一个小问题，那就是一个字符串究竟对应多少个byte 这个其实是根据具体的字符来看，像a,b,c 是两个字节，汉字的话有的三个，有的四个字节， 这种情况出现了一个新的数据类型`rune`其实他就是int32的别名， 我们知道1个byte 8bit 也就是1个字节。int32 有4个字节，这个长度可以表示任意的某一个字符。 











