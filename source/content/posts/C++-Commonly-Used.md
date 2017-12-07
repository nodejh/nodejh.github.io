+++
title = "C++ 常用操作"
description = "C++ Commonly Used"
date = 2017-07-30T01:44:54+08:00
tags = ["C++"]
categories = ["C++"]
draft = true
+++

最近在用 C++ 刷算法题。由于对 C++ 不熟悉，很多语法都是边查边用。为了方便查阅和记忆，所以在此记录下来。

## 字符串转数字

```cpp
string str="0100";
int i = stoi(str); // 100
```


## 数字转字符串及长度

```cpp
#include <string>

std::string s = std::to_string(42);
int length = s.length();
int len=s.size();
```

- [Easiest way to convert int to string in C++](https://stackoverflow.com/questions/5590381/easiest-way-to-convert-int-to-string-in-c)

## 求字符串长度

```cpp
char s[3]="abc";
cout<<strlen(s)<<endl; // 3
``

## 翻转字符串

```cpp
string rev(string str)
{
  long length=str.lengt();
  int start=0, end=lengt-1;
  while(start<end) {
    str[start]=start[end];
    start++;
    end--;
  }
  return str;
}
```

## 翻转数字

```cpp
long long res=0;
while(x) {
  res = res*10 + x%10;
  x /= 10;
}
return (res > INT_MAX || res < INT_MIN) ? 0 : res;
```

- [7. Reverse Integer](https://leetcode.com/problems/reverse-integer/description/)
