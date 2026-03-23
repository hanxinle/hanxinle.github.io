---
layout: post
title: 状态模式实现与内存泄漏问题解决
date: 2023-01-14 14:33:51
categories:
- 编程语言
tags:
- 设计模式
- C++
---
# 1 引言
## 1.1 写作背景
我看的设计模式资料使用 C# 和 Java 实现，这两种语言不需要处理内存泄漏问题。后续发现按照资料仿写的 C++ 代码有内存泄漏的情况，于是尝试解决这个问题，对 C++ 有新的理解，故记录之。<!-- more -->
## 1.2 参考资料
1. [本文参考的单例模式写法](https://github.com/jaredtao/DesignPattern/tree/master/code/Behavior/State)
2. [补充实现](https://github.com/JakubVojvoda/design-patterns-cpp/blob/master/state/State.cpp)
3. [两种状态模式实现，第二种与资料1类似](https://github.com/liu-jianhao/Cpp-Design-Patterns/tree/master/State)
4. 书籍-大话设计模式
5. headfirst设计模式
6. 经典设计模式黑皮书.
7. [source code of Design Patterns in Modern cpp](https://github.com/Apress/design-patterns-in-modern-cpp)
8. [Visual Leak Detector](https://kinddragon.github.io/vld/)


# ２ 问题的发现与检测
## 2.1 发现和分析问题
状态模式有个特点，即调用者调用状态A下的func1，其状态可**自动**切换为状态B，我在第一版的实现时重点在实现原型（从 Java / C# 翻译代码），
状态改变的代码是如下形式：

```
Class User;
class IState(){
public:
    virtual func(Context * ctx) = 0;
}；

class State1:public IState {
public:
    func(Context * ctx) {
        // some ops
        ctx->setState(new State2);   //leak
     }
}；

class State2:public IState {
public:
    func(Context * ctx) {
        // some ops
        ctx->setState(new State1);   //leak
    }
}；

Class User {
public:
    IState * stt_;
    void setState(ISate * p) {
        if (nullptr != stt_){
            delete stt_;
            stt_ = p;
        }
    }
}

// this param cause mem link
ctx->setState(new State1());
```
## 2.2 检测问题
上述实现，在切装状态的调用时，其会 new 个新对象，而没有释放，我尝试过在函数内部每次 new 对象传入函数，然后 delete 它，不出意外程序错误。检查内存泄漏的手段是在 vs2019 下实施的，样本代码如下：
```
#define _CRTDBG_MAP_ALLOC
#include <stdlib.h>
#include <crtdbg.h>
// include other files.


int main(int argc,char **argv)  {

    // my test code here.
    // init User * pu;
    // cause mem-leak.
    pu->setState(new sa);



    _CrtDumpMemoryLeaks();
    return 0;
}
``` 
更新信息：写完初稿后不久，我开始使用 vld，其介绍和安装信息见参考资料 8，工具已经上传到 GitHub 仓库内，可以按照一般的添加库方法在 vs2019 内配置和使用，记得将 bin 路径下所有文件拷贝到 Debug 路径（.exe 所在路径），配置文件.ini 文件也可以一并拷贝，

# 3 使用单例模式
这种写法是改变传入 setState 的对象指针的生成方法，在状态模式中，每种状态的紧紧需要该状态的一个实例即可表示当前主体所处于的状态，考虑引入单例模式，并且在引入的单例模式中，使用静态变量的写法如下，这段代码后面的篇幅再补充另一种单例写法并且阐述一些线程安全的问题。
```
#define _CRTDBG_MAP_ALLOC
#include <stdlib.h>
#include <crtdbg.h>
#include <iostream>
#include <thread>
#include <chrono>
#include <memory>

using namespace std;

class User;

class base {
public:
    virtual void func1(User *u) = 0;
    virtual void func2(User *u) = 0;
    virtual ~base() {}
};

class sa : public base {
public:
   
    static sa &getInstance() {
        static sa r;
        return r;
    }
 
    void func1(User *u);
    void func2(User *u);

private:
    sa() {}
};

class sb : public base {
public: 
    static sb &getInstance() {
        static sb r;
        return r;
    }
    
    void func1(User *u);
    void func2(User *u);

private:
    sb() {}
};

class User {
public:
    User() { p_ = nullptr; }
    base *p_; // ok
    void setState(base *p) { p_ = p; }
    void func1() { p_->func1(this); }

    void func2() { p_->func2(this); }

    ~User() {}
};

void sa::func1(User *u) {
    cout << "sa::func1" << endl;

    u->setState(&sb::getInstance());
}

void sb::func1(User *u) {
    cout << "sb::func1" << endl;

    u->setState(&sa::getInstance());
}

void sa::func2(User *u) {
    cout << "sa::func2" << endl;

    u->setState(&sb::getInstance());
}

void sb::func2(User *u) {
    cout << "sb::func2" << endl;

    u->setState(&sa::getInstance());
}

int main() {
    shared_ptr<User> pu = make_shared<User>();

    pu->setState(&sa::getInstance());

    for (int i = 0; i < 20; i++) {

        if (i % 2 == 0) {
            pu->setState(&sb::getInstance());
            pu->func2();
            pu->func1();
        } else {
            pu->setState(&sa::getInstance());
            pu->func1();
            pu->func2();
        }
        this_thread::sleep_for(chrono::seconds(1));
    }
    pu.reset();
    cout << "test done." << endl;
    system("pause");
    _CrtDumpMemoryLeaks();

    return 0;
}
```
上述代码还可以将 getInstance 函数重新实现为返回指针的形式，这样 setState 可以传入参数 sa::getInstance() 的形式，下面代码只包含核心的写法和用法，不包含main函数。单例模式有线程安全的问题需要考量，在这里不再论述。
```
class sa : public base {
public:
    static sa *getInstance() {
        if (pa_ == nullptr) {
            return new sa;
        } else {
            return pa_;
        }   
    }
private:
    static sa* pa_;
    sa(){ pa_ = nullptr;}
};
sa * sa::pa_ = nullptr; 
// .....
    puser->setState(sa::getInstance());
// .....

```

# 4 使用智能指针
写法变为 User 拥有了 share_ptr<IState> 变量替换原始指针，在 setState 中，参数传入派生类指针，写法是make_shared<Inherited Class>().下面是可运行的测试代码：
```
#define _CRTDBG_MAP_ALLOC
#include <stdlib.h>
#include <crtdbg.h>
#include <iostream>
#include <thread>
#include <chrono>
#include <memory>

using namespace std;

class User;

class base {
public:
    virtual void func1(User *u) = 0;
    virtual void func2(User *u) = 0;
    virtual ~base() {}
};

class sa : public base {
public:
    sa() {}

    
    void func1(User *u);
    void func2(User *u);
};

class sb : public base {
public:
    sb() {}
 
    void func1(User *u);
    void func2(User *u);
};

class User {
public:
    User() { p_ = nullptr; }
    shared_ptr<base> p_;
    void setState(shared_ptr<base> p) { p_ = p; }

    void func1() { p_->func1(this); }
    void func2() { p_->func2(this); }

    ~User() {}
};

void sa::func1(User *u) {
    cout << "sa::func1" << endl;

    u->setState(make_shared<sb>());
}

void sb::func1(User *u) {
    cout << "sb::func1" << endl;

    u->setState(make_shared<sa>());
}

void sa::func2(User *u) {
    cout << "sa::func2" << endl;

    u->setState(make_shared<sb>());
}

void sb::func2(User *u) {
    cout << "sb::func2" << endl;

    u->setState(make_shared<sa>());
}

int main(int argc, char **argv) {
    shared_ptr<User> pu = make_shared<User>();

    pu->setState(make_shared<sa>());

    for (int i = 0; i < 20; i++) {

        if (i % 2 == 0) {
            pu->setState(make_shared<sb>());
            pu->func2();
            pu->func1();
        } else {
            pu->setState(make_shared<sa>());
            pu->func1();
            pu->func2();
        }
        this_thread::sleep_for(chrono::seconds(1));
    }
    pu.reset();
    cout << "test done." << endl;
    system("pause");
    _CrtDumpMemoryLeaks();

    return 0;
}

```
# 5 其它研究

查看俄文作者的《Design Pattern with Modern C++》(2018)后发现，在上例的 func1 或者 func2 中有一种写法是保留 new 对象传参：
```
void sa::func1(User *u){
    pu->setState(new sb());
    delete this;
}
```

经过测试，发现这种写法在程序运行时会出现野指针导致崩溃，即需要使用的指针指向的对象已经释放。

并且，在 main 中，出现对 setState 的调用后，其 new 形式传参的内存泄漏时不能避免的。

另外，对于 CrtDumpMemoryLeaks() 的位置如果放置在 system 前，则会出现在 return 0 程序结束前，过早检查资源的情况。

---
# 1 引言
## 1.1 写作背景
我看的设计模式资料使用 C# 和 Java 实现，这两种语言不需要处理内存泄漏问题。后续发现按照资料仿写的 C++ 代码有内存泄漏的情况，于是尝试解决这个问题，对 C++ 有新的理解，故记录之。<!-- more -->
## 1.2 参考资料
1. [本文参考的单例模式写法](https://github.com/jaredtao/DesignPattern/tree/master/code/Behavior/State)
2. [补充实现](https://github.com/JakubVojvoda/design-patterns-cpp/blob/master/state/State.cpp)
3. [两种状态模式实现，第二种与资料1类似](https://github.com/liu-jianhao/Cpp-Design-Patterns/tree/master/State)
4. 书籍-大话设计模式
5. headfirst设计模式
6. 经典设计模式黑皮书.
7. [source code of Design Patterns in Modern cpp](https://github.com/Apress/design-patterns-in-modern-cpp)
8. [Visual Leak Detector](https://kinddragon.github.io/vld/)


# ２ 问题的发现与检测
## 2.1 发现和分析问题
状态模式有个特点，即调用者调用状态A下的func1，其状态可**自动**切换为状态B，我在第一版的实现时重点在实现原型（从 Java / C# 翻译代码），
状态改变的代码是如下形式：

```
Class User;
class IState(){
public:
    virtual func(Context * ctx) = 0;
}；

class State1:public IState {
public:
    func(Context * ctx) {
        // some ops
        ctx->setState(new State2);   //leak
     }
}；

class State2:public IState {
public:
    func(Context * ctx) {
        // some ops
        ctx->setState(new State1);   //leak
    }
}；

Class User {
public:
    IState * stt_;
    void setState(ISate * p) {
        if (nullptr != stt_){
            delete stt_;
            stt_ = p;
        }
    }
}

// this param cause mem link
ctx->setState(new State1());
```
## 2.2 检测问题
上述实现，在切装状态的调用时，其会 new 个新对象，而没有释放，我尝试过在函数内部每次 new 对象传入函数，然后 delete 它，不出意外程序错误。检查内存泄漏的手段是在 vs2019 下实施的，样本代码如下：
```
#define _CRTDBG_MAP_ALLOC
#include <stdlib.h>
#include <crtdbg.h>
// include other files.


int main(int argc,char **argv)  {

    // my test code here.
    // init User * pu;
    // cause mem-leak.
    pu->setState(new sa);



    _CrtDumpMemoryLeaks();
    return 0;
}
``` 
更新信息：写完初稿后不久，我开始使用 vld，其介绍和安装信息见参考资料 8，工具已经上传到 GitHub 仓库内，可以按照一般的添加库方法在 vs2019 内配置和使用，记得将 bin 路径下所有文件拷贝到 Debug 路径（.exe 所在路径），配置文件.ini 文件也可以一并拷贝，

# 3 使用单例模式
这种写法是改变传入 setState 的对象指针的生成方法，在状态模式中，每种状态的紧紧需要该状态的一个实例即可表示当前主体所处于的状态，考虑引入单例模式，并且在引入的单例模式中，使用静态变量的写法如下，这段代码后面的篇幅再补充另一种单例写法并且阐述一些线程安全的问题。
```
#define _CRTDBG_MAP_ALLOC
#include <stdlib.h>
#include <crtdbg.h>
#include <iostream>
#include <thread>
#include <chrono>
#include <memory>

using namespace std;

class User;

class base {
public:
    virtual void func1(User *u) = 0;
    virtual void func2(User *u) = 0;
    virtual ~base() {}
};

class sa : public base {
public:
   
    static sa &getInstance() {
        static sa r;
        return r;
    }
 
    void func1(User *u);
    void func2(User *u);

private:
    sa() {}
};

class sb : public base {
public: 
    static sb &getInstance() {
        static sb r;
        return r;
    }
    
    void func1(User *u);
    void func2(User *u);

private:
    sb() {}
};

class User {
public:
    User() { p_ = nullptr; }
    base *p_; // ok
    void setState(base *p) { p_ = p; }
    void func1() { p_->func1(this); }

    void func2() { p_->func2(this); }

    ~User() {}
};

void sa::func1(User *u) {
    cout << "sa::func1" << endl;

    u->setState(&sb::getInstance());
}

void sb::func1(User *u) {
    cout << "sb::func1" << endl;

    u->setState(&sa::getInstance());
}

void sa::func2(User *u) {
    cout << "sa::func2" << endl;

    u->setState(&sb::getInstance());
}

void sb::func2(User *u) {
    cout << "sb::func2" << endl;

    u->setState(&sa::getInstance());
}

int main() {
    shared_ptr<User> pu = make_shared<User>();

    pu->setState(&sa::getInstance());

    for (int i = 0; i < 20; i++) {

        if (i % 2 == 0) {
            pu->setState(&sb::getInstance());
            pu->func2();
            pu->func1();
        } else {
            pu->setState(&sa::getInstance());
            pu->func1();
            pu->func2();
        }
        this_thread::sleep_for(chrono::seconds(1));
    }
    pu.reset();
    cout << "test done." << endl;
    system("pause");
    _CrtDumpMemoryLeaks();

    return 0;
}
```
上述代码还可以将 getInstance 函数重新实现为返回指针的形式，这样 setState 可以传入参数 sa::getInstance() 的形式，下面代码只包含核心的写法和用法，不包含main函数。单例模式有线程安全的问题需要考量，在这里不再论述。
```
class sa : public base {
public:
    static sa *getInstance() {
        if (pa_ == nullptr) {
            return new sa;
        } else {
            return pa_;
        }   
    }
private:
    static sa* pa_;
    sa(){ pa_ = nullptr;}
};
sa * sa::pa_ = nullptr; 
// .....
    puser->setState(sa::getInstance());
// .....

```

# 4 使用智能指针
写法变为 User 拥有了 share_ptr<IState> 变量替换原始指针，在 setState 中，参数传入派生类指针，写法是make_shared<Inherited Class>().下面是可运行的测试代码：
```
#define _CRTDBG_MAP_ALLOC
#include <stdlib.h>
#include <crtdbg.h>
#include <iostream>
#include <thread>
#include <chrono>
#include <memory>

using namespace std;

class User;

class base {
public:
    virtual void func1(User *u) = 0;
    virtual void func2(User *u) = 0;
    virtual ~base() {}
};

class sa : public base {
public:
    sa() {}

    
    void func1(User *u);
    void func2(User *u);
};

class sb : public base {
public:
    sb() {}
 
    void func1(User *u);
    void func2(User *u);
};

class User {
public:
    User() { p_ = nullptr; }
    shared_ptr<base> p_;
    void setState(shared_ptr<base> p) { p_ = p; }

    void func1() { p_->func1(this); }
    void func2() { p_->func2(this); }

    ~User() {}
};

void sa::func1(User *u) {
    cout << "sa::func1" << endl;

    u->setState(make_shared<sb>());
}

void sb::func1(User *u) {
    cout << "sb::func1" << endl;

    u->setState(make_shared<sa>());
}

void sa::func2(User *u) {
    cout << "sa::func2" << endl;

    u->setState(make_shared<sb>());
}

void sb::func2(User *u) {
    cout << "sb::func2" << endl;

    u->setState(make_shared<sa>());
}

int main(int argc, char **argv) {
    shared_ptr<User> pu = make_shared<User>();

    pu->setState(make_shared<sa>());

    for (int i = 0; i < 20; i++) {

        if (i % 2 == 0) {
            pu->setState(make_shared<sb>());
            pu->func2();
            pu->func1();
        } else {
            pu->setState(make_shared<sa>());
            pu->func1();
            pu->func2();
        }
        this_thread::sleep_for(chrono::seconds(1));
    }
    pu.reset();
    cout << "test done." << endl;
    system("pause");
    _CrtDumpMemoryLeaks();

    return 0;
}

```
# 5 其它研究

查看俄文作者的《Design Pattern with Modern C++》(2018)后发现，在上例的 func1 或者 func2 中有一种写法是保留 new 对象传参：
```
void sa::func1(User *u){
    pu->setState(new sb());
    delete this;
}
```

经过测试，发现这种写法在程序运行时会出现野指针导致崩溃，即需要使用的指针指向的对象已经释放。

并且，在 main 中，出现对 setState 的调用后，其 new 形式传参的内存泄漏时不能避免的。

另外，对于 CrtDumpMemoryLeaks() 的位置如果放置在 system 前，则会出现在 return 0 程序结束前，过早检查资源的情况。

最后，对于 main 中使用 shared_ptr<User>，后续测试其一直保持 use_count 保持为 1，导致 User 对象资源不释放，在程序结束前添加 pu.reset() 将其引用技术置为0，后续测试无内存泄漏问题。