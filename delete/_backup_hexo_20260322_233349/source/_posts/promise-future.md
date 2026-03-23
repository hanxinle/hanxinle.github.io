---
title: C++ 特性之 promise/future 示例
date: 2023-06-27 19:26:49
tags:
- C++
---

# 参考资料

https://en.cppreference.com/w/cpp/thread/promise

# 正文

今天面试问到一个 C++ 一个异步转为同步的特性，

<!-- more -->

C++ 中可以通过 std::future 和 std::promise 实现同步。基本思路是:

1. 使用 std::promise 生成一个 promise 对象,该对象可在未来生成一个值或抛出异常。
2. 调用 promise 对象的 get_future 方法获取对应的 future 对象。该 future 对象可以获取 promise 对象设置的value或exception。
3. promise 对象传递给设置值或抛出异常的线程。
4. 线程通过调用 future 对象的 get 方法来获取值或异常,该调用会阻塞直到 promise 对象设置为止。
5. 设置值或抛出异常的线程通过 promise 对象的 set_value 或 set_exception 来设置结果。
6. get 方法获取结果后解除阻塞,主线程继续执行。这样,通过 future/promise,我们可以实现两个线程之间同步的值传递,主线程将阻塞等待结果。

示例1：

```c++
#include <future>
#include <iostream>

int doSomeWork() {
    std::promise<int> prom;
    std::future<int> fut = prom.get_future();

    auto f = [&prom]() {
        prom.set_value(42); // 设置结果
    };

    std::thread t(f);
    auto result = fut.get(); // 阻塞等待结果
    t.join();
    return result;
}

int main() {
    auto result = doSomeWork();
    std::cout << "The result is: " << result << '\n';
}
```



示例2（来自参考文献）：

```C++
#include <vector>
#include <thread>
#include <future>
#include <numeric>
#include <iostream>
#include <chrono>

void accumulate(
    std::vector<int>::iterator first, std::vector<int>::iterator last, std::promise<int> accumulate_promise) {
    int sum = std::accumulate(first, last, 0);
    std::cout << "1 accumulate running..." << std::endl;
    accumulate_promise.set_value(sum); // Notify future
}

void do_work(std::promise<double> barrier) {
    std::this_thread::sleep_for(std::chrono::seconds(1));

    std::cout << "5 thread do_work running   barrier.set_value()." << std::endl;

    barrier.set_value(0.1);
}

int main() {
    // Demonstrate using promise<int> to transmit a result between threads.
    std::vector<int> numbers = { 1, 2, 3, 4, 5, 6 };
    std::promise<int> accumulate_promise;
    std::future<int> accumulate_future = accumulate_promise.get_future();
    std::thread work_thread(accumulate, numbers.begin(), numbers.end(), std::move(accumulate_promise));

    // future::get() will wait until the future has a valid result and retrieves it.
    // Calling wait() before get() is not needed
    // accumulate_future.wait();  // wait for result
    accumulate_future.wait();
    std::cout << "2 result=" << accumulate_future.get() << '\n';
    work_thread.join(); // wait for thread completion

    // Demonstrate using promise<void> to signal state between threads.
    std::promise<double> barrier;
    std::future<double> barrier_future = barrier.get_future();
    std::cout <<  "3 before running thread do work." << std::endl;

    std::thread new_work_thread(do_work, std::move(barrier));
    std::cout << "4 after unning thread do work.   barrier_future.wait(); " << std::endl;

    barrier_future.wait();
    std::cout << "6 double = " << barrier_future.get() << std::endl;
    new_work_thread.join();
    std::cout << "7 after join." << std::endl;

    return 0;
}


```

