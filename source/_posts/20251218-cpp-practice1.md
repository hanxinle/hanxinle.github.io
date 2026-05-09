---
title: C++沉思(1)
date: 2025-12-18 09:36:27
tags:
- C++
---

|    日期    | 版本 | 作者 | 变更 |
| :--------: | :--: | :--: | :--: |
| 2025-12-18 | V1.0 | Lex  | 初稿 |



# 文档背景

一次技术沟通中，发现与我做不同项目的同行，在C++的机制和操作系统 API 上特别有心得，而这是我近两年的工作和学习中用不上，也没有深入学的内容，交流后对我有很大启发，很过瘾。

这些内容解释了很多原理，填补了我的认知空白，也间接指导了我后续的实践，非常开心有这样的交流，后面我拉了个微信群，只拉取了 C++ 开发在里面，把靠谱的 C++ 都拉进群。

<!-- more -->

# 术语解释

- **Dump（转储文件）**：进程崩溃时保存的内存快照，可用于事后调试分析
- **调用约定**：函数参数传递顺序、栈清理责任、名字修饰方式的统一规范
- **信号槽**：Qt 的一种对象间通信机制，类似于观察者模式
- **DLL（动态链接库）**：Windows 系统的共享库，可在运行时加载
- **this 指针**：指向当前对象的指针，是成员函数的隐含参数
- **MD/MT**：Visual Studio 的运行时库链接方式

# 参考资料

- 《Windows 高级编程》
- MSDN: MiniDumpWriteDump
- 《程序员的自我修养》



---

# 1 语言机制

## 1.1 Windows 怎么收集 dump，单进程崩溃怎么执行，多进程崩溃怎么收集？

### 基本概念

Dump 文件是进程崩溃时内存状态的快照，包含线程信息、堆栈信息、模块信息等。Windows 提供 `DbgHelp.dll` 来生成 Dump 文件。

### 单进程崩溃收集

**方法一：使用 SetUnhandledExceptionFilter（代码方式）**

```cpp
#include <windows.h>
#include <dbghelp.h>
#include <iostream>
#include <ShlObj.h>

#pragma comment(lib, "dbghelp.lib")

LONG WINAPI UnhandledExceptionHandler(EXCEPTION_POINTERS* pExceptionInfo) {
    // 获取文档路径
    TCHAR szPath[MAX_PATH];
    SHGetFolderPath(NULL, CSIDL_PERSONAL, NULL, 0, szPath);
    
    // 生成文件名（带时间戳）
    SYSTEMTIME st;
    GetSystemTime(&st);
    TCHAR szFileName[MAX_PATH];
    wsprintf(szFileName, TEXT("%s\\crash_%04d%02d%02d_%02d%02d%02d.dmp"),
             szPath, st.wYear, st.wMonth, st.wDay, 
             st.wHour, st.wMinute, st.wSecond);
    
    // 创建 Dump 文件
    HANDLE hFile = CreateFile(szFileName, GENERIC_WRITE, 0, NULL, 
                              CREATE_ALWAYS, FILE_ATTRIBUTE_NORMAL, NULL);
    
    if (hFile != INVALID_HANDLE_VALUE) {
        MINIDUMP_EXCEPTION_INFORMATION mdei;
        mdei.ThreadId = GetCurrentThreadId();
        mdei.ExceptionPointers = pExceptionInfo;
        mdei.ClientPointers = FALSE;
        
        // 写入 Dump
        MiniDumpWriteDump(GetCurrentProcess(), GetCurrentProcessId(), 
                         hFile, MiniDumpNormal, &mdei, NULL, NULL);
        CloseHandle(hFile);
        
        std::cout << "Dump saved to: " << szFileName << std::endl;
    }
    
    return EXCEPTION_EXECUTE_HANDLER;
}

int main() {
    SetUnhandledExceptionFilter(UnhandledExceptionFilter);
    
    // 触发崩溃测试
    int* p = nullptr;
    *p = 1;  // 空指针访问崩溃
    
    return 0;
}
```

