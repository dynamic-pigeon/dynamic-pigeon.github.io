---
title: C++ search 函数
author: Dynamic_Pigeon
avatar: https://s3.bmp.ovh/imgs/2024/10/08/ca40e9bf152dbbf0.jpg
authorLink: dynamic-pigeon.github.io
authorAbout: 一个好奇的人
authorDesc: 一个好奇的人
categories: 技术
comments: true
date: 2024-10-01 20:34:15
updated: 2024-10-01 20:34:15
tags:
    - C++
keywords: search
description:
photos: https://s3.bmp.ovh/imgs/2024/10/01/d254935c80511b82.png
---
## 简介

std::search 函数在 C++ 很早的版本就开始存在了，但是在 C++17 中添加了一些重载和辅助类，这大大提高了这个函数的使用价值。这篇文章主要来介绍下面这个重载。

```c++
template< class ForwardIt, class Searcher >
ForwardIt search( ForwardIt first, ForwardIt last,
                  const Searcher& searcher );
```

|||
| ---- | ---- |
|**元素名称**|**元素意义**|
|first, last| 要检验的元素范围 —— 一般是迭代器 |
|searcher|封装搜索算法和搜索模式的搜索器|

这是从 C++17 开始给 std::search 添加的函数重载，Searcher 是搜索器，接下来就是这个函数最有用的地方了，C++ std 提供了三种搜索器：

|||
| ---- | ---- |
| default_searcher (C++17) |  标准 C++ 库搜索算法实现 |
| boyer_moore_searcher (C++17) | Boyer-Moore 搜索算法实现 |
| boyer_moore_horspool_searcher (C++17) | Boyer-Moore-Horspool 搜索算法实现 |

注意到标准库提供了 BM 算法的搜索器，BM 算法一直被称为最快的字符串搜索算法，在实际表现中一般比 KMP 快 2～3 倍（暴力算法在大部分时候其实表现也比 KMP 好）。但是由于 BM 算法的最坏时间复杂度是 $O(nm)$，所以不建议在算法竞赛中使用，但是在生产环境中，BM 算法无疑是最佳选择。

## 使用举例

```c++
#include <functional>
#include <iostream>
#include <string>

int main() {
  std::string s = "never";
  // 用 s 创建一个搜索器
  std::boyer_moore_searcher searcher(s.begin(), s.end());

  std::string text = "never say never";
  // 在 text 中搜索 s
  auto result = std::search(text.begin(), text.end(), searcher);
  if (result != text.end()) {
    std::cout << "The text contains the substring \"never\" at position "
              << result - text.begin() << '\n';
  } else {
    std::cout << "The text does not contain the substring \"never\"\n";
  }

  // 在另一个文本中搜索 s
  text = "always say always";
  result = std::search(text.begin(), text.end(), searcher);
  if (result != text.end()) {
    std::cout << "The text contains the substring \"never\" at position "
              << result - text.begin() << '\n';
  } else {
    std::cout << "The text does not contain the substring \"never\"\n";
  }

  return 0;
}
```

输出结果

```text
The text contains the substring "never" at position 0
The text does not contain the substring "never"
```

## 参考链接

[cppreferrence.com](https://zh.cppreference.com/w/cpp/algorithm/search)
