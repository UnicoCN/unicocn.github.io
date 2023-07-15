---
title: 理解初始化列表
description: Understand initializer list
slug: cpp-understand-initializer-list
date: 2023-07-15 10:00:00+0000
image: cover.png
categories:
    - Study
tags:
    - C++
    - Technology
---

# 理解初始化列表

## 问题引入

阅读下面代码

```cpp
#include <iostream>
#include <vector>

using namespace std;

int main()
{
	vector<vector<int>> tmp{{10, 10}, {20, 20}, {30, 20}};
  	tmp.push_back({1, 2, 3}); // works
  
	auto list = {1, 2, 3, 4, 5};
	tmp.emplace_back(list); // works
  	// tmp.emplace_back({1, 2, 3, 4, 5}); // error
  
  	vector<int> v = {1, 2, 3};
  	v = {1, 2, 3};
  	vector<int>a{1, 2, 3};
	return 0;
}

```

Insights:

```cpp
#include <iostream>
#include <vector>

using namespace std;

int main()
{
  std::vector<std::vector<int, std::allocator<int> >, std::allocator<std::vector<int, std::allocator<int> > > > tmp = std::vector<std::vector<int, std::allocator<int> >, std::allocator<std::vector<int, std::allocator<int> > > >{std::initializer_list<std::vector<int, std::allocator<int> > >{std::vector<int, std::allocator<int> >{std::initializer_list<int>{10, 10}, std::allocator<int>()}, std::vector<int, std::allocator<int> >{std::initializer_list<int>{20, 20}, std::allocator<int>()}, std::vector<int, std::allocator<int> >{std::initializer_list<int>{30, 20}, std::allocator<int>()}}, std::allocator<std::vector<int, std::allocator<int> > >()};

  tmp.push_back(std::vector<int, std::allocator<int> >{std::initializer_list<int>{1, 2, 3}, std::allocator<int>()});

  std::initializer_list<int> list = std::initializer_list<int>{1, 2, 3, 4, 5};

  tmp.emplace_back<std::initializer_list<int> &>(list);

  std::vector<int, std::allocator<int> > v = std::vector<int, std::allocator<int> >{std::initializer_list<int>{1, 2, 3}, std::allocator<int>()};

  v.operator=(std::initializer_list<int>{1, 2, 3});

  std::vector<int, std::allocator<int> > a = std::vector<int, std::allocator<int> >{std::initializer_list<int>{1, 2, 3}, std::allocator<int>()};

  return 0;
}

```

为什么`tmp.emplace_back({1, 2, 3, 4, 5})`会Error？

## 理解初始化列表

首先对于上面这个问题，不理解的点在于：
- {1, 2, 3} 是一个 initializer_list<int>
- vector<int>有参数为initializer_list<int>的构造参数
- 同时 vector<int> v({1, 2, 3}) 是可以的
- 那么我们使用emplace_back将{1, 2, 3}作为 args 完美转发给vector<int>的构造函数为什么会报错？

> 解释见 https://en.cppreference.com/w/cpp/language/list_initialization#Notes
>
- 简单来说，{1, 2, 3}不是一个expression，因此也没有类型，decltype({1, 2}) is ill-formed
- 因此Args类型推断会无效，因此v.emplace_back({1, 2, 3})不会生效
- 为什么 auto x = {1, 2, 3}，x的类型会被推断为initializer_list?
- 答案: 规定的特例

为什么v.push_back({1, 2, 3}) 可以？
> 解释见 https://en.cppreference.com/w/cpp/language/overload_resolution#Implicit_conversion_sequence_in_list-initialization
>
简单来说，是一种重载的隐式转换,可以将其转换为initializer_list


> Written by Jiacheng Hu, at Zhejiang University, Hangzhou, China.