**方法二：使用 Windows Error Reporting（WER）**

无需代码，系统自带。崩溃时自动收集，保存在：
```
C:\ProgramData\Microsoft\Windows\WER\ReportQueue\
```

启用方法：注册表 `HKLM\Software\Microsoft\Windows\Windows Error Reporting\LocalDumps`

### 多进程崩溃收集

多进程场景（如主进程+子进程）需要父进程监控子进程：

```cpp
#include <windows.h>
#include <tlhelp32.h>
#include <dbghelp.h>

// 创建子进程并监控
void CreateAndMonitorProcess(const char* exePath) {
    STARTUPINFOA si = {sizeof(si)};
    PROCESS_INFORMATION pi;
    
    if (CreateProcessA(exePath, NULL, NULL, NULL, FALSE, 
                       DEBUG_PROCESS, NULL, NULL, &si, &pi)) {
        
        DEBUG_EVENT de;
        while (WaitForDebugEvent(&de, INFINITE)) {
            if (de.dwDebugEventCode == EXIT_PROCESS_DEBUG_EVENT) {
                // 子进程退出，收集 Dump
                HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, 
                                              FALSE, de.dwProcessId);
                if (hProcess) {
                    char dumpPath[MAX_PATH];
                    sprintf(dumpPath, "child_dump_%d.dmp", de.dwProcessId);
                    
                    HANDLE hFile = CreateFileA(dumpPath, GENERIC_WRITE, 0, 
                                              NULL, CREATE_ALWAYS, 0, NULL);
                    if (hFile != INVALID_HANDLE_VALUE) {
                        MiniDumpWriteDump(hProcess, de.dwProcessId, hFile,
                                         MiniDumpNormal, NULL, NULL, NULL);
                        CloseHandle(hFile);
                    }
                    CloseHandle(hProcess);
                }
            }
            ContinueDebugEvent(de.dwProcessId, de.dwThreadId, 
                               DBG_CONTINUE);
        }
        
        CloseHandle(pi.hProcess);
        CloseHandle(pi.hThread);
    }
}
```

### 实际应用建议

| 场景 | 推荐方案 | 说明 |
|------|----------|------|
| 客户端应用 | SetUnhandledExceptionFilter | 用户体验好，可自定义保存路径 |
| 服务端/守护进程 | 父进程监控 | 子进程崩溃不影响主进程 |
| 生产环境 | WER + 日志 | 系统级别，无需额外代码 |

---

## 1.2 函数调用约定是指什么？

### 基本概念

调用约定（Calling Convention）是函数调用时的一套规则，包括：
1. 参数传递顺序（从左到右 vs 从右到左）
2. 栈清理责任（调用方 vs 被调方）
3. 名字修饰规则（编译器内部标识）
4. this 指针传递方式（C++ 成员函数）

### 常见调用约定

| 调用约定 | 参数传递 | 栈清理 | 名字修饰 | 备注 |
|----------|----------|--------|----------|------|
| `__cdecl` | 右→左 | 调用方 | `_functionName` | C/C++ 默认 |
| `__stdcall` | 右→左 | 被调方 | `_functionName@N` | Windows API |
| `__fastcall` | ECX, EDX → 栈 | 被调方 | `@functionName@N` | 快速调用 |
| `__thiscall` | ECX + 栈 | 被调方 | C++ 编译器决定 | 成员函数默认 |
| `__vectorcall` | XMM 寄存器 | 被调方 | `functionName@@` | SIMD 优化 |

### 代码示例

```cpp
// 显式指定调用约定
int __cdecl cdeclFunc(int a, int b);    // 调用方清理栈
int __stdcall stdcallFunc(int a, int b); // 被调方清理栈（Windows API）
int __fastcall fastcallFunc(int a);     // 用 ECX, EDX 传参

// 混用可能导致栈不平衡崩溃
#ifdef _MSC_VER
    #define MYAPI __stdcall
#else
    #define MYAPI __cdecl
#endif

extern "C" int MYAPI MyExportFunction(int param);
```

