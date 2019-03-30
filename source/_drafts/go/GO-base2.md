[toc]

#### 教程
- [go源码](https://go.googlesource.com/go)
- 启动本地的go教程
  - go tool tour或下载教程源码编译执行


#### basics
##### packages
- 每一个go程序由包组成
- 程序启动运行于main包
- 按照约定，包名和导入路径的最后一个元素相同，math/rand包就由开头语句是package rand的文件组成

##### imports
多个import用()包括
```go
import ("fmt"
"math"
)
也可写作为
 import "fmt"
 import “math”
```
一般推荐用上面的风格
###### 别名

##### Exported Names
- 在go中，如果名称是大写的，它就是一个出口的名称
比如Pi，它是从math包导出的
pi不以大写字母开头，因此它没有被导出
- 当引入包时，只能引用导出的名称
- 任何没有被导出的名称在包外是不能访问的

##### Functions
- 一个函数可以包含0个或多个参数
- 类型在变量名称的后面
```go
package main
import "fmt"
func add(x int, y int) int {
	return x + y
}
func main() {
	fmt.Println(add(42, 13))
}
```
- 当两个或更多**连续的**函数参数名称共享一个类型时，可以省略到最后一个类型如 `x int ,y int` 可以简写为 `x,y int`
- 函数可以返回任意个的结果，如下
```go
package main
import "fmt"
func swap(x, y string) (string, string) {
	return y, x
}
func main() {
	a, b := swap("hello", "world")
	fmt.Println(a, b)
}
```

###### Named return values （指定返回值）
- go的返回值可以被指定，返回值被像变量一样定义在函数的开头
- 这些名字应该被用来记录返回值的含义
- 不带参数的return语句返回指定的返回值
- 直接return的语句应该仅被用在短函数中，在长的函数里它会危害可读性
```go
func split(sum int) (x, y int) {
	x = sum * 4 / 9
	y = sum - x
	return
}
```
- **引用传递**是指调用函数时将**实际参数的地址**传入到函数中，那么在函数中对参数的修改，将影响到实际参数
- **值传递**是指在调用函数时将**实际参数**复制一份传递到函数中，这样在函数中如果对参数进行修改，将不会影响到实际参数

##### Variables
- var语句声明变量的清单，类型还是在后面
- var语句可以是包级别或是函数级别
- var语句的声明可以包含多个初始化，对每个变量的初始化
- 如果存在初始化器，则可以省略类型;变量将采用初始化器的类型。

```go
var name int
var name,test int
var name,test int = 1,2
var c, python, java = true, false, "no!"
```
###### Short variable declarations
- 在函数里，可以使用:= short赋值语句来代替带有隐式类型的var声明。
- 在函数外，每一个语句都一个关键字(var,func...)开头,因此:=结构不可用

###### 变量作用域
- 函数体内定义的变量称为局部变量，作用域只在函数体内，参数和返回值也是局部变量
- 函数外定义的变量称为全局变量，可以在整个包甚至外部包（被导出后）使用，
- 全局变量和局部变量相同时，优先使用局部变量

##### Basic types 基本类型
- bool
- string
- int int8 int16 int32 int64
- uint uint8 uint16 uint32 uint64 **uintpr**
- byte // uint8 别名
- rune // int32的别名，
- float32 float64
- **complex64 complex128**
- 像import语句一样，变量的声明也可用括号包裹
 ``` go
  var ( toBe bool = false
    maxInt uint64 = 1<<64-1
    z complex128 = cmplx.Sqrt(-5+12i)
  )
 ```
 - int ,uint,uintptr类型通常是在32操作系统32个字节，在64位操作系统64个字节
 - 当你要使用一个数值类型的值时，你应该选择int，除非有特殊原因用才有长度的或无符号的数值类型

 - golang的字符称为rune，等价于C中的char，可直接与整数转换，rune实际是整型，必需先将其转换为string才能打印出来，否则打印出来的是一个整数
 ```go
 c:='a'
 fmt.Println(c)
 fmt.Println(string(c))
 fmt.Println(string(97))
 ```

 ###### Zero values
 - 变量没有明确的初始化值，会被给赋予zero value
 - 数值类型的值  是0
 - 布尔类型的值 是false
 - 字符串类型的是"" 空字符串
 ```go
var i int
var f float64
var b bool
var s string
 ```

###### Type conversion
- 不像C语言，go里面不同类型的值之间的赋值，需要显式的转化
``` go
var i int = 42
var f float64 = float64(i)
var u uint = uint(f)
```

###### Type inference
- 当什么一个变量没有执行一个明确的类型（使用:=或者var = 表达式），变量的类型会从右边的表达式的值推断出
- 当 变量声明的右边被指定类型了，新的变量和它相同
- 当右边是没有指定类型的常量，会根据常量值的精度，选定什么类型
```go
i := 42           // int
f := 3.142        // float64
g := 0.867 + 0.5i // complex128
fmt.Printf("v is of type%T ") // 获取类型
```
##### Constants
- 常量像变量一样声明，但是用const关键字声明
- 可以是字符，字符串，布尔，数值类型
- 常量不能用:= 声明
```go
const World string = "世界"
```
###### Numeric Constants
- 数值类型常量是高精度的值
- 没有指定类型的常量根据它的内容来选定类型
- int类型可以存储最大64字节的数值，有时更少

```go
const(
    Big=1<<100
    Small = Big>>99
)
```
