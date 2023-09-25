---
title: 理解 Copy Elision
description: Understand Copy Elision(RVO & NRVO)
slug: understand-copy-elision
date: 2023-09-25 01:00:00+0000
image: cover.png
categories:
    - Study
tags:
    - C++
---

## 概念解释

什么是copy elision？什么是RVO？什么是NRVO？

- copy elision是C++中对于函数返回值的优化机制，可以减少多余的构造/析构操作，提升效率
- RVO（return value optimization），返回值优化，是copy elision的一种形式
- NRVO（named return value optimization），具名返回值优化，是copy elision的一种形式

## 一个例子理解copy elision

```cpp
class Obj {
public:
    Obj() { cout << "Constructed\n"; }
    ~Obj() { cout << "Deconstructed\n"; }
    Obj(const Obj& r) { cout << "Copy constructed\n"; }
    Obj(Obj&& r) { cout << "Move constructed\n"; }
    Obj& operator=(const Obj& r) { cout << "Copy assignment\n"; return *this; }
    Obj& operator=(Obj&& r) { cout << "Move assignment\n"; return *this; }
};

Obj f1() {
    return Obj(); // RVO
}

Obj f2() {
    Obj t;
    return t; // NRVO
}

Obj f3() {
    Obj t;
    return std::move(t);
}

int main() {
    {
        Obj tmp = f1();
    }
    cout << "---------------\n";
    {
        Obj tmp = f2();
    }
    cout << "---------------\n";
    {
        Obj tmp = f3();
    }
    return 0;
}
```

Q：函数的输出是什么？

### 情况1: C++11，关闭copy elision

```cpp
clang++ rvo.cc -std=c++11 -fno-elide-constructors -o rvo
```

```cpp
Constructed
Move constructed
Deconstructed
Move constructed
Deconstructed
Deconstructed
---------------
Constructed
Move constructed
Deconstructed
Move constructed
Deconstructed
Deconstructed
---------------
Constructed
Move constructed
Deconstructed
Move constructed
Deconstructed
Deconstructed
```

解释：

没有任何优化，可以发现，出现了一次默认构造，两次移动构造，两次析构（最后一次是tmp的析构，不用在意，下同）

两次移动构造分别是临时对象移动到返回值，以及返回值移动到tmp

### 情况2: C++17，关闭copy elision

```cpp
clang++ rvo.cc -std=c++17 -fno-elide-constructors -o rvo
```

```cpp
Constructed
Deconstructed
---------------
Constructed
Move constructed
Deconstructed
Deconstructed
---------------
Constructed
Move constructed
Deconstructed
Deconstructed
```

解释：

f1只出现了一次构造，f2和f3多了一次移动构造和析构

C++17默认实现了RVO，而没有强制要求编译器实现NRVO，因此尽管关闭了copy elision，在RVO情况下，仍然进行了优化

还可以发现C++17在关闭copy elision时仍然比关闭了copy elision的C++11优化了一次移动构造和析构，优化了返回值移动到tmp的那一次

### 情况3: C++11，开启copy elision

```cpp
clang++ rvo.cc -std=c++11 -o rvo
```

```cpp
Constructed
Deconstructed
---------------
Constructed
Deconstructed
---------------
Constructed
Move constructed
Deconstructed
Deconstructed
```

### 情况4: C++17，开启copy elision

```cpp
clang++ rvo.cc -std=c++17 -o rvo
```

```cpp
Constructed
Deconstructed
---------------
Constructed
Deconstructed
---------------
Constructed
Move constructed
Deconstructed
Deconstructed
```

解释：

当不关闭copy elision时，C++11和C++17的表现是一样的，可以发现当使用了move时，反而产生了负优化，多了临时对象移动到返回值那一次

> Written by Jiacheng Hu, at Zhejiang University, Hangzhou, China.
