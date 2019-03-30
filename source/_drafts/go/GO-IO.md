##### Reader
- io包定义了io.Reader接口
- 这个表示数据流的读端
- [doc](https://golang.org/pkg/io/#Reader)
- go的标准库里包含这些接口的实现，包含files，network connections，compressors,ciphers等等
- Reader接口有一个Read方法
 ```go
 func (T) Read(b []byte) (n int, err error)
 ```
- Read 给定byte的slice ，返回bytes的数量和错误值，当流结束，函数返回io.EOF错误
```go
import (
	"fmt"
	"io"
	"strings"
)
func main() {
	r := strings.NewReader("Hello, Reader!")

	b := make([]byte, 8)
	for {
		n, err := r.Read(b)
		fmt.Printf("n = %v err = %v b = %v\n", n, err, b)
		fmt.Printf("b[:n] = %q\n", b[:n])
		if err == io.EOF {
			break
		}
	}
}
```

- 常见的模式是 io.Reader 包装另一个io.Reader，以某种方式修改流
- 例如，gzip.NewReader函数接受io.Reader(压缩数据流)并返回 gzip.Reader,这个Reader也实现了io.Reader(解压的数据流)
- 实践

##### package bufio
- 包bufio实现缓冲I/O，它包装了 io.Reader 和 io.Writer 对象，创建另一个对象（Reader or Writer）也实现了接口,为文本I/O提供缓冲和一些帮助。
- 初始化NewReader，`func NewReader(rd io.Reader) *Reader`
  - 返回并创建一个Reader,buffer的长度默认，(这个Reader定义在bufio包下，是io.Reader对象的实现，具体可看源码)
- ReadLine方法读取一行，很多调用者使用ReadBytes('\n') or ReadString('\n') ,如果一行太长，isPrefix会被设置，行的开头被返回，剩余的部分会在未来的调用中返回，当行的最后部分被返回时，isPrefix是false。返回的缓冲区仅在下次调用ReadLine之前有效。ReadLine要么返回一个非空行，要么返回一个错误。

##### package ioutil
- ReadFile
   - 根据文件名称读取文件，返回文件内容，正常的调用是返回的err==nil,不是EOF

##### os.File
- 在 Go 语言中，文件使用指向 os.File 类型的指针来表示的，也叫做文件句柄。
