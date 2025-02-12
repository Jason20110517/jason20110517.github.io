---
layout: post
title: "从 C++98 到 C++20，寻觅甜甜的语法糖们"
date:   2024-2-3
tags: [C++,OI]
comments: true
author: jason20110517

---

>  谔谔，谔谔……

于是一时兴起，写了这么一篇博客，记录一些比较平时比较少见的 / 冷门的 / 超前的语法知识，供大家学习参考。

本人水平有限，若有遗漏或错误请大家毫不吝啬地指出，谢谢！

- 选取 OI 中可能用到的，但比较冷门的语法、函数。
- 以版本为线进行了梳理（日后有时间将会以其它角度再写一些内容）。
- 实用至上，尽量少地探究语法背后的本质。
- 将不断更新，完善。

如果不确定能不能用，请见 [《关于NOI系列活动中编程语言使用限制的补充说明》](https://www.noi.cn/xw/2021-09-01/735729.shtml)

upd on 2024/1/17: for saving


## 目录

### C++98

- algorithm 库
  - 一些常见且比较重要的函数
  - 一些比较冷门的函数
  - GNU 私货


- numeric 库
- functional 库
- cmath 库
- `__builtin` 家族


#### C++11

- 零零散散的语法糖
  - auto
  - lambda 表达式
  - range-for
- STL
  - emplace
  - shrink_to_fit
- algorithm 库
- numeric 库
- iterator 库
- functional 库
- random 库
- cmath 库

#### C++14

- 零零散散的语法糖
  - auto
- functional 库

#### C++17

- 零零散散的语法糖
  - 结构化绑定
  - 类模板的实参推导
  - if 和 switch 中进行初始化
  - using 多个名称
- STL
  - set/map 的 merge
- algorithm 库
- numeric 库
- iterator 库
- cmath 库

#### C++20

- 零零散散的语法糖
  - 三路比较运算符
  - range-for 的初始化语句和初始化器
  - 规定有符号整数以补码实现
- STL
- bit 库
- ranges 库
- numbers 库





## C++ 98
### algorithm 库

`algorithm`  中有大量的函数，这些函数都位于命名空间 `std` 中。

最常见的有 `max/min`、`swap`、`sort`、`unique`、`lower_bound/upper_bound`、`reverse`。

#### 一些常见且比较重要的函数

- `find(bg,ed,val)`

  - 返回指向第一个等于 $val$ 的元素的指针。
  - 时间复杂度 $O(n)$。**后文所有序列操作的函数，都以 $n$ 代指 $ed-bg$。**

- `fill(bg,ed,val)`

  - 将 $[bg,ed)$ 之间的所有元素赋值为 $val$。

  - 复杂度为 $O(n)$，常数略大于 `memset`。

    在 $val=0/-1/\texttt{0x3f}$  时常数与 `memset` 相同。（存疑）

  - 常用于弥补 `memset` 无法赋值的问题，如赋值一个数组为 $1$。

- `copy(bg1,ed1,bg2)`

  - 将 $[bg_1,ed_1)$ 中的元素复制到 $bg_2$。
  - 复杂度 $O(n)$。

- `stable_sort(bg,ed)`

  - 对 $[bg,ed)$ 进行 **稳定** 排序。

  - 时间复杂度 $O(n\log n)$，要 $O(n)$ 的额外空间。

    当空间不足时使用 $O(n\log^2n)$ 的双调排序。

- `nth_element(bg,md,ed)`：

  - 将 $[bg,ed)$ 中的内容重新分配顺序，小于（等于） `md` 的元素在 $[bg,md)$，大于（等于） `md` 的元素都在 $(md,ed)$。
  - 时间复杂度 $O(n)$，需要 $O(n)$ 的额外空间。
  - 第四个参数为比较函数，若不传则默认 `less<>()`。
  - 常用于线性求序列中的第 $k$ 大，常用于实现替罪羊树。

- `max_element/min_element(bg,ed)`

  - 返回指向 $[bg,ed)$ 中最大 / 最小的元素的指针。

  - 时间复杂度 $O(n)$。

  - 第三个参数可传入比较函数。

  - 求数组最大值就可以直接写：`*max_element(a+1,a+n+1)`

- `random_shuffle(bg,ed)`

  - 打乱 $[bg,ed)$ 中元素的顺序。
  - 时间复杂度 $O(n)$。
  - 第三个参数传入一个函数 `func(int n)`，这个函数的返回值是一个 $[1,n]$ 中的随机整数。
  - 在未传入第三参数时，若 $ed-bg>\texttt{RAND\_MAX}$ 那么其**随机性将无法保证**。
  - 在 `C++14` 中被弃用，在 `C++17` 中被废除，`C++11` 之后都**应当以 `shuffle` 替代之**。

- `next_permutation(bg,ed)`

  - 将 $[bg,ed)$ 更改为下一个排列，并返回 $1$。

    若 $[bg,ed)$ 已经是最后一个排列，那么返回 $0$。

    下一个排列的定义为字典序严格大于当前排列的下一个排列。

  - 时间复杂度 $O(n)$。

  - 事实上 $[bg,ed)$ 不一定要是排列，可以存在相同元素。

  - 常用于枚举全排列：

    ```cpp
    iota(p,p+n,0);
    do{
     //do something   
    }while(next_permutation(p,p+n));
    ```

  - `prev_permutation(bg,ed)` 可以求出上一个排列。




#### 一些比较冷门的函数

- `merge(bg1,ed1,bg2,ed2,bg3)`

  - $[bg_1,ed_1)$ 和 $[bg_2,ed_2)$ 是两个有序序列，对其进行归并并存入 $bg_3$。

    **不能够原地归并**，若要原地归并请使用 `inplace_merge`。

  - 时间复杂度 $O(ed_1-bg_1+ed_2-bg_2)$。

  - 可以传入第六参数作为比较函数。

- `inplace_merge(bg,md,ed)`

  - 将 $[bg,md)$ 和 $[md,ed)$ 归并排序，并存入 $bg$。

  - 时间复杂度 $O(n)$，需要 $O(n)$ 的额外空间。

    当空间不足时使用 $O(n\log n)$ 的原地排序。

    常数较小，可能比手写的还快。

  - 可以传入第四个参数作为比较函数。

  - 常用于 CDQ 分治等分治算法，非常好用。

- `count(bg,ed,val)`

  - 返回 $[bg,ed)$ 中等于 $val$ 的元素的个数。
  - 时间复杂度 $O(n)$。

- `count_if(bg,ed,func)`

  - 返回 $[bg,ed)$ 中使得函数 `func` 返回值为 `true` 的元素的个数。
  - 时间复杂度 $O(n\times f)$，$f$ 为 `func` 的复杂度。
  - 常用的 `func` 有：`islower/isupper/isdigit` 等。

- `replace(bg,ed,val1,val2)`

  - 将 $[bg,ed)$ 中所有等于 $val_1$ 的元素替换为 $val_2$。
  - 时间复杂度 $O(n)$。

- `for_each(bg,ed,func)`

  - 对 $[bg,ed)$ 中所有元素执行函数 `func`。
  - 时间复杂度 $O(n\times f)$。
  - 其实没啥用，就是能压行。

- `transform(bg1,ed1,bg2,bg3,func)`

  - 对 $[bg_1,ed_1)$ 和 $[bg_2,bg_2+ed_1-bg_1)$ 中的元素依次执行二元函数 `func`，并将返回值存入 $bg_3$。
  - 时间复杂度 $O(n\times f)$。

- `rotate(bg,md,ed)`

  - 将 $[bg,ed)$ 循环至 $md$ 处元素位于 $bg$。

  - 时间复杂度 $O(n)$

  - 例子：

    ```cpp
    vector<int>a={1,2,3,4,5};
    rotate(a.begin(),a.begin()+1,a.end());
    //a={2,3,4,5,1}
    vector<int>b={1,2,3,4,5};
    rotate(b.rbegin(),b.rbegin()+1,b.rend());
    //b={5,1,2,3,4}
    ```



#### GNU 私货

有一些以双下划线开头的函数并未在 C++ 标准中，是 GNU 的私货，在 NOI 系列赛事中可以使用。

- `__lg(x)`
  - 返回 $\lfloor \log_2x\rfloor$ 。
  - 时间复杂度 $O(1)$。
  - 常用于实现倍增、二进制运算等。
- `__gcd(x,y)`
  - 返回 $\gcd(x,y)$。
  - 复杂度是对数级别的，常数较小。
  - 注意，返回值的符号 **不一定** 是正。
  - 在 C++17 之前都是很常用的。




### numeric 库

这里的函数，真的很好用。

- `accumulate(bg,ed,val)`

  - 将 $[bg,ed)$ 中的所有所有元素与初始值 $val$ 相加，返回这个和。

  - 时间复杂度 $O(n)$。

  - 可以传入第四个参数作为加法。

  - 可以用于求序列和，但注意，该函数返回值与 $val$ 类型一致，意味着你要注意 `long long` 的问题：

    ```cpp
    accumulate(bg,ed,0);//返回值是 int，可能溢出
    accumulate(bg,ed,0ll);//返回值是 long long
    ```

- `inner_product(bg1,ed1,bg2,val)`

  - 将 $[bg_1,ed_1)$ 和 $[bg_2,bg_2+ed_1-bg_1)$ 对应位置一一相乘再与初始值 $val$ 相加，返回这个和。
  - 时间复杂度 $O(n)$。
  - 可以传入第五、六个参数分别作为加法和乘法。
  - 用于做向量内积。

- `partial_sum(bg1,ed1,bg2)`

  - 对 $[bg_1,ed_1)$ 做前缀和并存入 $[bg_2,bg_2+ed_1-bg_1)$。
  - 时间复杂度 $O(n)$。
  - 可以传入第四个参数作为加法。
  - 可以原地求前缀和。

- `adjacent_difference(bg1,ed1,bg2)`

  - 对 $[bg_1,ed_1)$ 求差分并存入 $[bg_2,bg_2+ed_1-bg_1)$。
  - 时间复杂度 $O(n)$。
  - 可以传入第四个参数作为减法。
  - 可以原地差分。




### functional 库

常见的函数有 `less<>/greater<>` 等。

事实上，大部分运算 / 比较也在这里：

```cpp
plus<>;//x+y
minus<>;//x-y
multiplies<>;//x*y
divides<>;//x/y
modulus<>;//x%y
negate<>;//-x

equal_to<>;//x==y
not_equal_to<>;//x!=y
greater<>;//x>y
less<>;//x<y
greater_equal<>;//x>=y
less_equal<>;//x<=y

logical_and<>;//x&&y
logical_or<>;//x||y
logical_not<>;//!x

bit_and<>;//x&y
bit_or<>;//x|y
bit_xor<>;//x^y
//注意 bit_not(~x) 是 C++14 的哦~
```



### cmath 库

- `fabs(x)`

  - 返回 $|x|$。
  - 注意，对浮点运算请都使用 `fabs` 而不是 `abs`，因为有可能你调用的是 `abs(int)`。

- `fmod(x,y)`

  - 返回 $x\bmod y=x-y\lfloor\frac xy\rfloor$。
  - 在一些三角函数的地方可能会用到对 $\pi$ 取模。

- `exp(x)`

  - 返回 $e^x$。
  - 在 `double` 内，$x$ 的有效区间为 $[-708.4,709.8]$。

- `log(x)`

  - 返回 $\ln x$。

  - 当 $x\in (-\infty,0]$ 时报错。

  - 对数家族还有：`log10(x)` 和 `log2(x)`。

    请注意，`log2(x)` 是 C++11 的。

- `ceil(x)`

  - 返回 $\lceil x\rceil$
  - 其返回值依旧是浮点类型，输出时注意格式。

- `floor(x)`

  - 返回 $\lfloor x\rfloor$
  - 其返回值依旧是浮点类型，输出时注意格式。





### `__builtin` 家族

这里的内容并不在 C++ 标准中，全部都是 GNU 的私货，若使用其它编译器则可能无法通过编译。

如果 $x$ 的类型是 `long long`，请务必使用 `__builtin_xxxll(x)`（如 `__builtin_popcountll(x)`），否则将可能造成 FST 等严重后果。

- `__builtin_popcount(x)`

  - 返回 $x$ 在二进制下 $1$ 的个数。

  - 时间复杂度有说 $O(\log\log x)$ 的，也有说 $O(1)$ 的。

    一定比手写的快。

- `__builtin_parity(x)`

  - 返回 $x$ 在二进制下 $1$ 个数的奇偶性。
  - 时间复杂度 $O(1)$，快于 `__builtin_popcount(x)&1`。

- `__builtin_ffs(x)`

  - 返回二进制下最后一个 1 是从后往前第几位。
  - 时间复杂度 $O(1)$。

- `__builtin_ctz(x)`

  - 返回二进制下后导零的个数，$x=0$ 时UB。
  - 时间复杂度 $O(1)$。

- `__builtin_clz(x)`

  - 返回二进制下前导零的个数，$x=0$ 时UB。
  - 时间复杂度 $O(1)$。



## C++ 11

从 C++98 到 C++11 是一次重大的飞跃，许许多多非常实用的函数与语法如雨后春笋般出现。

### 零零散散的语法糖

#### auto

`auto` 在 C++98 中是一个较偏僻冷门的东西，因此在 C++11 中直接将其抛弃，并赋予其新生。

- `auto` 可以用来声明一个变量，其类型由初始化的内容定义。

  ```cpp
  auto a=1;//a is int
  auto b=2.0;//b is double
  set<int>s;
  auto it=s.end();//it is set<int>::iterator
  ```

  因此 `auto` 声明变量时必须有初始化内容，否则将 CE。

  同时，也可以声明为其它变量的引用：

  ```cpp
  auto x=true;//x is bool
  auto&y=x;//y is reference of x
  y=false;
  if(x)
      throw;//will not run
  ```

  常用于声明一些类型名很长的变量，如：`set<pair<int,int>,greater<pair<int,int>>>::iterator`

- `auto` 用于声明有固定返回值类型的函数，在 C++11 时需要尾随返回类，但在 C++14 及之后就不需要了：

  ```cpp
  auto func(double x)->double{
    return x*2-1;
  }//C++11
  
  auto func2(double x){
    return x*2-1;
  }//C++14
  ```

- 也可以用来声明一个 lambda 函数，见下文 lambda 表达式 部分。





#### lambda 表达式

lambda 相当于一个函数，其形式多变，但都是由若干部分组成：

```cpp
[captures](params)->T{body}
```

其中 `captures` 填捕获的方式，`params` 填传入的参数，`T` 填返回值类型，`body` 就是函数主体。

- `captures`

  有两种填法：`&` 和 `=`，不填时，局部的 lambda 将不会进行捕获，全局的 lambda 默认为 `&`。

  `&` 表示这个表达式捕获的内容是引用的形式，而 `=` 表示捕获的内容是复制且只读的。

  ```cpp
  int x=1;
  [=](){x=2;}();//CE,向只读变量‘x’赋值
  [&](){x=2;}();//OK,x will be 2
  //上面 lambda 最后的括号是对其进行调用
  ```

  这里有坑：请尽量使用 `&`，考虑如下程序片段：

  ```cpp
  vector<int>v(n),o(n);
  for(int&x:v)cin>>x;
  iota(o.begin(),o.end(),0);
  sort(o.begin(),o.end(),[=](int x,int y){return v[x]<v[y];});
  ```

  其作用是按 $v$ 的大小对其下标 $o$ 排序，但注意使用的是 `=`，这意味着每次调用 lambda 时都将 $v$ 拷贝了一份，一共拷贝 $n\log n$ 次，直接 TLE。

- `params`

  和普通函数一样。

- `T`

  若此处省略，则其返回值将为 `auto`：

  ```cpp
  [](int x){return x*2.0;}(3);//will return double 6.0
  ```

也可使用 `auto` 将 lambda 表达式作为一个可调用的函数：

```cpp
auto sqr=[](double x){
    return x*x;
};
```

那么每次调用 `sqr(r)` 都将返回 $r\times r$ 的值。

这样写的函数将自带 `inline`，比写 `inline` 短多了。



#### range-for

`for` 循环中轻松地遍历容器（`vector`、`set`、`list` 等）：

```cpp
vector<int>v={5,1,3,4,2};
for(int x:v)cout<<x<<' ';
//output:5 1 3 4 2
//按顺序遍历 v 中的每一个元素，并赋值给 x。
```

也可以将 $x$ 声明为引用：

```cpp
vector<int>v(n);
for(int&x:v)cin>>x;
//读入一个长度为 n 的序列
```

注意，$x$ 的类型必须与 $v$ 中元素的类型保持一致，否则会 CE。

也可以使用 `auto` 进行声明。

注意，若遍历数组，那将从头到尾遍历一遍，不推荐。

用处极为广泛，常用于对于 `vector` 存图的遍历等。



### STL 

#### emplace

在很多 STL 容器中都出现了一个新的函数——`emplace`，如：

```cpp
set<int>s;
s.insert(1);//C++98
s.emplace(1);//C++11

queue<int>q;
q.push(2);//C++98
q.emplace(2);//C++11

vector<int>v;
v.push_back(3)//C++98
v.emplace_back(3);//C++11

deque<int>dq;
dq.push_front(4)//C++98
dq.emplace_front(4);//C++11
```

`emplace` 相比原先的 `insert/push/push_back/push_front` 区别是，`emplace` 通过调用元素的构造函数来进行插入，所以它会比之前的函数更快一些。

因此也产生了使用上的区别：

```cpp
set<pair<int,int>>s;
s.insert(make_pair(1,2));//C++98
s.emplace(1,2);//C++11
```

更加便于书写。

但这要求用户自己的类型必须含有构造函数：

```cpp
struct A{
    int x;
};
queue<A>q;
q.emplace(1);//CE，A 没有构造函数

struct B{
    int x;
    B(int _x=0):x(_x){}
};
queue<B>p;
p.emplace(1);//OK，B 有构造函数
```



#### shrink_to_fit

`vector/deque/string/basic_string` 的 `shrink_to_fit` 可以使其 capacity 调整为 size 的大小，如：

```cpp
vector<int>v={1,2,3};
cout<<v.size()<<' '<<v.capacity()<<'\n';
v.clear();
cout<<v.size()<<' '<<v.capacity()<<'\n';
v.shrink_to_fit();
cout<<v.size()<<' '<<v.capacity()<<'\n';
/*
output:
3 3
0 3
0 0
*/
```

常用于 `clear()` 之后释放内存。



### algorithm 库

- `shuffle(bg,ed,gen)`
  - 打乱 $[bg,ed)$ 的顺序，`gen` 是一个随机数生成器（mt19937）。
  - 时间复杂度 $O(n)$。
  - C++11 之后请尽量使用 `shuffle` 而不是 `random_shuffle`
- `is_sorted(bg,ed)`
  - 判断 $[bg,ed)$ 是否升序排序。
  - 时间复杂度 $O(n)$。
- `minmax(a,b)`
  - 返回一个 `pair<>`，其 `first` 为 $\min(a,b)$，`second` 为 $\max(a,b)$。
  - 时间复杂度 $O(1)$。
  - 常用于无向图存边的去重问题。
- `max(l)/min(l)`
  - $l$ 是一个初始化列表，这个函数返回 $l$ 中最大 / 最小的元素。
  - 时间复杂度 $O(size_l)$。
  - 在多个元素求最大 / 最小时非常好用：`max({a,b,c})`
- `minmax(l)`
  - $l$ 是一个初始化列表，这个函数返回一个 `pair<>`，其 `first` 为 $l$ 中最小的元素，`second` 为 $l$ 中最大的元素。
  - 时间复杂度 $O(size_l)$。
  - 若要求一个序列 / 容器中最小和最大的元素，请使用 `minmax_element`。
- `minmax_element(bg,ed)`
  - 返回一个 `pair<>`，其 `first` 为 $[bg,ed)$ 中最小的元素，`second` 为 $[bg,ed)$ 中最大的元素。
  - 时间复杂度 $O(n)$。




### numeric 库

- `iota(bg,ed,val)`
  - 将 $[bg,ed)$ 中的元素依次赋值为 $val,val+1,val+2,\cdots$
  - 时间复杂度 $O(n)$。
  - 常用于给并查集初始化。




### iterator 库

- `prev(it)/next(it)`

  - 返回迭代器 $it$ 的前驱 / 后继。

  - 复杂度与 `++/--` 的复杂度相同，取决于容器。

  - 可以更方便的实现许多内容：

    ```cpp
    set<int>s={1,2,3,4,5};
    auto i=s.lower_bound(3);
    ++i;
    auto j=i;
    --i;
    //这是很麻烦的
    auto j=next(i);//很方便
    ```

  - 请习惯在 C++98 环境下使用 `next` 的同学们注意：这会导致声明一个叫 `next` 的变量时出错。

- `begin(container)/end(container)`

  - 返回容器 $\texttt{container}$ 的  `begin()` 和 `end()`。

  - 复杂度取决于容器。

  - 作用就是相比原先要短一个字节：

    ```cpp
    set<int>t;
    auto i=t.begin();
    auto i=begin(t);//你比一比谁更短
    ```



### functional 库

- `function` 

  声明方式：`function<R(Args...)>f;`

  其中 `R` 为返回值，`Args...` 为形参，`f` 为名称。

  一个 `function` 可以作为函数直接使用。

  ```cpp
  function<int(int,int)>f[4]={
    [](int x,int y){return x+y;},
    [](int x,int y){return x-y;},
    [](int x,int y){return x*y;},
    [](int x,int y){return x/y;}
  };
  //声明一个数组，每个元素都是一个 function<int(int,int)>
  
  for(int i=0;i<4;++i)
      cout<<f[i](5,2)<<' ';
  //output:7 3 10 2
  ```

  常用来代替函数指针。




### random 库

用于代替垃圾的 `rand`。

- `mt19937` 

  - 一个类型，作为随机数生成器。

  - 其构造函数应传入一个数字作为随机种子，使用时用法和 `rand` 相同。

  - 生成 `unsigned int` 范围内的随机数。

    ```cpp
    mt19937 gen(123456);//123456 为随机种子
    int x=gen()%10000;//x will in [0,10000)
    uint32_t y=gen();//normally y will in [0,2^32)
    ```

  - 随机种子通常为时间相关，下面的非常常用，建议背过

    ```cpp
    mt19937 gen(chrono::system_clock::now().time_since_epoch().count());
    //chrono 在头文件 <chrono> 中
    ```

- `uniform_int_distribution<T>(L,R)`

  - 随机数发生器，每次调用时传入一个 `mt19937`，返回一个 $[L,R]$ 之间的整数，类型为 `T`，若 `T` 为空则默认为 `int`。

- `uniform_real_distribution<T>(L,R)`

  - 随机数发生器，每次调用时传入一个 `mt19937`，返回一个 $[L,R]$ 之间的实数，类型为 `T`，若 `T` 为空则默认为 `double`。

```cpp
mt19937 gen(chrono::system_clock::now().time_since_epoch().count());
uniform_int_distribution<>dist(1,1000);
int x=dist(gen);// x is a random int in [1,1000]
uniform_real_distribution<>dist2(-10,10);
double y=dist2(gen);// y is a random double in [-10,10]
```




### cmath 库

- `exp2(x)`
  - 返回 $2^x$。
- `log2(x)`
  - 返回 $\log_2(x)$。
- `hypot(x,y)`
  - 返回 $\sqrt{x^2+y^2}$。
  - 常用于求两点之间的距离，非常方便。
- `NAN`，`INFINITY`
  - 两个 `define` 出的量，由编译器实现定义。
