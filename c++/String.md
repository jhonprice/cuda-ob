要将整行输入读取到字符串中，最好使用 `std::getline()` 函数。 `std::getline()` 需要两个参数：第一个是 `std::cin` ，第二个是你的字符串变量。
- 若使用 `std::getline()` 读取字符串，请使用 `std::cin >> std::ws` 输入操纵符忽略前导空白符。这需要在每次 `std::getline()` 调用时执行，因为 `std::ws` 不会在调用间保留。
- `std::getline()` 不会忽略前导空白字符。如果你希望它忽略前导空白字符，请将 `std::cin >> std::ws` 作为第一个参数传入。它在遇到换行符时停止提取。

```cpp
#include <iostream>
#include <string> // For std::string and std::getline

int main()
{
    std::cout << "Enter your full name: ";
    std::string name{};
    std::getline(std::cin >> std::ws, name); // read a full line of text into name

    std::cout << "Enter your favorite color: ";
    std::string color{};
    std::getline(std::cin >> std::ws, color); // read a full line of text into color

    std::cout << "Your name is " << name << " and your favorite color is " << color << '\n';

    return 0;
}
```

`std::string::length()` 返回的是**无符号整数**值（很可能是 `size_t` 类型）。若要将长度赋值给 `int` 变量，应当进行 `static_cast` 转换，以避免编译器关于有符号/无符号转换的警告
- 在 C++20 中，也可以使用`std::ssize()`函数来获取`std::string`的长度，结果以大型有符号整数类型（通常是`std::ptrdiff_t`）表示


不要通过值传递 `std::string` ，因为这会生成一个开销巨大的副本。
- 初始化一个 `std::string` 的成本很高
- 在大多数情况下，应使用 `std::string_view`

当返回语句的表达式解析为以下任一情况时，可以按值返回 `std::string`
- 类型为 `std::string` 的局部变量
- 通过另一个函数调用或运算符按值返回的 `std::string
- 作为返回语句一部分创建的 `std::string` 临时对象

`std::string` 支持一种称为**移动语义**的功能，它允许在函数结束时将被销毁的对象通过值返回，而无需进行复制
如果返回 C 风格字符串字面量，请改用 `std::string_view` 返回类型