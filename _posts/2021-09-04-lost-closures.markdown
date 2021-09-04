---
layout:     post
title:      "Framegraph and the lost closures along the way"
date:       2021-09-04 10:40:00
author:     "YunHsiao"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - C++
---

## Problem Statement

最近在调试 framegraph 时发现 execute 函数按值捕获的闭包完全捕捉不到东西，这种蹊跷的问题一般都意味着我们的基础设施里有问题，一番调试后解决了这个隐藏很深的 bug，过程涉及不少 C++ 语言特性和功能模块的设计思考，很有意思。

抛去所有无关细节，整个问题可以准确地简化为如下代码：

```cpp
#include <iostream>
#include <memory>

class Executable {
public:
    virtual void execute() = 0;
};

template <typename ExecuteMethod>
class CallbackPass final : public Executable {
public:
    explicit CallbackPass(ExecuteMethod &&execute): mExecute(execute) {}
    void execute() override { mExecute(); }
private:
    ExecuteMethod mExecute;
};

class FrameGraph final {
public:
    template <typename ExecuteMethod>
    void addPass(ExecuteMethod &&execute) {
        mPass = std::make_unique<CallbackPass<ExecuteMethod>>(std::forward<ExecuteMethod>(execute));
    }
    void execute() { mPass->execute(); }
private:
    std::unique_ptr<Executable> mPass;
};

//////////////////////////////////////////////////

void setup(FrameGraph &fg) {
    std::string str{"IMPORTANT INFORMATION"};
    auto exec = [=]() {
        std::cout << str << std::endl;
    };
    fg.addPass(exec);
}

int main() {
    FrameGraph fg;
    setup(fg);
    fg.execute();
    return 0;
}
```

这段代码可以直接编译并复现问题，在这个小巧的测试例中，如果一切顺利的话我们应该看到 "IMPORTANT INFORMATION" 被打印输出，但实际在 playground 环境中运行却没有任何输出。

进一步的调试可以发现 exec 的闭包中捕获的值已完全错乱，无论捕获什么变量在执行时都无法正常访问。

继续阅读前可以先到比如 [这个 playground](https://code.sololearn.com/c7kV0PX783f8) 玩一玩，猜猜问题是什么~

## Situation Analysis

测试例展示了一轮典型的 framegraph 用法 —— 一帧内在管线所有阶段将要做的事的关键信息全注册到 fg，并以按值捕获的 lambda 形式提供最终执行时的 execute 函数。全部注册完成后由 fg 根据完整的全帧信息统一做全局优化，最后在合适的阶段执行 execute。

由于 execute 函数延迟执行的特点，所有其闭包内的局部对象必须按值捕获，lambda 本身在执行前也会被暂存在 fg 内。可以看到 `addPass` 函数通过 C++ 11 的标准完美转发，将 lambda 传给 `CallBackPass` 的显式单参构造函数，并通过 lambda 的移动构造函数存下来。看起来很不错啊。

那问题出在哪儿了呢。

看起来问题出在 setup 函数的写法上。由于 exec lambda 是先被声明为局部变量再传给 addPass，导致实际的调用栈是这样的：

```cpp
FrameGraph::addPass<lambda&>(lambda& &&execute) ->
CallbackPass<lambda&>::CallbackPass(lambda& &&execute) ->
lambda& mExecute(execute)
```

显然最后保存在 `CallbackPass` 中的是 lambda 的左值引用，在 setup 函数作用域结束后就变成了野指针，所以我们执行 execute 时所有闭包里的东西都失效了。

## Possible Solutions

现在我们知道了问题是什么，那么应该怎么改呢？

首先想到的最直接的方法是直接把 exec lambda 放在 `addPass` 函数参数里，或在调用时加上 std::move：

```cpp
// do this
fg.addPass([=]() { std::cout << str << std::endl; });
// or like this
auto exec = [=]() { std::cout << str << std::endl; };
fg.addPass(std::move(exec));
```

但这就对上层调用有了明确的要求，任何使用 addPass 的地方都需要自己确保传的是右值，否则就会访问到野指针。

这显然不够理想，所以我们可以尝试在编译期抛出这个错误：

```cpp
template <typename ExecuteMethod>
void addPass(ExecuteMethod &&execute) {
    static_assert(!std::is_reference_v<ExecuteMethod>, "The execute function should be passed as r-value");
    // ...
}
```

这样如果上层有不准确的用法会在编译期触发 assert，及时发现问题了。但再仔细想一下，我们例子里的用法真的有不可挽回的问题吗？似乎还有更好的办法来兼容这种用法，首先想到的是加一个针对左值的重载：

```cpp
template <typename ExecuteMethod>
void addPass(ExecuteMethod &execute) {
    addPass(std::move(execute));
}
```

这样是可以解决问题，但假设了外部传进来的 execute 函数都是可以被移动的。如果多个 pass 共用了同一个 exec lambda 呢？
所以看起来还是没有找准地方。

仔细想一下，最后的解决方案其实已经尽在咫尺了：

```cpp
template <typename ExecuteMethod_>
class CallbackPass final : public Executable {
public:
    using ExecuteMethod = std::remove_reference_t<ExecuteMethod_>;
    explicit CallbackPass(ExecuteMethod &execute): mExecute(execute) {}
    // ...
};
```

对于左值调用拷贝构造，对于右值调用移动构造，这样我们就能稳定健壮地处理所有用法了。
