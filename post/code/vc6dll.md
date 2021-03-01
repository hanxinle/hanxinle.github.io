# dll 文件的编写和使用

[返回首页](../../index.md) 

>装载、链接、库（特别是 *Linux* 平台相关内容）

> 以前根据本科毕业设计写过一篇关于 VC++6.0 创建 dll 共享的博客，这篇文字简述下 vs2017 下创建动态链接库的操作。

更新：

现在创建 dll 及使用 dll 不需要原来那么复杂的做法。使用 dll 在同一个解决方案中，创建使用 dll 的项目，设置好项目依赖并且配置好即可使用，无需加载、卸载 dll 过程。

在 vs2017 动态链接库项目中，自己以前经常程序莫名崩溃，一个原因是在解决方案中没有设置项目依赖关系，另一个原因是在 dll 项目中，需要在“项目属性-链接器-高级-导入库”中进行设置，导出 lib、dll 文件，使用时包含头文件，并且配置好包含路径、库路径、执行路径（dll  所咋路径）就不会再出错了。

## 1 *.dll* 不同创建方法 

### 1.1 *def* 方式 

*VC++6.0* 中，当写好了函数声明及实现文件后，可以添加 .def 文件， 注明要导出的函数即可。
```c++
LIBRARY "dlltest"

EXPORTS add@1
EXPORTS sub@2
...
```
vs2017 中，创建项目类型中的“动态链接库(DLL)”和这种方式类似，可以添加自己的接口文件和实现文件，添加 .def 文件，在 EXPORTS 下写明函数名即可。
```
LIBRARY

EXPORTS
add
sub
```
### 1.2 具有导出项方式

这是 vs2017 中另一种创建 DLL 的方式，该方式为外部导出方式，生成的文件可以给其它模块调用。在项目创建时可以选择“具有导出项的动态链接库(DLL)”。在一 .h 文件中，会有如下内容（DLL2 是工程名字）：
```
#ifdef DLL2_EXPORTS
#define DLL2_API __declspec(dllexport)
#else
#define DLL2_API __declspec(dllimport)
#endif
```
将自己编写的接口及实现文件可以拷贝到这个工程中，并且在接口文件上面添加上述内容，同时在需要导出的类、函数、变量等名字前添加 *DLL2_API*, 如果少量编码，将导出项写在 DLL 入口文件也可以，不需要标记 *DLL2_API*， 这个工程生的 .def 需要自己手动添加现有项，或者自行创建。


## 2 dll 调用

编译得到 .dll、.lib 文件后，放入相应位置，同一个解决方案中，可在解决方案属性中设置其它项目依赖该 dll 项目，遇到找不到库的情况，配置库目录及预加载的库。

### 2.1 显式链接——动态调用

显式链接是应用程序在执行过程中控制加载、卸载 *dll* 文件，具有更好的灵活性。加载 *dll* 获得句柄，获得函数地址，存储在函数指针中调用 dll 中函数，调用结束后释放句柄。

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

### 2.2 隐式链接——静态调用

隐式链接就是在程序开始执行时就将 *dll* 文件加载到应用程序当中，通过 comment 实现，如下所示：

```c++
#include <windows.h>
#include <stdio.h>

#pragma comment(lib，"MyDll.lib")

int main()  {
int result;
result=min(8,10);

printf("比较的结果为%d\n",result);

return 0;

}

```
（PS：该方式看似简单，看是我这边佛系编译，遇到问题真是头大）



[返回首页](../../index.md) 