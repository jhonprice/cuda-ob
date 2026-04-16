**限定作用域的枚举**（在 C++ 中常称为 enum class

```cpp
#include <iostream>
int main()
{
    enum class Color // "enum class" defines this as a scoped enumeration rather than an unscoped enumeration
    {
        red, // red is considered part of Color's scope region
        blue,
    };

    enum class Fruit
    {
        banana, // banana is considered part of Fruit's scope region
        apple,
    };

    Color color { Color::red }; // note: red is not directly accessible, we have to use Color::red
    Fruit fruit { Fruit::banana }; // note: banana is not directly accessible, we have to use Fruit::banana

    if (color == fruit) // compile error: the compiler doesn't know how to compare different types Color and Fruit
        std::cout << "color and fruit are equal\n";
    else
        std::cout << "color and fruit are not equal\n";

    return 0;
}
```

有作用域枚举（scoped enumerations）仅将枚举器置于枚举自身的作用域范围内。换句话说，有作用域枚举对其枚举器而言类似于一个命名空间。这种内置的命名空间机制有助于减少全局命名空间污染，并降低在全局作用域中使用有作用域枚举时可能出现的名称冲突风险
- 作用域枚举器不会隐式转换为整数
- 在某些情况下，将作用域枚举器视为整数值会很有用。在这些情况下，您可以使用 `static_cast` 显式地将作用域枚举器转换为整数。在 C++23 中，更好的选择是使用 `std::to_underlying()` （定义在 头文件中），它会将枚举器转换为该枚举的底层类型值。
- 也可以将整数 `static_cast` 为作用域枚举器