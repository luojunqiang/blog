
# 拦截 lstat 系统调用

## 实现

代码：

```
#include <sys/types.h>
#include <sys/stat.h>
#include <unistd.h>
 
int (__lxstat) (int ver, __const char *filename, struct stat *buf)
{
    return __xstat(ver, filename, buf);
}
 
int (__lxstat64) (int ver, __const char *filename, struct stat64 *buf)
{
    return __xstat64(ver, filename, buf);
}

```

构建：

```
gcc -shared -fPIC -o libitc_lstat.so itc_lstat.c
```

使用：

```
LD_PRELOAD=/usr/local/lib/libitc_lstat.so svn status
```

## 拦截系统函数然后调用原系统函数的方法

代码：

```
#define _GNU_SOURCE

#include <stdio.h>
#include <dlfcn.h>

FILE *fopen(const char *path, const char *mode) {
    printf("In our own fopen, opening %s\n", path);

    FILE *(*original_fopen)(const char*, const char*);
    original_fopen = dlsym(RTLD_NEXT, "fopen");
    return (*original_fopen)(path, mode);
}
```
> 
This shared library exports the fopen function that prints the path and then uses dlsym with the RTLD_NEXT pseudohandle to find the original fopen function. We must define the _GNU_SOURCE feature test macro in order to get the RTLD_NEXT definition from <dlfcn.h>. RTLD_NEXT finds the next occurrence of a function in the search order after the current library.

We can compile this shared library this way:
```
gcc -Wall -fPIC -shared -o myfopen.so myfopen.c -ldl
```


## 参考资料

- http://stackoverflow.com/questions/5478780/c-and-ld-preload-open-and-open64-calls-intercepted-but-not-stat64
- http://www.catonmat.net/blog/simple-ld-preload-tutorial/
- http://www.catonmat.net/blog/simple-ld-preload-tutorial-part-2/
- https://rafalcieslak.wordpress.com/2013/04/02/dynamic-linker-tricks-using-ld_preload-to-cheat-inject-features-and-investigate-programs/ (great!) 

## 备注

本来想通过拦截lstat来让subversion将连接文件视同真正的文件进行处理，基本可以实现。但目录的连接会有问题，subversion找上层父目录的时候会引起问题。
