## lambda基本使用
基本使用：
~~~
[捕获列表](参数列表) -> 返回类型 {
    函数体
};
~~~

测试代码：
~~~
#include <iostream>
int main()
{
    auto print = [](int x) {
        std::cout << x << std::endl;
    };
    print(42);

}
~~~
## 在lambda内部引用使用当前类
在C++17以前，如果你想在 lambda 表达式中使用当前类的成员变量或成员函数，你通常会捕获 this 指针。
测试代码：
~~~
#include <iostream>
class MyClass {
public:
    int value = 10;
    void doSomething() {
        auto lambda = [this]() {
            std::cout << this->value << std::endl;
        };
        lambda();
    }
};
int main()
{
    MyClass mc;
    mc.doSomething();
    auto print = [](int x) {
        std::cout << x << std::endl;
    };
    print(42);

}
~~~
输出结果：
~~~
10
42
~~~
这种方式的问题是，它捕获的是 this 指针，而不是对象本身。这意味着如果外部对象的生命周期结束，而 lambda 表达式仍在使用，就可能访问到无效的内存。这种问题在多线程或异步编程中尤为常见，可能导致难以调试的错误。

为了解决这个问题，C++17 引入了通过 *this 捕获当前对象的副本的能力。这样，lambda 表达式就拥有了当前对象的一个完整副本，从而避免了潜在的悬挂指针问题。

C++17及以后，测试代码：
~~~
#include <iostream>
#include <thread>
#include <future>

class MyClass {
public:
    int value = 10;
    void doSomething() {
        auto lambda = [*this]() {
            std::this_thread::sleep_for(std::chrono::seconds(1)); // 模拟异步操作
            std::cout << value << std::endl;
        };

        // 启动异步任务
        std::future<void> fut = std::async(std::launch::async, lambda);
    }
};

int main() {
    MyClass obj;
    obj.doSomething();
    return 0; // 对象 obj 的生命周期结束，但 lambda 中的副本仍然有效
}
~~~
输出结果：
~~~
Program returned: 0
Program stdout
10
~~~

## lambda递归调用自身
在C++23之前，传统 lambda 无法直接递归调用自身，因为 lambda 的类型在定义时尚未完全推导。例如：
~~~
auto factorial = [](int n) -> int {
    return (n <= 1) ? 1 : n * factorial(n-1); // 错误：factorial 尚未完全定义
};
~~~
