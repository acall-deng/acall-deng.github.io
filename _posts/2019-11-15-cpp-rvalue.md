---
type: article
title:  "C++ 右值引用"
categories: C++
tags:  C++ 
author: DengYuting
---
* content
{:toc}

# 前言  
在C语言中，左值是在表达式左边的值，右值是在表达式右边的值。  
在C++中，左值代表使用的是这个变量，而右值代表的是使用它的值。左值一般可以用左值引用，即&，用来作为别名或者在函数传参进入时保证其值能够透传出函数体，右值则可以用右值引用，即&&，可以用来接管右值，延长其生命周期。  
区分左值和右值的一个简单办法是：看能不能对表达式取地址，如果能，则为左值，否则为右值。 




# 右值的作用  

右值的作用主要有三点：（1. 减少内存赋值。 2. 延长右值生命周期。 3. 实现完美转发）   

1. 减少内存复制带来的消耗。在拷贝构造函数的参数中使用右值引用，可以达到移动需要被拷贝的变量而不是复制来避免无必要的资源浪费，从而提升程序的运行效率。其实在C++11中，STL的容器都实现了移动构造函数与移动赋值运算符，这将大大优化STL容器。    
    
举个例子，下面的这个代码为最原始的代码，未使用右值引用的情况下。在一次函数调用并返回值的过程中，至少会有（1.函数内部临时变量调用构造函数生成。2.返回值调用拷贝构造函数生成临时变量返回给调用者。 3. 调用者调用拷贝构造函数接收变量）共计三次构造函数的调用过程，具体如下。 

```cpp

class Person {

public:
    Person(int i) {
        cout << "constructer!"  << endl;
    }

    Person(const Person& p) {
        cout << "copy construct!" << endl;
    }

    ~Person() {
        cout << "deconstructer!" << endl;
    }
};

Person test(){
    Person tmp(1); // 1. 调用构造函数
    return tmp; // 2. 调用拷贝构造函数，返回拷贝后的类实例
} // 3. 类内tmp类实例的析构函数

int main() {

    Person a = test(); // 3. 调用拷贝构造函数
                       // 4. 返回值的析构函数
    return 0; // 5. a的析构函数
}

最终输出：
constructer!
copy construct!
deconstructer!
copy construct!
deconstructer!
deconstructer!

注意：
编译时去除了返回值优化-fno-elide-constructors，否则会只有下面两句，原因是编译器对其进行了优化，减少了复制，在此为了演示需要去除这种优化。
construct!
deconstructer!
```   

此时，如果将接收的返回值改为右值引用，则主函数中将不再需要调用拷贝构造函数接收返回值，原因是右值引用的情况下是直接接管返回值的，直接移动而不复制，如下所示。  

```cpp
#include <iostream>

using namespace std;

class Person {

public:
    Person(int i) {
        cout << "constructer!"  << endl;
    }

    Person(const Person& p) {
        cout << "copy construct!" << endl;
    }

    ~Person() {
        cout << "deconstructer!" << endl;
    }
};

Person test(){
    Person tmp(1); // 1. 调用构造函数
    return tmp; // 2. 调用拷贝构造函数复制一个实例用于返回
} // 3. 函数内部变量tmp析构

int main() {

    Person&& a = test(); // 直接接管函数返回的值，不需要调用构造函数了。
    return 0;
}

程序的输出为：
constructer!
copy construct!
deconstructer!
deconstructer!
```  

再进一步，类的拷贝构造函数如果也改为拷贝构造函数，即变为下面的这种形式，则虽然程序的输出仍然会是上面的四个步骤，但是在拷贝构造的过程当中，将不再出现类实例的整体构造与复制过程，新的类实例直接接管老实例的内部变量，无需重复生成。  

```cpp
class Person {

public:
    Person(int i) {
        cout << "constructer!"  << endl;
    }

    Person(const Person&& p) { // 此处变为右值引用
        cout << "copy construct!" << endl;
    }

    ~Person() {
        cout << "deconstructer!" << endl;
    }
};
```  
  
在第一段代码中，我们提出了编译器优化的问题，它的一个例子是"参考3"博客中开篇的第一个实例，代码段：std::vector<int> v = readFile(); 其正常的实现逻辑是调用拷贝构造函数生成v，但在编译器优化下可以直接把readFile()返回值生成在v的空间上，免去拷贝的过程。但是，如果代码段变为std::vector<int> v; v = readFile(); 两句话，把拷贝构造变成了拷贝赋值，那么一般来说编译器就还是会有复制过程了。  

实际上对于上面的这个过程，如果不添加-fno-elide-constructors编译参数，其实会发现，程序的输出只有两行：  

```
constructer!  
deconstructer!  
```  

我的理解是，现代编译器已经能够预知返回值最终的使用场景，即return的变量最终会赋值给main函数中的a变量，因此在return的过程中，直接将内部生成的tmp变量交由a接管，无需调用任何一次拷贝构造函数。经过个人实验，其效果等同于以下的程序：  

