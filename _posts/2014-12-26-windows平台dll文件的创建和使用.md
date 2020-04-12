---
layout:     post
title:      dll文件的编写和使用
subtitle:   学士论文所用方法记录
date:       2014-10-22
author:     小乐
header-img: img/post-main-page.png
catalog: true
tags:
    - Windows
    - 库
---

>装载、链接、库（特别是 *Linux* 平台相关内容）值得读一读


# 1 *.dll* 不同创建方法 

## 1.1 *def* 方式 

第一步，*def* 方式创建：*VC6* 找不到 *stdafx.h*,所以创建空工程，*stdafx.h*不包含在项目中不影响本项目。*dll*工程建立好后，新建一*.cpp* 文件，命名为 *Dlltest.cpp*，直接去复制一个其它的 *DllMain* 函数的内容，有*warning* 就添加

```c++
#include<windows.h> 
``` 

*DllMain*下直接写要定义的函数,如 

```c++
int add(int a,int b){ return (a+b);}
```

去工程目录，新建记事本文档，命名 *dlltest.def*。打开该文件进行编辑，输入以下内容：
```c++
LIBRARY "dlltest",EXPORTS add@1
```
(PS : 把所有要导出的函数都按照这一格式记录，序号依次是1,2,3,4...).

保存文件，这一步非常重要，保存后在该工程浏览窗口 *source file* 栏中点击添加文件，浏览找到*dlltest.def*。接下来，编译，链接，千万不要一边写一边链接，那样 *DLL* 改起来太麻烦。不小心生成了*DLL*，不想要了就把刚刚编译和链接的过程中生成的文件，能删了都删掉，再重新编译，链接。

第二步， *def*方式创建后的使用：*workstation* 中添加新的工程，*win32 console app* (找类似描述的选项)。设置成 *ACTIVE* 模式，右击设置。 好了在创建一个*call.cpp*. 输入 
```c++
#pragma comment (lib,"dllset.lib"),
_declspec (dllimport) int add(int a,int b)
```
把 *dllset.lib* 和 *dlltest.dll*拷贝靠该工程目录，然后像是正常调用函数一样使用就行了。
```c++
int sum = add(10,10); 
printf (10+10=20);
```
yeah!!!!!!!这就是动态使用，程序运行时自动启用*dll*文件，稍后自动释放资源。

## 1.2 export模式创建

第一步，老规矩，*VC6* 找不到 *stdafx.h*,所以创建空工程，*stdafx.h* 里面功能不是必须的，删掉即可。*DLL* 工程建立好后，新建一*.cpp* 文件，命名为 *Dlltest.cpp*，直接去复制一个其它的 *DllMain* 函数的内容，直接去copy一个DLLMAIN复制进去，有警告就添加
```c++
#include <windows.h> 
```
*DLLMAIN* 下编写要定义的函数。如
```C++
 int add(int a,int b){return(a+b);}
 ```
 添加一个 *codll.h* 在里面输入：
 ```c++
 #pragma once (lib,"路径名+文件名（双反斜杠显示）") extern “C”_declspec (dllexport) int add(int a,int b); 
 ```

第二步，使用 *workstation* 中添加新的工程，*win32 console app*（同上例）。设置成 *ACTIVE* 模式，右击设置，创建 *call.cpp*， 输入 
```c++
#progma comment （lib,"dllset.lib"),extern “C”_declspec(dllimport)
```
把*dllset.lib*和*dlltest.dll*拷贝到该工程目录下，add即可被使用，用法如上一节所述。

下面是显式调用过程概述，创建一个可以 指向 *add* 的函数指针 *p*。用 *Loadlibrary*（*DLL*路径+文件名）获得返回句柄。用*p*实例一个对象 *startAdress* 使用 *GetProcAdress* (句柄，函数名字)获得起始地址 *addr*，*int i* = *addr* 就是在调用 *add*，别忘了释放资源 *free (句柄 )*。

# 2 其他说明

