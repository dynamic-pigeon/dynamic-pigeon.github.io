---
title: C++ 高版本的一些对算竟有用的东西.
author: Dynamic_Pigeon
avatar: https://s3.bmp.ovh/imgs/2024/10/08/ca40e9bf152dbbf0.jpg
authorLink: chy669086.github.io
authorAbout: 一个好奇的人
authorDesc: 一个好奇的人
categories: 技术
comments: true
date: 2024-10-01 16:57:58
tags:
    - C++
keywords:
description:
photos: https://s3.bmp.ovh/imgs/2024/10/01/89a65a3569f8a057.png
---
# C++ 高版本的一些对算竟有用的东西.
## 总括
现在已经是 2024 年了，C++17 标准都已经是 7 年前的东西了，这几年 C++17 及以上的标准被越来越多的选手使用，这篇文章主要是来总结一下我学习算法一年来了解到的一些高版本好用的语法和标准库类与函数。

## C++17
### 一些数学函数

- std::gcd()
- std::lcm()

在 C++17 之前，就已经有人用 gcc 私货 std::__gcd 来求最大公因数，但是自从 C++17 标准发布之后，std::gcd 光荣转正，使用 stein 算法，时间复杂度 $O(\log(n))$。
```c++
template< class M, class N >
constexpr std::common_type_t<M, N> gcd( M m, N n );

template< class M, class N >
constexpr std::common_type_t<M, N> lcm( M m, N n );
```

### 结构化绑定
对于一个结构体（**只能是结构体！**），可以用形如 `auto [a, b, ...] = value` 的形式进行绑定。

例如：
```c++
#include <cassert>
#include <utility>

struct node {
    int l, r;
    double p;
};

int main() {
    node t{10, 3, 6.0};
    auto [l, r, p] = t;
    assert(l == 10);
    assert(r == 3);
    assert(p == 6.0);


    std::pair<int, int> pr{1, 2};
    auto [x, y] = pr;
    assert(x == 1);
    assert(y == 2);

    return 0;
}
```
C++ 的结构化绑定只能绑一层，比如说