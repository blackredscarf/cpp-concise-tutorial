# 现代 C++ 特性
介绍一些比较零散的 C++ 新特性。

## 初始化列表
```cpp
double sum(initializer_list<double> il){
    double tot = 0;
    for(auto p = il.begin(); p != il.end(); p++){
        tot += *p;
    }
    return tot;
}

int main(){
    double res = sum({1.5, 3.5, 1.2});
    cout << res << endl;    // 6.2
}
```
类也可以：
```cpp
class Node{
public:
    Node(initializer_list<double> il){

    }
};

int main(){
    Node no{1.3, 2.3, 7.8};
}
```
这种传参的好处是你可以传入若干个参数。在设计API时会非常好用。

## 禁用方法
C++11意识到用户可能不想使用类中某些默认的成员函数，这时候你可以禁用掉它们。
```cpp
IntArray(IntArray &&) = delete;
```
你还可以禁用自己定义的函数。
```cpp
input(int a) { ... };
input(double a) = delete;
```

## 函数参数绑定
使用`std::bind`可以绑定某一个函数，预先填充一些参数进去来简化参数传递。
```cpp
double power(double a, double b){
    return pow(a, b);
}

auto power2 = bind(power, placeholders::_1, 2.0);

int main(){
    cout << power2(3) << endl;  // 9
}
```
`std::placeholders`表示参数传递占位符，`_1`是一个序号，表示绑定之后传入的第一个参数，同理还有`_2`, `_3`等等。

## 结构绑定
C++17强化了对元组和pair的支持，可以在函数内以`{a, b}`这样的形式返回一个元组或pair。

同时新增了一个名为Structured Bindings的特性，以`[a, b]`这种方式接收一个元组并按顺序绑定到变量中。
```cpp
#include <tuple>
using namespace std;

tuple<int, bool> foo() {
    return {10, false};
}

int main() {
    auto [a, b] = foo();
    cout << a  << " " << b << endl;
    return 0;
}
```
结构绑定对结构体也有同样作用，
```cpp
struct Foo {
    int age;
    string name;
};

int main() {
    auto f = Foo {23, "John"};
    auto& [age, name] = f;
    cout << age  << " " << name << endl;
}
```
map的迭代器获取值返回的是pair，所以也可以用该方式进行绑定。
```cpp
map<string, int> get_map() {
    return {
        {"C++", 22},
        {"Java", 34},
        {"Python", 45}
    };
}

int main() {
    for(auto& [k, v] : get_map()) {
        cout << k << " " << v << endl;
    }
    return 0;
}
```