# C++ Templates 读书笔记
## 类模板的特化
可以用模板实参来特化类模板。和函数模板的重载类似，通过特化类模板，你可以优化基于某种特定类型的实现，或者克服某种特定类型在实例化类模板时多出现的不足。
* 为了特化一个类模板，你必须在起始处声明一个template<>，接下来声明用来特化类模板的类型。这个类型被用作模板实参，且必须在类名的后面直接指定。
```c++
template<>
class Stack<std::string> {
  ...
}
``` 
* 进行类模板的特化时，每个成员函数都必须重新定义为普通函数，原来模板函数中的每个T也相应地被进行特化的类型取代：
```c++
void Stack<std::string>::push (std::string const& elem)
{
    elems.push_back(elem);
}
```

## 包含模型(inclusion model)
按照cpp的习惯：
* 类和其他类型都被放在一个头文件中。
* 对于全局变量和(非内联)函数，只有声明放在头文件中，定义则位于dot-C文件。
我们习惯于作出以下编写：
```c++
//basic/myfirst.hpp
#ifndef MYFIRST_HPP
#DEFINE MYFIRST_HPP

//模板声明
template <typename T>
void print_typeof (T const&)

#endif //MYFIRST_HPP
```
```C++
//basics/myfirst.cpp
#include <iostream>
#include <typeinfo>
#include "myfirst.hpp"

//模板的实现/定义
template <typename T>
void print_typeof (T const& x)
{ 
    std::cout << typeid(x).name() << std::endl;
}
```
当我们在另一个dot-C文件里使用这个模板，并且把模板声明包含进这个文件：
```c++
//basics/myfirstmain.cpp
#include "myfirst.hpp"

//使用模板
int main()
{
    double ice = 3.0;
    print_typef(ice); // 调用参数类型为double的函数模板
}
```
大部分C++编译器都会顺利地接受这个程序，但是链接器可能会报错，提示找不到函数print_typeof()的定义。<br>
解决方法：使用包含模型或者分离模型
* 包含模型 (三种方法)<br>
(1)通过把
```c++
#include "myfirst.cpp"
```
添加到myfirst.hpp的末尾，(2)或者在每个使用模板的dot-C文件都包含myfirst.cpp。(3)再或者完全不要myfirst.cpp，然后重写myfirst.hpp，让它同时包含模板声明和模板定义。
* 分离模型(separation model)
又称导出模型(export template)
```c++
//使用export关键字 内容略
```
## 模板参数
### 类型参数(使用得最多)
类型参数是通过关键字typename或者class引入的：它们两者几乎是等同的。关键字后面必须是一个简单的标识符，后面用逗号来隔开下一个参数声明，等号(=)代表接下来的是缺省模板参数，一个封闭的尖括号(>)表示参数化字句的结束。
### 非类型参数
非类型参数表示的是：在编译期或者链接期可以确定的常值。这种参数的类型必须是下面的一种:
* 整形或者枚举类型。
* 指针类型（包含普通对象的指针类型、函数指针类型、指向成员的指针类型）。
* 引用类型（指向对象或者指向函数的引用都是允许的）
### 模板的模板参数
模板的模板参数是代表类模板的占位符（placeholder）。它的声明和类模板的声明很类似，但不能使用关键字struct和union。
```c++
template <template<typename T,
                  typename A = MyAllacator> class Container>
class Adaptation {
    Container<int> storage; //隐式等同于Container<int, MyAllocator>
    ...
};
```

### List参数
有时候，我们希望可以把具有几个类型的列表看成一个单一的模板实参，并用这个单一的实参进行传递。通常情况下，使用这种列表会有两个目的：声明一个参数个数不固定的函数，或者定义一种具有成员个数不固定的类型结构。
例如，我们希望定义一个能够计算任意多个值中最大者的模板。一种可能的声明语法是：使用省略号标记，从而说明最后一个模板参数的含义是允许匹配任意个数的实参。
```c++
#include <iostream>

template <typename T, ... list>
T const& max (T const, T const&, list const&);
int main()
{
    std::cout << max(1, 2, 3, 4) << std::endl;
}

template <typename T> inline
T const& max (T const& a, T const& b)
{
    return a < b ? b : a; // 我们用于求普通的二元最大值的操作
}

//模板函数的实现,使用重载函数模板
template <typename T, ... list> inline
T const& max (T const& a, T const& b, list const& x)
{
    return max(a, max(b, x));
}
```
调用max(1,2,3,4)的步骤：<br>
由于具有4个参数，所以具有二元参数的max()并不能匹配，于是就选择了参数为T=int与list = int，int的第2个模板。这等于调用第1个实参为1、第2个实参为max(2,3,4)的二元函数模板max()。接下来调用max(2,3,4)，这也不能和二元参数的max()进行匹配，于是我们要调用T=int与list=int的list参数版本。最后一次的子表达式是max(b,x),它可以扩展成max(3,4),于是选择二元模板，该递归结束。

