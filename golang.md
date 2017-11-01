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

记录从开源项目中的使用便捷的util或自己常用的util

### [k8s](https://github.com/kubernetes)

[RecoverFromPanic](https://github.com/kubernetes/kubernetes/blob/master/staging/src/k8s.io/apimachinery/pkg/util/runtime/runtime.go#L151): panic中恢复，并打印堆栈
[Runner](https://github.com/kubernetes/kubernetes/blob/master/pkg/util/async/runner.go): 异步执行一组func
[Exponential Backoff](https://github.com/kubernetes/kubernetes/tree/master/pkg/util/goroutinemap): 指数回退算法Go实现及其用法
