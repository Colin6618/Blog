---
title: sizeof 偶遇指针和数组
date: 2013-09-02 22:40:32
tags: C 编程
---

> 记录几个我觉得很有意思的题目，先看这个函数

```c
#include "stdafx.h"
#include <iostream>
using namespace std;

void UpperCase( char str[] ) // 将 str 中的小写字母转换成大写字母
{
  for( size_t i=0; i<strlen(str);i++)
  {
    if( 'a'<=str[i] && str[i]<='z' )
      str[i] -= ('a'-'A' );
  }
}

int _tmain(int argc, _TCHAR* argv[])
{
  char str[] = "aBcDe";
  cout << "str字符长度为: " << sizeof(str) << endl << sizeof(str[0]) << endl;
  UpperCase( str );
  cout << str << endl;
  system("pause");
  return 0;
}

```

函数内的sizeof有问题。根据语法，sizeof如用于数组，只能测出静态数组的大小，因为其在编译时确定，无法检测动态分配的或外部数组大小，
因此sizeof不能用来返回动态分配的内存空间的大小。函数外的str是一个静态定义的数组，因此其大小为6，函数内的str实际只是一个指向字符串的指针，没有任何额外的与数组相关的信息，
因此sizeof作用于上只将其当指针看，一个指针为4个字节，因此返回4。
   注意:数组名作为函数参数时,退化为指针.
        数组名作为sizeof()参数时,数组名不退化,因为sizeof不是函数。
而strlen 要在运行时才能计算。参数必须是字符型指针（char*）。当数组名作为参数传入时，实际上数组就退化成指针，sizeof对于指针和数组计算的结果不一样。

输出结果：
str字符长度为: 6
1
4
ABCDE
请按任意键继续. . .

再看这段
```c
char arr[10] = "What?";
int len_one = strlen(arr);
int len_two = sizeof(arr);
cout << len_one << " and " << len_two << endl;
//一个区别：sizeof返回定义arr数组时，编译器为其分配的数组空间大小，不关心里面存了多少数据。strlen只关心存储的数据内容，不关心空间的大小和类型。
char * parr = new char[10];
int len_one = strlen(parr);
int len_two = sizeof(parr);
int len_three = sizeof(*parr);
cout << len_one << " and " << len_two << " and " << len_three << endl;

```
    输出结果：23 and 4 and 1
    点评：第一个输出结果23实际上每次运行可能不一样，这取决于parr里面存了什么（从parr[0]开始知道遇到第一个NULL结束）；第二个结果实际上本意是想计算parr所指向的动态内存空间
的大小，但是事与愿违，sizeof认为parr是个字符指针，因此返回的是该指针所占的空间（指针的存储用的是长整型，所以为4）;第三个结果，由于*parr所代表的是parr所指的地址空间存放的
字符，所以长度为1.
