1   什么是文件？IO的对象是什么？

文件是操作系统对外部设备与磁盘数据的抽象, 是一组有序字节流的集合

IO的对象是文件, 输入是从文件读取数据到内存, 输出是把内从中的数据写入到文件中

2   Linux 系统中的文件有几种类型？分别是什么？

普通文件-, 目录文件d, 链接文件l, 套接字文件s, 块设备文件b, 字符设备文件c, 管道文件p

3   什么是标准IO库？

标准IO库是C语言标准库中为开发者提供的一组输入输出的函数, 针对内核接口做了简单的封装, 加入了缓冲区概念, 在对于大文件和需要频繁访问的场景能够减少IO次数, 提高效率, 同时没有用户态切换, 减少系统开销

4   什么是文件流？

文件流是将文件中的字符进行移入移出的操作, 将这一操作抽象为流, 文件流按照字符编码处理, 注重文本语义

5   什么是文件指针？如何得到文件指针？

FILE指针是指向FILE对象的内存地址, 所有文件操作都基于这个指针进行, FILE指针可以通过fopen()进行创建, 可以通过fclose()进行销毁

6   标准IO的缓存机制有几种？分别是什么？如何刷新缓存？

行缓冲: 用于终端设备, 如stdin, Linux默认大小1KB, 遇到\n, 程序关闭, 缓冲区切换, 缓冲区满, fflush强制刷新, 文件关闭

全缓冲: 用于文件读写, Linux默认大小为4KB, 程序关闭, 缓冲区切换, 缓冲区满, fflush强制刷新, 文件关闭

无缓冲: 无缓冲区大小, 不做临时存储, 直接写入硬件

7  32位系统中的全缓存 行缓存 缓存区大小是多少？

全缓存 4KB, 行缓存1KB

8  fgets和gets的区别？

char *fgets(char *s, int size, FILE *stream);

从文件流中读取字符串, 最大size-1个, 自动添加\0, 包含换行符,成功返回字符串指针, 失败返回NULL

char *gets(char *s);

从stdin中读取字符串, 遇到换行或\0结束, 包含换行符, 自动添加\0, 成功返回字符串指针, 失败返回NULL

9   如何设置文件指针的位置？如何获取文件指针的位置？

标准IO可以通过fseek函数设置文件指针位置, 通过rewind函数将光标重置到文件开头, 通过ftell函数获取文件指针的位置

文件IO可以通过lseek函数设置文件指针位置

10 如何重定向文件流的方向？（从一个文件指向另一个文件）

FILE *freopen(const char *path, const char *mode, FILE *stream);

如 freopen("log.txt", "w", stdout);

标准IO中可以使用freopen函数将文件流重定向到另一个文件中, 如可以stdout重定向到log.txt中, 后续使用printf可以直接将打印保存到文件而不是输出到控制台

恢复可以使用freopen("/dev/tty", "w", stdout);

文件IO可以使用dup2函数实现

dup2(fd, 1)可以将fd赋值到stdout, 标准输出指向test.txt

11 使用标准IO库的哪一个函数可以判断文件指针指向文件末尾？

可以使用feof函数判断文件指针是否指向末尾, 需要注意的是, 使用读取时, 虽然指针指向文件末尾的字符, 如果读取还未发生, 此时feof不会认为文件到达末尾, 只有当读取真正完成后才会人文到达末尾

12 使用标准IO库函数实现:文件拷贝（写出伪代码即可）

```c
#include <stdio.h>

// 用fgetc和fputc实现cp命令
int main(int argc, char *argv[])
{
    if (argc != 3)
    {
        printf("Please check arguments\n");
        printf(("usage: %s sourcefile destinationfile\n", argv[0]));
        return -1;
    }
    FILE *src = NULL;
    FILE *dest = NULL;
    if ((src = fopen(argv[1], "r")) == NULL)
    {
        perror("Please check source file");
        return -1;
    }
    if ((dest = fopen(argv[2], "w")) == NULL)
    {
        perror("Please check destination file");
        fclose(src);
        return -1;
    }
    char ch = 0;
    int write_return = 0;
    while ((ch = fgetc(src)) != EOF)
    {
        write_return = fputc(ch, dest);
    }
    if (write_return == EOF)
    {
        perror("Copy failed");
        fclose(src);
        fclose(dest);
        return -1;
    }
    printf("Copy finished\n");
    fclose(src);
    fclose(dest);
    return 0;
}
```

13  如何获取文件的属性值？

可以通过stat, lstat和fstat函数, 三个函数的返回值一样都是stat结构体

stat可以通过文件路径查询, 遇到符号链接时会自动获取链接的目标文件的属性

lstat则会获取符号链接本身的属性

fstat可以通过文件描述符来获取文件的属性, 遇到符号链接时会获取链接的目标文件的属性

14 对目录的操作有哪些函数？为什么没有写目录的函数？

DIR *opendir(const char *name); 通过路径打开目录, 返回DIR *指针

