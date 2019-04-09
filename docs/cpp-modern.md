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



