---
layout: post
title: "[洛谷日报#265]关于 C++ 未定义行为的一些事"
date:   2020-2-19
tags: [OI,C++]
comments: true
author: jason20110517
---

> Q：萌新求助，为啥不开 `-O2` 能 AC，开了 `-O2` 就 RE/TLE/WA 了？
>
> A：你的程序可能有 UB。
>
> Q：萌新求助，为啥本地数据能过，提交就全 RE/TLE/WA 了？
>
> A：你的程序可能有 UB。
>
> Q：所以啥是 UB？

未定义行为（Undefined Behavior，UB），是一类对程序**无任何限制**的行为。

在 C/C++ 中，未定义行为的例子非常地多，我们在下文将会提到一些例子。

## 1 和其他概念的辨析

在 C/C++ 中，和**未定义行为**容易混淆的概念有两个，**实现定义行为**，**未指明行为**。这里先对这几个概念做一个辨析。

- **实现定义行为**：程序的行为随实现而变动，**遵从标准的实现必须为每个这样的行为的效果提供文档**。一个例子是 `int` 在不同环境下的大 小（标准规定为**至少 16 位**，现在大多数环境下均为 32 位）。
- **未指明行为**：程序的行为随实现而变动，**而不要求遵从标准的实现为每个行为的效果提供文档**。虽然行为在变动，但它产生的结果均应该是合法的。一个例子是变量分配的方式和位置（可以把一次定义的不同变量分配到一片连续的空间，当然也可以分开分配）。
- **未定义行为**：对程序的行为**无任何限制**。前两类行为的结果都要求是合法的，而对于未定义行为，则不要求程序做任何合法，有意义的事情。一个例子是访问非法内存。

## 2 为什么会有未定义行为？

一个正常的 C/C++ 程序的行为都应该是合法的，像未定义行为这样的操作不应该在程序中出现。

有人也许会奇怪：那为什么不去检测未定义行为，将未定义行为视为语法或语义错误而终止编译呢？

事实上对未定义行为的检测并没有检测语法和语义错误那么容易。有些未定义行为的检测比较容易（比如访问未初始化的变量），但诸如带符号整数溢出这样的行为，因为它并不一定会发生，在编译阶段检测它就困难不少。

而假如编译器要考虑这些未定义行为的话，则不利于程序优化。

因此将一些操作指定为未定义行为，编译器就不必再考虑这些操作，从而利于程序优化。

这也是一些程序在没有优化的情况下行为正常，开启优化选项之后出现不期待的结果的原因。

## 3 一些未定义行为的例子

### 3.1 带符号整数算术溢出

```cpp
#include <iostream>
using namespace std;
int main()
{
 int x;
 cin>>x;
 if(x+1<x)
  cout<<"Overflow!"<<endl;
 else
  cout<<"Not overflow!"<<endl;
 return 0;
}
```

试着输入 2,147,483,647（2³¹-1）看看程序结果吧！

如果你编译的时候开启了 `-O2` 优化选项，或者你的编译器版本比较高的话，你可能会发现预期的输出 `Overflow!` 没有出现。

为什么呢？因为带符号整数溢出是未定义行为，从而编译器不会考虑这种情况。在忽略这种情况的前提下，`x+1<x` 一定为假，从而上面这段程序事实上与下面这段程序等价：

```cpp
#include <iostream>
using namespace std;
int main()
{
 int x;
 cin>>x;
 cout<<"Not overflow!"<<endl;
 return 0;
}
```

你也许会奇怪，为什么不将带符号整数溢出的行为给一个明确的定义呢？

事实上，给带符号整数溢出下定义会带来很多不必要的开销。

我们来看下面这个程序：

```cpp
#include <iostream>
using namespace std;
int main()
{
 int x;
 cin>>x;
 cout<<x*2/2<<endl;
 return 0;
}
```

如果带符号整数溢出是有定义的，那我们就要老实执行一次乘法和一次除法运算。

而把它定为未定义行为，编译器就可以直接优化成这样：

```cpp
#include <iostream>
using namespace std;
int main()
{
 int x;
 cin>>x;
 cout<<x<<endl;
 return 0;
}
```

瞬间减少了不少开销，对吧？当操作次数很多的时候效果当然更加明显。

需要注意的是，**无符号整数溢出不是未定义行为**。也就是说：

