---
type: article
title:  "C++ 11 智能指针"
categories: C++
tags:  C++  
author: DengYuting
---
* content
{:toc}

# 主要优势

> 自动管理指针多引用的情况，在没有引用的时候清空指针和所指对象，类似于Java垃圾回收机制，从而无需手动delete

# 指针概述  
> 包含在头文件 #include \<memory\>中，主要为三类指针：  

1. shared_ptr 共享指针/强指针，拥有所指对象的指针，当此指针被销毁时，所指的对象也会被销毁
2. weak_ptr 弱指针，由强指针管理，本身不拥有对象，作用为避免出现指针回路造成的内存泄露（后面讲）
3. unique_ptr 独占指针，一个对象只能被一个unique_ptr拥有，能够转移所有权，超出作用域后释放，但是可以作为return value传递回调用的函数  

<!--more-->

# 使用时注意
智能指针不是C++内置的，需要遵循一些原则来保证不会出现错误。  
1. 所指向的对象需要可以通过new/delete来动态申请和销毁，不要将其指向函数栈中的内容。
2. 每个对象只能被一个manager object管理单元管理，因为当一个管理单元销毁时会同时销毁指向的对象，出现多个管理单元时会出现多次delete导致内存泄露。可以(1)在对象创建时就赋值给智能指针。(2)使用make_shared来创建新的shared_ptr指向这个对象。
3. 可以使用get()函数来获取原始的指针，但是不建议这么使用，如果需要转换类型的话可以使用static_cast，后面会讲。  

# Shared_ptr  
---  
同一个对象可被多个shared_ptr指针指向。  
创建的时候包含一个manager object（管理单元）用来计算指向这个对象的指针个数，同时包含shared count（指向该元素的shared_ptr的个数）和 weak count（指向该元素的weak ptr的个数）  

