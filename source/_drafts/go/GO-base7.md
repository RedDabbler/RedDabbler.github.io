[toc]
#### Methods
- go没有类，然而，可以**在类型上定义方法**
- 方法是带有特殊接受器参数的函数
- 接受者出现在func关键字和方法名字之间的参数列表中
```go
type Vertex struct {
	X, Y float64
}
func (v Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}
func main() {
	v := Vertex{3, 4}
	fmt.Println(v.Abs())
}
```
##### Methods are functions
- 方法是带有特殊接受器参数的函数
```go
func Abs(v Vertex) float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}
func main() {
	v := Vertex{3, 4}
	fmt.Println(Abs(v))
}
```
- 上面的abs写做为常规的函数，函数体没有任何改变
- **也可以声明方法在非结构类型上,如下面的MyFloat类型是个非结构类型**
- **只能声明 类型被定义在同一个包接受器的方法，不能声明接收器类型定义在另一个包的方法**
```go
type MyFloat float64

func (f MyFloat) Abs() float64 {
	if f < 0 {
		return float64(-f)
	}
	return float64(f)
}

func main() {
	f := MyFloat(-math.Sqrt2)
	fmt.Println(f.Abs())
}
```
##### Pointer receivers
- 可以声明方法在指针的接收器上，接收者类型具有某种类型T的文字语法*T
- 但是**T自己不能是一个指针比如*int**
- 带指针类型的接收器的方法可以修改接收器指向的值。因此，当方法需要修改他们的接收器时，指针接收器比值接收器更常用
- 值接收器不会修改接收器的值
- 值接收器时，方法操作的是**值操作器的拷贝**
- 函数的参数和上面的接收器类似，用指针类型来修改参数的值，非指针类型的参数，函数操作的是**参数值的拷贝**

```go
func (v *Vertex) Scale(f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}
func ScaleFunc(v *Vertex, f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}
```
##### Methods and pointer indirection
- 上面的2个程序，可能会注意到，带指针类型参数的函数必须采用指针,如下
``` go
var v Vertex
ScaleFunc(v, 5)  // Compile error!
ScaleFunc(&v, 5) // OK
```
- 当带有指针接收器的方法被调用时，可以采用值类型,也可用指针类型
```go
var v Vertex
v.Scale(5)  // OK
p := &v
p.Scale(10) // OK
```
- 上面的v是一个值而不是一个指针，带指针接收器的方法`(v.Scale(5))`仍然可以被调用，这是因为go将自动解析`v.Scale(5)` 为`(&v).Scale(5)`


- 另一个例子，如下程序
```go
func (v Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func AbsFunc(v Vertex) float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}
```
- 带值参数的函数必须采用指定类型的值
```go
var v Vertex
fmt.Println(AbsFunc(v))  // OK
fmt.Println(AbsFunc(&v)) // Compile error!
```
- 当带有值接收器的方法被调用时，也是可以采用值或指针
```go
var v Vertex
fmt.Println(v.Abs()) // OK
p := &v
fmt.Println(p.Abs()) // OK
```
- 与上面一样道理，p.Abs()方法的调用被解析为(*p).Abs()


###### choosing a value or pointer receiver
- 要使用指针接收器有2个原因
  - 方法可以修改接收器指向的值
  - 避免每个方法都拷贝值，比如，如果接收器是一个大的结构体时，这种方式更有效率
- 比如下面的方法，即使Abs方法没有修改它的接收器，Scale和Abs方法仍然采用指针接收器
  ```go
  func (v *Vertex) Scale(f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
  }

  func (v *Vertex) Abs() float64 {
    return math.Sqrt(v.X*v.X + v.Y*v.Y)
  }
  ```
- 总之，对于所有的方法来说，接收器类型要么是值，要么是指针，不会是2者的混合