```cpp
#include <iostream>

using namespace std;

class Person {

public:
    Person(int i) {
        cout << "constructer!"  << endl;
    }

    Person(const Person&& p) {  // 使用右值引用
        cout << "copy construct!" << endl;
    }

    ~Person() {
        cout << "deconstructer!" << endl;
    }
};

Person& test(){  // 返回左值引用
    Person tmp(1);
    return tmp;
}

int main() {

    Person& a = test();
    return 0;
}

输出：
constructer!
deconstructer!
```


2. 延长临时值的生命周期，这一点就如上面的第二段代码，下面截取了一小段，此时main函数直接接管了函数的返回值，则返回值不会被析构，且其有效作用域与a的作用域相同了。     
   
```cpp
int main() {

    Person&& a = test(); // 直接接管函数返回的值，不需要调用构造函数了。
    return 0;
}
```  

3. 完美转发  

C++11引入了完美转发：在函数模板中，完全依照模板的参数的类型（即保持参数的左值、右值特征），将参数传递给函数模板中调用的另外一个函数。C++11中的std::forward正是做这个事情的，他会按照参数的实际类型进行转发。(即完美转发就是创建一个函数，该函数可以接收任意类型的参数，然后将这些参数按原来的类型转发给目标函数。原因是左值只能接收变量不能接收值，之前无法实现完美转发的原因见参考5) ，我们现在看下面的例子：  

```cpp
void processValue(int& a){ cout << "lvalue" << endl; }
void processValue(int&& a){ cout << "rvalue" << endl; }
template <typename T>void forwardValue(T&& val)
{
    processValue(std::forward<T>(val)); //照参数本来的类型进行转发。
}
void Testdelcl()
{   
    int i = 0;
    forwardValue(i); //传入左值 
    forwardValue(0);//传入右值 
}
输出：
lvaue 
rvalue
```  

std::forward的一个实现如下，即对变量进行转换。这里需要注意的是，<span style="color:red">一个右值引用被赋值之后将变为左值</span>，因此在这里的static_cast保证了在转发的时候右值还是右值，而不会因为右值引用被赋值之后成为左值的问题：  

```cpp
// 目标函数
void foo(const string& str);   // 接收左值
void foo(string&& str);        // 接收右值

template <typename T>
void wrapper(T&& param)
{
    foo(std::forward<T>(param));  // 完美转发
}


template<typename T> 
T&& forward(typename remove_reference<T>::type& param) 
{
    return static_cast<T&&>(param);
}
```

# 右值所有权的转移(std::move)  

这一点在智能指针unique_ptr的时候也用到了，用于转移其所有权，且在转移后原始的变量不再拥有值的所有权，unique_ptr变为空，在这里的右值转移之后也为空。swap(a,b)借助std::move的一种实现如下：  

```cpp
template <typename T>
void swap(T& a, T& b)
{
    T tmp{a};  // 调用拷贝构造函数，这里有可被优化的资源消耗存在
    a = b;     // 复制赋值运算符
    b = tmp;     // 复制赋值运算符
}

// 使用std::move进行优化：

template <typename T>
void swap(T& a, T& b)
{
    T temp{std::move(a)};   // 调用移动构造函数
    a = std::move(b);       // 调用移动赋值运算符
    b = std::move(tmp);     // 调用移动赋值运算符
}
```

> 可以这么理解，std::move的语义其实是将变量（左值）转换为了右值，然后对其所有权进行了移动。

# 使用const的情况下，即可以接收左值也可以接收右值。这是一个万金油的类型  

```cpp
void fun(const int& clref)
{
    cout << "l-value const reference\n";
}
```  

但是，这么做也存在问题，即其由于是const类型，因此在使用上其实是受到了约束的，哪怕传入的是一个左值引用，也不能改变其内部的值，这在使用上是受限的，而且常量左值也不能赋值给非常量右值，具体可以看参考5中的文章，在谈及完美转发之前的问题的时候有提到为什么const int&不是完美的转发。

# 最后，一段程序简要看看右值引用与左值引用  

```cpp
// 假设Class1为有定义的类型
Class1 f1(){...}
void f2(const Class1&){...}
void f2(Class1&&){...}
int main(){
    Class1 c1;
    f2(c1); //调用f2(Class1&)
    f2(f1()); //调用f2(Class1&&)
}
```

# 参考  

1. <a href="https://cloud.tencent.com/developer/article/1062982"> [专业技术]从四行代码看右值引用，原始来源：《程序员》2015年1月刊 </a>
2. <a href="https://www.jianshu.com/p/27ad3af4033b"> C++ 右值引用 </a>
3. <a href="https://toutiao.io/posts/rseuap/preview">漫谈 C++11 利器之右值引用（move 语义 & Perfect Forwarding）</a>
4. <a href="https://blog.csdn.net/cxsmarkchan/article/details/50792007"> swap函数的高效实现：右值引用和move </a>
5. <a href="https://blog.csdn.net/ink_cherry/article/details/74573225"> C++完美转发 </a> // 此文中列出了7中转发方案并解释了为何右值引用才能真正解决此问题，写的很好