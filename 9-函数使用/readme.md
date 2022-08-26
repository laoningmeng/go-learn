# 函数的使用

## 1. 函数是一等公民

这句话的意思是函数享有和其他数据类型相同的权利，更通俗一点就是你可以像使用变量那样使用函数，可以把函数当作一个变量，一个变量值，函数也是一种数据类型。



```go
type Printer func(content string)(n int, err error) // type 定义了一个printer的新类型
```