## 2.1 隐式链接——静态调用

隐式链接就是在程序开始执行时就将 *DLL* 文件加载到应用程序当中。实现隐式链接很容易，只要将导入函数关键字*_declspec(dllimport)* 函数名等写到应用程序相应的头文件中就可以了。下面的例子通过隐式链接调用 *MyDll.dll* 库中的 *Min* 函数。首先生成一个项目为*TestDll*，在 *DllTest.h*、 *DllTest.cpp* 文件中分别输入如下代码：

```c++
//Dlltest.h

#pragma comment(lib，"MyDll.lib")

extern "C"_declspec(dllimport) int Max(int a,int b);
extern "C"_declspec(dllimport) int Min(int a,int b);

//TestDll.cpp
#include"Dlltest.h"
int main()  {
int a;
a=min(8,10)printf("比较的结果为%d\n",a);

return 0;

}

```

在创建 *DllTest.exe* 文件之前，要先将 *MyDll.dll* 和 *MyDll.lib* 拷贝到当前工程所在的目录下面，也可以拷贝到 *windows* 的 *System* 目录下。如果 *DLL* 使用的是 *def* 文件，要删除 *TestDll.h* 文件中关键字  *extern "C"* 。*TestDll.h* 文件中的关键字 *Progam commit* 是要 *Visual C++* 的编译器在 *link* 时，链接到 *MyDll.lib* 文件，当然，开发人员也可以不使用 *#pragma comment(lib，"MyDll.lib")* 语句，而直接在工程的 *Setting* -> *Link* 页的* Object/Moduls* 栏填入 *MyDll.lib* 既可。(注：*def* 创建的 *.dll* 文件可以供给

## 2.2 显式链接——动态调用

显式链接是应用程序在执行过程中随时可以加载 *DLL* 文件，也可以随时卸载 *DLL* 文件，这是隐式链接所无法做到的，所以显式链接具有更好的灵活性，对于解释性语言更为合适。不过实现显式链接要麻烦一些。在应用程序中用 *LoadLibrary* 或 *MFC* 提供的 *AfxLoadLibrary* 显式的将自己所做的动态链接库调进来，动态链接库的文件名即是上述两个函数的参数，此后再用 *GetProcAddress()* 获取想要引入的函数。自此，你就可以象使用如同在应用程序自定义的函数一样来调用此引入函数了。在应用程序退出之前，应该用 *FreeLibrary* 或 *MFC* 提供的*AfxFreeLibrary* 释放动态链接库。下面是通过显式链接调用 *DLL* 中的 *Max* 函数的例子。

```c++

int main(void)  {

typedef int(*pMax)(int a,int b);
typedef int(*pMin)(int a,int b);

HINSTANCE hDLL;
pMax MaxHDLL=LoadLibrary("MyDll.dll");//加载动态链接库MyDll.dll文件；
Max=(pMax)GetProcAddress(hDLL,"Max");
int maxV=Max(5,8);

printf("比较的结果为%d\n",maxV);
FreeLibrary(hDLL);//卸载MyDll.dll文件；

return 0;
}

```

在上例中使用类型定义关键字 *typedef* ，定义指向和*DLL* 中相同的函数原型指针，然后通过 *LoadLibray()* 将 *DLL* 加载到当前的应用程序中并返回当前 *DLL* 文件的句柄，然后通过 *GetProcAddress()* 函数获取导入到应用程序中的函数指针，函数调用完毕后，使用*FreeLibrary()* 卸载 *DLL* 文件。在编译程序之前，首先要将DLL文件拷贝到工程所在的目录或 *Windows* 系统目录下。

# 致谢

最后，感谢王小民老师在我本科学习阶段程序设计、论文撰写上的指导。  
2011年您的《C++程序设计》这门课带我体会到编程的神奇与美感，那时拿着您的代码乐不可支，一直以来都钦佩您程序设计时的认真态度，这些都深深影响了当时初入编程大门的我。

 -- 小乐