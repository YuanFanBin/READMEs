# Golang

## 常用包

### [labstack/echo](https://github.com/labstack/echo)

```sh
$ go get -u labstack/echo
$ go install labstack/echo
```

相关资料：
* [echo](https://echo.labstack.com/) - 官网

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

- 1. 开源项目
- 2. 问答社区
- 3. 个人项目

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
