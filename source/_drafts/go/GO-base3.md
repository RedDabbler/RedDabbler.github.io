[toc]
#### basics
##### for
- go只有一个循环的结构，for循环
- for循环有三部分组成，有分号分割
  - 初始化部分：在第一次循环之前执行
  - 条件表达式：在每次循环之前比较
  - 最后的表达式：每次循环之后执行
- 初始化语句通常是一个剪短的变量声明，声明的这些变量尽在for循环语句中存在
- 一旦条件表达式返回false，循环将会停止
- 不像其他语言比如C，java，js，它没有圆括号包裹for语句，而且一定要有{}
- 初始化语句和最后的表达式是可选的
```go
sum := 1
for i := 0; i < count; i++ {

}
for ; sum < 1000; {
		sum += sum
	}
```
- 可以删除分号，c的while在go里面是for
``` go
for sum<1000{
 //
}
```
- 死循环,省略条件
``` go
for{

}
```

##### IF
- 像for循环一样，if语句不需要()包裹，{}不能省略
```go
if x>0 {
    //
}
```
- 像for一样，if语句可以在条件之前以一个短语句为开始执行
```go
if x=0;x>0 {

}
```
- 变量的类型仅仅在if的范围内(包含else语句)可见
- var 声明不允许出现在for定义里，如下是不合法的，if语句也是
``` go
for var i = 0; i < count; i++ { //
}
```
##### Switch
- switch语句是if-else语句的简写
- go只运行选中的用例，不会运行接下来的用例
- break语句在go语言**自动被提供**在每个用例的结尾
- switch用例**不必一定是常量**，包含的值不必是一定是数值
```go
switch os := runtime.GOOS; os {
	case "darwin":
		fmt.Println("OS X.")
	case "linux":
		fmt.Println("Linux.")
	default:
		// freebsd, openbsd,
		// plan9, windows...
		fmt.Printf("%s.", os)
    }
```

- 用例比较按照从开头到结尾的顺序，当碰到用例时结束
- 没有条件表达式的switch语句和switch true一样,这种是一个简洁的方式来写if-else语句
```go
switch {
	case t.Hour() < 12:
		fmt.Println("Good morning!")
	case t.Hour() < 17:
		fmt.Println("Good afternoon.")
	default:
		fmt.Println("Good evening.")
	}
```

##### Defer
- defer语句推迟函数的执行到包含的函数**返回才执行**
```go
// 结果： hello\n world
func main() {
	defer fmt.Println("world")
	fmt.Println("hello")
}
```
- 推迟的调用参数立刻被赋值，但是函数的调用直到包含它的函数返回了才执行
- 多个推迟函数的调用时，推迟的函数调用被push到一个栈里
- 当函数返回，它推迟的调用会以**后进先出**的顺序执行
```go
for i := 0; i < 10; i++ {
	defer fmt.Println(i)
}
fmt.Println("done")
```
- defer **[blog](https://blog.golang.org/defer-panic-and-recover)**
