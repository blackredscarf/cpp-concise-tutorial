# 常量
## const
当你把一个变量加上const后，意味着不能改变这个值，只能访问。编译器层面的实现原理是在编译时发现这个变量是常量，就直接把这个变量直接以符号表中的值进行替换。
```cpp
const int a = 999;

int main() {
	cout << a << endl;
	cout << 999 << endl;  // 编译后
}
```

## constexpr
C++11提供了新的关键字constexpr，表示编译时求值，就是在编译的时候就把结果求出来，作用是允许把数据置于只读内存中以及提升性能。
```cpp
constexpr double a1 = 7;
constexpr double b1 = 8;
constexpr double n = a1 + b1;   // a1 + b1 称为 constant expressions 常量表达式
```
```cpp
constexpr double sum(double a, double b){ return a + b; };
constexpr double a1 = 7;
constexpr double b1 = 8;
constexpr double sm = sum(a1, b1);
```
如果你是调包使用函数，你可能不知道这个函数到是不是constant expressions，如果不是，你就不能constexpr变量去接收。这时你可以直接写const，不管函数到是不是constant expressions都可以编译通过。注意，const和constexpr必须声明其中一个，否则，变量将不在编译时求值。
```cpp
const double sm = sum(a1, b1);
```
在C++11中，constexpr function 只能有一条语句，C++14允许多条。

## constexpr 类
*see [Class](cpp-class.md)*

当一个类保证不需要被修改时，你可以用 constexpr 声明其构造函数和getter。
```cpp
class Rectangle { 
    int _h, _w; 
public: 
    // A constexpr constructor 
    constexpr Rectangle (int h, int w) : _h(h), _w(w) {} 
    constexpr int getArea () const { return _h * _w; } 
}; 
  
int main() { 
    const Rectangle obj(10, 20); 
    cout << obj.getArea(); 
    return 0; 
} 
```

这个[教程](https://www.geeksforgeeks.org/understanding-constexper-specifier-in-c/) 做了一个测试，使用了constexpr比不使用constexpr，提高了5倍性能，但也只是从0.003s到0.017s的提升，如果只执行一次，这点时间几乎可以忽略不计。但从合理利用语言特性的角度上看，你可以在任何函数或不被修改的对象上加上constexpr。

