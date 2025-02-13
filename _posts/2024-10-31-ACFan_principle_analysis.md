---
layout: post
title: "ACFan 原理解析"
date:   2024-10-31
tags: [洛谷,OI,C++,作弊]
comments: true
author: jason20110517
---
本文章将简单介绍ACFan的工作原理。

## 首先上一段代码

> Talk is cheap, show me the code.      ——莱纳斯·~~马克思~~

```cpp
#include <iostream>
using namespace std;

void WA() {printf("this_is_absolutely_a_wrong_answer_2333");}
void TLE() {while(1);}
void MLE() {while(1) new long long[2333];}
void RE() {throw;}
void OLE() {while(1) printf("i_am_a_junk_string_isnt_it");}

int main() {
    int a,b;
    cin>>a>>b;
    switch(a) {
        case 1: WA(); break;
        case 2: TLE(); break;
        case 3: MLE(); break;
        case 4: RE(); break;
        default: OLE(); break;
    }
    return 0;
}
```

请仔细阅读上面的程序，并试想一下：如果把这段代码提交到POJ的“简单A+B”题目中，结果是Wrong Answer（不用试了，真的是WA），那么能说明什么问题呢？

**对，可以依此断定代码中`case 1`分支被触发，从而推断出 POJ 的第一组数据的第一个数是“1”！**

类似地，把`switch(a)`改成`switch(b)`就可以获得（下文称这种通过提交代码来获得输入数据的行为为"dump"）POJ“简单A+B"题目的第一组数据的第二个数。

这便是ACFan最基础的工作原理：你的代码根据数据“遥控”运行结果（WA/TLE/MLE……），再通过运行结果反推数据。得到数据后你就可以手算出结果，或者用极慢的模拟算法在本地算出正确结果，从而通过该题目。

（改成`switch(b)`后结果是TLE，从而我们得到了这道题目第一组的全部测试数据："1 2"）

## 处理多组数据

你可能会说，光得到第一组数据有什么用？AC一道题需要通过**所有**数据呀。

别着急，我们来分析一下OJ是如何测评你的代码的。根据直觉，可以推测出OJ判题目的算法大致如下：

>1    编译你的代码，如果出错或出现不允许的指令，则返回CE或非法调用错误
>
>2    对于每组数据的 input 和 output：
>
>2.1    运行你的程序，输入input
>
>2.2    在运行过程中监视程序状态，如果超时、超内存、超输出限制、崩溃则相应返回TLE、MLE、OLE、RE
>
>2.3    判断输出是否正确，如果与样例输出不同则返回WA（如果仅空白字符不同则返回PE）
>
>3    返回AC

因此，只要对`main`略做修改，让其在正经代码（`switch`那块）之前做一个判断、pass掉第一组数据即可开始dump第二组数据：

```cpp
int main() {
    int a,b;
    cin>>a>>b;
    if(a==1 && b==2) {
        cout<<3<<endl;
        return 0;
    }
    //后面的switch不做修改
}
```

如果你感兴趣，可以用这种方法试着通过POJ上的“简单A+B”。

顺便说一下，POJ的“简单A+B”有5组测试数据，它们依次是：

>1  2
>
>1  4
>
>1  3
>
>3  3
>
>7  8


## ACFan的适用范围及缺陷

- 目标OJ必须能给出错误类型（WA还是MLE还是TLE还是什么），如果只给一个“Unaccepted”就没辙了。
- 每一组数据都必须是不变且顺序固定的。
- 只有你知道了第1至N组数据及它们的**正确**输出，才能开始获取下一组数据。说的通俗一点，就是“在你pass掉前一组数据之前，你对下一组数据一无所知”。
- 由于上面那条性质，你无法事先判断数据有几组。
- ACFan需要向OJ提交大量的代码才能dump出全部数据，因此ACFan不适用于数据量极大的题目。同理，提交答案需要输验证码的OJ也不太好办。
- 你用手算或用模拟方式都解不出来的题目，用ACFan也没用。

## 一个鲁棒性强的代码

上面贴的那些代码有一个问题，就是：它们只适用于“简单A+B”一个题目。就是说，当你面对一个新的问题时，你还得重头为它写一个dump代码。这种基于“人工智能”的代码可不好，我们需要一个**适用于所有题目**的！

首先抛开效率，上一个完整版的demo（这基本上就是ACFan目前正在使用的代码，为了可读性我稍微排了下版）：

