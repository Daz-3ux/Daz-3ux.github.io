---
title: C++ Leetcode 常用数据结构浅析
author: daz
pubDatetime: 2023-10-30T08:11:18Z
featured: false
draft: false
tags:
  - C++
  - language lawyer
ogImage: ""
description: "C++ and Leetcode"
---

![](https://img-blog.csdnimg.cn/img_convert/54e60afdf2764a07539da3136f3ce3e4.png)
本作品采用[知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议](https://creativecommons.org/licenses/by-nc-sa/4.0/)进行许可。

---

# C++ 中常用数据结构

## 数组

- 数组是存放在`连续内存`空间上的相同类型数据的集合
  - C++ 数组的内存空间是连续的,包括 `二维数组`
- vector 底层实现是 array, vector 是容器

## 链表

- 单向链表

```c++
struct ListNode {
  int val;
  ListNode *next;
  ListNode(int x = 0): val(x), next(nullptr) {}
}
```

- 双向链表
  - 多用于 LRU

```c++
struct List {
  int key, val;
  List *pre, *next;
  List(int k = 0, int v = 0) : key(k), value(v) {}
}
```

![数组 VS 链表](https://code-thinking-1253855093.file.myqcloud.com/pics/20200806195200276.png)

## 哈希表

- 常见的三种哈希结构
  - 数组
    - 常用于可限定数组范围的情况
      - 比如说都是小写字母
  - set
    - 常用 unordered_set
      - 底层为哈希表
    - 具有过滤作用
  - map
    - 常用 unordered_map
      - 底层为哈希表

![set](https://raw.githubusercontent.com/Daz-Bot/Img-hosting/master/host/202310302045525.png)

![map](https://raw.githubusercontent.com/Daz-Bot/Img-hosting/master/host/202310302045820.png)

## 栈与队列

- Linux 中 GCC 使用的 STL 是 Silicon Graphics Computer Systems 公司实现的 `SGI STL`

- stack
  - stack `不是容器`, 而是容器适配器
    - 栈使用底层容器完成其所有的工作,对外提供统一接口
    - 底层容器是可插拔的
  - 栈中元素必须保持先进后出,所以不提供走访功能,也不提供`迭代器`
  - SGI STL 中栈底层默认为 deque
    - 也可以是 vector 或 list
- queue
  - 不是容器
  - 先进先出,无迭代器
  - 底层默认为 deque
