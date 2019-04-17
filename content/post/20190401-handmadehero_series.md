---
title: "Handmade Hero Video Lessons"
author: "ykdu"
tags: ["handmadehero"]
weight: 0
date: 2019-04-01T18:39:47+08:00
<!--draft: true-->
---

[Handmade Hero Project](https://handmadehero.org/watch), Handmade Game from the very beginning

<!--more-->


国外的一位游戏开发者在[Twitch](https://www.twitch.tv/handmade_hero)上面直播从零开始写一个游戏。


## Day 001 settting up with Windows build & QA


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

* win32 API is written in C, which can not overload functions?
  - why MSDN show

* win32 surfix:
  - FuncEX: 兼容使用，用于处理
  - FuncA: using ANSI char
  - FuncW: using wide char, unicode
  
* win32 game client:
  - create & register windows