### 实际应用

**问题：** DLL 导出函数如果约定不匹配，会导致栈错误

```cpp
// DLL 导出（被调用方）
extern "C" __declspec(dllexport) int __stdcall Add(int a, int b) {
    return a + b;
}

// 调用方必须匹配
typedef int(__stdcall *AddFunc)(int, int);
HMODULE h = LoadLibrary("mydll.dll");
AddFunc fn = (AddFunc)GetProcAddress(h, "Add@8");  // @8 表示参数占用8字节
```

---

## 1.3 Qt 的信号槽怎么传递数据结构？例如跨线程的时候如何传递？

### 基本原理

Qt 信号槽默认是同步的（直接调用），跨线程时通过**事件队列**异步传递。信号发送时，参数会被拷贝到事件队列，由目标线程的事件循环处理。

### 传递方式

```cpp
// 方式一：直接传递（栈上的局部变量需要注意生命周期）
emit mySignal(someData);  // 如果是值类型，会自动拷贝

// 方式二：使用指针（需要手动管理生命周期）
emit mySignal(&someData);  // 危险！对象可能在接收前被销毁

// 方式三：使用 QSharedPointer（推荐）
emit mySignal(QSharedPointer<MyData>::create(args));

// 方式四：Qt 5.7+ 使用 Qt::QueuedConnection 显式跨线程
emit mySignal(args);  // 自动处理
```

### 跨线程传递示例

```cpp
// MyData.h
class MyData {
public:
    int id;
    QString name;
    QVector<double> values;  // 可以传递复杂数据结构
    Q_OBJECT
};

// Worker 线程
class Worker : public QObject {
    Q_OBJECT
public slots:
    void doWork(const MyData& data) {
        // 在线程中处理数据
        qDebug() << "Processing:" << data.name;
    }
signals:
    void workDone(const MyData& result);
};

// 主线程
class MainWindow : public QWidget {
    Q_OBJECT
public:
    void startTask() {
        MyData data;
        data.id = 1;
        data.name = "Test";
        data.values = {1.0, 2.0, 3.0};
        
        // 跨线程发送信号，参数会被自动拷贝/序列化
        // Qt::QueuedConnection 会将参数放入事件队列
        Qt::ConnectionType type = Qt::QueuedConnection;
        
        // 方法一：直接 emit，Qt 自动处理跨线程
        // 内部会使用 Qt::QueuedConnection
        emit startWorkSignal(data);  // data 会被拷贝
        
        // 方法二：显式指定队列连接
        connect(this, &MainWindow::startWorkSignal,
                m_worker, &Worker::doWork, Qt::QueuedConnection);
        
        // 方法三：使用 Q_ARG 队列参数（Qt 4 风格）
        QMetaObject::invokeMethod(m_worker, "doWork",
            Qt::QueuedConnection, Q_ARG(MyData, data));
    }
    
signals:
    void startWorkSignal(const MyData&);
    
private:
    Worker* m_worker;
};
```

### 关键点总结

| 场景 | 处理方式 | 注意事项 |
|------|----------|----------|
| 基础类型(int/QString) | 自动拷贝 | 默认即可 |
| 自定义类型 | 需要注册到 Qt 元对象系统 | 使用 Q_DECLARE_METATYPE |
| 指针/引用 | 谨慎使用 | 考虑生命周期 |
| 大数据量 | 移动语义/QSharedPointer | 减少拷贝开销 |
| 跨线程 | Qt::QueuedConnection | 自动处理线程切换 |

**重要：** 自定义类型需要注册：

```cpp
// 注册自定义类型
Q_DECLARE_METATYPE(MyData);

// 在 connect 前调用（如果跨线程）
qRegisterMetaType<MyData>("MyData");
```

---

## 1.4 怎么加载不同路径下的 dll？

### 方法一：LoadLibrary 显式加载

