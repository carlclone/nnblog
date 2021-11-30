# go 源码阅读技巧

从一门课程学来的阅读技巧, 感觉很实用记录一下

- 先确定库提供的功能 (对外暴露的接口)

`go doc net/http| grep "^func"`

先忽略 New 和 Set 开头的, 然后关注我们想看的部分,创建 HTTP 服务(包含 Serve 字样的)

假如某个库没有 go doc 的话, 可以用 goland 的正则搜索功能

`^func (?=[A-Z])(?=(?!Test))(?=(?!Example))(?=(?!Benchmark))(?=(?!New))(?=(?!Set))`

`^func (\(.*\))? (?=([A-Z]))(?=(?!Test))(?=(?!Example))(?=(?!Benchmark))(?=(?!New))(?=(?!Set))`

```
func CanonicalHeaderKey(s string) string
func DetectContentType(data []byte) string
func Error(w ResponseWriter, error string, code int)
func Get(url string) (resp *Response, err error)
func Handle(pattern string, handler Handler)
func HandleFunc(pattern string, handler func(ResponseWriter, *Request))
func Head(url string) (resp *Response, err error)
func ListenAndServe(addr string, handler Handler) error
func ListenAndServeTLS(addr, certFile, keyFile string, handler Handler) error
func MaxBytesReader(w ResponseWriter, r io.ReadCloser, n int64) io.ReadCloser
func NewRequest(method, url string, body io.Reader) (*Request, error)
func NewRequestWithContext(ctx context.Context, method, url string, body io.Reader) (*Request, error)
func NotFound(w ResponseWriter, r *Request)
func ParseHTTPVersion(vers string) (major, minor int, ok bool)
func ParseTime(text string) (t time.Time, err error)
func Post(url, contentType string, body io.Reader) (resp *Response, err error)
func PostForm(url string, data url.Values) (resp *Response, err error)
func ProxyFromEnvironment(req *Request) (*url.URL, error)
func ProxyURL(fixedURL *url.URL) func(*Request) (*url.URL, error)
func ReadRequest(b *bufio.Reader) (*Request, error)
func ReadResponse(r *bufio.Reader, req *Request) (*Response, error)
func Redirect(w ResponseWriter, r *Request, url string, code int)
func Serve(l net.Listener, handler Handler) error
func ServeContent(w ResponseWriter, req *Request, name string, modtime time.Time, ...)
func ServeFile(w ResponseWriter, r *Request, name string)
func ServeTLS(l net.Listener, handler Handler, certFile, keyFile string) error
func SetCookie(w ResponseWriter, cookie *Cookie)
func StatusText(code int) string
```


- 库主要的结构体

`go doc net/http| grep "^type" | grep struct`

`^type (?=[A-Z]).+struct`

```
type Client struct{ ... }     负责 HTTP 客户端
type Cookie struct{ ... }     Cookie
type ProtocolError struct{ ... } 
type PushOptions struct{ ... }
type Request struct{ ... }   请求
type Response struct{ ... }  响应
type ServeMux struct{ ... }  路由
type Server struct{ ... }    HTTP 服务端
type Transport struct{ ... } 控制传输过程
```


了解每部分负责的功能

- 想要细看哪一部分功能 , 如创建 HTTP 服务

使用思维导图从入口处一层层记录下来,只记录核心代码,并对每个核心代码继续解析


## 参考

[手把手带你写 web 框架 01 net/http：使用标准库搭建Server并不是那么简单]()