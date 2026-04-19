
当实例化类模板对象时，编译器可以根据对象初始化器的类型推导出模板类型（这称为类模板参数推导，简称 CTAD）
```cpp
#include <utility> // for std::pair

int main()
{
    std::pair p1 { 3.4f, 5.6f }; // deduced to pair<float, float>
    std::pair p2 { 1u, 2u };     // deduced to pair<unsigned int, unsigned int>

    return 0;
}
```

在大多数情况下，CTAD 可以直接使用。然而，在某些情况下，编译器可能需要一些额外的帮助来正确推导模板参数。
向编译器提供一个**推导指南**（deduction guide），用于告诉编译器如何为给定的类模板推导模板参数。
```cpp
template <typename T, typename U>
struct Pair
{
    T first{};
    U second{};
};

// Here's a deduction guide for our Pair (needed in C++17 only)
// Pair objects initialized with arguments of type T and U should deduce to Pair<T, U>
template <typename T, typename U>
Pair(T, U) -> Pair<T, U>;

int main()
{
    Pair<int, int> p1{ 1, 2 }; // explicitly specify class template Pair<int, int> (C++11 onward)
    Pair p2{ 1, 2 };           // CTAD used to deduce Pair<int, int> from the initializers (C++17)

    return 0;
}
```
我们是在告诉编译器：如果它看到一个带有两个参数（类型分别为 `T` 和 `U` ）的 `Pair` 声明，则应将其推导为 `Pair<T, U>` 类型。

```cpp
template <typename T>
struct Pair
{
    T first{};
    T second{};
};

// Here's a deduction guide for our Pair (needed in C++17 only)
// Pair objects initialized with arguments of type T and T should deduce to Pair<T>
template <typename T>
Pair(T, T) -> Pair<T>;

int main()
{
    Pair<int> p1{ 1, 2 }; // explicitly specify class template Pair<int> (C++11 onward)
    Pair p2{ 1, 2 };      // CTAD used to deduce Pair<int> from the initializers (C++17)

    return 0;
}
```
我们的推导指南将一个 `Pair(T, T)` （一个具有两个 `T` 类型参数的 `Pair` ）映射到一个 `Pair<T>`

> C++20 增加了编译器为聚合类型自动生成推导指南的能力，因此只需为了兼容 C++17 才需要提供推导指南。

> `std::pair` （以及其他标准库模板类型）自带预定义的推导指南，这就是为什么我们上面使用 `std::pair` 的示例在 C++17 中无需手动提供推导指南也能正常编译的原因。



模板参数也可以被赋予默认值。当未显式指定模板参数且无法推导时，将使用这些默认值。
```cpp
template <typename T=int, typename U=int> // default T and U to type int
struct Pair
{
    T first{};
    U second{};
};

template <typename T, typename U>
Pair(T, U) -> Pair<T, U>;

int main()
{
    Pair<int, int> p1{ 1, 2 }; // explicitly specify class template Pair<int, int> (C++11 onward)
    Pair p2{ 1, 2 };           // CTAD used to deduce Pair<int, int> from the initializers (C++17)

    Pair p3;                   // uses default Pair<int, int>

    return 0;
}
```

CTAD 不适用于非静态成员初始化
- 当使用非静态成员初始化来初始化类类型的成员时，CTAD 在此上下文中无法工作。必须显式指定所有模板参数
```cpp
#include <utility> // for std::pair

struct Foo
{
    std::pair<int, int> p1{ 1, 2 }; // ok, template arguments explicitly specified
    std::pair p2{ 1, 2 };           // compile error, CTAD can't be used in this context
};

int main()
{
    std::pair p3{ 1, 2 };           // ok, CTAD can be used here
    return 0;
}
```

CTAD 代表类模板参数推导（class template argument deduction），而非类模板形参推导，因此它仅推导模板**实参**的类型，而不推导模板形参。
- 因此，类模板参数推导（CTAD）不能用于函数参数。
```cpp
#include <iostream>
#include <utility>

void print(std::pair p) // compile error, CTAD can't be used here
{
    std::cout << p.first << ' ' << p.second << '\n';
}

int main()
{
    std::pair p { 1, 2 }; // p deduced to std::pair<int, int>
    print(p);

    return 0;
}
```