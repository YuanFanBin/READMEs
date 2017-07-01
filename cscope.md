# cscope

## vim

```vim
" ------------------------------------------------------------------------------
" [cscope Setting]
" --------------------------------------
if has("cscope")
    set csprg=/usr/bin/cscope
    set csto=1
    set cst
    set nocsverb
    " add any database in current directory
    if filereadable("cscope.out")
        cs add cscope.out
    endif
    set csverb
endif

nmap <LEADER>d :cs find g <cword><CR>       " 查找定义
nmap <LEADER>s :cs find s <cword><CR>       " 查找symbol
nmap <LEADER>c :cs find c <cword><CR>       " 查找calling
```

参考资料：
* [程序员的利器 – cscope](http://easwy.com/blog/archives/advanced-vim-skills-cscope/)

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