```cpp
#include <cstdio>
#include <cstring>
#define _C if(unsigned(p)!=strlen(b)) return false

int p=0; //输入数据的字符串长度

void WA(){printf("this_is_absolutely_a_wrong_answer_2333");}
void TLE(){while(1);}
void MLE(){while(1)new long long[2333];}
void RE(){throw;}
void OLE(){while(1)printf("i_am_a_junk_string_isnt_it");}

bool OKfull(char *a,char *b){ //比较两个字符串是否相等，本来能用strcmp，但为了和OKnorm和OKlite保持一致性就没有使用
    _C; //首先比较a和b的长度是否一致，不一致则 return false
    for(int x=0;x<p;x++)
        if(a[x]!=b[x])
            return false;
    return true;
}
//本来这里还有OKnorm和OKlite两个函数，目的是提高效率、减少代码提交次数，后文会介绍，此处暂时删去

int main() {
    char i[625];
    while(scanf("%c",i+p)!=EOF) if(i[p]!='\r') p++; //将 \r\n 替换为 \n 的奇技淫巧
    i[p]='\0';
    
    //【这是模板代码，对于不同的目的，会把不同的代码塞到这个位置，下文会将此处称为“base位置”】
    
    return 0;
}
```

输入数据会被读到字符串`i`中，`p==strlen(i)`。

**要想获取输入数据的长度**，需要将以下代码插入到base位置：

```cpp
if(p<n+j) WA();
else if(p<n+2*j) TLE();
else if(p<n+3*j) MLE();
else if(p<n+4*j) RE();
else OLE();
```

其中`i`和`j`要由“攻击者”的电脑上的自动化程序（以下称为“framework”）算出。计算方式如下（python代码）：

```python
n,j=0,125
while j>=1:
    #【将把n,j带入后的代码提交到OJ】
    r=【从OJ获取到的结果，其中WA=0, TLE=1, MLE=2, RE=3, OLE=4】
    n, j = n+r*j, j//5
# 至此得到了输入数据长度：n
```

值得注意的是，如果`j`的初值为`L`，那么这段程序只适用于输入数据长度小于`5*L`的情况。

**要想获取第idx位输入数据的内容**，base位置代码如下：

```cpp
if(i[idx]<n+j) WA();
else if(i[idx]<n+2*j) TLE();
else if(i[idx]<n+3*j) MLE();
else if(i[idx]<n+4*j) RE();
else OLE();
```

framework部分的代码如下：

```python
n,j=5,25 #一般来说，每一个输入的字符的ASCII都在[5,130)之间
while j>=1:
    r=【带入n,j,idx，提交到OJ并获取结果】
    n, j = n+r*j, j//5
# 第idx位的数据的ASCII是i
```

**要想pass掉一组已知input和正确output的数据**，把这段代码加到base位置前面：

```cpp
if(OKfull(i, (char*)input)) {
    printf("%s",output);
    return 0;
}
```

注意：保证上述代码output正确性尤为重要，否则后面提交到OJ的所有代码的结果都会因此变成WA，从而干扰接下来所有dump结果。因此需要向OJ提交一段测试代码在pass每一组数据之前检验手算出来的output是否正确。代码略。

**framework部分的完整算法如下**：

>1    通过4次提交，获取输入数据的长度L
>
>2    对于 idx ← [1 .. L]：
>
>2.1    通过3次提交获取输入数据的第idx位
>
>3    至此全部input已被获取到，提示攻击者输入output
>
>4    通过1次提交检验output是否正确，如不正确跳回3，如已经Accepted了则退出
>
>5    将bypass代码加到base位置前面，回到1，开始dump下一组数据

至此，ACFan的概念验证完成。

## 提高运行效率，减少提交次数

通过分析上述代码，我们可以发现：使用这种“朴素”代码，要想通过一个样例数量为`N`、每组样例的输入长度为`L (L<625)`的题目，至少需要向OJ提交`N*(3*L+5)`次代码，其中为了pass每一组样例：

- 需要提交`4`次代码来获取输入长度`L`
- 需要提交`3*L`次代码来获取每一位数据
- 需要提交至少`1`次来验证用户手算出的结果是否正确

显然，这样的效率是极低的。低效率的坏处有：

- 慢
- 被管理员发现从而被封号的几率增加
- 由于OJ抽风（比如OJ判题系统卡壳导致MLE变TLE）导致dump结果有误的概率增加

