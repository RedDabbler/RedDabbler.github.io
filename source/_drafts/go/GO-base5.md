[toc]
#### basics
##### Range
- 通过遍历一个slice或一个map，for循环形成Range
- 当遍历slice时，每个迭代返回2个值，第一个值是索引，第二个是这个索引值对应元素的拷贝
```go
var pow = []int{1, 2, 4, 8, 16, 32, 64, 128}
for i, v := range pow {
	fmt.Printf("2**%d = %d\n", i, v)
}
```
- 可以通过 _ 来省略索引或者对应的值
- 如果只想要索引，可以完全删掉",value"

```go
for i := range pow {
	pow[i] = 1 << uint(i) // == 2**i
}
for _, value := range pow {
	fmt.Printf("%d\n", value)
}
```
- Map遍历
```go
type IPAddr [4]byte
func main() {
	hosts := map[string]IPAddr{
		"loopback":  {127, 0, 0, 1},
		"googleDNS": {8, 8, 8, 8},
	}
	for name, ip := range hosts {
		fmt.Printf("%v: %v\n", name, ip)
	}
}
```

- 练习

 实现 Pic 。它返回一个长度为 dy 的 slice，其中每个元素是一个长度为 dx 且元素类型为8位无符号整数的 slice。当你运行这个程序时， 它会将每个整数作为对应像素的灰度值（好吧，其实是蓝度）并显示这个 slice 所对应的图像。灰度值自由选择，几个有意思的选择包括 (x+y)/2、x*y 和 x^y 。
```go
package main
import "golang.org/x/tour/pic"
func Pic(dx, dy int) [][]uint8 {
	a := make([][]uint8,dy)
	for x:= range a{
		b:= make([]uint8,dx)
		for y:= range b{
			b[y] = uint8((x+y))
		}
		a[x] = b
	}
	return a
}
func main() {
  pic.Show(Pic)
}
```
##### Maps
- map的zero value是nil，nil的map 没有key值，也没有key值可以被添加
- make函数可以返回一个给定类型的map，并且被初始化可被使用
```go
type Vertex struct {
	Lat, Long float64
}
var m map[string]Vertex;
m = make(map[string]Vertex)
m["Bell Labs"] = Vertex{40.68433, -74.39967,}
```
###### Map literals
- 和struct的字面量类似，但是map的key必须有
```go
var m = map[string]Vertex{
	"Bell Labs": Vertex{
		40.68433, -74.39967,
	},
	"Google": Vertex{
		37.42202, -122.08408,
	},
}
```
- 如果顶级类型是个类型名，那么可以从元素的字面量中省略
```go
var m = map[string]Vertex{
	"Bell Labs":{40.68433, -74.39967},
	"Google": {37.42202, -122.08408},
}
```
- **Vertex的逗号是什么鬼**
###### Mutating Maps
- 更新或者插入map的元素
  m[key]=elem
- 获取元素 elem = m[key]
- 删除元素 delete(m,key)
- 用2个值的指派方式测试key值是否存在 elem,ok = m[key]
  - 如果key在m中，ok值是true，如果不在，ok是false
  - 如果key不存在map中，elem的值是对应元素的zero value
  - 如果elem和ok没有被声明，可以用简短的声明格式： elem,ok:=m[key]

##### Function values
- 函数也是值，他们可以像其他值一样被传递
- 函数值可以被用作是函数的参数，和返回值
```go
func compute(fn func(float64, float64) float64) float64 {
	return fn(3, 4)
}

func main() {
	hypot := func(x, y float64) float64 {
		return math.Sqrt(x*x + y*y)
	}
	fmt.Println(hypot(5, 12))

	fmt.Println(compute(hypot))
	fmt.Println(compute(math.Pow))
}

```
###### closures
- 闭包是一个在函数体外部引用变量的函数值
- 函数可以访问和分配到引用的变量，在这种场景下，函数被绑定到变量上
- 下面函数返回一个闭包，每一个闭包绑定到它拥有的sum变量
```go
func adder() func(int) int {
	sum := 0
	return func(x int) int {
		sum += x
		return sum
	}
}
```
- [function_literals](https://golang.org/ref/spec#Function_literals)
