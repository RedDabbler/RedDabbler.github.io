[toc]

#### 常用包doc
- [strings.Field](https://golang.org/pkg/strings/#Fields)
- [gzip](https://golang.org/pkg/compress/gzip/#NewReader)
- [flag](https://golang.org/pkg/flag/)

#### package regexp
```go
var validPath = regexp.MustCompile("^/(edit|save|view)/([a-zA-Z0-9]+)$")
```
- regexp.MustCompile 会解析和编译正则表达式，返回regexp.Regexp
- MustCompile 和Compile 有很大不同，MustCompile将会panic当表达式失败，但是Compile会返回一个error作为第二个参数

#### package flag
- 命令行参数解析，是标准库的一个包
##### 定义 flags 有两种方式
- flag.Xxx()，其中 Xxx 可以是 Int、String 等；返回一个相应类型的指针，如：
``` go
var ip = flag.Int("flagname", 1234, "help message for flagname")
```
- flag.XxxVar()，将 flag 绑定到一个变量上，如：
```go
var flagvar int
flag.IntVar(&flagvar, "flagname", 1234, "help message for flagname")
```
##### 解析 flag
- 在所有的 flag 定义完成之后，可以通过调用 flag.Parse() 进行解析。
- 命令行 flag 的语法有如下三种形式：
  ```shell
   -flag // 只支持bool类型
   -flag=x
   -flag x // 只支持非bool类型
  ```
- Usage：这是一个函数，用于输出所有定义了的命令行参数和帮助信息（usage message）。一般，当命令行参数解析出错时，该函数会被调用。我们可以指定自己的 Usage 函数，即：flag.Usage = func(){}


#### [package http](https://godoc.org/net/http)
- http 包提供Http client 和Sever的实现
- get，head，post，postform方法调用http请求
##### client
- client 必须关闭reponse body，当完成请求后，如下
``` go
resp, err := http.Get("http://example.com/")
if err != nil {
	// handle error
}
defer resp.Body.Close()
body, err := ioutil.ReadAll(resp.Body)
```
- 控制HTTP客户端头、重定向策略和其他设置，创建client客户端
- 控制代理，TLS配置，keep-alives, compression, and other settings, 创建Transport（传输）
- 客户端和传输对于多个goroutines的并发使用是安全的，并且为了提高效率，应该只创建一次并重新使用。
##### server
- ListenAndServe使用给定的地址和handler(处理程序)启动HTTP服务器。处理程序(handler)通常为nil，这意味着使用DefaultServeMux。Handle和HandleFunc向DefaultServeMux添加处理程序
```go
http.Handle("/foo", fooHandler)
http.HandleFunc("/bar", func(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello, %q", html.EscapeString(r.URL.Path))
})
log.Fatal(http.ListenAndServe(":8080", nil))
```
- 为了更多控制服务的行为，可通过创建一个自定义服务
```go
s := &http.Server{
	Addr:           ":8080",
	Handler:        myHandler,
	ReadTimeout:    10 * time.Second,
	WriteTimeout:   10 * time.Second,
	MaxHeaderBytes: 1 << 20,
}
log.Fatal(s.ListenAndServe())
```
