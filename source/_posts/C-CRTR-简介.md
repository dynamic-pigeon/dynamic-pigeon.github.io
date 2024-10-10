---
title: C++ CRTR 简介
author: Dynamic_Pigeon
avatar: https://s3.bmp.ovh/imgs/2024/10/08/ca40e9bf152dbbf0.jpg
authorLink: chy669086.github.io
authorAbout: 一个好奇的人
authorDesc: 一个好奇的人
categories: 技术
comments: true
date: 2024-10-05 22:08:24
updated: 2024-10-05 22:08:24
tags:
    - C++
keywords: CETR 继承
description:
photos: http://cdnjson.com/images/2024/10/05/image.png
---
最近看群友讨论到了“虚函数就是历史遗留问题，现在完全可以用 CRTR 代替”，好奇，就去搜索了什么是 CRTR，结果发现这么早的一个技术我现在才知道，真是落后时代 10 余年。

## 虚函数与动态链接

C++ 的多态是通过虚函数实现的，虚函数的实质是在对象中开辟了一个空间来存储函数指针（虚函数表 `vtable`），调用这个函数是通过这个指针去访问，而不能在编译期确定。

比如：

```c++
class Animal {
public:
  virtual void jump() = 0;
};

class Cat : public Animal {
public:
  void jump() override {
    std::cout << "cat jump\n";
  }
};

int main() {
  Animal *p = new Cat;
  p->jump();
  return 0;
}
```

调用 `p->jump()` 的时候，程序会去虚函数表寻找那个指针，在这个程序中最终会调用 `Cat::jump()`。

但是查表终究是有代价的，有没有兼顾多态和效率的做法呢，于是，CRTR(Curiously Recurring Template Pattern) 应运而生。

## 通过模板实现静态绑定

首先复习一个概念，C++ 的模板是编译期多态，是**零成本抽象**。我们在子类和父类中定义同一个函数，但是不声明为虚函数，通过编译期多态来实现绑定。

```c++
template <typename T>
class Base {
public:
  void show() const {
    static_cast<const T*>(this)->show();
  }
};

class Driver : public Base<Driver> {
public:
  void show() const {
    std::cout << "This is class Driver." << std::endl;
  }
};

int main() {
  Base<Driver> *p = new Driver;
  p->show();
  return 0;
}
```

这是一个简单的例子，有下列特点

- 基类是模板类，用来接收子类的名字，因此继承类似于 `ClassName : public Base<ClassName>`。
- 基类的函数中，用 `static_cast<>` 将基类的指针转换成子类的指针实现绑定。

运行上述程序，得到结果 `This is class Driver.` 说明我们成功进行了子类与父类的绑定，实打实的调用了子类的 `show()` 函数。

## 有什么用，怎么用

众所周知，多态一般需要类似 `std::vector<Base*>` 的形式，但是由于我们的基类是模板类，不能通过这种方式多态，不如说，没有多态，因为 `Animal<Cat>*` 和 `Animal<Dog>*` 是两个完全不同的指针。

***那怎么办？***

考虑到一次查表的开销不是特别大，虚函数带来的大量开销主要是多层继承带来的巨大链条带来的查询开销，继承链很短的话开销其实是在预期内的。

那我们用一个模板基类继承一个普通基类，这个普通基类使用虚函数，但是模板基类使用静态绑定，派生类去继承这个模板基类，那么不论如何，我们都只需要查一次表就可以定位到对应的函数，大大降低了多次继承带来的开销。

```c++
class Animal {
public:
  virtual void say() const = 0;
  virtual ~Animal() {}
};

template <typename T>
class Animal_CRTR : public Animal {
public:
  void say() const override {
    static_cast<const T*>(this)->say();
  }
};

class Cat : public Animal_CRTR<Cat> {
public:
  void say() const {
    std::cout << "I'm a cat.\n";
  }
};

class Dog : public Animal_CRTR<Dog> {
public:
  void say() const {
    std::cout << "I'm a dog.\n";
  }
};

int main() {
  std::vector<Animal*> v;
  v.push_back(new Dog);
  v.push_back(new Cat);
  for (auto x : v) {
    x->say();
  }
  return 0;
}
```

输出结果

```test
I'm a dog.
I'm a cat.
```

不过写起来确实很麻烦，而且没有 override 带来的部分编译提示（字打错了跑哪里哭去），但是这样写我们同时赢得了多态和效率（同时代码量极致膨胀）。

~~为什么有人还在用继承啊~~
