为了解决`std::string`初始化（或复制）成本高昂的问题:
- `std::string_view` 提供对现有字符串（C 风格字符串、`std::string` 或另一个 `std::string_view`）的只读访问，而无需进行复制。
- **只读**意味着我们可以访问和使用被查看的值，但不能修改它。
- 将新字符串赋值给 `std::string_view` 会使其转而查看新字符串，而不会以任何方式修改先前查看的字符串

当你需要只读字符串时，特别是作为函数参数时，应优先选择 `std::string_view` 而不是 `std::string` 。


C++不允许将 `std::string_view` 隐式转换为 `std::string`
```cpp
#include <iostream>
#include <string>
#include <string_view>

void printString(std::string str)
{
	std::cout << str << '\n';
}

int main()
{
	std::string_view sv{ "Hello, world!" };

	// printString(sv);   // compile error: won't implicitly convert std::string_view to a std::string

	std::string s{ sv }; // okay: we can create std::string using std::string_view initializer
	printString(s);      // and call the function with the std::string

	printString(static_cast<std::string>(sv)); // okay: we can explicitly cast a std::string_view to a std::string

	return 0;
}
```

默认情况下，双引号字符串字面量是 C 风格字符串字面量。
- 我们可以通过在双引号字符串字面量后添加 `sv` 后缀来创建类型为 `std::string_view` 的字符串字面量。 
- 在双引号字符串字面量后添加 `s` 后缀来创建类型为 `std::string` 的字符串字面量

```cpp
#include <iostream>
#include <string>      // for std::string
#include <string_view> // for std::string_view

int main()
{
    using namespace std::string_literals;      // access the s suffix
    using namespace std::string_view_literals; // access the sv suffix

    std::cout << "foo\n";   // no suffix is a C-style string literal
    std::cout << "goo\n"s;  // s suffix is a std::string literal
    std::cout << "moo\n"sv; // sv suffix is a std::string_view literal

    return 0;
}
```


与 `std::string` 不同， `std::string_view` 完全支持 constexpr：涉及**constexpr变量**
- `constexpr std::string_view` 成为需要字符串符号常量时的首选。

在以下情况下使用 `std::string` 变量：
- 需要一个可以修改的字符串。
- 需要存储用户输入的文本。
- 需要存储一个返回 `std::string` 的函数的返回值。

在以下情况下使用 `std::string_view` 变量：
- 需要对已存在于其他位置的字符串的部分或全部内容进行只读访问，且该字符串在 `std::string_view` 使用完成前不会被修改或销毁。
- 需要一个用于 C 风格字符串的符号常量。
- 需要持续查看返回 C 风格字符串或非悬空 `std::string_view` 的函数的返回值。

在以下情况下使用 `std::string` 函数参数：
- 需要修改传入的字符串参数，而不影响调用者。
- 正在使用 C++14 或更早的语言标准，并且还不习惯使用引用

在以下情况下使用 `std::string_view` 函数参数：
- 函数需要处理非空终止字符串。
- 函数需要一个只读字符串

在以下情况下使用 `const std::string&` 函数参数：涉及**左值引用**
- 正在使用 C++14 或更旧的语言标准，并且函数需要一个只读字符串来工作（因为 `std::string_view` 直到 C++17 才可用）
- 正在调用需要 `const std::string` 、 `const std::string&` 或 const C 风格字符串的其他函数（因为 `std::string_view` 可能不是以空字符结尾的）。

以下情况下使用 `std::string&` 函数参数：
- 正在使用一个 `std::string` 作为输出参数
- 正在调用需要 `std::string&` 或非常量 C 风格字符串的其他函数

以下情况下使用 `std::string` 返回类型：
- 返回值是 `std::string` 局部变量或函数参数。
- 返回值是一个函数调用或运算符，它通过值返回一个 `std::string`

在以下情况下使用 `std::string_view` 返回类型：
- 函数返回一个 C 风格字符串字面量或已用 C 风格字符串字面量初始化的局部 `std::string_view`
- 函数返回一个 `std::string_view` 参数

涉及：通过引用返回和通过地址返回
在以下情况下使用 `std::string_view` 返回类型：
- 为 `std::string_view` 成员编写访问器时。
在以下情况下使用 `std::string&` 返回类型：
- 该函数返回一个 `std::string&` 参数。
在以下情况下使用 `const std::string&` 返回类型：
- 函数返回一个 `const std::string&` 参数。
- 为 `std::string` 或 `const std::string` 成员编写访问器
- 函数返回一个静态的（局部或全局） `const std::string` 。

