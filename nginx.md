# Nginx的11个阶段

## 1. [11个阶段详解][1]

1.NGX\_HTTP\_POST\_READ\_PHASE

    接受完请求头之后的第一个阶段，它位于uri重写之前，实际上很少有模块会注册在该阶段，默认的情况下，该阶段被跳过。

2.NGX\_HTTP\_SERVER\_REWRITE\_PHASE

    server级别的uri重写阶段，也就是该阶段执行处于server块内，location块外的重写指令，前面章节已经说明在读取请求头的过程中nginx会根据host及端口找到对应的虚拟主机配置。

3.NGX\_HTTP\_FIND\_CONFIG\_PHASE

    寻找location配置阶段，该阶段使用重写之后的uri来查找对应的location，值得注意的是该阶段可能会被执行多次，因为也可能有location级别的重写指令。

4.NGX\_HTTP\_REWRITE\_PHASE

    location级别的uri重写阶段，该阶段执行location基本的重写指令，也可能会被执行多次。

5.NGX\_HTTP\_POST\_REWRITE\_PHASE

    location级别重写的最后一阶段，用来检查上阶段是否有uri重写，并根据结果跳转到合适的阶段。

6.NGX\_HTTP\_PREACCESS\_PHASE

    访问权限控制的前一阶段，该阶段在权限控制阶段之前，一般也用于访问控制，比如限制访问频率，链接数等。

7.NGX\_HTTP\_ACCESS\_PHASE

    访问权限控制阶段，比如基于IP黑名单的权限控制，基于用户名密码的权限控制等。

8.NGX\_HTTP\_POST\_ACCESS\_PHASE

    访问权限控制的后一阶段，该阶段根据权限控制阶段的执行结果进行相应处理。

9.NGX\_HTTP\_TRY\_FILE\_PHAST

    try\_files指令的处理阶段，如果没有配置try\_files指令，则该阶段被跳过。

10.NGX\_HTTP\_CONTENT\_PHASE

    内容生成阶段，该阶段产生响应，并发送到客户端。

11.NGX\_HTTP\_LOG\_PHASE

    日志记录阶段，该阶段记录访问日志。


## 2. [openresty对应使用的阶段][2]

阶段                     | 常量                              | 配置
------------------------ | --------------------------------- | ------------------------------------------------------
读取请求内容阶段         | NGX\_HTTP\_POST\_READ\_PHASE      |
Server请求地址重写阶段   | NGX\_HTTP\_SERVER\_REWRITE\_PHASE |
配置查找阶段             | NGX\_HTTP\_FIND\_CONFIG\_PHASE    |
location请求地址重写阶段 | NGX\_HTTP\_REWRITE\_PHASE         | rewrite\_by\_lua, rewrite\_by\_lua\_file, set\_by\_lua
请求地址重写提交阶段     | NGX\_HTTP\_POST\_REWRITE\_PHASE   |
访问权限检查准备阶段     | NGX\_HTTP\_ACCESS\_PHASE          |
访问权限检查阶段         | NGX\_HTTP\_ACCESS\_PHASE          |
访问权限检查提交阶段     | NGX\_HTTP\_POST\_ACCESS\_PHASE    | access\_by\_lua, access\_by\_lua\_file
配置项try\_files处理阶段 | NGX\_HTTP\_TRY\_FILE\_PHASE       |
内容产生阶段             | NGX\_HTTP\_CONTENT\_PHASE         | content\_by\_lua, content\_by\_lua\_file
日志模块处理阶段         | NGX\_HTTP\_LOG\_PHASE             | log\_by\_lua, log\_by\_lua\_file

[1]: http://tengine.taobao.org/book/chapter_12.html#id8
[2]: http://blog.csdn.net/liujiyong7/article/details/37692027