所以，我们要想办法改进这种朴素的方式。显而易见，要想效率更高但不失鲁棒性，可以尝试优化**字符集**（原先的字符集是`[ascii 5 .. ascii 130)`，未免太大了点）。

- 比如，“简单A+B”及类似的问题的输入可能包括11种字符：`1,2,3,4,5,6,7,8,9,0,空白字符（空格或回车）`。这种情况我们称之为**lite字符集**。

对于lite字符集的输入，用三次提交就可以获取两个字节的内容，因为`5^3 >= 11^2`

- 有一些题目也会包括少量符号或16进制数。对于这些仅包括`0,1,2,3,4,5,6,7,8,9,空格,回车,X,+,-,*,/,=,.,A,B,C,D,E,F`（字母不区分大小写）这25种字符但不属于lite字符集的，我们称之为**norm字符集**。

对于norm字符集的输入，用两次提交就可以获取一个字节的内容。

- 其它情况不好优化，只能使用朴素算法。这样的情况被我们称作**full字符集**。

**通过以下代码可以判断输入数据的字符集**：

```cpp
char norm[]="0123456789 \nX+-*/=.ABCDEFabcdef", //norm中小写字母会被替换为大写
     lite[]="0123456789 \n"; //lite字符集中换行符会被替换为空格
bool nf=1,lf=1; //norm标记和lite标记

for(int x=0;x<p;x++) {
    if(nf && strchr(norm,i[x])==NULL) nf=0;
    if(lf && strchr(lite,i[x])==NULL) lf=0;
}

if(!nf) WA(); //非norm，为full字符集
else if(!lf) TLE(); //norm但非lite，为norm字符集
else OLE(); //norm且lite，为lite字符集
```

**norm字符集的dump代码**：

```cpp
char *u="0123456789 \nX+-*/=.ABCDEF",
    *r=strchr(u,(i[idx]>96 && i[idx]<123 ? i[idx]-32 : i[idx])); //大小写转换
//r-u是i[idx]在u中的index

if(r-u<n+j) WA();
else if(r-u<n+2*j) TLE();
else if(r-u<n+3*j) MLE();
else if(r-u<n+4*j) RE();
else OLE();
```

**lite字符集的dump代码**：

```cpp
char *u="0123456789 ",
*r1=strchr(u,(i[idx]=='\n' ? ' ' : i[idx])),
*r2=(idx+1==p ? u+10 : strchr(u,(i[idx+1]=='\n' ? ' ' : i[idx+1])) );
//当idx==p-1，即i[idx]是字符串最后一位时，dump i[idx+1]没有意义

int r=(r1-u)*11+(r2-u);
if(r<n+j) WA();
else if(r<n+2*j) TLE();
else if(r<n+3*j) MLE();
else if(r<n+4*j) RE();
else OLE();
```

值得注意地，由于norm字符集中小写字母会被转换成大写、lite字符集中换行符会被转换成空格，在pass代码中也要略作修改。还记得前面的“完整版demo”里提到过的`OKnorm`和`OKlite`吗？它们的实现方式如下：

```cpp
bool OKnorm(char *a,char *b) {
    _C;
    for(int x=0;x<p;x++)
        if((a[x]>96 && a[x]<123 ? a[x]-32 : a[x])!=b[x])
            return false;
    return true;
}
bool OKlite(char *a,char *b){
    _C;
    for(int x=0;x<p;x++)
        if((a[x]=='\n' ? ' ' : a[x])!=b[x])
            return false;
    return true;
}
```

在经过上述优化后，**framework部分的真·完整版算法如下**：

>1    通过一次提交，获取适用于输入数据的最佳字符集C
>
>2    通过4次提交，获取输入数据的长度L
>
>3    对于 idx ← [1 .. L]：
>
>3.1    如果C==full，通过3次提交获取输入数据的第idx位
>
>3.2    如果C==norm，通过2次提交获取输入数据的第idx位
>
>3.3    如果C==lite且idx位在上一次循环已经获取到了，continue
>
>3.4    否则，通过3次提交获取输入数据的第idx位和(idx+1)位（如有）
>
>4    至此全部input已被获取到，提示攻击者输入output
>
>5    通过1次提交检验C字符集下的output是否正确，如不正确跳回3，如已经Accepted了则退出
>
>6    将C字符集下的bypass代码加到base位置前面，回到1，开始dump下一组数据

全文完，[ACFan.zip](https://www.luogu.com.cn/fe/api/problem/downloadAttachment/c55q23py)。
