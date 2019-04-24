---
title: "Google Sanitizers"
author: "ykdu"
tags: ["memcheck", "intro", "tool"]
weight: 10
date: 2019-04-21
<!--draft: true-->
---

[Google Sanitizaers](https://github.com/kasicass/blog/blob/master/cpp/2019_02_13_google_sanitizers.md)
目前相关的工具代码都已经迁移到了[LLVM](http://llvm.org)编译器架构上。本文将针对相关工具的使用以及官方文档中的实现介绍进行翻译。

<!--more-->

# Intro

- [Official Doc](https://github.com/google/sanitizers/wiki)

## [Address Sanitizer](https://github.com/google/sanitizers/wiki/AddressSanitizer)

快速的内存错误检查工具，可以找到C\C++程序当中的{heap, stack, global} buff overflow, use-after-{free, return, scope},
Initialization order bugs, Memory leaks。
分别从[LLVM](http://llvm.org/)的3.1版本以及[GCC](http://gcc.gnu.org/)4.8版本开始，
AddressSanitizer(ASan)已经被集成。
ASan不仅支持Linux-like以及Unix-like操作系统，还支持多种不同的架构。

- Average slowdown ~2x
- 工具由LLVM插桩模块以及一个动态的运行库组成（替换了malloc）
<!-- - [Algorithm Design](https://github.com/google/sanitizers/wiki/AddressSanitizerAlgorithm) -->
<!-- - [Comparison with Others](https://github.com/google/sanitizers/wiki/AddressSanitizerComparisonOfMemoryTools) -->

## How to USE

由于相关的功能以及融合到clang+llvm当中，相关的开启和关闭也十分的便捷。
<!-- Simply compile and link your program with -faddress-sanitizer flag.  -->
<!-- To get a reasonable performance add -O1 or higher.  -->
<!-- To get nicer stack traces in error messages add -fno-omit-frame-pointer.  -->
<!-- To get perfect stack traces you may need to disable inlining (just use -O1) and [tail call elimination](https://www.geeksforgeeks.org/tail-call-elimination/) (-fno-optimize-sibling-calls). -->

### Turning on 

需要在编译和连接的时候传入-faddress-sanitizer参数，
能够通过添加-O1或者更高的优化层级来提升性能
能够通过添加-fno-omit-frame-pointer参数想错误信息中添加更加友好的栈跟踪信息
能够通过禁止内联函数（-O1）以及尾调用消除可以获得完整的栈追踪信息

### Turning off

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

上面的代码在使用clang编译前端进行编译的时候将会关闭对函数的插桩操作。

## Algorithm Design (Short Version)

* Author: Mitch Phillips 
* Translator: ykdu

使用了不同的运行时库替换了原有的malloc以及free函数的相关实现。
算法的基础思想是将malloc分配的内存周围以及free释放的内存标记为poisoned（red zones）。
程序中所有的内存的访问都将会插桩成为，先判断内存地址是否已经被标记为poisoned或者是否在red zones里面。
编译器具体对代码做的修改可以表现为如下：
```c++
// before
*address = ...; // or: *** = *address;
// after
if (IsPoisoned(address)) {
  ReportError(address, kAccessSize, kIsWrite);
}
*address = ...;  // or: ... = *address;
```
实现这个算法有至少存在两个问题需要解决，如何实现一个性能良好的IsPoisoned方法与一个简洁严谨的ReportError（提升报错的准确率）


### Memory Mapping and Instrumentation

将虚拟内存划分为不相交的两个部分：

* Main application memory(Mem): 这部分的内存属于被常规的应用代码使用的部分
* Shadow memory(Shadow): 这部分保存着shadow values（metadata）。
将Mem部分的内存标记为Poisoned的时候，意味对应的shadow memory也将会进行标记。
当程序访问Mem部分内存的时候，Sanitizer将会把Mem部分内存地址快速转换为shadow memory地址，然后进行Posioned标志位校验。
相关的插桩结果如下图所示：

```
shadow_address = MemToShadow(address);
if (ShadowIsPoisoned(shadow_address)) {
  ReportError(address, kAccessSize, kIsWrite);
}
```

#### Mapping Algorithm

对于任意对齐的8字节Mem内存而言只会存在9种不同的状态，如下所示。因此只需要使用1字节shadow内存便可以为Mem内存中8字节进行标记（单射）。

1. 所有的8字节（4字）被标记为unpoisoned，对应的shadow内存为0
2. 所有的8字节（4字）被标记为poisoned，对应的shadow内存被标记为负数
3. 实际上`malloc`会返回非负整数个8个字节对齐的内存区域，
因此当起始的k字节被标记为unpoisoned而剩下的8-k字节被标记为poisoned，对应的shadow内存被标记为k。
其中起始的k字节可以被当作，分配的合法内存，剩下的8-k的字节将会作为合法分配内存的周围内存被标记为poison。
例如`malloc(13)`将会返回两个8字节（4字）对齐的内存块，其中第一个完整的8字节（4字）与之后起始5个字节被标记为unpoisoned，
剩下的3字节将会标记为poisoned。

此时插桩的结果如下面的代码清单所示：
```
byte *shadow_address = MemToShadow(address);
byte shadow_value = *shadow_address;
if (shadow_value) {
  if (SlowPathCheck(shadow_value, address, kAccessSize)) {
    ReportError(address, kAccessSize, kIsWrite);
  }
}
// Check the cases where we access first k bytes of the qword
// and these k bytes are unpoisoned.
bool SlowPathCheck(shadow_value, address, kAccessSize) {
  last_accessed_byte = (address & 7) + kAccessSize - 1;
  return (last_accessed_byte >= shadow_value);
}
```

如果`MemToShadow(address)`访问的shadow内存的地址属于`ShadowGap`那么程序将会报错并且退出。

#### Report Error

`ReportError`将会默认作为一个函数被插桩到目标程序当中，实际上还是存在某些更加高效并间接的手段，
这里只会讨论默认的处理手段。

1. 将被访问的地址复制到`%rax(%eax)`（这里是不是写错了？EAX不是RAX下半部分吗？）地址。
2. 运行[`ud2(opcode: 0Fh 0Bh)`指令](https://docs.microsoft.com/en-us/cpp/intrinsics/ud2?view=vs-2019)，触发`SIGILL`信号。
3. 紧接着`ud2`指令生成一个1字节的执行，包括访问类型以及访问大小。所有的三条执行将会额外需要5-6字节的机器码



#### Stack

为了能够捕获stack buffer overflow错误，`AddressSanitizer`将会在需要访问的stack变量的前后插入redzone。
具体的插桩样例如下面的伪代码清单。
在需要监控的栈变量a[8]的前后分别插入一个4字节的对齐的`redzone`（`redzone1`与`redzone3`）以及为了弥补4字节对齐中需要的三个字节空间(`redzone2`)

```
// Original code:

void foo() {
  char a[8];
  ...
  return;
} // Original code ends

// Instrumented code:
void foo() {
  char redzone1[32];  // 32-byte aligned
  char a[8];          // 32-byte aligned
  char redzone2[24];
  char redzone3[32];  // 32-byte aligned
  int  *shadow_base = MemToShadow(redzone1);
  shadow_base[0] = 0xffffffff;  // poison redzone1
  shadow_base[1] = 0xffffff00;  // poison redzone2, unpoison 'a'
  shadow_base[2] = 0xffffffff;  // poison redzone3
  ...
  shadow_base[0] = shadow_base[1] = shadow_base[2] = 0; // unpoison all
  return;
} // Instrumented code ends

```


#### Unaligned accesses

如果通过相关指针操作访问的一部分在unpoisoned区域而剩下的一部分在poisoned部分，那么这样的部分越界的非法access将不会被捕获，如下伪代码清单所示。
(一个性能消耗较大的解决方案： https://github.com/google/sanitizers/issues/100)

```
int *x = new int[2]; // 8 bytes: [0,7].
int *u = (int*)((char*)x + 6);
*u = 1;  // Access to range [6-9]
```
