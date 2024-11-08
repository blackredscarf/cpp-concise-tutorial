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

注意一个细节，我们把模板函数方法的声明和定义都写在了头文件里，是因为模板函数最终会根据你的使用情况生成多种类型的实现，如果你定义在了`.cpp`源文件中，那么别的文件在include头文件后就无法引用到正确类型的类或函数。 如果真的想要定义在源文件中，可以在源文件中进行显式具体化。

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
在使用某些特殊类型时，模板类中的一些行为可能和该类型不符。这时候，我们使用相同的类名，针对该类型，重新定义一份。还有另一种叫法称之为模版特化。
```cpp
template class MyArray<Node, 10>{
    // ...
};
```

### 2.4 部分具体化
所谓部分具体化就是假设当用户输入一个指定类型时就调用对应的重载类。还有另一种叫法称之为模版偏特化。
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
将一个模板类作为另一个模板类的模板参数。
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

## 5. 友元模板函数
友元函数属于外部函数，你需要额外声明一个模板给它。
```cpp
template<typename U>
friend ostream& operator<<(ostream& os, DynamicArray<U>& da);
```
```cpp
template<typename U>
ostream& operator<<(ostream& os, DynamicArray<U>& da) {
    // ...
    return os;
}
```

## 6. 模板基类成员的访问
<div id="template-protected"></div>
我们知道类的公有和保护成员能够被其派生类直接访问，而当该类的基类是一个模板类时，GCC编译器将不允许派生类直接访问，而VC编译器却可以。这是因为GCC编译器遵循一个标准，当父类是一个模板类，而派生类访问变量时没有指定其模板参数，则不会去查找父类的作用域。这个和编译器查找变量名的实现有关，比如GCC认为程序员自己声明清楚，可以节省编译时查找的时间，提高编译速度。

```cpp
template <typename T>
class A {
protected:
    int a_;
};

template <typename T>
class B : public A<T> {
protected:
    void test() {
        a_ = 1; // error
    }
};
```
解决办法有两个，一个是用 using 声明一下。
```cpp
template <typename T>
class B : public A<T> {
    using A<T>::a_;
protected:
    void test() {
        a_ = 1;
    }
};
```
另一个办法是使用this指针访问。
```cpp
template <typename T>
class B : public A<T> {
protected:
    void test() {
        this->a_ = 1;
    }
};
```


## 总结
C++的模板是C++最复杂的特性之一，虽然让不少人吐槽，但类似于STL等库中确实大量用到了这些特性，说明这些特性并非无缘无故加进去的。标准委员会那帮家伙从来就没奢望每个人都去精通它，它是给需要用到它的人用的。




