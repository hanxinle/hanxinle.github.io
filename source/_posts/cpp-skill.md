---
title: C/C++ 技术随笔
date: 2023-07-09 22:10:37
tags:
- C++
---





1. 单例模式的目的是确保唯一实例，确切的说是为了在应用程序的进程空间中确保唯一实例，刚刚在视频处理器的使用中被这种设计小小震撼一下：

   <!-- more -->

```
class Single {
public:
    static Single *Get() {
        static Single re;
        return &re;
    }
    void Set(int x, int y) {
        a = x;
        b = y;
    }

    void Print() { std::cout << a << " " << b << std::endl; }

private:
    int a = 0;
    int b = 0;
    Single() {}
    Single(const Single &) {}
    Single &operator=(const Single &) {}
};

int main() {
    auto s = Single::Get();
    s->Set(12, 13);
    s->Get()->Print();
    std::thread([]() {
        auto sa = Single::Get();
        sa->Set(01, 3);
           }).join();
    s->Get()->Print();
}

// output
12 13
1 3
```