![](https://raw.githubusercontent.com/acall-deng/acall-deng.github.io/master/_posts/_img/2019-11-12-cpp-smart_ptr/2019-11-12-cpp-shared_ptr_1.png)

## 基本概述  
每个shared_ptr在创建的时候会产生一个manager object（管理单元），用来处理引用计数的信息，当shared count为0时对象会被销毁但是管理单元保留，当shared count和weak count同时为0时管理对象也会被销毁

## 代码使用  

当超过作用域的时候 shared_ptr会释放，指向的元素的shared count会减一，当为0时delete指向的对象。  

```cpp
class Thing {
    public:
        void defrangulate();
};
ostream& operator<< (ostream&, const Thing&);
...
// a function can return a shared_ptr
shared_ptr<Thing> find_some_thing();
// a function can take a shared_ptr parameter by value;
shared_ptr<Thing> do_something_with(shared_ptr<Thing> p);
...
void foo()
{
    // the new is in the shared_ptr constructor expression:
    shared_ptr<Thing> p1(new Thing);
    ...
    shared_ptr<Thing> p2 = p1; // p1 and p2 now share ownership of the Thing
    ...
    shared_ptr<Thing> p3(new Thing); // another Thing
    p1 = find_some_thing(); // p1 may no longer point to first Thing
    do_something_with(p2);
    p3->defrangulate(); // call a member function like built-in pointer
    cout << *p2 << endl; // dereference like built-in pointer
    // reset with a member function or assignment to nullptr:
    p1.reset(); // decrement count, delete if last
    p2 = nullptr; // convert nullptr to an empty shared_ptr, and decrement count;
}
```  


## 复制对象（3种方式）  


```cpp
class Base {};
class Derived : public Base {};
...
shared_ptr<Derived> dp1(new Derived); // 1.创建对象赋值，会出现两个内存申请，一次为Derived对象，一次是shared_ptr的管理单元
shared_ptr<Base> bp1 = dp1;  // 2.直接赋值
shared_ptr<Base> bp2(dp1);  // 3.赋值
shared_ptr<Base> bp3(new Derived);
```  
### 使用make_shared优化内存分配  

```cpp
shared_ptr<Thing> p(make_shared<Thing>()); // only one allocation!
shared_ptr<Thing> p (make_shared<Thing>(42, "I'm a Thing!"));  // 带参数的  
```  


> <font color=##FF0000>此时需注意：</font>下面的代码中，shared_ptr<Base> bp3(new Derived);这一句将一个Derived对象给了Base对象的指针。  
这样的后果是：manager object在delete这个对象的时候会调用Derived类的析构函数，但是使用get方法获取原始指针的时候会返回Base类的指针。

上述提到的两次申请内存原因是在创建对象先后顺序的不同导致的，但是，在使用了make_shared变为一次申请内存之后，会导致被指向的对象需要在所有的weak_ptr全部销毁之后才会被delete，本来只需要shared count=0就会销毁，延长了对象停留在内存中的时间，在资源敏感型的程序中需要注意：  

- shared_ptr<Thing> p(new Thing); // 申请了两次内存    
  
![](https://raw.githubusercontent.com/acall-deng/acall-deng.github.io/master/_posts/_img/2019-11-12-cpp-smart_ptr/2019-11-12-cpp-shared_ptr_2.png)   

- shared_ptr<Thing> p(make_shared<Thing>());  // 只申请一次内存    

![](https://raw.githubusercontent.com/acall-deng/acall-deng.github.io/master/_posts/_img/2019-11-12-cpp-smart_ptr/2019-11-12-cpp-shared_ptr_3.png)    


## 替代get方法进行类型转换

```cpp
shared_ptr<Base> base_ptr (new Base);
shared_ptr<Derived> derived_ptr;
// if static_cast<Derived *>(base_ptr.get()) is valid, then the following is valid:
derived_ptr = static_pointer_cast<Derived>(base_ptr);
```  

# Weak_ptr  
---  

## 创建用法  
只能从shared_ptr中创建  

```cpp
shared_ptr<Thing> sp(new Thing);
weak_ptr<Thing> wp1(sp); // construct wp1 from a shared_ptr
weak_ptr<Thing> wp2; // an empty weak_ptr - points to nothing
wp2 = sp; // wp2 now points to the new Thing
weak_ptr<Thing> wp3 (wp2); // construct wp3 from a weak_ptr
weak_ptr<Thing> wp4
wp4 = wp2; // wp4 now points to the new Thing.
```  

## 使用weak_ptr 引用对象  
由于weak_ptr不能直接引用，所以需要使用lock函数取出引用的对象后赋值给shared_ptr变量，在lock取出对象指向该对象在使用完毕之前不会被释放，也可以使用wp.expired()来判断对象是否被释放。   

```cpp
shared_ptr<Thing> sp2 = wp2.lock();  // 取出对象，并且在使用完毕之前不释放这个对象,如果对象已经没了，sp2=false

if(wp.expired()) {  //使用expired方法判断指向的对象是否还存在
```  

> <font color=##FF0000>此时需注意：</font>在多线程的程序中，判断expired和lock的间隙中对象仍有可能被释放，因此建议使用： expired判断 -> shared_ptr = lock() -> 判断shared_ptr是否为空  

当通过weak_ptr来作为shared_ptr构造函数的参数时，如果weak_ptr已经expired，则会抛出异常bad_weak_ptr

```cpp
void do_it(weak_ptr<Thing> wp){
    shared_ptr<Thing> sp(wp); // construct shared_ptr from weak_ptr
    // exception thrown if wp is expired, so if here, sp is good to go
    sp->defrangulate(); // tell the Thing to do something
}
...
try {
    do_it(wpx);
}
catch(bad_weak_ptr&)
{
    cout << "A Thing (or something else) has disappeared!" << endl;
}
```

## 特殊情况（类内this指针，使用enable_shared_from_this）  

问题代码：  
```cpp
class Thing {
    public:
        void foo();
        void defrangulate();
};

void transmogrify(shared_ptr<Thing>);
int main()
{
    shared_ptr<Thing> t1(new Thing); // start a manager object for the Thing
    t1->foo();
    ...
    // Thing is supposed to get deleted when t1 goes out of scope
}
...
void Thing::foo()
{
    // we need to transmogrify this object
    shared_ptr<Thing> sp_for_this(this); // danger! a second manager object!
    transmogrify(sp_for_this);
}
...
void transmogrify(shared_ptr<Thing> ptr)
{
    ptr->defrangulate();
    /* etc. */
}
```  

其中存在的问题是，当调用foo函数时，语句shared_ptr<Thing> sp_for_this(this); 创建了一个新的shared_ptr指向this的类，从而在此段代码中，main函数中的t1和foo函数中的sp_for_this都是指向了同一个Thing对象但是却有两个manager object，从而在析构的时候会出现二次delete导致内存移除，正确的想法是下面这张图，通过使用一个weak_ptr指向不增加shared count.
  
![](https://raw.githubusercontent.com/acall-deng/acall-deng.github.io/master/_posts/_img/2019-11-12-cpp-smart_ptr/2019-11-12-cpp-weak_ptr_1.png)  

修改后的代码如下所示，在需要使用this指针来创建新ptr的情况下需要继承enable_shared_from_this<Thing>类，从而在函数中使用shared_from_this()来创建一个weak_ptr，进一步获取到其所指的对象。  

```cpp
class Thing : public enable_shared_from_this<Thing> {
    public:
    void foo();
    void defrangulate();
};
int main()
{
    // The following starts a manager object for the Thing and also
    // initializes the weak_ptr member that is now part of the Thing.
    shared_ptr<Thing> t1(new Thing);
    t1->foo();
    ...
}
...
void Thing::foo()
{
    // we need to transmogrify this object
    // get a shared_ptr from the weak_ptr in this object
    shared_ptr<Thing> sp_this = shared_from_this();
    transmogrify(sp_this);
}
...
```  

# Unique_ptr  
---  

Unique_ptr所指的对象只能被一个指针指向，该指针指向的内容可以通过（1）std::move函数,(2)函数返回值的方式来转移所有权，但是不能通过复制的方式来获取到对象。例如下面的方法就是不允许的。  

```cpp
unique_ptr<Thing> p1 (new Thing); // p1 owns the Thing
unique_ptr<Thing> p2(p1); // error - copy construction is not allowed.
unique_ptr<Thing> p3; // an empty unique_ptr;
p3 = p1; // error, copy assignment is not allowed.
```

## 使用方法  

1. 通过直接构造的方法获取到一个unique_ptr  

```cpp
void foo ()
{
    unique_ptr<Thing> p(new Thing); // p owns the Thing
    p->do_something(); // tell the thing to do something
    defrangulate(); // might throw an exception
} // p gets destroyed; destructor deletes the Thing
```  

2. 通过返回值的方法接收一个unique_ptr并转移所有权  

```cpp
//create a Thing and return a unique_ptr to it:
unique_ptr<Thing> create_Thing()
{
    unique_ptr<Thing> local_ptr(new Thing);
    return local_ptr; // local_ptr will surrender ownership
}
void foo()
{
    unique_ptr<Thing> p1(create_Thing()); // move ctor from returned rvalue
    // p1 now owns the Thing
    unique_ptr<Thing> p2; // default ctor'd; owns nothing
    p2 = create_Thing(); // move assignment from returned rvalue
    // p2 now owns the second Thing
}
```  

3. 通过std::move方式转移所有权  

```cpp
unique_ptr<Thing> p1(new Thing); // p1 owns the Thing
unique_ptr<Thing> p2; // p2 owns nothing
// invoke move assignment explicitly
p2 = std::move(p1); // now p2 owns it, p1 owns nothing
// invoke move construction explicitly
unique_ptr<Thing> p3(std::move(p2)); // now p3 owns it, p2 and p1 own nothing
```  

## 其他  

1. unique_ptr可以包含在标准的容器里，使用起来没什么不同，移出容器内的指针（erase）或者是清空（clear）都会使得所指向的对象同时被销毁，如果有发生所有权转移的话，容器内可能出现空指针，注意判断。  
2. unique_ptr也有make_unique函数，但是由于它没有manager_object，因此这个操作不会带来任何性能上的提升


# 使用智能指针成环的情况(注意)  
---  


虽然智能指针在很多情况下可以解决动态内存管理的问题，但是仍会出现shared_ptr创建的类成环，导致在出了作用域之后仍然不释放对象的问题。例如：  

```cpp
#include <iostream>
#include <memory>
using namespace std;

class father;
class son;

class father {
public:
    father() {
        cout << "father !" << endl;
    }
    ~father() {
        cout << "~father !" << endl;
    }
    void setSon(shared_ptr<son> s) {
        son = s;
    }
private:
    shared_ptr<son> son;
};


class son {
public:
    son() {
        cout << "son !" << endl;
    }
    ~son() {
        cout << "~son !" << endl;
    }
    void setFather(shared_ptr<father> f) {
        father = f;
    }
private:
    shared_ptr<father> father;
};

void test() {
    shared_ptr<father> f(new father());
    shared_ptr<son> s(new son());
    f->setSon(s);
    s->setFather(f);
}

int main()
{
    test();
    return 0;
}
```  

> 此时，在程序结束之后只会出现 Father！ Son！这两个构造函数的输出，两个对象都没有被析构，原因是这两个类里面各有一个shared_ptr强指针相互指着，shared count不为0因此不会被释放。  

解决方案是把其中一个shared_ptr改为weak_ptr即可，例如下面把Father类的ptr改为了弱指针，那么在出了程序的作用域之后，会经过以下几个过程：  
1. test函数中shared_ptr<son> s 和 shared_ptr<father> f 首先销毁。  
   从而son的shared count=0，weak count=1  （father类中的弱指针指着）
   fahter的shared count=1，weak count=0  (son类中的强指针指着)  

2. 由于son中的shared_count = 0,因此son析构，类中的shared_ptr<father> father;也会被销毁，从而fahter的shared count=0，weak count=0，father也被析构  

3. 综上，最后打印出来的语句为  
   father !  
   son !  
   \~son !  
   \~father !  
   

```cpp
class father {
public:
    father() {
        cout << "father !" << endl;
    }
    ~father() {
        cout << "~father !" << endl;
    }
    void setSon(shared_ptr<son> s) {
        son = s;
    }
private:
    //shared_ptr<son> son;
    weak_ptr<son> son; // 用weak_ptr来替换
};
```  

# 最后  

综上可以看出，虽然指针可以避免大部分的问题，但是例如成环的情况。上例较为简单，一开始可以预见成环从而使用弱指针，但在程序较复杂规模较大的情况下是仍有可能出现问题的，使用时还是需要注意。  

# 参考   
- Using C++11’s Smart Pointers (David Kieras, EECS Department, University of Michigan)  
- <a href="https://blog.csdn.net/love_hot_girl/article/details/21161507"> CSDN: std::make_shared有啥用 </a>  
- <a href="https://blog.csdn.net/gcs6564157/article/details/70144846"> 用weak_ptr解决shared_ptr的环形引用问题 </a>