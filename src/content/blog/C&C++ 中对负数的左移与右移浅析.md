---
title: C 与 C++ 语言中对负数的左移与右移浅析
author: daz
pubDatetime: 2023-04-16T13:00:47Z
featured: false
draft: false
tags:
  - C&&C++
  - language lawyer
ogImage: ""
description: "C/C++'s detail"
---

![](https://img-blog.csdnimg.cn/img_convert/54e60afdf2764a07539da3136f3ce3e4.png)
本作品采用[知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议](https://creativecommons.org/licenses/by-nc-sa/4.0/)进行许可。

---

[迁移博客]

# C 与 C++ 语言中对负数的左移与右移浅析

## Demo

- 小组今年的面试题有一道题目专门考了 `UB` 行为，所以最近就对 UB 行为比较感兴趣，发现关于这方面确实是自己一直以来遗漏掉的一个地方。
- 在另一道面试题中，考到了负数的位移`位运算`，关于负数的左移与右移到底属于什么行为引出了一些讨论。

- 原题如下

```c
int main(void)
{
    unsigned char a = 4 | 7;
    a <<= 3;
    unsigned char b = 5 & 7;
    b >>= 3;
    unsigned char c = 6 ^ 7;
    c = ~c;
    unsigned short d = (a ^ c) << 3;
    signed char e = -63;
    e <<= 2;

    printf("a:%d , b:%d , c:%d , d:%d\n", a, b, c, (char)d);
    printf("e:%#x\n", e);
}
```

- 题目的题解在此就不展开了，这篇博客主要讨论一下负数的左移与右移

## 参阅标准

### ISO/IEC 9899

根据此标准的 `6.5.7 Bitwise shift operators` 一节来看：

- 对负数的`左移`属于 `UB` 行为。
- 对负数的`右移`属于 `implementation-defined` 行为。

- _关于 UB 与 implementation-defined 的区别,请参阅:_

  - [Undefined, unspecified and implementation-defined behavior](https://stackoverflow.com/questions/2397984/undefined-unspecified-and-implementation-defined-behavior)

- 但是此时有一个新的问题，如果只参阅 ISO 的标准去判断 UB 行为，那么 UB 行为就有些过于**广泛**了，所以还需要去参阅一下编译器的实现。

### Microsoft C++ 语言参考

根据微软的 C++ 编译器实现，在 X86/X64 的架构下：

- 对负数的`左移`，如果影响到符号位，则结果是不确定的，为 UB。
- 对负数的`右移`，为 implementation-defined。**Microsoft C++ 编译器使用符号位填充空出的位，但是无法保证其他实现也会这样执行。**

---

## 参考

- [ISO/IEC 9899:201x](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1548.pdf)
- [左移和右移运算符（<< 和 >>）](https://learn.microsoft.com/zh-cn/cpp/cpp/left-shift-and-right-shift-operators-input-and-output?view=msvc-170#right-shifts)
- [附录：C 未定义行为成因列表](https://github.com/Qihoo360/safe-rules/blob/main/c-ub-list.md)
