---
title: C++ string类：find()和find_first_of()
date: 2020-02-03T05:57:18.602Z
coverImage: 
categories: 
    - cpp
    - string
tags: 
    - cpp
    - c++
---
<!-- toc -->
在c++中`find()`和`find_first_of()`经常容易被混淆。一句话概括`find()`和`find_first_of()`：`find()`在s中找子串第一次出现的位置，`find_first_of()`在s中找一个数组中任意一个元素第一次出现的位置

<!-- more -->

## 异同
- 共同点：
查找成功时返回所在位置，失败返回string::npos的值，string::npos一般是MAX_INT（即2^32 - 1）

- 差异：
    - `find()`: 查找字符串中第一次出现字符c、字符串s的位置；
    - `find_first_of()`: 查找字符串中字符c、字符数组s中任意一个字符第一次出现的位置。

## 示例

``` c++
string haystack = "helloworld";
string needle = "world";
cout << haystack.find_first_of(needle) << endl; //2, index of first 'l'
cout << haystack.find(needle) << endl; //5, index of first "world"

needle = "";
cout << haystack.find_first_of(needle) << endl;//string::npos, 因为字符数组s为空，haystack中找不到空字符（区别于'\0'）
cout << haystack.find(needle) << endl;//0, 空串
```

## 重载函数

``` c++
int find(char c, int pos = 0) const;//从pos开始查找字符c在当前字符串的位置
int find(const char *s, int pos = 0) const;//从pos开始查找字符串s在当前串中的位置
int find(const char *s, int pos, int n) const;//从pos开始查找字符串s中前n个字符在当前串中的位置
int find(const string &s, int pos = 0) const;//从pos开始查找字符串s在当前串中的位置

int find_first_of(char c, int pos = 0) const;//从pos开始查找字符c第一次出现的位置
int find_first_of(const char *s, int pos = 0) const; //从pos开始查找当前串中第一个在s的前n个字符组成的数组里的字符的位置
int find_first_of(const char *s, int pos, int n) const;
int find_first_of(const string &s,int pos = 0) const;
```

类似的，还有`rfind()`和`find_last_of()`，以及`find_first_not_of()`, `find_last_not_of()`.
