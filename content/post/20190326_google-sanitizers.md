---
title: "Google Sanitizers"
author: "ykdu"
tags: ["memcheck", "intro", "tool"]
weight: 10
date: 2019-03-26T21:56:43+08:00
draft: true
---

[Google Sanitizaers](https://github.com/kasicass/blog/blob/master/cpp/2019_02_13_google_sanitizers.md)
目前相关的工具代码都已经迁移到了[LLVM](http://llvm.org)编译器架构上

<!--more-->

# Intro

- [Official Doc](https://github.com/google/sanitizers/wiki)

## [Address Sanitizer](https://github.com/google/sanitizers/wiki/AddressSanitizer)

快速的内存错误检查工具，可以找到C\C++当中的{堆，栈，全局}buff overflow, use-after-{free, return, scope},
Initialization order bugs, Memory leaks。
分别从[LLVM](http://llvm.org/)的3.1版本以及[GCC](http://gcc.gnu.org/)4.8版本开始，
AddressSanitizer(ASan)已经被集成。
ASan不仅支持Linux-like以及Unix-like操作系统，还支持多种不同的架构。

- Average slowdown ~2x
- 工具由LLVM插桩模块以及一个动态的运行库组成（替换了malloc）
- [Algorithm Design](https://github.com/google/sanitizers/wiki/AddressSanitizerAlgorithm)
- [Comparison with Others](https://github.com/google/sanitizers/wiki/AddressSanitizerComparisonOfMemoryTools)

### How to USE

```
Simply compile and link your program with -faddress-sanitizer flag. 
To get a reasonable performance add -O1 or higher. 
To get nicer stack traces in error messages add -fno-omit-frame-pointer. 
To get perfect stack traces you may need to disable inlining (just use -O1) and tail call elimination (-fno-optimize-sibling-calls).
```

Turniing off instrumentation
```
#if defined(__clang__) || defined (__GNUC__)
# define ATTRIBUTE_NO_SANITIZE_ADDRESS __attribute__((no_sanitize_address))
#else
# define ATTRIBUTE_NO_SANITIZE_ADDRESS
#endif
...
ATTRIBUTE_NO_SANITIZE_ADDRESS
void ThisFunctionWillNotBeInstrumented() {...}
```