![](https://cdn.luogu.com.cn/upload/image_hosting/hu0gfpdv.png)

```cpp
#include <iostream>
using namespace std;
int main()
{
 unsigned x;
 cin>>x;
 if(x+1<x)
  cout<<"Overflow!"<<endl;
 else
  cout<<"Not overflow!"<<endl;
 return 0;
}
```

如果你输入 4,294,967,295（2³²-1），会发现它的行为是符合预期的（输出 `Overflow!`）。

### 3.2 越界访问

越界访问是一件让人很头大的事情。

众所周知，C/C++ 并不会进行越界检查，因此越界访问造成的后果可能有这几种：

- 访问非法内存而导致程序运行时错误（RE）；
- 意外访问程序里的其他变量导致 [暴力写挂](https://www.luogu.com.cn/problem/P4565)；

为啥不进行越界检查呢？在多数情况下，越界检查的成本并不小，而且进行了越界检查会丧失不少程序优化的机会。因此最划算的决定就是不进行越界检查。

### 3.3 无可视副作用的无限循环

在开始讲这个之前，先引出 C++ 的一个原则，**如同原则**（as if），这是不少优化的基础。

简单来说，在不影响程序运行结果的前提下，允许编译器进行一些代码转换。当然，未定义行为造成的影响是例外，因为这种行为不会被编译器考虑。

因此，可以通过消除冗余代码来达到优化的目的。

因为无可视副作用的无限循环是未定义行为，编译器有时可以直接将它优化掉。

```cpp
#include <iostream>
using namespace std;
int f()
{
 unsigned cnt=0;
 while(1)
  if(cnt<0)return true;
 return false;
}
int main()
{
 if(f())//停机问题（大雾
  cout<<"This program has been terminated."<<endl;
 else
  cout<<"Some strange things happened!"<<endl;
 return 0;
}
```

在 g++ 8.3 下进行编译，程序进入了死循环，而在 clang++ 6.0.0 下进行编译，程序则无输出终止。

查了下汇编代码，发现 `main()` 函数在 clang++ 下被优化成了空。

也就是说，这一无限循环因为是未定义行为，而被编译器视为冗余代码而消除。

但是，包含空语句的无限循环是个例外。自 C++26 标准起，其不再是为未定义行为。

```cpp
while(true); // 该循环自 C++26 标准起不再是未定义行为
// 上述循环的循环体会被替换为 yield() 函数调用，即交出当前线程执行权。
```

### 3.4 无法确定的运算顺序

现在据说还有不少教材还在考这种奇怪的东西。

```cpp
#include <iostream>
using namespace std;
int main()
{
 int x=1;
 cout<<(x++ + ++x)<<endl;
 return 0;
}
```

根据 C++11 起的 [“按顺序早于”规则](https://zh.cppreference.com/w/cpp/language/eval_order)，这种式子用一句比较拗口的话来说，就 是：标量对象上的一项副作用相对于同一标量对象上的另一副作用为无顺序，则其行为未定义。

在这个式子中，`x++` 的副作用和 `++x` 的副作用相比，可以在前面发生，也可以在后面发生，因此它是未定义的。

### 3.5 访问未初始化变量

```cpp
#include <iostream>
using namespace std;
int main()
{
 int x;
 if(x)
  cout<<"True"<<endl;
 if(!x)
  cout<<"False"<<endl;
 return 0;
}
```

据说在比较旧的 g++ 版本上编译会出现两个都输出的奇怪现象。不过这既然是未定义行为，怎么样的输出都是合理的。

## 4 如何检测未定义行为？

在编译期检测未定义行为难度并不小，原因在上文已经说的很清楚了。

不过在运行期，还是有一些方法来捕捉程序的未定义行为的。这将在一定程度上方便我们的调试。

一种非常实用的检测未定义行为的方式是使用 clang 的诊断模式。

只需在编译选项中加上一条：`-fsanitize=undefined` 即可开启。

（注：Linux 下的较新版本的 g++ 也提供这一编译选项，而 NOI Linux 自带的 g++ 4.8.4 并不支持该编译选项）

```cpp
#include <iostream>
using namespace std;
int a[10005];
int main()
{
 if(a[10005])
  cout<<42<<endl;
 return 0;
}
```

对于上面这个出现数组越界的程序，运行的结果将是：

```plain
a.cpp:6:5: runtime error: index 10005 out of bounds for type 'int [10005]'
```

PS：Codeforces 也支持了这个功能，可以在 Custom Test 中选择 Clang++17 Diagnostics 开启。

## 5 总结

作为 C/C++ 的一大特色，未定义行为让不少人都头疼不已。未定义行为不可预测的特点，使调试的难度加大了不少。

避开未定义行为的关键是养成良好的编程习惯。当然一些辅助的检测手段对于消除未定义行为也能起到非常大的帮助。

## Reference

- [未定义行为 - cppreference.com](https://zh.cppreference.com/w/cpp/language/ub)
- [每个 C 程序员都该知道关于未定义行为的事 #1/3](http://blog.llvm.org/2011/05/what-every-c-programmer-should-know.html)       
- [每个 C 程序员都该知道关于未定义行为的事 #2/3](http://blog.llvm.org/2011/05/what-every-c-programmer-should-know_14.html)    
- [每个 C 程序员都该知道关于未定义行为的事 #3/3](http://blog.llvm.org/2011/05/what-every-c-programmer-should-know_21.html)
