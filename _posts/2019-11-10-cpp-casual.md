---
type: article
title:  "C++ 随笔"
categories: C++
tags:  C++  
author: DengYuting
---


## glibc qsort 库函数多线程环境下的 core dump 问题

https://www.felix021.com/blog/read.php?1951

https://blog.csdn.net/yockie/article/details/51683515

出现原因：

> 为了对 qsort 尽可能优化，qsort 用到了物理内存页大小等参数，下列为代码片段，会出现多线程竞争的问题导致除0错误

解决方案:

> 在主程序（线程未启动时）中先调用qsort一次，为static变量赋上值。
>
> 需要注意的：初始化时，n*s必须大于1024，否则pagesize没进行初始化。

```c++
if (phys_pages == 0)
{
    phys_pages = __sysconf (_SC_PHYS_PAGES);
    //__sysconf函数在sysdeps/posix/sysconf.c中
    //_SC_PHYS_PAGES对应到函数__get_phys_pages()
    //位于文件sysdeps/unix/sysv/linux/getsysstats.c中
    //通过phys_pages_info()打开/proc/meminfo来读取内存信息
    //(这就定位到了qsort打开文件的问题)

    if (phys_pages == -1)
        /* Error while determining the memory size.  So let's
            assume there is enough memory.  Otherwise the
            implementer should provide a complete implementation of
            the `sysconf' function.  */
        phys_pages = (long int) (~0ul >> 1);

    /* The following determines that we will never use more than
        a quarter of the physical memory.  */
    phys_pages /= 4;

    pagesize = __sysconf (_SC_PAGESIZE);
}
//注意，上面这一段if会产生竞争，出现线程安全安全：
//如果两个线程都调用qsort，当线程1获取了phys_pages之后，线程2
//才到达if，线程2就会跳过这一段代码，直接执行下面的if语句——
//而此时pagesize尚未初始化（=0），于是就会出现除零错误，导致
//core dump
```





## C++ 类成员冒号初始化问题

https://blog.csdn.net/zj510/article/details/8135556

> 冒号初始化为初始化时赋值，对于 const 或者 引用 来说只能通过此方式来改动
>
> 作为对比：正常函数体中用 "=" 赋值的过程有两个步骤，第一是实例化对象/创建变量，第二进行赋值

```c++
class A
{
    public:
        A(int& c) : _b(2), _c(c) // 此处为变量初始化
        {
            _a = 1;
        }
    
    private:
        int _a;
        const int _b;
        int& _c;
};
```



## C++ \__attribute__  和 void() 用法

\__attribute__ 基本作用：https://blog.csdn.net/qq_29343201/article/details/51815445

\__attribute__(unused) 作用：https://satanwoo.github.io/2017/06/01/FBTweak/

这个前面出来了`__attribute__((used))`，它的作用是告诉编译器，我声明的这个符号是需要保留的。我们在开发iOS的过程中，常常会遇到有时候会报警告`xxx unused`，在某些优化的情况下，编译器甚至都不报警告，直接将我们进行了剔除，**这样在编译后(预处理、编译、汇编)生成的目标文件里就不存在我们这个符号。**



## C++ 预编译中使用##和#的作用

```c++
#define TEST_SHARP(name) \
    int name##_test(1)
// ##的作用为拼接，编译结果为 int Johnny_test(1)

#define TEST_SHARP(name) \
    int name##_test(#name，name)
// #的作用为添加引号，编译结果为 int Johnny_tst("Johnny", Johnny)

int main()
{
    Test_SHARP(Johnny); //不加引号
}
```

