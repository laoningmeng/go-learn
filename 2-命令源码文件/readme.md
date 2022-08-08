# 命令源码文件



## 1. 源码文件分类

* 命令源码文件
* 库源码文件
* 测试源码文件



## 2. 什么是命令源码文件

简单的说就是可以在命令行中直接执行的文件，也就是我们常见的入口文件，具有一些固定的结构

```go
package main
import "fmt"

func main(){
  fmt.Println("hello")
}
```

上面就是一个最简单的命令源码文件了，他能通过`go run main.go` 直接运行， 如果想把它填进环境变量，可以直接通过`go install main.go`就可以了。

## 3. 接收参数

我们常见的类似git ,就是一个命令行命令，他们往往都能接受用户输入的参数， 下面我们看一下go 语言中如何接受参数的。

这里其实有两种方法，一种是os.Args

下面是实例

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	args := os.Args
	for index, value := range args {
		fmt.Println("index:", index, "value:", value)
	}
}
// go run main.go name=zhangsan age=18
/**
index: 0 value: /var/folders/8h/247_050d5k36lkjpcd7gcb240000gn/T/go-build728859727/b001/exe/main
index: 1 value: name=zhangsan
index: 2 value: age=18
*/

```



上面是一个简单的获取参数的方法，通过上面的例子也可以看出， 通过os.Args获取的参数是保存在`var Args[] string` 的切片数据结构，且第一个参数是执行的文件地址， 参数都是string, 无法解析键值对关系



下面是方式二， 利用专门解析命令行的库`flag`

这里有两种解析变量的方法，一种是通过引用的方式赋值变量，一种是直接返回结果

* 通过引用的方式直接赋值给变量的方式通常是xxxVar的函数，下面是一个举例

  ```go
  var name string
  flag.StringVar(&name, "name", "default name", "usage description")
  flag.Parse()
  fmt.Println(name)
  ```

  这里需要说明一下，`flag.Parse()` 这个函数用于flag的变量定义以后，变量被访问之前。

  

  

* 下面是通过直接返回值的方式

  ```go
  var name = flag.String("name","default name", "usage desc")
  flag.Parse()
  fmt.Println(*name) // 这里需要说明一些，flag.String 返回的类型是*string 获取其中的值，需要使用*string 进行获取一些，否则打印的是这个变量的内存地址。
  ```

  



其实还有方式三，就是利用开源的库， 比如cobra 







## 4.  如何自定usage 的使用说明



采用直接覆盖赋值

```go
flag.Usage=func(){
  fmt.Println("随便打印一些东西")
}
```

这样我们在运行`go run main.go --help` 

结果为：

```go
//随便打印一些东西
```

我们发现 平时我们打he l p 的时候下面会列出各个参数的用法说明， 如何解决？ 可以加上下面的内容

```go
flag.Usage=func(){
  fmt.Println("随便打印一些东西")
  flag.PrintDefaults() // 加一个这个就好了
}
```



结果为:

```go
随便打印一些东西
  -name string
        hello (default "default name")
  -name1 string
        用法 (default "default name")

```





## 5. 更深一层的CommandLine



我们通过goland 的跳转功能，跳转到flag.String, flag.Usage中，我们发现，我们更改的都是一个叫做CommandLine的数据结构， 所以上面的写法还可以换成更底层一些的写法

```go
flag.CommandLine = flag.NewFlagSet("", flag.ExistOnError) // ExistOnError 是告诉程序如果参数不正确时的异常码
flag.CommandLine.Usage = func(){
  fmt.Println("xxxx")
  flag.PrintDefaults()
}
```





## 6. 接受自定义的数据结构



我们打开flag.go的包，上面有一段关于接受自定义数据格式的描述

```go
Or you can create custom flags that satisfy the Value interface (with
	pointer receivers) and couple them to flag parsing by
		flag.Var(&flagVal, "name", "help message for flagname")
	For such flags, the default value is just the initial value of the variable.
```

这段话的意思就是如果传递自定义的数据接口，需要实现一些Value的接口，然后使用flag.Var() 获取参数

```go

type Value interface {
	String() string // 序列化
	Set(string) error // 反序列化
}
```



下面是一个简单的实现：

```go
package main

import (
	"encoding/json"
	"flag"
	"fmt"
)

type Info struct {
	Age  int
	Name string
}

func (info *Info) String() string {
	var desc string = "姓名：" + info.Name + "年龄为" + (string)(info.Age)
	fmt.Println(desc)
	return desc
}
func (info *Info) Set(v string) error {
	fmt.Println("set 的数据为", v)
	return json.Unmarshal([]byte(v), &info)
}

func main() {
	var info Info
	flag.Var(&info, "info", "自定义的数据结构")
	flag.Parse()
	fmt.Println("自定义的数据结构", info)
}

```



运行：

```shell
go run main.go -info='{"Age":18,"Name":"zhangsan"}'
```

