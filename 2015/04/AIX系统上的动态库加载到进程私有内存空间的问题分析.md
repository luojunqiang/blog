# AIX系统上的动态库加载到进程私有内存空间的问题分析

 #IBM AIX shared library loaded in private section problem

## 概述

动态库(shared library)的权限如果是other不可读取的，则系统会将该动态库装载到进程私有内存中，而不是系统共享的内存中。
如果动态库自身的权限正确但其依赖的其他动态库的权限有问题，也会导致该动态库装载到进程私有内存中。

## 问题现象

> 按前期ESID分段的说明，现网的libpubclass.1.0.so是加载到私有数据段，而实验室和CBE16是加载到共享段。
```
    ESID ：
    0x80000000-0x8FFFFFFF        private load
    0x90000000-0x9FFFFFFF        shared library text
```

现网`genld –l -d`输出如下：
```
Proc_pid: 6422732   Proc_name: startapp.1.0
        100000000    3e71        110000a4a     856  startapp.1.0
  9fffffff0000000    e0ea  9fffffff000e0ea       0  /usr/ccs/bin/usla64
  90000000153a500    14f4  9001000a0655208     1e8  /usr/lib/libc_r.a[aio_64.o]
  800000001612000  27b4918  8001000a3878fd3  21edf5  /TimesTen/tt1121/lib/libttco.so
  800000000279000  139882f  8001000a37a030c   cfb4c  /TimesTen/tt1121/lib/libtten.so
  800000000242000   363b2  8001000a38703f8    6d58  /home/cbe/bin/TtEnDb.so          <<== private load
  800000000212000   2f99b  8001000a3796068    9f50  /home/cbe/bin/shrsub.so          <<== private load
  900000000473300   16f1c  9001000a01188a8    dea8  /usr/lib/libsrc.a[shr_64.o]
  900000000459b00   18e56  9001000a04022c8    4af0  /usr/lib/libcorcfg.a[shr_64.o]
  900000001377a00   bc225  9001000a0578dc8   36f81  /usr/lib/liblvm.a[shr_64.o]
  9000000003eed00   1e22f  9001000a010dde8    a158  /usr/lib/libodm.a[shr_64.o]
  900000000443a80   152a5  9001000a03ea5d0    71a0  /usr/lib/libcfg.a[shr_64.o]
  900000000eaf180   607b8  9001000a05b0c60    d654  /usr/lib/libperfstat.a[shr_64.o]
  90000000153c280   5419a  9001000a06561a8   19236  /usr/lib/libcurses.a[shr42_64.o]
  8000000001c5000   4c5ad  8001000a006502c   1bb04  /home/cbe/bin/libgcpsC.a                <<== private load
  900000000429700   1985f  9001000a026cc20    cf88  /usr/lib/libiconv.a[shr4_64.o]
  900000095542000  a48ef7c  8001000a00818ed  371408b  /home/cbe/bin/libpubclass.1.0.so      <<== private load
  90000000040d3a0     af3  9001000a010c948     1a0  /usr/lib/libcrypt.a[shr_64.o]
  900000000000980  3ecd2c  9001000a0000bb0  10b428  /usr/lib/libc.a[shr_64.o]
  90000000049e000   41a58  9001000a027c000   8c014  /usr/lib/libpthreads.a[shr_xpg5_64.o]
  8000000001ae100   16756  8001000a0000524    2014  /home/cbe/bin/libgcc_s.a[shr.o]            <<== private load
  800000000000100  1ad48e  8001000a0003391   615ff  /home/cbe/bin/libstdc++.a[libstdc++.so.6]  <<== private load
  900000000ab8000     2fb  9001000a03aa000       0  /usr/lib/libdl.a[shr_64.o]
```

## 分析过程

```
## pid=385028 chmod=0750
genld -ld output:
  800000000000000  ddeaa2c  8001000a00006cf  2fef541  /ttadm/billtest/build/bakCBS_BASE/app/V300R005C12/lib/libcbs.so
svmon -lP output:  
    Vsid      Esid Type Description              PSize  Inuse   Pin Pgsp Virtual
  1e8f17  80000000 work private load text            s   5878     0    0  5892 

## pid=495942 chmod=0750
genld -ld output:
  800000000000000  ddeaa2c  8001000a00006cf  2fef541  /ttadm/billtest/build/bakCBS_BASE/app/V300R005C12/lib/libcbs.so
svmon -lP output:  
    Vsid      Esid Type Description              PSize  Inuse   Pin Pgsp Virtual
  137e36  80000000 work private load text            s   5891     0    0  5892 

----

## pid=590134 chmod=0755
genld -ld output:
  900000021de7000  ddeaa2c  9001000a86126cf  2fef541  /ttadm/billtest/build/bakCBS_BASE/app/V300R005C12/lib/libcbs.so
svmon -lP output:  
    Vsid      Esid Type Description              PSize  Inuse   Pin Pgsp Virtual
  1270b2  90000002 work shared library text          m   1914     0    0  1914 

## pid=528528 chmod=0755
genld -ld output:
  900000021de7000  ddeaa2c  9001000a86126cf  2fef541  /ttadm/billtest/build/bakCBS_BASE/app/V300R005C12/lib/libcbs.so
svmon -lP output:  
    Vsid      Esid Type Description              PSize  Inuse   Pin Pgsp Virtual
  1270b2  90000002 work shared library text          m   1914     0    0  1914
```

访问权限为750时，动态库加载到了进程私有内存空间，其Vsid是不同的，说明占用了不同的内存。
而权限为755时，动态库加载到系统共享的内存空间，其Vsid是相同的。

## 相关的工具

- svmon
- slibclean
- procldd
- procmap
- genkld
- genld 

## 参考资料

- http://www.ibm.com/developerworks/aix/library/au-slib_memory/
- Developing and Porting C and C++ Applications on AIX
  http://www.redbooks.ibm.com/redbooks/pdfs/sg245674.pdf
