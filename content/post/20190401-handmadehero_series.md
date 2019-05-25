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
	* alloc a big size of array on stack => stack overflow (x86 visual studio 1MB, compile with given stack size
	* 指针的地址实际上需要配置地址空间才有效
	* 这个哥们目前讲解的内容有些十分的基础，包括一下C语言的基础语法规则
	* pointer aliasing
		
## Day 006
	* input from a game paddle
	* DirectInput and XInput
	* 实际上一个设备可以plug多个不同的设备（手柄），在game loop中需要循环每一个设备，并且接受相关的按键消息才可以
	* 在连接相关的静态库的时候需要注意相关静态库的环境需求，如果最后编译的可执行文件运行的环境不支持链接的静态库的load条件，
	那么整个可执行程序都不会load，所以有时候关注兼容性的时候需要注意
	* 写完代码之后，可以通过review代码检查自己的代码风格是否符合自己的期望
	* 不要在编码的阶段考虑过多的low-level optimisation，只要不写太多的灾难性的代码一切都是可以接受的

## Day 007
	* alt-f4 will provoke `WM_CLOSE` message before `WM_KEYDOWN` message
	* be careful with bool and integer when commes to compiler optimasation
	* DirectSound maintain a buffer to contain the audio data, which is palyed loop buffer-wide.
	when comes the newcomming data, the start location should be ahead the playing position.
	* `HRESULT`, `winerror.h`, checking with SUCCEESS
	* 载入dll库，创建DirectSound对象，设置cooperative level，创建primary/secondary buff then play it
	
## Day 008
	* TO CHECH
		- what is primary buffer and secondary buffer??
		- what is waveformt variable member means??
		- SoundBuffer: square wave
			- ToneHz, how many full circles in one second
			- ToneVolume, the value in the sound buffer, 
			- SamplesPerSecond, 采样率，数据信号和模拟信号之间的转化，每一秒的波形由这么多个离散的数字信号构成
			- BytePerSample，一个sample需要包括所有的channel，每一个channel的volume大小使用一个uint16描述
			- what is channel

## Day 009 
	Variable-Pitch Sine Wave Output
	* Review DirectSound init and square wave
	* Story Time: Indi Game Jam, I have no idea what is that, 调试Sound Code 相关的时候需要一个音调比较精确的耳机
   	* 一个重复填充的问题，由于声卡或者相关设计包括DirectSound库如何计算以及更新GetCurrentPosition方法我们不得而知，因此是可以去假设这个position的更新存在一个延迟（在老的平台上面的确也是这样的）。
	* square wave is hard for ear
	* math library will also not be used, handmade hero will implement one itself
	* int fix_point float_point 24:55-42:25
	* gap problem, bug found
		- PlayCursor is always the same as ByteToLock, PlayCursor did not advanced, because play are called every frame
		- not using SUCCEEDED
		- upgrade the warning level
		- Sound Skip
	- lower Lantency
		+ if your write to much to the buffer, next time you write some new data, which must be played when the previous data is played
		
### TOPICS
	* basic conceptions
	* 调整了day008的代码
	* 控制了写入的buffer的大小， 由于这个实际上是个ring buffer，因此减少一次写入的内容可以解决延迟问题
	* audio skip的优化问题
	* fix point 以及 float point，IEEE754
	* Thanksgiving holiday

## Day 010 Query Performance Counter and RDTSC
	* The Interl Achitecture Reference Manual
	* RDTSC， READ Time-Stamp Counter, processor's time-stamp counter for a CPU clock to EDX:EAX
		- Counter for CPU instruction clock
			- #define TIME_CONSUME ((double)(end - beg) / CLOCKS_PER_SEC), end = clock(), this might be Wall time
		* Wall clock time - time as it passes in the real world. Measured in seconds.
		* Processor time - how many cycles? this is related to wall clock time by processor frequency, but for a long time now frequency varies a lot and quickly.
		
### TOPIC
	* RDTSC: Read Time-Stamp Counter, processor's time-stamp counter for CPU instruction clock to EDX:EAX
		- 有些系统会记录CPU的clock计数，这样记录的时间差是所有在这个cpu上面执行过的操作的clock circle
		- 有些会根据进程进行划分，只记录某一特定进程的时钟数
		- processor clock
		* 3.2 Billion of those per second, this might variable due to the performance of cpu might change, 电源管理
		* Discontinous Values, 多核的计数器的值没有即使同步
		* 目前有一些主板提供了专门的timing device精度上面会优于 RDTSC
	* QueryPerformanceCounter, wall-clock
		- MSDN recommended
			- use `QueryPerformanceCounter` and `QueryPerformanceFrequency`, 他们都是API调用时候的性能会比RDSTC要慢
			-
