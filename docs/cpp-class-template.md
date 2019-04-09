# c++ 类模板

基本功能和[函数模板](cpp-function.md#function-template)差不多，但比函数模板更加强大。

## 1. 指定参数的模板
头文件如下:
```cpp
#ifndef HELLOWORLD_TCLASS_H
#define HELLOWORLD_TCLASS_H

template <typename T, int n>
class MyArray{
    T arr[n];
public:
    MyArray() = default;
    explicit MyArray(T* v);
    virtual T &operator[](int i);
};

template<typename T, int n>
MyArray<T, n>::MyArray(T* v) {
    for(int i = 0; i < n; i++){
        arr[i] = v[i];
    }
}

template<typename T, int n>
T &MyArray<T, n>::operator[](int i) {
    return arr[i];
}

#endif //HELLOWORLD_TCLASS_H
```
```cpp
#include "tclass.h"

int main(){
    MyArray<double, 10> ma = MyArray<double, 10>();
}
```
注意到，每一个实现函数的头顶都加入了`template`的定义。而且相比函数模板，你可以传入一个额外的指定类型的参数，比如说数组，就传入一个整型作为长度。

## 2. 一些概念
### 2.1 隐式实例化
指定所需类型。
```cpp
MyArray<double, 10> ma;
```
当你创建对象时，编译器会根据你传入的类型创建一个对应类型的MyArray类。

### 2.2 显式实例化
直接在模板中指出所需类型。
```cpp
template class MyArray<double, 10>;
```
补充以上定义后，编译器会生成一个类型为double的MyArray类。

### 2.3 显式具体化
在使用某些特殊类型时，模板类中的一些行为可能和该类型不符。这时候，我们使用相同的类名，针对该类型，重新定义一份。
```cpp
template class MyArray<Node, 10>{
    // ...
};
```

### 2.4 部分具体化
所谓部分具体化就是假设当用户输入一个指定类型时就调用对应的重载类。
```cpp
#include<iostream>
#include<typeinfo>
using namespace std;

template <typename T1, typename T2>
class PartialTest{
public:
    void print(){
        cout << "1: " << typeid(T1).name() << " 2: " << typeid(T2).name() << endl;
    }
};

template <typename T1>  // 只定义一个T1
class PartialTest<T1, int>{ // 默认一个int
public:
    void print(){
        cout << "1: " <<  typeid(T1).name() << " 2: defualt int" << endl;
    }
};
```
使用时：
```cpp
PartialTest<double, double> p1;
p1.print();    // 1: d 2: d

PartialTest<double, int> p2;
p2.print();    // 1: d 2: defualt int
```

## 3. 模板作为参数
```cpp
template <typename T>
class MiniBox {

};

template <template <typename T> class U, typename V>
class BigBox {
    U<V> m;
};
```
使用：
```cpp
BigBox<MiniBox, double> bm;
```


## 4. 模板别名
C++11中加入了模板别名，我们能够对模板的某些参数做一些预设值，来简化我们的使用。

比如，我们定义一个默认是12个元素的array的别名。直接使用别名定义即可。
```cpp
template <typename T>
using arr12 = std::array<T, 12>;
arr12<double> a;
arr12<int> a;
```
又比如，定义一个默认是double的array的别名。
```cpp
template <int n>
using arrDouble = std::array<double, n>;
arrDouble<10> a1;
arrDouble<10> a2;
```

## 总结
C++的模板是C++最复杂的特性之一，虽然让不少人吐槽，但类似于STL等库中确实大量用到了这些特性，说明这些特性并非无缘无故加进去的。标准委员会那帮家伙从来就没奢望每个人都去精通它，它是给需要用到它的人用的。




