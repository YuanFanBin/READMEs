
ngx.print("ok");
```

V
```
标签（空格分隔）： openresty

---
[toc]


## openresty在业务中的运用
### nginx.conf 基本配置
```conf
http {
    ...
    lua_package_path "/aaa/bbb/ccc/?.lua;;";                 # 指定项目跟路径
    lua_package_cpath "/usr/local/openresty/lualib/?.so;;";  # 指定.so或.lua扩展路径
    lua_shared_dict dict 128m;                               # 本机cache
    init_by_lua_file "/aaa/bbb/ccc/init.lua";                # 指定项目的init文件，加载使用的基本模块
    lua_code_cache on;		                                 # 开启lua cache开关，默认开启，不建议关闭
    server {
        ...
        location /hello {
            content_by_lua 'ngx.print("hello")';
        }
        ...
    }
    ...
}
```

### openresty的处理阶段
|阶段|可用配置|
|----|--------|
|location请求地址重写阶段|rewrite_by_lua_xxx, set_by_lua_xxx|
|访问权限检查提交阶段|access_by_lua_xxx|
|内容产生阶段|content_by_lua_xxx|
|日志模块处理阶段|log_by_lua_xxx|

还有一些其他的处理阶段如：header_filter_by_xxx, body_filter_by_xxx，但我们的业务中涉及较少，不过多提及。

### 访问控制，ip限制
若对接口有权限或者ip等限制可在 `access_by_lua_xxx` 中加入相应规则，限制请求。

```conf
location /api {
    acccess_by_lua '
        if 用户登录验证失败 then
            ngx.print("登录失败")
            ngx.exit(ngx.http_ok)
        end
        if cookie失效 then
            ngx.print("cookie 失效")
            ngx.exit(ngx.http_ok)
        end
        ...
    ';
}
```

接口若有加解密过程使用如下方式更加合适：
```conf
location /api {
    access_by_lua_file      decrypt.lua;        # 对body数据解密
    content_by_lua_file     process.lua;        # 逻辑处理，产生内容
    body_filter_by_lua_file encrypt.lua;        # 对body数据加密
    log_by_lua_file         statistics.lua;     # 若有统计需求的日志，在此处记录
}
```

### 关于一些坑

1. openresty中的坑

```lua
ngx.print("hello openresty");   -- 响应结果无换行符
ngx.say("hello openresty");     -- 响应结果有换行符
```

若在加解密接口中，若出现解密失败，有可能是使用了 `ngx.say` 在body中增加了额外的换行符

openresty自带的cjson库中，对空表的encode结果为默认为JSON对象，并非所期望的JSON数组，需要手动关闭空表解析为对象的开关。

```lua
local tb = {}
cjson.encode(tb)        --> {}
cjson.encode_empty_table_as_object(false) -- 关闭空表=>JSON对象
cjson.encode(tb)        --> []
```

[记一次踩坑|空table应该编码为数组还是对象](http://answerywj.com/2017/06/16/table-encode-as-array-or-object/)

[编码为 array 还是 object](https://wiki.jikexueyuan.com/project/openresty-best-practice/array-or-object.html)

2. lua中的坑

lua中的数组（表）以索引 [1] 开始，并非其他语言中的 [0] 开始。
lua & luajit 只支持 32bit 运算，不支持 64bit 运算。
