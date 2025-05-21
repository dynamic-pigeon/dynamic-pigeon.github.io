---
title: C++ ADL 简介
author: Dynamic_Pigeon
avatar: https://s3.bmp.ovh/imgs/2024/10/08/ca40e9bf152dbbf0.jpg
authorLink: dynamic-pigeon.github.io
authorAbout: 一个好奇的人
authorDesc: 一个好奇的人
categories: 技术
comments: true
date: 2024-11-20 15:49:21
updated: 2024-11-20 15:49:21
tags:
    - C++
keywords:
    - C++
    - ADL
description:
photos: https://s3.bmp.ovh/imgs/2024/11/20/adcf194d90bbd779.png
---

前两天水群水到的，突然感觉开发了新大陆。

## ADL 简介

相信很多人应该都写过类似这样的代码

```C++
#include <algorithm>
#include <vector>

int main() {
  std::vector<int> a{1, 2, 3, 4};
  sort(a.begin(), a.end());
  return 0;
}
```

这里并没有 `using namespace std` 却可以不用显示指定 `std::sort` 来调用，应该不止我一个人在第一次写出这个并意识到的时候感到惊讶。

更重要的是，以下代码会 `CE`

```C++
#include <algorithm>

int main() {
  int a[10]{};
  sort(a, a + 10);
  return 0;
}
```

**难道是 `std::vector` 的问题？**

没错，就是因为 `std::vector`！

**接下来让我来正式介绍一下 ADL(argument-dependent lookup)！**

实参依赖查找（ADL），又称 Koenig 查找，是一组对函数调用表达式（包括对重载运算符的隐式函数调用）中的无限定的函数名进行查找的规则。在通常无限定名字查找所考虑的作用域和命名空间之外，还会在它的各个实参的命名空间中查找这些函数。[[1]](#reference1)

简单来说就是，在进行函数调用的时候，先会在当前作用域（名称空间）寻找定义，找不到就会在实参的命名空间中寻找定义，还找不到就会报错。

在这些调用中，包含类自身，基类，外围类和最内层类的外层命名空间（当然，对于基础类型，如int之类其关联集为空）；如果类模板已经特化实参，则还增加对实参类的命名空间及以其为成员的类。

!!! note 注解
    对于更多的细节，请查阅 [cppreference.com](https://zh.cppreference.com/w/cpp/language/adl)

然后我们就可以写出这样的代码：

```C++
#include <algorithm>
#include <vector>

class vec : public std::vector<int> {
 public:
  vec() : std::vector<int>() {}
  vec(std::initializer_list<int> il) : std::vector<int>(il) {}
};

int main() {
  vec a{1, 2, 3, 4};
  sort(a.begin(), a.end());
  return 0;
}
```

`sort` 调用的时候，发现当前没有定义，然后到实参定义域中找，没有找到，又去他的基类找，基类是 `std::vector<int>`，在 `std` 名称空间中，于是找到了 `std::sort` 进行调用。

## ADL 的用途

通过前面的例子来看，这个东西好像除了增加阅读难度（完全不知道哪里来的函数）之外就没有用了，那么接下来举一个例子：

```C++
#include <algorithm>
#include <iostream>
#include <vector>
 
namespace test {
class Demo {
public:
  Demo(int a) : a(a) {}
  int get() const { return a; }

  friend std::ostream &operator<<(std::ostream &os, const Demo &d) {
    os << d.a;
    return os;
  }
private:
  int a;
};
}
 
int main() {
  test::Demo d(114514);
  std::cout << d << '\n';
  return 0;
}
```

`std::cout <<` 调用的时候通过 ADL 找到了位于 `test` 名称空间的 `std::ostream &operator<<(std::ostream &os, const Demo &d)`，大大的简化了代码。

ADL 允许函数模板在不同名称空间定义，而不需要显示的指定作用域或者全局空间限定。

再来看一个例子：

```C++
namespace N1 {

struct X {};

void f(X);

}

namespace N2 {

void f(N1::X);

}

void g() {
  N1::X x;
  f(x); // 调用 N1::f
  N2::f(x); // 调用 N2::f

  using N2::f;
  f(x); // 错误，Call to 'f' is ambiguous，同时存在了 N1::f 和 N2::f
}
```

可以发现 `using` 的函数并不被当作当前作用域，而被当作了与 N1 同级的函数。

## 总结

ADL 是 C++ 的一个重要语法概念，不过带来便利的同时也带来的一些不确定性（万一调用调飞了就全毁了），不过这个和允许函数重载是一样的，好用与否，还是在于使用的人。

## 参考文献

<span id="reference1"></span>
[1] https://zh.cppreference.com/w/cpp/language/adl 