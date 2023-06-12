---
title: 理解 push_back ｜ emplace_back
description: Understand push_back ｜ emplace_back
slug: cpp-emplace_back-vs-push_back
date: 2023-06-12 10:00:00+0000
image: cover.png
categories:
    - Study
tags:
    - C++
    - Technology
    - Programming Language
---


# 什么是push_back和emplace_back

- push_back和emplace_back是C++11引入的成员函数，用于对vector容器的末尾进行添加元素
- push_back的函数签名

| void push_back( const T& value ); | (until C++20) |
| --- | --- |
| constexpr void push_back( const T& value ); | (since C++20) |
| void push_back( T&& value ); | (since C++11) (until C++20) |
| constexpr void push_back( T&& value ); | (since C++20) |

- emplace_back的函数签名

| template< class... Args >void emplace_back( Args&&... args ); | (since C++11) (until C++17) |
| --- | --- |
| template< class... Args >reference emplace_back( Args&&... args ); | (since C++17) (until C++20) |
| template< class... Args >constexpr reference emplace_back( Args&&... args ); | (since C++20) |

# push_back和emplace_back的异同点

1. push_back仅支持传入一个参数，而emplace_back支持传入一个可变参数列表
2. push_back支持传入左值（常量左值引用捕获）和右值（右值引用捕获），emplace_back也支持传入左值和右值
3. 对于传入的左值，二者均调用一次constructor、一次copy constructor和一次destructor
4. 对于传入的右值，二者均调用一次constructor、一次move constructor和一次destructor
5. emplace_back可以传入构造函数所需的一系列参数，支持in-place construct，而push_back不支持，因此若传入构造函数所需的一系列参数，则只会调用一次constructor（详见例子）
6. insert和emplace的异同和push_back和emplace_back类似

# 常见误区

1. emplace_back相比于push_back可以提升效率，原因是可以减少不必要的构造函数调用

对于直接传入左值或者右值的情况，emplace_back并无法提升效率，二者都会创建一个临时对象，通过该临时对象进行拷贝/移动构造，之后再销毁该临时对象

2. emplace_back支持右值引用，而push_back不支持

从函数签名可以看到push_back也是支持右值引用的，同时也支持移动语义（调用移动构造函数）

# 例子

```cpp
#include <vector>
#include <cassert>
#include <iostream>
#include <string>

struct President {
    std::string name;
    std::string country;
    int year;

    President(std::string p_name,std::string p_country, int p_year)
        : name(std::move(p_name)), country(std::move(p_country)), year(p_year)
    {
        std::cout << "I am being constructed.\n";
    }

    President(President&& other)
        : name(std::move(other.name)), country(std::move(other.country)), year(other.year)
    {
        std::cout << "I am being moved.\n";
    }

    President& operator=(const President& other) = default;
};

int main() {
    std::vector<President> elections;
    std::cout << "emplace_back:\n";
    auto& ref = elections.emplace_back("Nelson Mandela", "South Africa", 1994);
    assert(ref.year == 1994 && "uses a reference to the created object (C++17)");
        
    std::vector<President> reElections;
    std::cout << "\npush_back:\n";
    reElections.push_back(President("Franklin Delano Roosevelt", "the USA", 1936));
        
    std::cout << "\nContents:\n";
        for (President const& president: elections)
    std::cout << president.name << " was elected president of "
                      << president.country << " in " << president.year << ".\n";
    for (President const& president: reElections)
        std::cout << president.name << " was re-elected president of "
                        << president.country << " in " << president.year << ".\n";
}
/*
Output：
emplace_back:
I am being constructed.
 
push_back:
I am being constructed.
I am being moved.
 
Contents:
Nelson Mandela was elected president of South Africa in 1994.
Franklin Delano Roosevelt was re-elected president of the USA in 1936.
*/
```


# Reference

[std::vector<T,Allocator>::push_back - cppreference.com](https://en.cppreference.com/w/cpp/container/vector/push_back)

[std::vector<T,Allocator>::emplace_back - cppreference.com](https://en.cppreference.com/w/cpp/container/vector/emplace_back)

[push_back vs emplace_back](https://stackoverflow.com/questions/4303513/push-back-vs-emplace-back)

> Written by Jiacheng Hu, at Zhejiang University, Hangzhou, China.
