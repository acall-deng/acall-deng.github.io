---
layout: post
title:  "C++ explicit关键字使用"
categories: C++
tags:  C++  
author: DengYuting
---
* content
{:toc}

## 说在前面

explicit 只对构造函数有效，用来避免隐式类型转换。且只对仅含有一个参数的类构造函数有效，因为多于两个的时候是不会发生隐式转换的（除非只有一个参数需要赋值，其他的参数有默认值）。





## 用法


首先定义一个类：  
```
Class String{
    String (int n); // 分配n个字节空间给字符串
    String (const char* p);  // 用字符串p的值初始化字符串
}
```

正常初始化的方法：
```
String s1(10);  // 10个字节长度的字符串
String s2("Hello world!") // s2的初始值为 Hello world
```

隐式转换的写法：
```
String s3 = 10;  // 编译通过，分配10个字节长度的字符串
String s4 = 'a'; // 编译通过，分配int('a')个字节长度的字符串
String s5 = "a"; // 编译通过，调用的是String (const char* p)
```

使用 explicit 关键字：
```
Class String{
    explicit String (int n); // 分配n个字节空间给字符串
    String (const char* p);  // 用字符串p的值初始化字符串
}

此时：
String s3 = 10;  // 编译不通过，不允许隐式转换类型
String s4 = 'a'; // 编译不通过，不允许隐式转换类型
```

## 使用 explicit 的好处

当出现下面的场景时，explicit 关键字能够在编译阶段给出错误：
```
class   A   {  
    A(int   a);  
};  
int   Function(A   a);  
```

此时，若要调用Function(2)，则会隐式转换2为 A 类型，显然不是我们想要的，从而可以使用 explicit 关键字修饰A 的构造函数避免隐式转换的问题。并且也可以避免String s4 = 'a'; 这种奇怪的赋值语句出现。

## 参考

https://www.cnblogs.com/cutepig/archive/2009/01/14/1375917.html  
https://blog.csdn.net/guoyunfei123/article/details/89003369
