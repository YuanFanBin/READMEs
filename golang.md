# Golang

## Tips And Tricks

[Go-advices](https://github.com/cristaloleg/go-advices)

[Go的50度灰：Golang新开发者要注意的陷阱和常见错误](http://colobu.com/2015/09/07/gotchas-and-common-mistakes-in-go-golang/)

## Blog

[Go Concurrency Patterns: Pipelines and cancellation](https://blog.golang.org/pipelines)

- [About When Not to Do Microservices](http://blog.christianposta.com/microservices/when-not-to-do-microservices/)
    - [Low-risk Monolith to Microservice Evolution Part I](http://blog.christianposta.com/microservices/low-risk-monolith-to-microservice-evolution/)
        - [Blue-green Deployments, A/B Testing, and Canary Releases](http://blog.christianposta.com/deploy/blue-green-deployments-a-b-testing-and-canary-releases/)
    - [Low-risk Monolith to Microservice Evolution Part II](http://blog.christianposta.com/microservices/low-risk-monolith-to-microservice-evolution-part-ii/)
    - [Low-risk Monolith to Microservice Evolution Part III](http://blog.christianposta.com/microservices/low-risk-monolith-to-microservice-evolution-part-iii/)

[Golang Guide: A List of Top Golang Frameworks, IDEs & Tools](https://medium.com/@quintinglvr/golang-guide-a-list-of-top-golang-frameworks-ides-tools-e7c7866e96c9)

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
