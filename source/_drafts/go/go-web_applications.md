[toc]
#### 最简单最简单的web程序
```go
import (
    "fmt"
    "log"
    "net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hi there, I love %s!", r.URL.Path[1:])
}

func main() {
    http.HandleFunc("/", handler)
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```
- http.HandleFunc 函数告诉http包用handler函数处理所有请求到根路径`/`的请求
- `handler`方法的类型是`http.HandlerFunc`，它包含`http.ResponseWriter`和`http.Request`参数
  - http.ResponseWriter 封装http服务器的响应，通过写入到它，我们发送数据到HTTP客户端
  - http.Request 是一个表示http客户端请求得数据接口 r.URL.Path是请求url的路径部分，[1:]表示`创建从第一个字符到末尾的路径的子片。`,它会从路径名称里删掉/,因此输入`http://localhost:8080/money/test`，会得到`Hi there, I love money/test!`
- `http.ListenAndServe` 指定程序监听端口在8080，这个函数会一直阻塞等待直到整个程序结束，它总是返回error,因为只有当有error发生时，它才返回
- 为了记录上面的错误，我们用`log.Fatal`函数来包装它

#### 添加处理方法
- 添加新的handler为viewHandler，可以截取/view
```go
func viewHandler(w http.ResponseWriter, r *http.Request) {
    title := r.URL.Path[len("/view/"):]
    p, _ := loadPage(title)
    fmt.Fprintf(w, "<h1>%s</h1><div>%s</div>", p.Title, p.Body)
}
```
- main方法里可以添加下面代码，可以用来处理/view/路径对应的请求
```go
http.HandleFunc("/view/", viewHandler)
```

##### 保存页面
- save页面后根据url的路径重定向到view页面
```go
func saveHandler(w http.ResponseWriter, r *http.Request) {
    title := r.URL.Path[len("/save/"):]
    body := r.FormValue("body")
    p := &Page{Title: title, Body: []byte(body)}
    p.save()
    http.Redirect(w, r, "/view/"+title, http.StatusFound)
}
```
#### html/template 包
- 它属于go标准库，我们可以使用html/template来保存html在一个独立的文件，允许我们只修改编辑页面的布局而不用修改底层代码
- 如创建 edit.html
```html
<h1>Editing {{.Title}}</h1>
<form action="/save/{{.Title}}" method="POST">
<div>
<textarea name="body" rows="20" cols="80">
{{printf "%s" .Body}}
</textarea>
</div>
<div><input type="submit" value="Save"></div>
</form>
```
- go 底层
```go
func editHandler(w http.ResponseWriter, r *http.Request) {
    title := r.URL.Path[len("/edit/"):]
    p, err := loadPage(title)
    if err != nil {
        p = &Page{Title: title}
    }
    t, _ := template.ParseFiles("edit.html")
    t.Execute(w, p)
}
```
- t.Execute 执行模板，并写生成的html到`http.ResponseWriter`,`.Title`表示对应着p.Title

##### 模板指令
- 模板指令用**双花括号**括起来。`printf "%s" .Body`是一个函数调用，它将.Body输出为一个**字符串，而不是一个字节流**，这与对fmt.Printf的调用相同。
- html/template包有助于确保模板操作只**生成安全且正确的html**。例如，它自动转义任何大于号(>)，用`&gt;`替换它，以确保用户数据不会损坏表单HTML。

##### 处理不存在页面
- 如果你访问 /view/APageThatDoesntExist，如下代码，您将看到一个包含HTML的页面,**忽略返回值的错误**并继续尝试用空数据填充模板
```go
func renderTemplate(w http.ResponseWriter, tmpl string, p *Page) {
    t, _ := template.ParseFiles(tmpl + ".html")
    t.Execute(w, p)
}
```
- 相反，如果返回的页面不存在，它将重定向客户端到编辑页面，以便创建内容，如下，`http.Redirect`添加302状态码和地址header到响应

```go

func viewHandler(w http.ResponseWriter, r *http.Request) {
    title := r.URL.Path[len("/view/"):]
    p, err := loadPage(title)
    if err != nil {
        http.Redirect(w, r, "/edit/"+title, http.StatusFound)
        return
    }
    renderTemplate(w, "view", p)
}
```

##### 错误处理
- 上面的代码有几处是当出现异常时，忽略错误，这不是一个好的实践
- 应该处理错误并返回错误信息给用户。服务器将按照我们想要的方式运行，并且可以通知用户。
```go
func renderTemplate(w http.ResponseWriter, tmpl string, p *Page) {
    t, err := template.ParseFiles(tmpl + ".html")
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    err = t.Execute(w, p)
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
    }
}
```
- http.Error 函数发送一个特殊的状态码和错误信息

##### 模板缓存
- 一个更好的方式是当服务启动时调用ParseFiles，解析所有的模板为单个*模板
- 然后使用[ExecuteTemplate](https://golang.org/pkg/html/template/#Template.ExecuteTemplate)方法着色特定的模板
  ``` go
  var templates = template.Must(template.ParseFiles("edit.html", "view.html"))
  err := templates.ExecuteTemplate(w, tmpl+".html", p)
  ```
  - template.Must是一个方便的包装，当传递一个非nil错误值时panic，否则返回的*模板未被修改
  - 恐慌在这里是适当的;如果不能加载模板，惟一明智的做法是退出程序
  - ParseFiles函数接受任何数量的字符串参数来标识模板文件，并将这些文件解析为以**基本文件名**命名的模板。如果我们要向程序添加更多的模板，我们将把它们的名称添加到ParseFiles调用的参数中。

##### validation
- 可用regexp包对路径验证
##### function_literals and Closures
- 在上面的hanlder有很多重复的代码，比如路径的验证等
- 对于上面的viewHandler,editHandler,saveHandler,我们可以定义一个封装的方法，它返回`http.HandlerFunc`函数类型,(**适合传递到函数`http.HandleFunc`**)
```go
func makeHandler(fn func (http.ResponseWriter, *http.Request, string)) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		// Here we will extract the page title from the Request,
		// and call the provided handler 'fn'
	}
}
```
- 返回的的函数被称为闭包，因为它**将定义在它外面的值封装起来**。
- **fn变量被这个闭包封装**，fn变量可以是save, edit, 或者 view handler
- 因此下面的代码，可以首先验证路径，当通过时，才会执行fn函数
```go
func makeHandler(fn func(http.ResponseWriter, *http.Request, string)) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        m := validPath.FindStringSubmatch(r.URL.Path)
        if m == nil {
            http.NotFound(w, r)
            return
        }
        fn(w, r, m[2])
    }
}
func main() {
    http.HandleFunc("/view/", makeHandler(viewHandler))
    http.HandleFunc("/edit/", makeHandler(editHandler))
    http.HandleFunc("/save/", makeHandler(saveHandler))

    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

#### 参考文档
- [doc](https://golang.org/doc/articles/wiki/)
- [talks](https://talks.golang.org/2012/simple.slide#2)
- [最终代码](https://golang.org/doc/articles/wiki/final.go)
