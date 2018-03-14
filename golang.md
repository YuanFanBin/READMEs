# Golang

[The Go Programming Language Specification](https://golang.org/ref/spec)

## Video

[Go Concurrency Patterns](https://www.youtube.com/watch?v=f6kdp27TYZs)
[Advanced Go Concurrency Patterns](https://www.youtube.com/watch?v=QDDwwePbDtw&feature=youtu.be)

## Tips And Tricks

[Go-advices](https://github.com/cristaloleg/go-advices)

[Go的50度灰：Golang新开发者要注意的陷阱和常见错误](http://colobu.com/2015/09/07/gotchas-and-common-mistakes-in-go-golang/)

[Rob Pike's 5 Rules of Programming](http://blog.codonomics.com/2017/09/rob-pikes-5-rules-of-programming.html)

[Introduction to bufio package in Golang](https://medium.com/golangspec/introduction-to-bufio-package-in-golang-ad7d1877f762): ReadSlice**!**, ReadBytes**!!**

    - [In-depth introduction to bufio.Scanner in Golang](https://medium.com/golangspec/in-depth-introduction-to-bufio-scanner-in-golang-55483bb689b4)

## Blog

[Go Concurrency Patterns: Pipelines and cancellation](https://blog.golang.org/pipelines)

[About When Not to Do Microservices](http://blog.christianposta.com/microservices/when-not-to-do-microservices/)

    - [Low-risk Monolith to Microservice Evolution Part I](http://blog.christianposta.com/microservices/low-risk-monolith-to-microservice-evolution/)

        - [Blue-green Deployments, A/B Testing, and Canary Releases](http://blog.christianposta.com/deploy/blue-green-deployments-a-b-testing-and-canary-releases/)

    - [Low-risk Monolith to Microservice Evolution Part II](http://blog.christianposta.com/microservices/low-risk-monolith-to-microservice-evolution-part-ii/)

    - [Low-risk Monolith to Microservice Evolution Part III](http://blog.christianposta.com/microservices/low-risk-monolith-to-microservice-evolution-part-iii/)

同类问题：[小小的公共库，大大的耦合，你痛过吗？](https://mp.weixin.qq.com/s?__biz=MjM5ODYxMDA5OQ==&mid=2651960650&idx=1&sn=7c63fdc50a130e1d9fc3e5b6791ce01f&chksm=bd2d00968a5a89801861bd1665ad60fe240b5ff3d6eab984e70938d9cfd6502aca8de5686e0c&scene=25#wechat_redirect)

[Golang Guide: A List of Top Golang Frameworks, IDEs & Tools](https://medium.com/@quintinglvr/golang-guide-a-list-of-top-golang-frameworks-ides-tools-e7c7866e96c9)

#### [🐣 Go Funcs — Baby-Gopher’s Visual Guide - Inanc Gumus](https://blog.learngoprogramming.com/golang-funcs-params-named-result-values-types-pass-by-value-67f4374d9c0a)

阅读总结：重点看一下 **Naming funcs** 这个小节

摘录部分代码：

```go
// Be a Minimalist
// Not this:
func CheckProtocolIsFileTransferProtocol(protocolData io.Reader) bool
// This:
func Detect(in io.Reader) Name {
  return FTP
}
// Not this:
func CreateFromIncomingJSONBytes(incomingBytesSource []byte)
// This:
func NewFromJSON(src []byte)

// Name in MixedCaps
// This:
func runServer()
func RunServer()
// Not this:
func run_server()
func RUN_SERVER()
func RunSERVER()

// Acronyms should be all uppercase:
// Not this:
func ServeHttp()
// This:
func ServerHTTP()

// Choose descriptive param names
// Not this:
func encrypt(i1, a3, b2 byte) byte
// This:
func encrypt(privKey, pubKey, salt byte) byte
// Not this:
func Write(writableStream io.Writer, bytesToBeWritten []byte)
// This:
func Write(w io.Writer, s []byte)
// Types make it clear, no need for the names

// Use verbs
// Not this:
func mongo(h string) error
// This:
func connectMongo(host string) error
// If it's in Mongo package, just:
func connect(host string) error

// Use is / are
// Not this:
func pop(new bool) item
// This:
func pop(isNew bool) item

// Omit types in the name
// Not this:
func show(errorString string)
// This:
func show(err string)

// Getters and Setters
// Not this:
func GetName() string
// This:
func Name() string
// Not this:
func Name() string
// This:
func SetName(name string)
```


#### [Go Defer Simplified with Practical Visuals - Inanc Gumus](https://blog.learngoprogramming.com/golang-defer-simplified-77d3b2b817ff)

阅读总结：

```go
type Car struct {
	model string
}

func (c Car) PrintModel() { 		// #1
// func (c *Car) PrintModel() { 	// #2
	fmt.Println(c.model)
}
func main() {
	c := Car{model: "DeLorean DMC-12"}
	defer c.PrintModel()
	c.model = "Chevrolet Impala"
}
```

```sh
$ go run test.go    #1
Chevrolet Impala
$ go run test.go    #2
DeLorean DMC-12
```

其他小节知识点都易理解，也不易出错。**Defer Methods** 小节知识需要对 **Method** 有所理解，指针接收（对象地址）与非指针接收（对象拷贝）所产生的行为不同。

可以阅读一下 [The Zoo of Go Functions](https://blog.learngoprogramming.com/go-functions-overview-anonymous-closures-higher-order-deferred-concurrent-6799008dde7b) 这篇文章，加深对函数，方法，接口理解。

拓展资料：[Defer, Panic, and Recover](https://blog.golang.org/defer-panic-and-recover), [The Zoo of Go Functions](https://blog.learngoprogramming.com/go-functions-overview-anonymous-closures-higher-order-deferred-concurrent-6799008dde7b), [★ Ultimate Guide to Go Variadic Functions](https://blog.learngoprogramming.com/golang-variadic-funcs-how-to-patterns-369408f19085)

#### [The Zoo of Go Functions - Inanc Gumus](https://blog.learngoprogramming.com/go-functions-overview-anonymous-closures-higher-order-deferred-concurrent-6799008dde7b)

阅读总结： Just Reading...

#### [★ Ultimate Guide to Go Variadic Functions - Inanc Gumus](https://blog.learngoprogramming.com/golang-variadic-funcs-how-to-patterns-369408f19085)

阅读总结：函数可变参数的应用( **...** )

摘录几个例子

1.

```go
func ToIP(parts ...byte) string {
    parts = append(parts, make([]byte, 4-len(parts))...)
    return fmt.Sprintf("%d.%d.%d.%d", 
    parts[0], parts[1], parts[2], parts[3])
}
```

```sh
ToIP(255)   // 255.0.0.0
ToIP(10, 1) // 10.1.0.0
ToIP(127, 0, 0, 1) // 127.0.0.1
```

2.

```go
type formatter func(s string) string

func format(s string, fmtrs ...formatter) string {
  for _, fmtr := range fmtrs {
    s = fmtr(s)
  }
  return s
}

format(" alan turing ", trim, last, strings.ToUpper)

// output: TURING
```

拓展：[Variadic function](https://rosettacode.org/wiki/Variadic_function) - 各种语言的可变入参

#### [Golang 任务队列策略 -- 读《JOB QUEUES IN GO》](http://www.cnblogs.com/artong0416/p/7883381.html)

阅读总结：

```golang
package main

import (
	"context"
	"fmt"
	"time"
)

func worker(ctx context.Context, jobChan <-chan int) {
	for {
		select {
		case <-ctx.Done(): // 利用context.Context包的cancel功能
			fmt.Println("ctx done")
			return
		case job := <-jobChan:
			fmt.Printf("Job %d\n", job)
			time.Sleep(100 * time.Millisecond)
		}
	}
}

func producer(jobChan chan<- int) {
	for i := 1; i <= 100; i++ {
		jobChan <- i
	}
	close(jobChan)
}

func main() {
	var ctx context.Context
	var cancel context.CancelFunc
	jobChan := make(chan int, 100)

	// #1 带有取消功能的 context
	ctx, cancel = context.WithCancel(context.Background())
	producer(jobChan)
	go worker(ctx, jobChan)
	select {
	case <-time.After(500 * time.Millisecond):
		cancel() // #1.1 这里也可以用channel来代替，不必使用context
	}
	time.Sleep(500 * time.Millisecond)

	// #2 带有超时功能的 context
	// ctx, cancel = context.WithDeadline(context.Background(), time.Now().Add(300*time.Millisecond))
	// ctx, cancel = context.WithTimeout(context.Background(), 300*time.Millisecond)
	// defer cancel()
	// producer(jobChan)
	// go worker(ctx, jobChan)
	// time.Sleep(500 * time.Millisecond)

	// #1, #2 虽然取消了，但可能丢弃掉了jobChan中的剩余任务
}
```

若想消费完成channel中数据后再执行退出操作需要这样做： **`close(jobChan)`** 这样可以让任务队列不再接收新任务，当前channel中任务利用 **`for job := range jobChan {...}`** 即可全部读出

#### [Golang Workers / Job Queue](https://gist.github.com/harlow/dbcd639cf8d396a2ab73)

- [Handling 1 Million Requests per Minute with Go](http://marcio.io/2015/07/handling-1-million-requests-per-minute-with-golang/)**!!**
- [Writing worker queues, in Go](http://nesv.github.io/golang/2014/02/25/worker-queues-in-go.html)

#### [Go语言中实现基于 event-loop 网络处理](http://colobu.com/2017/11/29/event-loop-networking-in-Go/)

扩展：[tidwall/evio](https://github.com/tidwall/evio): Fast event-loop networking for Go, [valyala/fasthttp](https://github.com/valyala/fasthttp)

## Web 框架

- [labstack/echo](https://github.com/labstack/echo): [echo 官网](https://echo.labstack.com/)

- [kataras/iris](https://github.com/kataras/iris): [iris 官网](https://iris-go.com/)

资讯：

[Top 6 web frameworks for Go as of 2017](https://dev.to/speedwheel/top-6-web-frameworks-for-go-as-of-2017-34i)

## 微服务框架

- [go-kit/kit](https://github.com/go-kit/kit)

## 各种其他包

### [jinzhu/gorm](https://github.com/jinzhu/gorm)

```sh
$ go get -u github.com/jinzhu/gorm
$ go install github.com/jinzhu/gorm
```

相关资料：
* [GORM - English](http://jinzhu.me/gorm/) - 文档（英文）
* [GORM - Chinese](https://jasperxu.github.io/gorm-zh/) - 文档（中文）

### [cihub/seelog](https://github.com/cihub/seelog)

```sh
$ go get -u github.com/cihub/seelog
$ go install github.com/cihub/seelog
```

### [golang/glog](https://github.com/golang/glog)

```sh
$ go get -u https://github.com/golang/glog
$ go install https://github.com/golang/glog
```

### [go-redis/redis](https://github.com/go-redis/redis)

```sh
$ go get -u github.com/go-redis/redis
$ go install github.com/go-redis/redis
```

### [golang/gomock](https://github.com/golang/gomock)

```sh
go get github.com/golang/mock/gomock
go get github.com/golang/mock/mockgen
```

### [parnurzeal/gorequest](https://github.com/parnurzeal/gorequest)

```sh
go get github.com/parnurzeal/gorequest
```

### [google/go-querystring](https://github.com/google/go-querystring)

```go
import "github.com/google/go-querystring/query"

type Options struct {
  Query   string `url:"q"`
  ShowAll bool   `url:"all"`
  Page    int    `url:"page"`
}

opt := Options{ "foo", true, 2 }
v, _ := query.Values(opt)
fmt.Print(v.Encode()) // will output: "q=foo&all=true&page=2"
```

## util

摘录从以下来源获得的util

- 开源项目
- 问答社区
- 个人项目

### [k8s](https://github.com/kubernetes)

[RecoverFromPanic](https://github.com/kubernetes/kubernetes/blob/master/staging/src/k8s.io/apimachinery/pkg/util/runtime/runtime.go#L151): panic中恢复，并打印堆栈

[Runner](https://github.com/kubernetes/kubernetes/blob/master/pkg/util/async/runner.go): 异步执行一组func

[Exponential Backoff](https://github.com/kubernetes/kubernetes/tree/master/pkg/util/goroutinemap): 指数回退算法Go实现及其用法

### [StackOverflow](https://stackoverflow.com)

[RandString](https://stackoverflow.com/questions/22892120/how-to-generate-a-random-string-of-a-fixed-length-in-golang): 随机字符串

摘录部分源代码：

```golang
const (
	letterBytes   = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ~@#%^&*(){}[]|"
	letterIdxBits = 7                    // 7 bits to represent a letter index
	letterIdxMask = 1<<letterIdxBits - 1 // All 1-bits, as many as letterIdxBits
	letterIdxMax  = 63 / letterIdxBits   // # of letter indices fitting in 63 bits
)

var src = rand.NewSource(time.Now().UnixNano())

// RandString 生成随机字符串
func RandString(n int) string {
	b := make([]byte, n)
	// A src.Int63() generates 63 random bits, enough for letterIdxMax characters!
	for i, cache, remain := n-1, src.Int63(), letterIdxMax; i >= 0; {
		if remain == 0 {
			cache, remain = src.Int63(), letterIdxMax
		}
		if idx := int(cache & letterIdxMask); idx < len(letterBytes) {
			b[i] = letterBytes[idx]
			i--
		}
		cache >>= letterIdxBits
		remain--
	}
	return string(b)
}
```

## misc

[sync/atomic.Value](https://golang.org/pkg/sync/atomic/#Value): 轮询查询配置 - （应用点：线上debug日志开关）

[Share memory by communicating; don't communicate by sharing memory.](https://golang.org/pkg/sync/atomic)

```go
// 确保SomeStruct满足接口SomeInterface的实现
var _ SomeStruct = (*SomeInterface)(nil)
```

[Go 1.9 sync.Map揭秘](http://colobu.com/2017/07/11/dive-into-sync-Map/)