```cpp
#include <windows.h>

typedef int(*AddFunc)(int, int);

int main() {
    // 方法1：绝对路径
    HMODULE h = LoadLibraryA("C:\\mylib\\mydll.dll");
    
    // 方法2：相对路径（相对于当前工作目录）
    h = LoadLibraryA(".\\mydll.dll");
    
    // 方法3：使用环境变量路径
    const char* envPath = getenv("MYDLL_PATH");  // 用户设置环境变量
    if (envPath) {
        std::string dllPath = std::string(envPath) + "\\mydll.dll";
        h = LoadLibraryA(dllPath.c_str());
    }
    
    if (h) {
        AddFunc fn = (AddFunc)GetProcAddress(h, "Add");
        if (fn) {
            int result = fn(1, 2);
        }
        FreeLibrary(h);
    }
    
    return 0;
}
```

### 方法二：SetDllDirectory / AddDllDirectory（推荐）

```cpp
#include <windows.h>

int main() {
    // 方法1：设置默认搜索路径（替换系统搜索路径）
    SetDllDirectoryA("C:\\mylib\\dlls");
    
    // 方法2：添加到搜索路径列表开头（推荐，保留系统路径）
    AddDllDirectory(L"C:\\mylib\\custom_dlls");
    AddDllDirectory(L"C:\\program files\\myapp\\plugins");
    
    // 之后 LoadLibrary 会先搜索这些路径
    HMODULE h = LoadLibraryA("mydll.dll");  // 自动找到
    
    return 0;
}
```

### 方法三：修改可执行文件目录

```cpp
#include <windows.h>

int main() {
    // 获取可执行文件所在目录
    char path[MAX_PATH];
    GetModuleFileNameA(NULL, path, MAX_PATH);
    
    // 去掉文件名，获取目录
    std::string dir = path;
    size_t pos = dir.rfind('\\');
    if (pos != std::string::npos) {
        dir = dir.substr(0, pos);
    }
    
    // 添加到 DLL 搜索路径
    std::string dllPath = dir + "\\dlls";
    SetDllDirectoryA(dllPath.c_str());
    
    return 0;
}
```

### 方法四：延迟加载（Delay Load）

Visual Studio 项目属性 → 链接器 → 输入 → 延迟加载的 DLL

```cpp
// 编译时不需要 .lib 文件，运行时加载
#pragma comment(linker, "/delayload:mydll.dll")

// 使用方式一样，但首次调用时才加载
int result = Add(1, 2);  // 首次调用时才 LoadLibrary
```

### 实际应用对比

| 方法 | 优点 | 缺点 |
|------|------|------|
| 绝对路径 | 明确 | 不灵活，迁移麻烦 |
| SetDllDirectory | 简单全局生效 | 需管理员权限修改 |
| AddDllDirectory | 安全，推荐 | Windows 7+ |
| 延迟加载 | 开发简单 | 运行时才报错 |
| 环境变量 | 用户可配置 | 需设置环境变量 |

---

## 1.5 this 指针是怎么传递的？

### 基本原理

this 指针是成员函数的隐含参数，在 C++ 中通常通过寄存器或栈传递。

### 不同调用约定的传递方式

```cpp
class MyClass {
public:
    // 隐含实现类似：void __thiscall method(MyClass* this, int a, int b)
    void method(int a, int b);
    
    // static 函数没有 this
    static void staticMethod(int a);
};
```

| 调用约定 | this 传递位置 | 说明 |
|----------|---------------|------|
| `__thiscall` | ECX 寄存器 | 32位 MSVC 默认 |
| x64 调用约定 | RCX 寄存器 | 64位默认 |
| GCC/Clang | RDI 寄存器 | 64位 System V ABI |

### 32位 vs 64位 示例