## 模板的多态
C++动多态，是通过继承和虚函数来实现。借助模板的多态为静多态。
### 优点和缺点
#### C++动多态的优点
* 能够优雅地处理异类集合。
* 可执行代码的大小通常比较小（因为只需要一个多态函数，但对于静态语言而言，为了处理不同的类型，必须生成多个不同的模板实例）。
* 可以对代码进行完全编译；因此并不需要发布实现源码（但是，分发模板库通常都需要同时分发模板实现的源代码）。
#### C++静多态的优点
* 可很容易地实现内建类型的集合。更广义地说，并不需要通过公共基类来表达接口的共同性。
* 所生成的代码效率通常比较高（因为并不存在通过指针的间接调用，而且，可以进行演绎的非虚拟函数具有更多的内联机会）。
* 对于只提供部分接口的具体类型，如果在应用程序中只是使用到这一部分接口，那么也可以使用该具体类型；而不必在乎该类型是否提供其他部分的接口。

## trait与policy
policy类和trait是两种C++程序设计机制，它们有助于对某些额外参数的管理，这里的额外参数是指：在具有工业强度的模板设计中所出现的参数。
### fixed traits
求两个指针之间元素的总和
```c++
template <typename T>
inline
T accum(T const* beg, T const* end)
{
    T total = T();
    while(beg != end){
        total += *beg;
        ++beg;
    }
    return total;
}
```
我们的输出的类型是依据T，即我们实例化的时候，但当我们在操作完+后得到的total可能因为类型范围限制得不到我们期望的值，又或者说当我们想根据不同的T输出不同类型的total时，这样设计就十分受限。<br>
我们可以模板特化来解决：
```c++
template<typename T>
class AccumulationTraits;

template<>
class AccumulationTraits<char> {
    public:
      typedef int AccT;
};

template<>
class AccumulationTraits<short> {
    public:
      typedef int AccT;
};

template<>
class AccumulationTraits<int> {
    public:
      typedef long AccT;
};

template<>
class AccumulationTraits<unsigned int> {
    public:
      typedef unsigned long AccT;
};

template<>
class AccumulationTraits<float> {
    public:
      typedef double AccT;
};
```
在我们定义的时候
```c++
template <typename T>
inline
typename AccumulationTraits<T>::AccT accum(T const* beg, T const* end)
{
    // 返回值的类型是一个元素类型的trait
    typedef typename AccumulationTraits<T>::AccT AccT;
    AccT total = T();
    while(beg != end){
        total += *beg;
        ++beg;
    }
    return total;
}
```
我们把这个存储累加和的类型成为T的trait。
### policy和policy类
我们把累积和求和等价起来，针对accum的所有操作，唯一需要改变的只是total += \*beg操作。于是，我们就把这个操作成为该累积过程的一个policy。因此，一个policy类就是一个提供了一个接口的类，改接口能够在算法中应用一个或多个policy。
```c++
template <typename T, 
         typename Policy = SumPolicy,
         typename Traits = AccumulationTraits<T> >
class Accum {
    public:
        typedef typename Trait::AccT AccT;
        static Acct accum (T const* beg, T const* end) {
            AccT total = Traits::zero(); // 初始化的 value trait
            while (beg != end) {
                Policy::accumulate(total, *beg);
                ++beg;
            }
            return total;
        }
};
```
Policy类
```c++
class SumPolicy {
   public:
   template<typename T1, typename T2>
   static void accumulate (T1& total, T2 const & value) {
       total += value;
   }
};

class MultPolicy {
   public:
   template<typename T1, typename T2>
   static void accumulate (T1& total, T2 const & value) {
       total *= value;
   }
};
```
```c++
// 调用
Accum<int, MultPolicy>::accum(&num[0], &num[5]);
```
### trait和policy的区别
#### New Short Oxford English Dictionary 的定义
* trait n... （名词）：用来刻划一个事物的（与众不同的）特性
* policy n... （名词）：为了某种有益或者有利目的而采用的一些列动作。
#### Modern C++ Design 中的声明
* policy和trait具有许多共同点，但是policy更加注重于行为，而trait则更加注重于类型。
