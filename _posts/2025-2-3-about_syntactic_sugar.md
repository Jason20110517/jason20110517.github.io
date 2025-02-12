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