```cpp
class Calculator {
public:
    int add(int a, int b) { return a + b; }
    
private:
    int m_value;
};

// 32位汇编（MSVC）:
// mov ecx, [esp+4]    ; this 指针放入 ecx
// mov eax, [ecx]      ; 读取 m_value

// 64位汇编（MSVC）:
// mov rcx, rdx        ; this 指针（第一个隐藏参数）
// mov eax, [rcx]      ; 读取 m_value
```

### 虚函数的 this 调整

```cpp
class Base {
public:
    virtual void vfunc() {}
};

class Derived : public Base {
public:
    void vfunc() override {}
};

Base* p = new Derived();
// p 可能是指向 Derived 对象的 Base 子对象的指针
// 虚函数调用需要先调整 this 指针到完整的 Derived 对象
```

编译器会生成类似代码：
```cpp
// 原始代码
p->vfunc();

// 实际生成
void* adjusted_this = p;  // 可能需要调整
((void(*)(void*))(*(void**)(*(void**)adjusted_this)[0])(adjusted_this);
```

### 实际应用

**问题：** 回调函数如何获取 this？

```cpp
// 错误的做法：普通函数没有 this
void callback() { 
    // 无法访问成员变量
}

// 正确做法：使用 lambda / std::bind / 成员函数指针
class MyClass {
public:
    void init() {
        // 方式1: lambda（推荐）
        auto cb = [this]() { handleCallback(); };
        registerCallback(cb);
        
        // 方式2: std::bind
        auto cb2 = std::bind(&MyClass::handleCallback, this);
        registerCallback(cb2);
        
        // 方式3: 成员函数指针
        using Callback = void(*)(MyClass*);
        Callback fn = &MyClass::staticCallback;
        fn(this);
    }
    
    void handleCallback() { /* 可以访问 this */ }
    
    static void staticCallback(MyClass* p) { p->handleCallback(); }
};
```

---

## 1.6 32位、64位系统，int、long 是多少字节，哪些数据类型的字节变了？哪些不变？为什么不变？

### 数据类型字节大小

| 类型 | 32位 | 64位 | 变化？ |
|------|------|------|--------|
| char | 1 | 1 | 不变 |
| short | 2 | 2 | 不变 |
| int | 4 | 4 | 不变 |
| long | 4 | **8** | **变化** |
| long long | 8 | 8 | 不变 |
| float | 4 | 4 | 不变 |
| double | 8 | 8 | 不变 |
| pointer | 4 | **8** | **变化** |
| size_t | 4 | **8** | **变化** |
| ptrdiff_t | 4 | **8** | **变化** |

### 变化的根本原因

1. **指针变为 8 字节**：为了支持 64 位地址空间（2^64 = 16EB 寻址能力）
2. **long 变为 8 字节**（仅 Windows）：Windows 采用 LLP64 模型，long 保持 4 字节；Linux/macOS 采用 LP64 模型，long 变为 8 字节

### LP64 vs LLP64 模型

```
LP64 (Linux, macOS, BSD):
- long (Long) = 64-bit
- pointer = 64-bit

LLP64 (Windows):
- long long = 64-bit  
- long = 32-bit (保持兼容)
- pointer = 64-bit
```

### 代码示例

```cpp
#include <iostream>
#include <cstdint>

int main() {
    std::cout << "sizeof(int): " << sizeof(int) << std::endl;           // 4
    std::cout << "sizeof(long): " << sizeof(long) << std::endl;        // 4(Win) / 8(Linux)
    std::cout << "sizeof(long long): " << sizeof(long long) << std::endl; // 8
    std::cout << "sizeof(void*): " << sizeof(void*) << std::endl;       // 4 / 8
    std::cout << "sizeof(size_t): " << sizeof(size_t) << std::endl;     // 4 / 8
    std::cout << "sizeof(ptrdiff_t): " << sizeof(ptrdiff_t) << std::endl;
    
    // 使用固定宽度整数避免移植问题
    int64_t i64 = 123;     // 始终 8 字节
    int32_t i32 = 123;     // 始终 4 字节
    uint64_t u64 = 456;
    
    // size_t 是最安全的数组索引类型
    std::vector<int> vec(10);
    for (size_t i = 0; i < vec.size(); ++i) {}  // 避免警告
    
    return 0;
}
```

