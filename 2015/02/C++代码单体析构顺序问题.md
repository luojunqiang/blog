# C++代码单体析构顺序问题

 #c++ #singleton #order

## 问题

底层的MemPool单体类已经析构，而其他单体类析构中还要用MemPool，导致程序core dump。

## 解决方案

一个相对简单的解决方案，std::cout等也是用的这种方案。

http://en.wikibooks.org/wiki/More_C%2B%2B_Idioms/Nifty_Counter

## 其他方案

### Loki

loki::singleton，通过atexit注册析构顺序，改造较大。

### 按析构阶段进行析构

按析构阶段进行析构的方案，类似loki的方式。

- http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.21.7490&rep=rep1&type=pdf
- http://www.drdobbs.com/controlling-the-destruction-order-of-sin/184403709?pgno=7


## 参考资料

- 详细说明了c++的相关知识<br>
https://isocpp.org/wiki/faq/ctors

- 介绍loki singleton<br>
http://www.drdobbs.com/creating-dynamic-singletons-the-loki-li/184401943

