---
title: Golang 的接口型函数
author: daz
pubDatetime: 2023-05-06T02:15:08Z
featured: false
draft: false
tags:
  - Golang
  - ServeHTTP
ogImage: ""
description: "go's interface function"
---

![](https://img-blog.csdnimg.cn/img_convert/54e60afdf2764a07539da3136f3ce3e4.png)
本作品采用[知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议](https://creativecommons.org/licenses/by-nc-sa/4.0/)进行许可。

---

# Golang 的`接口型函数`

### demo

##### **接口型函数的定义**

```go
// A Getter loads data for a key.
// 接口
type Getter interface {
	Get(key string) ([]byte, error)
}

// A GetterFunc implements Getter with a function.
// 接口型函数
type GetterFunc func(key string) ([]byte, error)

// Get implements Getter interface function
// 接口型函数实现的方法
func (f GetterFunc) Get(key string) ([]byte, error) {
	return f(key)
}
```

- 定义了一个接口 Getter
  - 包含了一个方法 Get
- 定义了一个函数类型 GetterFunc
  - 参数与返回值 与 Getter 中 Get 方法一致
  - 定义了 Get 方式,并在 Get 中调用自己: 实现了接口 `Getter`
  - 所以 GetterFunc 是一个接口的函数类型,简称为 `接口型函数`

##### **实现接口示例**

- 使用场景: GetFromSource 从某数据源获取结果,接口 Getter 是其中一个参数

```go
func GetFromSource(getter Getter, key string) []byte {
	buf, err := getter.Get(key)
	if err == nil {
		return buf
	}
	return nil
}
```

**注意,此时 getter.Get() 尚未实现,我们可以通过以下方法实现**

- `通过匿名函数实现 Getter 接口`

```go
GetFromSource(GetterFunc(func(key string) ([]byte, error) {
	return []byte(key), nil
}), "hello")
```

- 通过普通函数实现 Getter 接口

```go
func test(key string) ([]byte, error) {
	return []byte(key), nil
}

func main() {
    GetFromSource(GetterFunc(test), "hello")
}
```

- 通过结构体实现 Getter 接口

```go
type MyGetter struct{}

func (g MyGetter) Get(key string) ([]byte, error) {
    // 实现获取数据的逻辑
    return []byte(key), nil
}

func main() {
    getter := MyGetter{}
    data := GetFromSource(getter, "hello")
    fmt.Println(string(data))
}
```

- 通过接口嵌套实现 Getter 接口

```go
type Getter interface {
    Get(key string) ([]byte, error)
}

type MyGetter struct {
    Getter
}

func (g *MyGetter) Get(key string) ([]byte, error) {
    // 实现获取数据的逻辑
    return []byte(key), nil
}

func main() {
    getter := &MyGetter{}
    data := GetFromSource(getter, "hello")
    fmt.Println(string(data))
}
```

### 价值

- 只要实现了接口的任何类型（除了 interface 类型以外）都可以作为参数传递给函数使用
  - 既能传入函数作为参数,也能传入实现了该接口的结构体作为参数...
- 定义一个函数类型 F，并且实现接口 A 的方法，然后在这个方法中调用自己。这是 Go 语言中将其他函数（参数返回值定义与 F 一致）转换为接口 A 的常用技巧

### 使用场景

- 广泛使用
- 包括`标准库`
  - 经典例子为 net/http 下的 Handler 和 HandlerFunc

##### **标准库 net/http/server.go 中**

- Handler 的定义

```go
// 接口类型
type Handler interface {
	ServeHTTP(ResponseWriter, *Request)
}

// 接口型函数
type HandlerFunc func(ResponseWriter, *Request)

func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
	f(w, r)
}
```

- http.Handle: 映射请求路径及其处理函数
  - Handle 的定义如下

```go
func Handle(pattern string, handler Handler)
```

- 其第二个参数类型即接口 Handler,用法可以如下

```go
func home(w http.ResponseWriter, r *http.Request) {
	w.WriteHeader(http.StatusOK)
	_, _ = w.Write([]byte("hello, index page"))
}

func main() {
	http.Handle("/home", http.HandlerFunc(home))
	_ = http.ListenAndServe("localhost:8000", nil)
}
```

--

- 通常,还会使用另外一个函数 http.HandlerFunc
  - 其定义如下

```go
func HandleFunc(pattern string, handler func(ResponseWriter, *Request))
```

- 其第二个参数为普通函数类型,可直接将 home 传给 HandleFunc

```go
func main() {
	http.HandleFunc("/home", home)
  /*
    ListenAndServer 的第二个接口也是接口类型
    传入 nil 为默认路由
    传入 实现好的[Handler接口] 就可以自定义路由
  */
	_ = http.ListenAndServe("localhost:8000", nil)
}
```

- 由 HandleFunc 内部实现可知,两种写法其实`等价`

```go
func (mux *ServeMux) HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
	if handler == nil {
		panic("http: nil handler")
	}
	mux.Handle(pattern, HandlerFunc(handler))
}
```

---

- 除此之外,想要理解 ServeHTTP 当然最好参考经典的 `类echo` 服务器

```go
package main

import (
	"log"
	"net/http"
)

type server int

func (h *server) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	log.Println(r.URL.Path)
	w.Write([]byte("Hello World!"))
}

func main() {
	var s server
	http.ListenAndServe("localhost:9999", &s)
}
```

- 此处再重复一次 Handler 的定义

```go
// 接口类型
type Handler interface {
	ServeHTTP(ResponseWriter, *Request)
}

// 接口型函数
type HandlerFunc func(ResponseWriter, *Request)

func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
	f(w, r)
}
```

- **详细解释**:
- 定义
  - Handler 是接口类型,定义了一个 ServeHTTP 方法
  - HandlerFunc 是一个函数类型,接受两个参数
  - 为了让 HandlerFunc 类型`实现 Handler`接口,我们需要定义一个 ServeHTTP 方法,其已经被定义为`f(w, r)`
    - 其功能很简单:只是`调用函数 f 并传入相同参数`
    - `任何 HandlerFunc 类型的函数都可以作为 Handler 接口使用`
- 实现
  - server 类型实现了 Handler 接口
    - 因为其定义了 ServeHTTP 方法
    - 意味着可以使用 server 类型的变量作为一个 Handler
    - &s 是一个指向 s 的指针，因为在 Go 中可以使用指针接收者来实现接口。

---

参考:

- [动手写分布式缓存](https://geektutu.com/post/geecache-day2.html)
- [Go 接口型函数的使用场景](https://geektutu.com/post/7days-golang-q1.html)
