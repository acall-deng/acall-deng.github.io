---
type: article
title:  "C++ 随笔"
categories: C++
tags:  C++  
author: DengYuting
---

- 20191212第二次更新

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

## rand 随机数种子问题

在同一个程序中多次调用rand()会产生不同的随机数，但是每次运行程序会发现产生的随机数相同。下面的程序则是这种情况。  
原因是默认情况下都是从相同的随机数序列中取出数据的，所以每次运行这段程序产生的10个数字都是相同的，实测哪怕是重新编译或者是重新建立一个代码段生成的也是一样的10个数字。当加入随机数种子之后，则每次都会生成一个新的随机数序列。

```c++
#include <iostream>

using namespace std;

int main() {
    //srand(time(NULL)); 或者
    //std::srand(unsigned(std::time(0))); 下节中说明为什么相同
    for (int i=0; i<10; i++) {
        int rand_num = rand() % 100;
        cout << rand_num << endl;
    }
}
```

## C++ __thread 线程局部存储(Thread-Local Storage)  

gnu: https://gcc.gnu.org/onlinedocs/gcc-3.3.1/gcc/Thread-Local.html  

被修饰的变量在每个线程中都会有一个单独的实例，从而线程之间不串扰，std::addressof()可以在线程运行时获取到变量的地址，在线程结束之后失效。

- 需要linker/dynamic linker/system libraries(libc.so & libpthread.so)支持，所以不是任何时候都生效
- 可单独使用，或者与extern/static连用，连用情况下出现在extern/static的后面  
- 可作用于全局/文件作用域static/函数作用域static/类内static变量，不能用于局部或非静态数据成员
- 静态初始化不能引用线程局部变量的地址。
- 在C++中，如果存在用于线程局部变量的初始化过程，则该初始化过程必须是ANSI / ISO C ++标准5.19.2中定义的常量表达式。
  
```c++
//用法：
__thread int i;
extern __thread struct state s;
static __thread char *p;
```