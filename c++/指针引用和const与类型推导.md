
默认情况下，类型推导会从类型中丢弃 `const`
- 可以通过在推导类型的定义中添加 `const` （或 `constexpr` ）限定符来重新应用 const（或 constexpr）
- 当我们说类型推导会丢弃 const 限定符时，它仅*丢弃顶层 const*。底层 const 不会被丢弃。


类型推导还会*丢弃引用*：
- 如果你希望推导出的类型是引用，可以在定义处重新添加引用符号

**顶层 const** 是直接应用于对象本身的 const 限定符
```cpp
const int x;    // this const applies to x, so it is top-level
int* const ptr; // this const applies to ptr, so it is top-level
// references don't have a top-level const syntax, as they are implicitly top-level const
```

**底层 const** 是应用于被引用或被指向对象的 const 限定符
```cpp
const int& ref; // this const applies to the object being referenced, so it is low-level
const int* ptr; // this const applies to the object being pointed to, so it is low-level
```
- *对 const 值的引用始终是底层 const。*
- 指针可以具有顶层 const、底层 const，或两者兼有



> *去掉引用可能会将底层 const 变为顶层 const*： `const std::string&` 是底层 const，但去掉引用后得到 `const std::string` ，这是一个顶层 const。


如果初始化器是对 const 的引用，则首先丢弃该引用（若适用则重新应用），然后从结果中丢弃任何顶层 const。

> 如果你想要一个 const 引用，即使并非严格必要，也应重新添加 `const` 限定符，因为这能明确你的意图并有助于防止错误。

```cpp
#include <string>

const std::string& getConstRef(); // some function that returns a const reference

int main()
{
    auto ref1{ getConstRef() };        // std::string (reference and top-level const dropped)
    const auto ref2{ getConstRef() };  // const std::string (reference dropped, const dropped, const reapplied)

    auto& ref3{ getConstRef() };       // const std::string& (reference dropped and reapplied, low-level const not dropped)
    const auto& ref4{ getConstRef() }; // const std::string& (reference dropped and reapplied, low-level const not dropped)

    return 0;
}
```

constexpr 不属于表达式的类型，因此不会由 `auto` 推导得出。
- 在定义 const 引用（例如 `const int&` ）时，const 修饰的是被引用的对象，而非引用本身。
- 在定义指向 const 变量的 constexpr 引用时（例如 `constexpr const int&` ），我们需要同时应用 `constexpr` （适用于引用）和 `const` （适用于被引用的类型）。

当我们在方程中引入 `const` 时， `auto` 和 `auto*` 的行为存在差异
- 如果你需要 const 指针、指向 const 的指针或指向 const 的 const 指针，即使并非严格必要，也应重新应用 `const` 限定符，因为这能明确你的意图并有助于防止错误。
- 在推导指针类型时，请考虑使用 `auto*` 。在这种情况下使用 `auto*` 可以更清楚地表明我们正在推导指针类型，借助编译器的帮助确保不会推导出非指针类型，并让你对 const 拥有更多控制权。
```cpp
#include <string>

int main()
{
    std::string s{};
    const std::string* const ptr { &s };

    auto ptr1{ ptr };  // const std::string*
    auto* ptr2{ ptr }; // const std::string*

    auto const ptr3{ ptr };  // const std::string* const
    const auto ptr4{ ptr };  // const std::string* const

    auto* const ptr5{ ptr }; // const std::string* const
    const auto* ptr6{ ptr }; // const std::string*

    const auto const ptr7{ ptr };  // error: const qualifer can not be applied twice
    const auto* const ptr8{ ptr }; // const std::string* const

    return 0;
}
```

[总结](https://www.learncpp.com/cpp-tutorial/type-deduction-with-pointers-references-and-const/)