---
title: "Handmade Hero Video Lessons"
author: "ykdu"
tags: ["handmadehero"]
weight: 0
date: 2019-04-01T18:39:47+08:00
<!-- draft: true -->
---

[Handmade Hero Project](https://handmadehero.org/watch), Handmade Game from the very beginning

<!--more-->


国外的一位游戏开发者在[Twitch](https://www.twitch.tv/handmade_hero)上面直播从零开始写一个游戏。


## Day 001 settting up with Windows build & QA

* 内容总结：
	1. 搭建Windows平台的开发环境，使用VS的command line tools
	2. 构建的脚本自己编写，这样可以学习被VS等IDE屏蔽的细节
	3. MSDN函数的reference查询（调用方式，包含头文件，包含库文件），撰写一个WinMain，以及HelloWorld Messagebox。
	
***

* 微软的VC++工具 + visual studio debugger 是视频中两个主要使用的工具

* visual studio = debuger, shortcut from cmd devenv
* subst:
    * 将一个文件夹目录 
    * Associates a path with a drive letter. If used without parameters, subst displays the names of the virtual drives in effect.
* 讲了一个关于intel amd的 x64 和 x86的故事
* startup 启动目录
* shortcut + shell
* `cl` and `vcvarsall` `vcvarsall x64`
* pushd popd
* c runtime library runs the program not windows
    * handmade will not call anything from it, but it must be linked, to stack staffs

```
win32_handmade.exe!WinMain(HINSTANCE__ * hInstance, HINSTANCE__ * hPrevInstance, char * lpCmdLine, int nShowCmd) 行 8	C++
[内联框架] win32_handmade.exe!invoke_main() 行 102	C++
win32_handmade.exe!__scrt_common_main_seh() 行 288	C++
kernel32.dll!BaseThreadInitThunk()	未知
ntdll.dll!00007ff8f6b53691()	未知
```

+ Q: Why start from scratch ? 有人问Miblo为为什么要从这么基础的地方开始而不选择去使用引擎
    * 对于一个严谨的游戏开发者而言需要学会游戏制作当中大大小小的所有事情是很重要的。
    * 如果从引擎开始，那么对很多基础知识的认识可能会出现偏差，引擎会产生一个功能强大的中间层，
    并且将真正的游戏代码逻辑功能与开发者隔离开。开发者学会的是使用引擎，而不是制作游戏。
    * Knowing all stuffs underneath will make you stronger even in an Engine.
    * Miblo的观点：如果没有人去学习如何制作轮子（引擎）而只是去使用轮子（引擎），那么以后谁来制作新的轮子（引擎）。

## Day 002 win32 windows

* 内容总结
	1. 在第一课只有一个对话框的基础上，创建的窗口，相关的属性
	2. 为相关的客户端窗口注册相关的消息回调函数
	3. 注意Windows希望自己掌控Message的dispatch
	4. 相应WM_PAINT消息，利用BeginPaint以及EndPaint，对窗口背景进行绘制（纯色）
	
---

* win32 API is written in C, which can not overload functions?
  - why MSDN show

* win32 surfix:
  - FuncEX: 兼容使用，用于处理
  - FuncA: using ANSI char
  - FuncW: using wide char, unicode
  
* win32 game client:
  - create & register windows
  - loop

## Day 003 Allocating a Backbuffer

* 内容总结
	1. 基于Day002，处理剩余部分需要处理的`Windows Message`
	2. 希望可以使用自己绘制的buff对客户端的窗口进行填充，而不是只填充白色或者黑色
	3. 今天Casey貌似有点卡壳了，1个小时时间讲完了，只将创建buff的函数讲清楚了

---
* Resource acquisition is initialization (RAII)
  - Handle Resource all together aggregately instead of indivival
  - start to think construction desctruction acquisition and free in waves?
  
* 之前的可能可以设置客户端的背景颜色，不是白色就是黑色，第三天的相关工作

* Device Context & Memory Device Context
  * Device Context, 是windows用来屏蔽不同种设备绘制复杂度而提供的接口，
  可以让程序以一套的接口再不同设备是进行绘图

* QA:
	* 是否能够自己生成Buff再传递给Windows
		* 也许可以但是不是一个好方法
	* sleep will prevent melting the CPU
	* pre ANSI C didn't allow return structs, funny


## Day 004 
	* Correction about StretchDIBits() BitBlt()
		* BitBlt
			* relatived faster, 依赖win32 build-in的device context，速度会比较快速（内部优化）
			* 但是只能DC->DC，所以这个老哥开始想要使用一个memory dc来保存相关的buffer，但是最后没有讲完
		* StrechDIBits:
			* slower，不依赖win32相关内容，依赖一个`void`的buffer，可以使用openGL等方式生成
			* 貌似具有较高的自由度
			
			
	* VirtualAlloc VirtualFree VirtualProtect
		* Virtual Alloc VirtualFree 很好理解
		* VirtualProtect可以通过将内存设置为不能读写，来检测 user after free bug
		* 区分
			* VirtualAlloc, low-level
			* HeapAlloc
		
	* 我真的佩服这个老哥，老是refactor变量的名字
	
	* 他的WM_SIZE之后的确只是重新绘制新增的部分，而我这边会绘制整个屏幕
	
	* 处理字节计算的时候需要注意运算溢出的问题
	
### QA
	* using 0 instead of NULL
		* less definition is better
	* malloc and new and HeapAlloc
		* ??
	* streaming for two years 
	* for writecopy and COW
	* alpha

## Day 005
	* Overview all the week
	* pass pointer or pass value, if pass a large structvj
	* 处理指针时候需要注意当两个指针指向同一片地址
	* 实际上栈上的变量实际上都是一样的，只是作用域有所不同，对于C++而言还有一个构造析构函数的调用
	* access violation
		- 默认的程序的虚拟地址空间没有映射到任何的物理页面上面，除非调用了malloc/VirtualAlloc等
		- 当程序通过指针访问一个无效的页面时，cpu将会设置相关的异常标志位给操作系统，之后操作系统将会抛出异常，segmentation fault
		
## Day 006

## Day 007

## Day 008

## Day 009

## Day 010
