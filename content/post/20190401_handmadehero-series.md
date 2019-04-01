---
title: "Handmade Hero Video Lessons"
author: "ykdu"
tags: ["handmadehero"]
weight: 0
date: 2019-04-01T18:39:47+08:00
draft: true
---

[Handmade Hero Project](https://handmadehero.org/watch), Handmade Game from the very beginning

<!--more-->



## Day 001 settting up with Windows build

* visual studio = debuger
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
