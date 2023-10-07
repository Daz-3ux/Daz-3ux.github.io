---
title: Linux 下使用 JetBrains 系IDE
author: daz
pubDatetime: 2023-10-07T14:29:55Z
featured: false
draft: false
tags:
  - JetBrains
  - IDE
  - tips
ogImage: ""
description: "feat: fix bugs"
---

![](https://img-blog.csdnimg.cn/img_convert/54e60afdf2764a07539da3136f3ce3e4.png)
本作品采用[知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议](https://creativecommons.org/licenses/by-nc-sa/4.0/)进行许可。

---

## 背景

- 作为长期的 vscode 用户,在初次接触 Goland 后感觉 Goland 丑丑的,试用后发现还有 bug,但最后发现这并不是 Goland 的问题,在解决问题后,不得不感慨, Goland 真好用呀!
- 这篇博客提到的步骤适合所有的 JetBrains 系 IDE, 并非只解决 Goland 的问题!
- 我的环境

```text
OS: Arch Linux x86_64
Kernel: 6.5.5-arch1-1
DE: Plasma 5.27.8
```

## 具体问题

1. fcitx5 输入法候选框不跟随光标(只停留在左下角)
2. Markdown 文件无法正常预览
3. IDE 内无内置的[最小化, 全屏, 退出]

- 正常情况应该如下图
  ![下图](https://raw.githubusercontent.com/Daz-Bot/Img-hosting/master/host/202310072304025.png)

## Why

- Why
- [JetBrainsRuntime](https://github.com/JetBrains/JetBrainsRuntime) 负责字体渲染,`窗口支持`, `窗口/焦点子系统`, HiDPI, 性能等方面问题
  - JetBrains Runtime is a fork of OpenJDK available for Windows, Mac OS X, and Linux. It supports enhanced class redefinition (DCEVM), features optional JCEF, a framework for embedding Chromium-based browsers, includes a number of improvements in font rendering, keyboards support, windowing/focus subsystems, HiDPI, accessibility, and performance, provides better desktop integration and bugfixes not yet present in OpenJDK.
- JetBrainsRuntime 对 Linux64 的支持有问题,会造成 IDE 在这部分存在错误

### 如何解决

- 参考[此项目](https://github.com/AlanSune/JetBrainsRuntime-for-Linux-x64)

  - 手动方式:
    - fork 项目仓库
    - 使用 Github Actions 对 jbr 打入补丁
    - 直接替换 IDE 安装目录下的 jbr 目录
  - Arch Linux 用户: - 通过 `aur` 安装补丁包 - 在 IDE 中切换 Java 运行时 - 切换具体操作: 双击 shift 打开`到处搜索`然后选择此项
    ![此项](https://raw.githubusercontent.com/Daz-Bot/Img-hosting/master/host/202310072306532.png)

- 详细流程参照项目 [README](https://github.com/AlanSune/JetBrainsRuntime-for-Linux-x64#readme)

---

## 参考

1. [JetBrainsRuntime-for-Linux-x64](https://github.com/RikudouPatrickstar/JetBrainsRuntime-for-Linux-x64)