### 实际注意事项

```cpp
// 1. 跨平台代码避免使用 long 存储指针
long addr = (long)p;  // 32位可以，64位溢出
intptr_t addr = (intptr_t)p;  // 正确：始终能存指针

// 2. 格式化输出
printf("%zu\n", size);     // size_t 用 %zu
printf("%td\n", diff);      // ptrdiff_t 用 %td
printf("%lld\n", llval);   // long long 用 %lld

// 3. Windows/Linux 差异
#ifdef _WIN32
    #define PRId64 "lld"
#else
    #define PRId64 "ld"
#endif
printf("%" PRId64 "\n", value);

// 4. 内存对齐
struct Data {
    char c;    // 1 + 7 padding
    void* p;   // 8 (64位)
};
// 32位: 1+3 + 4 = 8
// 64位: 1+7 + 8 = 16
```

---

# 2 Windows/Visual Studio 相关

## 2.1 VS 中的 MD、MTD 这些调用选项的区别是什么？

### 基本概念

MD / MT 是 Visual Studio 链接器选项，决定运行时库（Runtime Library）的链接方式：

- **M** = **M**ultithreaded（多线程）
- **D** = **D**ynamic linking（动态链接）
- **T** = **S**tatic linking（静态链接）
- **D** = **D**ebug（调试版本）

### 选项对照表

| 编译选项 | 链接方式 | 运行时库 | 宏定义 | 特点 |
|----------|----------|----------|--------|------|
| /MD | 动态链接 | MSVCRxx.DLL | `_MT`, `_DLL` | 运行时依赖 DLL |
| /MT | 静态链接 | LIBCMT.lib | `_MT` | 静态链接，无外部依赖 |
| /MDd | 动态链接（调试） | MSVCRxxD.DLL | `_MT`, `_DLL`, `_DEBUG` | 调试版本 |
| /MTd | 静态链接（调试） | LIBCMTD.lib | `_MT`, `_DEBUG` | 调试版本 |

### 代码示例与影响

```cpp
// test.cpp
#include <stdio.h>
#include <stdlib.h>

int main() {
    printf("Hello, World!\n");
    
    // 某些函数在 debug 版本有额外检查
    int* p = (int*)malloc(100);
    if (!p) {
        return 1;  // /MDd 下会进入调试 allocator
    }
    free(p);
    
    return 0;
}
```

### 实际选择建议

| 场景 | 推荐选项 | 原因 |
|------|----------|------|
| 分发可执行文件 | /MT | 无需用户安装 VC++ 运行库 |
| 插件/DLL | /MD | 与主程序共享运行时 |
| 开发测试 | /MTd | 快速链接，独立调试 |
| 发布版 | /MT | 减少依赖，简化部署 |

### 常见问题

**1. 运行时库不匹配**

```
error LNK2005: "void * __cdecl operator new(unsigned int)" 
  (??2@YAPAXI@Z) 已经在 LIBCMT.lib 中定义
```

解决：统一所有项目的链接选项

**2. DLL 和 EXE 之间的内存分配**

```cpp
// DLL 中分配
void* AllocMem() {
    return malloc(100);  // 使用 MSVCRxx.DLL
}

// EXE 中释放（可能使用不同运行时库）
void FreeMem(void* p) {
    free(p);  // 可能崩溃！
}
```

解决：使用统一运行时库，或提供导出函数让 DLL 负责释放

**3. 调试版本 vs 发布版本混用**

```
MSVCR100D.dll (调试运行时) != MSVCR100.dll (发布运行时)
```

解决：确保加载的 DLL 与主程序使用相同的运行时库

### 项目配置位置

Visual Studio 中设置位置：
- **项目属性** → **C/C++** → **代码生成** → **运行时库**

或通过命令行：
```
/MD  /MDd  /MT  /MTd
```
