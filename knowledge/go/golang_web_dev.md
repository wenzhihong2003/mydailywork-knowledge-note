Golang Web开发
============
从 http://lanlingzi.cn/post/technical/2016/0515_go_web/ 来 


###  标准库[net/http]

采用Golang来开发Web应用或Rest接口的应用还是比较容易的。golang标准库就提供对Http协议的封装，主要涉及到net/http包，它包括了HTTP相关的各种函数、类型、变量等标识符。标准库的net/http是支持HTTP1.1协议，而目前Go1.6也支持HTTP2.0，包放在golang.org/x/net/http2,后续可能会移到标准库。

net/http库中主要涉及到如下几个类型与接口：

#### 1.1 Request结构体

封装了HTTP的请求消息，其结构如下，可以很方便的地取出Method，Header与Body。

```go
  type Request struct {
      Method string
      URL *url.URL
      Proto      string
      ProtoMajor int
      ProtoMinor int
      Header Header
      Body io.ReadCloser
      ContentLength int64
      TransferEncoding []string
      Close bool
      Host string
      Form url.Values
      PostForm url.Values
      MultipartForm *multipart.Form
      Trailer Header
      RemoteAddr string
      RequestURI string
      TLS *tls.ConnectionState
      Cancel <-chan struct{}
  }
```

#### 1.2 Response结构体

封装HTTP的响应消息，其结构如下，Response会关联Request。
```go
  type Response struct {
    Status     string
    StatusCode int
    Proto      string
    ProtoMajor int
    ProtoMinor int
    Header Header
    Body io.ReadCloser
    ContentLength int64
    TransferEncoding []string
    Close bool
    Trailer Header
    Request *Request
    TLS *tls.ConnectionState
 }
```

#### 1.3 Handler接口

用于构建Response。应用开发就编写各种实现该Handler接口的类型，并在该类型的ServeHTTP方法中编写服务器响应逻辑。

```go
 type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
 }
```

#### 1.4 ResponseWriter接口

即应用通过各种Handler操作ResponseWriter接口来构建Response。ResponseWriter实现了io.Writer接口，可以写入响应的Body，WriteHeader方法用于向HTTP响应信息写入状态码，但必须先于Writer方法调用。若不调用WriteHeader，使用Write方法会自动写入状态码http.StatusOK。

```go
 type ResponseWriter interface {
    Header() Header
    Write([]byte) (int, error)
    WriteHeader(int)
 }
```

#### 1.5 ListenAndServe函数

启动HTTP服务，需要构建Server对象，并调用该Server的ListenAndServe方法，Server是HTTP服务的主控器。期结构定义如下，应用可以设置HTTP监听的地址，配置TLS，以及一些其它参数配置。

```go
type Server struct {
    Addr           string        // TCP address to listen on, ":http" if empty
    Handler        Handler       // handler to invoke, http.DefaultServeMux if nil
    ReadTimeout    time.Duration // maximum duration before timing out read of the request
    WriteTimeout   time.Duration // maximum duration before timing out write of the response
    MaxHeaderBytes int           // maximum size of request headers, DefaultMaxHeaderBytes if 0
    TLSConfig      *tls.Config   // optional TLS config, used by ListenAndServeTLS
    TLSNextProto map[string]func(*Server, *tls.Conn, Handler)
    ConnState func(net.Conn, ConnState)
    ErrorLog *log.Logger
    disableKeepAlives int32     // accessed atomically.
    nextProtoOnce     sync.Once // guards initialization of TLSNextProto in Serve
    nextProtoErr      error
}
```
Server需要关注如下几个方法，从方法名就可能知道它的用途。

```go
func (srv *Server) ListenAndServe() error
func (srv *Server) Serve(l net.Listener) error
```

#### 1.6 ServeMux结构体

用于HTTP路由配置，其结构体定义如下：

```go
type ServeMux struct {
    mu    sync.RWMutex
    m     map[string]muxEntry
    hosts bool // whether any patterns contain hostnames
}
```
ServeMux有如下几个方法，用于配置HTTP与URL的映射关系。其实ServeMux也是实现ServeHTTP接口，其ServeHTTP方法完成了ServeMux的主要功能，即根据HTTP请求找出最佳匹配的Handler并执行之，它本身就是一个多Handler封装器，是各个Handler执行的总入口。

```go
func (mux *ServeMux) Handle(pattern string, handler Handler)
func (mux *ServeMux) HandleFunc(pattern string, handler func(ResponseWriter, *Request))
func (mux *ServeMux) Handler(r *Request) (h Handler, pattern string)
func (mux *ServeMux) ServeHTTP(w ResponseWriter, r *Request)
```

ServeMux的路由功能是非常简单的，其只支持路径匹配，且匹配能力不强，也不支持对Method的匹配。net/http包已经为我们定义了一个可导出的ServeMux类型的变量DefaultServeMux。net/http包也提供了注册Handler的方法，它其实也是操作DefaultServeMux：

调用http.Handle或http.HandleFunc实际上就是在调用DefaultServeMux对应的方法。
若ListenAndServe的第二个参数为nil，它也默认使用DefaultServeMux.
