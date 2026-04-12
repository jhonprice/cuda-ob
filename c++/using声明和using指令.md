名称可以是**限定名称**或**非限定名称**。
- **限定名称**是包含关联作用域的名称。
- 大多数情况下，名称通过作用域解析运算符（::）与命名空间进行限定
- **非限定名称**是不包含作用域限定符的名称

名称也可以通过类名使用作用域解析运算符（::）进行限定，或通过类对象使用成员选择运算符（. 或 ->）进行限定。例如：

```cpp
class C; // some class

C::s_member; // s_member is qualified by class C
obj.x; // x is qualified by class object obj
ptr->y; // y is qualified by pointer to class object ptr
```

using声明允许我们将非限定名称（不带作用域）作为限定名称的别名
```cpp
#include <iostream>

int main()
{
   using std::cout; // this using declaration tells the compiler that cout should resolve to std::cout
   cout << "Hello world!\n"; // so no std:: prefix is needed here!

   return 0;
} // the using declaration expires at the end of the current scope
```

另一种简化方法是使用 using 指令。using 指令允许在给定命名空间中的所有标识符，在 using 指令的作用域内无需限定即可引用。

using 声明或 using 指令在代码块内使用，那么这些名称仅适用于该代码块（遵循正常的块作用域规则）


> 优先使用显式命名空间限定符，而非使用声明。
> 完全避免使用指令（除了 `using namespace std::literals` 用于访问 `s` 和 `sv` 字面量后缀的情况）。在.cpp 文件中，包含头文件后可以使用声明。不要在头文件中使用声明（尤其是在头文件的全局命名空间中）。


`using` 关键字也用于定义类型别名，这与使用声明无关。: 涉及**类型别名**