[toc]
#### interface
- **接口类型**被定义为方法签名的集合
- 接口类型的值可以持有任何实现这些方法的值
```go
type Abser interface {
	Abs() float64
}
type MyFloat float64

func (f MyFloat) Abs() float64 {
	if f < 0 {
		return float64(-f)
	}
	return float64(f)
}

type Vertex struct {
	X, Y float64
}

func (v *Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

```
- 如上代码，MyFloat类型实现了Abs方法，即Abs方法定义在MyFloat类型,因此
```go
var a Abser
f := MyFloat(-math.Sqrt2)
a = f  // a MyFloat implements Abser
v := Vertex{3, 4}
a = &v //a *Vertex implements Abser
```
- 不能 赋值a =v ,因为Vertex没有实现Abser方法

##### Interfaces are implemented implicitly
- 类型实现接口的方式是通过是实现它的方法，但是没有明确的声明比如implements关键字，来表明这个意图，如上面的MyFloat类型和Vertex类型
- 隐式的接口分离了它的定义和实现
- 实现可以出现在任何没有预先安排的的包中
- 注意下面的main方法里的参数接受类型是I，而不是M
```go
type I interface {
	M()
}

type T struct {
	S string
}
// This method means type T implements the interface I,
// but we don't need to explicitly declare that it does so.
func (t T) M() {
	fmt.Println(t.S)
}

func main() {
	var i I = T{"hello"} // 用I类型来持有
	i.M()
}
```
##### Interface values
- 接口值可被认为是一个值和一个具体类型的元组： (value,type),如下面的例子
- 接口值包含特定底层具体类型的值。
- 在接口值上调用一个方法，在其底层类型上执行同名的方法。
``` go
func describe(i I) {
	fmt.Printf("(%v, %T)\n", i, i)
}
```
##### Interface values with nil underlying values
- 当接口自身包含的值是nil,对应的方法将会被nil 接口器调用
- 在某些语言里，这将会触发空指针异常，但在go里，编写方法优雅的处理使用nil接收者调用的问题是很常见的，像下面的M方法一样
```go
func (t *T) M() {
	if t == nil {
		fmt.Println("<nil>")
		return
	}
	fmt.Println(t.S)
}
```
- 注意：**包含nil具体值的接口值本身不是nil**,比如下面的i的第一次赋值为nil
``` go
func main() {
	var i I
	var t *T
	i = t
	describe(i)
	i.M()
	i = &T{"hello"}
	describe(i)
	i.M()
}
```
##### Nil interface values
- nil 接口值即不包含值，也不包含具体的类型
- 在nil接口上调用方法是一个运行时error，如下(i.M()调用)因为在接口的元组里没有类型表明哪一个具体的方法会被调用
```go
var i I
describe(i)
i.M()
```
##### empty interface
- 空接口是指没有指定方法的接口
- 空接口可能包含任何类型的值，因为每个类型都最少实现了zero methods
```go
var i interface{}
describe(i)
i = 42
describe(i)
i = "hello"
describe(i)
```
- 空接口被用来编码处理未知类型的值
- 例如，fmt.Print 接受任意数量的接口类型{}的的参数
```go
 func Print(a ...interface{}) (n int, err error)
```
#### Type assertions
- assertions断言提供方式访问接口值的底层具体值
```go
 t:=i.(T)
```
- 上面的语句断言接口值i包含具体类型T和分配底层T值给变量t
- 如果i不包含T，那么这个语句将触发一个恐慌（panic）
- 为了测试接口值是否包含指定的类型，类型的断言可以返回2个值，底层的值和一个布尔值，表明是否断言成功
  ```go
   t, ok := i.(T)
  ```
   - 如果i含有T，t的值就是底层的值，ok是true
   - 如果不是，ok是false，t是T类型的zero value，不会触发恐慌

- 注意和读取map值的语法的相似点
```go
s := i.(string)
fmt.Println(s)
s, ok := i.(string)
fmt.Println(s, ok)
```


#### Type switches
- switch类型是一个允许在系列中使用多个类型断言的构造。
- 类型选择像是一个switch语句，但是在一个类型转换的用例指定类型（不是值),这些值和和**给定接口值包含的类型**进行比较
```go
switch v := i.(type) {
case T:
    // here v has type T
case S:
    // here v has type S
default:
    // no match; here v has the same type as i
}
```
- 类型选择的声明和类型断言的声明相同，但是断言指定的类型被替换为关键字type,`i.(type)`
- switch 语句测试是否接口值i包含T类型或S
- 在T和S的每个用例，变量v将分别为T或S类型，并保存i所持有的值。
- 在默认的用例（没有匹配）中，变量v和i的接口类型和值相同


#### Stringers
- 在fmt包中定义的最常用的接口是Stringer，接口定义如下
```go
type Stringer interface {
	String() string
}
```
- Stringer是一个用来描述它自己是个string的接口
- fmt包(和许多其他包)寻找这个接口来打印值。如下Sprintf方法
```go
func (p Person) String() string {
	return fmt.Sprintf("%v (%v years)", p.Name, p.Age)
}

func main() {
	a := Person{"Arthur Dent", 42}
	z := Person{"Zaphod Beeblebrox", 9001}
	fmt.Println(a, z)
}
```
#### Errors
- go表示error状态用error值
- error接口类型和fmt.Stringer的构造类似
``` go
type error interface {
    Error() string
}
```
- 和fmt.Stringer 一样，fmt包在打印值时查找error接口
- 函数通常返回一个error值，调用函数的代码应该处理错误来测试是否error值是nil
```go
i, err := strconv.Atoi("42")
if err != nil {
    fmt.Printf("couldn't convert number: %v\n", err)
    return
}
fmt.Println("Converted integer:", i)
```
- nil的错误表示成功，不是nil的错误表示失败
```go
type MyError struct {
	When time.Time
	What string
}

func (e *MyError) Error() string {
	return fmt.Sprintf("at %v, %s",
		e.When, e.What)
}

func run() error {
	return &MyError{
		time.Now(),
		"it didn't work",
	}
}

func main() {
	if err := run(); err != nil {
		fmt.Println(err)
	}
}
```
>  body, _ := ioutil.ReadFile(filename) 用下划线(_)符号表示的“空白标识符”用来丢弃错误返回值(本质上，将值赋给none)。
