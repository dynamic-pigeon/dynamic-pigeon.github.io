---
title: 2022 ICPC 杭州区域赛 K 题题解
author: Dynamic_Pigeon
avatar: https://cdn.luogu.com.cn/upload/usericon/545317.png
authorLink: chy669086.github.io
authorAbout: 一个好奇的人
authorDesc: 一个好奇的人
categories: 题解
comments: true
date: 2024-10-02 17:01:08
tags: 
  - c++
  - ICPC
keywords:
description:
photos: http://cdnjson.com/images/2024/10/01/81197b85969320adbdf325c260404543.png
---
## 题目大意
给你 $n$ 个字符串，然后有 $q$ 次询问，每次询问给出一个字典序，请你输出在这个字典序下的逆序对个数。

## 题解
首先考虑暴力，每次询问可以 $O(|S|\log(n))$ 处理，显然会超时。

考虑优化，注意到两个字符串比较出结果一定是在某个位置发生的（称这个比出大小的比较为**决定性比较**），这两个字符串一定有一个相同的前缀（长度可能是 0）。那么可以把问题转换成判断某个字符和其他字符发生了几次**决定性**的的比较，那么对于每次询问我们和 $O(26 \times 26)$ 处理。

如何找到发生了几次决定性的比较呢，注意前面的一个性质，决定性比较之前的两个字符串一定是同前缀，这正好撞上了字典树的性质：相同前缀共用一条链。

我们只要统计每个节点的后继节点字符总数，就可以对每一个新插入的字符串进行决定性比较的统计。对于 `a` 和 `aa` 这种串，我们可以简单在每个串后面加一个字符，并人为将这个字符的字典序设置为最小
```c++
// 字段树节点
struct node {
  // 后继节点位置
  std::array<int, 27> nxt;
  // 后继节点个数统计
  std::array<int, 27> sum;
  node() {
    std::fill(all(nxt), -1);
    std::fill(all(sum), 0);
  }
};

// 决定性比较次数统计
int cnt[27][27];

struct Trie {
  std::vector<node> nodes;
  int root = 0;
  Trie() {
    nodes.emplace_back();
  }

  void insert(const std::string &s) {
    int cur = root;
    for (char c : s) {
      if (nodes[cur].nxt[c - 'a'] == -1) {
        nodes[cur].nxt[c - 'a'] = nodes.size();
        nodes.emplace_back();
      }
      // 统计这个串的这个位置决定性比较次数
      for (int i = 0; i < 27; i++) {
        cnt[c - 'a'][i] += nodes[cur].sum[i];
      }
      // 后继节点计数加一
      nodes[cur].sum[c - 'a']++;
      cur = nodes[cur].nxt[c - 'a'];
    }
  }
};
```
全部代码
```c++
#include <algorithm>
#include <array>
#include <cassert>
#include <cctype>
#include <cmath>
#include <cstdint>
#include <cstdio>
#include <cstring>
#include <iostream>
#include <ostream>
#include <string>
#include <vector>

#ifdef DYNAMIC_PIGEON
#include "algo/debug.h"
#else
#define debug(...) 114514
#endif

#define Dynamic_Pigeon 0

using i64 = std::int64_t;
using u64 = std::uint64_t;
using u32 = std::uint32_t;

constexpr i64 MOD = i64(1e9) + 7;
constexpr int INF = 1e9;
constexpr i64 INF_LONG_LONG = 1e18;

#define all(a) a.begin(), a.end()
#define rall(a) a.rbegin(), a.rend()

#define int i64

struct node {
  std::array<int, 27> nxt;
  std::array<int, 27> sum;
  node() {
    std::fill(all(nxt), -1);
    std::fill(all(sum), 0);
  }
};

int cnt[27][27];

struct Trie {
  std::vector<node> nodes;
  int root = 0;
  Trie() {
    nodes.emplace_back();
  }

  void insert(const std::string &s) {
    int cur = root;
    for (char c : s) {
      if (nodes[cur].nxt[c - 'a'] == -1) {
        nodes[cur].nxt[c - 'a'] = nodes.size();
        nodes.emplace_back();
      }
      for (int i = 0; i < 27; i++) {
        cnt[c - 'a'][i] += nodes[cur].sum[i];
      }
      nodes[cur].sum[c - 'a']++;
      cur = nodes[cur].nxt[c - 'a'];
    }
  }
};

void tizzytyt_SuKi() {
  int n, q;
  std::cin >> n >> q;

  Trie trie;

  for (int i = 0; i < n; i++) {
    std::string s;
    std::cin >> s;
    s.push_back('a' + 26);
    trie.insert(s);
  }

  while (q--) {
    std::string s;
    std::cin >> s;

    s = (char)('a' + 26) + s;

    i64 ans = 0;

    for (int i = 0; i < 27 - 1; i++) {
      for (int j =  i + 1; j < 27; j++) {
        ans += cnt[s[i] - 'a'][s[j] - 'a'];
      }
    }

    std::cout << ans << '\n';
  }
}

signed main() {
#ifdef DYNAMIC_PIGEON
  freopen("input.in", "r", stdin);
#else
  std::cin.tie(0)->sync_with_stdio(0);
#endif
  
  tizzytyt_SuKi();
  
  return Dynamic_Pigeon;
}
```