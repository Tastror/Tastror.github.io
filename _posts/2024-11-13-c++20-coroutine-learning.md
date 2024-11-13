---
layout: post
title: "C++20 协程学习"
date: 2024-11-13 12:24 +0800
categories: 计算机
---

这里提供一个单线程内（设备线程仅为模拟用）由主线程恢复另一协程的写法。

另外一个常用的例子是在线程 B 中恢复线程 A 的内容，通过传递 handle 进行 resume 实现。它需要修改 await_suspend 相关内容。

推荐的 c++ 协程学习网址（由他人推荐）<https://lewissbaker.github.io>

```cpp
#include <iostream>
#include <coroutine>
#include <chrono>
#include <thread>


// 异步函数返回类型：模板写法
template<typename T>
struct Async {
    struct promise_type {
        std::optional<T> result;
        Async get_return_object() {
            return Async{ std::coroutine_handle<promise_type>::from_promise(*this) };
        }
        std::suspend_never initial_suspend() { return {}; }
        std::suspend_always final_suspend() noexcept { return {}; }
        // 如果函数返回值是 void 就用这个
        // void return_void() {}
        void return_value(T v) { result = v; }
        void unhandled_exception() { std::terminate(); }
    };

    std::coroutine_handle<promise_type> handle;

    Async(std::coroutine_handle<promise_type> h) : handle(h) {}
    ~Async() { if (handle) handle.destroy(); }
    void resume() { if (!handle.done()) handle.resume(); }
    bool done() const { return handle.done(); }
    auto result() const { return handle.promise().result; }
};


// 异步等待子：模板写法
// 实现了最基本的要求：ready 返回 false 和 suspend 是 void 或 bool + return true
struct Awaitable {
    bool await_ready() const noexcept { return false; }
    void await_suspend(std::coroutine_handle<> handle) const {}
    void await_resume() const noexcept {}
};


// 模拟一个耗时的设备
class Device {
private:
    bool lock;
    int value;
public:
    Device() : lock(true), value(0) {}
    // 开一个后台线程模拟工作 1 秒然后返回的设备
    // 这个函数仅用于启动设备，并不进行任何等待，所以早点运行更好
    void start_work() {
        std::thread([&]() {
            std::this_thread::sleep_for(std::chrono::seconds(1));
            value = 42;
            lock = false;
        }).detach();
    }
    // 这个 poll 进去就会 spin_lock 了
    // 如果提前进去就会浪费时间，所以尽量等一会儿以后再进去
    int poll() {
        if (lock) std::cout << "waiting for device to complete...\n";
        while (lock) {
            std::this_thread::sleep_for(std::chrono::milliseconds(100));
        }
        return value;
    }
};


// 实现一个具体的异步协程函数
Async<int> async_operation() {
    std::cout << "starting async I/O operation...\n";

    Device device;
    device.start_work();

    co_await Awaitable();
    // 上面运行完就开始中断了，需要通过外界给个 resume() 恢复下面的内容

    auto value = device.poll();

    std::cout << "async I/O operation completed, value: " << value << '\n';

    co_return value;
}


int main(int argc, char const* argv[]) {
    auto start = std::chrono::system_clock::now();

    // 调用异步函数，必须接收返回值，否则返回值直接 destory 了
    auto io = async_operation();

    // 异步函数还没返回呢，所以这里是空的
    std::cout << "has result: " << io.result().has_value() << "\n";

    // 随便添加一个参数，就会提前 resume，模拟没有协程的情况
    if (argc > 1) io.resume();

    // 假装在做一些耗时的工作
    std::cout << "doing other works for 1.5 seconds\n";
    std::this_thread::sleep_for(std::chrono::milliseconds(1500));

    // 正确的使用方式：协程在其他函数工作一段时间后恢复，以最优化性能
    io.resume();

    std::cout << "has result: " << io.result().has_value() << "\n";
    std::cout << "result: " << io.result().value() << "\n";

    std::cout << "done\n";

    auto end  = std::chrono::system_clock::now();
    auto duration = duration_cast<std::chrono::microseconds>(end - start);
    std::cout << "time use: " << double(duration.count()) * std::chrono::microseconds::period::num / std::chrono::microseconds::period::den << "s" << std::endl;
    return 0;
}
```