# cscope

## Golang

```sh
$ find $GOPATH/src/ -name "*.go" > cscope.files
$ find $GOROOT/src/ -name "*.go" >> cscope.files
$ cscope -Rbkq
```

    -R: 在生成索引文件时，搜索子目录树中的代码
    -b: 只生成索引文件，不进入cscope的界面
    -k: 在生成索引文件时，不搜索/usr/include目录
    -q: 生成cscope.in.out和cscope.po.out文件，加快cscope的索引速度

参考资料：
* [cscope 支持 go 语言](http://studygolang.com/topics/209)