int closedir(DIR *dir);	通过DIR指针关闭路径, 防止内存泄漏

struct dirent *readdir(DIR *dirp);	读取目录中的文件信息, 返回一个dirent结构体指针

目录属于文件系统元数据, 目录项由内核维护, 如果提供写目录的函数让用户直接写入原始数据, 极易破坏文件系统结构, 造成数据丢失

15 什么是文件描述符？如何表示？最大范围是多少？遵循什么原则？

文件描述符时操作系统为每一个进程打开的文件所分配的标识, 使用int类型数据表示, 默认打开3个文件描述符: 0 stdin, 1 stdout, 2 stderr, 用户自定义打开的会从3开始分配, Linux默认范围为0-1023, 可以使用ulimit进行设置, 文件描述符分配遵循最小未占用原则

16 什么是函数库？Linux系统中的函数库有几种类型？有什么区别？

函数库是预先编译好的一堆函数的集合, 库文件是一种可执行代码的二进制形式, linux系统中的函数库有两种:

静态库: 程序编译阶段将静态库链接到程序当中, 为每一个使用静态库的程序创建副本, 适合对启动速度有要求的程序, 但程序整体占用空间大, 静态库更新后需要重新编译调用的程序, Linux下命名为libxxx.a

动态库: 程序编译阶段创建符号链接, 运行阶段将动态库加载到内存中, 所有调用程序共享同一个动态库, 程序占用空间小, 但是加载速度慢, 动态库除大版本更新外, 调用程序无需进行重新编译, linux下命名为libxxx.so

17 如何制作静态库？如何制作动态库？（写出步骤）

编译生成目标文件:gcc -c xxx.c -o xxx.o

制作静态库: ar -cr libxxx.a xxx.o 

调用程序编译时: gcc main.c -lxxx -Lxxx -o main

18 总结：标准IO和文件IO的区别？

标准IO: C标准库函数, 操作FILE *文件流, 有用户缓冲区, 可移植性好, 无需特殊处理错误, 适合频繁小数据读写, 会话结束后数据丢失

文件IO: 操作系统调用, 操作int 文件描述符fd, 没有用户缓冲区, 依赖操作系统, 移植性差, 需要处理错误, 适合大文件读写, 数据持久保存

19 使用文件IO函数实现:文件拷贝（写出伪代码即可）

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>

int main(int argc, char *argv[])
{
    if (argc != 3)
    {
        printf("Please check arguments\n");
        printf(("usage: %s sourcefime destinationfile\n", argv[0]));
        return -1;
    }
    int src = -1;
    int dest = -1;
    src = open(argv[1], O_RDONLY);
    if (src == -1)
    {
        perror("Please check source file");
        return -1;
    }
    dest = open(argv[2], O_WRONLY | O_CREAT | O_EXCL, 0666);
    if (dest == -1)
    {
        perror("Please check destination file");
        close(src);
        return -1;
    }
    char buffer[BUFSIZ];
    ssize_t read_return = 0;
    ssize_t write_return = 0;
    while ((read_return = read(src, buffer, sizeof(buffer))) > 0)
    {
        write_return = write(dest, buffer, read_return);
        if (write_return < 0)
        {
            perror("Write file failed");
            close(src);
            close(dest);
            return -1;
        }
    }
    if (read_return < 0)
    {
        perror("Read file failed");
        close(src);
        close(dest);
        return -1;
    }
    printf("Copy finished\n");
    close(src);
    close(dest);
    return 0;
}
```

20 什么是系统调用？

操作系统内核提供的底层接口, 如open, 运行于内核空间, 需切换CPU特权级, 直接操作硬件, 内核资源, 因为有用户态和内核太切换, 开销较大, 依赖操作系统内核, 移植性差

21 ？动态库生成和版本管理

创建与位置无关目标文件: gcc -fPIC xxx.c -o xxx.o

生成动态库: gcc -shared -Wl,-soname,libxxx.so.x xxx.o -o libxxx.so.x.y.z

-Wl,-soname,libxxx.so.x: 指定soname, 方便ld做版本管理

调用函数编译: gcc main.c -llibxxx -Lxxx -o main

库文件制作完成后, 需要通过如下方式更细, 保证库可以被加载:

- 配置LD_LIBRARY_PATH

  export LD_LIBRARY_PATH=$PWD, 仅对当前终端有效

- /etc/ld.so.conf

  配置完成后需要使用sudo ldconfig更新ld缓存

- 放到/usr/lib或/usr/local/lib下

  配置完成后需要使用sodo ldconfig更新缓存

# 错题分析

1. 在标准IO中, fgets函数读取一样数据时, 以下那个说法是正确的: 读取的字符串包含换行符\n, 会自动添加\0

2. 以下那些操作会影响文件指针的位置: fseek和fputc. fopen只做打开文件, 文件指针初始位置由打开模式决定, 不会改变其位置
3. feof在读取到文件结尾时立即返回真: feof仅在尝试读取超过文件结尾之后才返回真, 而不是仅仅到达文件末尾