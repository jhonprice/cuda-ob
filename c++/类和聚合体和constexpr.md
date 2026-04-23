与非成员函数一样，成员函数也可以通过使用 `constexpr` 关键字声明为 constexpr。constexpr 成员函数既可以在编译时求值，也可以在运行时求值。


聚合类型隐式支持 constexpr
```cpp
#include <iostream>

struct Pair // Pair is an aggregate
{
    int m_x {};
    int m_y {};

    constexpr int greater() const
    {
        return (m_x > m_y  ? m_x : m_y);
    }
};

int main()
{
    constexpr Pair p { 5, 6 };        // now constexpr
    std::cout << p.greater() << '\n'; // p.greater() evaluates at runtime or compile-time

    constexpr int g { p.greater() };  // p.greater() must evaluate at compile-time
    std::cout << g << '\n';

    return 0;
}
```
---
如果您希望类能够在编译时求值，请将成员函数和构造函数声明为 constexpr。
- 隐式定义的构造函数在可以定义为 constexpr 的情况下会自动成为 constexpr。显式默认化的构造函数必须显式声明为 constexpr。
- constexpr 是类接口的一部分，后续移除它将导致在常量上下文中调用该函数的代码出错。

在 C++ 中，**字面类型**是指任何可能在常量表达式中创建对象的类型。换句话说，除非某个类型符合字面类型的资格，否则该类型的对象不能是 constexpr。

**字面类型**的定义较为复杂，其摘要可在 cppreference 上找到。不过值得注意的是，字面类型包括：
- Scalar types (those holding a single value, such as fundamental types and pointers)  
    标量类型（即保存单个值的类型，例如基本类型和指针）
- Reference types  引用类型
- Most aggregates  大多数聚合类型
- Classes that have a constexpr constructor  
    具有 constexpr 构造函数的类
```cpp
#include <iostream>

class Pair
{
private:
    int m_x {};
    int m_y {};

public:
    constexpr Pair(int x, int y): m_x { x }, m_y { y } {} // now constexpr

    constexpr int greater() const
    {
        return (m_x > m_y  ? m_x : m_y);
    }
};

int main()
{
    constexpr Pair p { 5, 6 };
    std::cout << p.greater() << '\n';

    constexpr int g { p.greater() };
    std::cout << g << '\n';

    return 0;
}
```


当 constexpr 函数在编译时上下文中求值时，只能调用其他 constexpr 函数。

在 C++11 中，非静态 constexpr 成员函数隐式为 const（构造函数除外）。
- 然而，从 C++14 开始，constexpr 成员函数不再隐式为 const。这意味着，如果您希望一个 constexpr 函数成为 const 函数，则必须显式地将其标记为 const。

constexpr 非 const 成员函数可以修改类的数据成员，只要隐式对象不是 const 即可。即使该函数在编译时求值，这一规则依然成立。
```cpp
#include <iostream>

class Pair
{
private:
    int m_x {};
    int m_y {};

public:
    constexpr Pair(int x, int y): m_x { x }, m_y { y } {}

    constexpr int greater() const // constexpr and const
    {
        return (m_x > m_y  ? m_x : m_y);
    }

    constexpr void reset() // constexpr but non-const
    {
        m_x = m_y = 0; // non-const member function can change members
    }

    constexpr const int& getX() const { return m_x; }
};

// This function is constexpr
constexpr Pair zero()
{
    Pair p { 1, 2 }; // p is non-const
    p.reset();       // okay to call non-const member function on non-const object
    return p;
}

int main()
{
    Pair p1 { 3, 4 };
    p1.reset();                     // okay to call non-const member function on non-const object
    std::cout << p1.getX() << '\n'; // prints 0

    Pair p2 { zero() };             // zero() will be evaluated at runtime
    p2.reset();                     // okay to call non-const member function on non-const object
    std::cout << p2.getX() << '\n'; // prints 0

    constexpr Pair p3 { zero() };   // zero() will be evaluated at compile-time
//    p3.reset();                   // Compile error: can't call non-const member function on const object
    std::cout << p3.getX() << '\n'; // prints 0

    return 0;
}
```