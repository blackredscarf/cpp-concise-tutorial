# traits 类型萃取
在讲“类型萃取”之前，我们先来看看 typedef 。

## typedef
我们都知道使用 typedef 可以为非常多东西创建别名，包括类型，指针，函数等等。而且结合类和泛型非常好用。
```cpp
template <class T = int>
struct MyDef {
    typedef T myInt;
    typedef vector<T> myVec;
};

int main() {
    MyDef<>::myInt a = 3;
    MyDef<double>::myVec b = vector<double>();
}
```
在STL中，我们常常会把迭代器输入到算法函数中进行计算。下面写一个简单的例子，说明一下流程。
```cpp
template <class T>
struct Iterator {
    typedef T value_type;
    T* data_;
    Iterator(T* data): data_(data) {};
    T& operator*() const { return  *data_; };
    // ...
};

template <class T>
typename T::value_type // 函数返回值
todo(T iter) {
    return *iter;
}

int main() {
    Iterator<int> it(new int(4));
    cout << todo(it) << endl;   // 4
    return 0;
}
```
 todo() 函数的返回值加了一个 typename ，这是为了告诉编译器，我返回的是一个类型，而不是指针或者函数等。

但这么干会有一个问题，就是你无法传入一个指针类型。但STL的算法函数是兼容指针的。
```cpp
int* a = new int[5]{1, 2, 3, 4, 5};
reverse(a, a + 5);
```
那要怎么实现呢？答案就是类型萃取。

## 类型萃取
traits 表示“特性”的意思，而类型萃取就是为了萃取这些“特性”。萃取方法是在返回值中再加一个转换层。
```cpp
template <class T>
struct Iterator_traits {
    typedef typename T::value_type value_type;
};
```
返回值改为返回 Iterator_traits<T>::value_type 。
```cpp
template <class T>
typename Iterator_traits<T>::value_type // 函数返回值
todo(T iter) {
    return *iter;
}

int main() {
    Iterator<int> it(new int(4));
    cout << todo(it) << endl;   // 4
```
事实上，上面这里的结果都是返回`T::value_type`，并没有支持指针。而关键点在于，这个转换层是可以有多个版本的，即模板的偏特化 (partial specialization)。

当我加入下面两个特化以后，函数就能传入指针了。
```cpp
template <class T>
struct Iterator_traits<T*> {
    typedef T value_type;
};

template <class T>
struct Iterator_traits<const T*> {
    typedef T value_type;
};

int main() {
    cout << todo(new int[3]{1, 2, 3}) << endl; // 1
}
```
有了这个特点，你想获得任何一种返回值都可以，只要你去重载这个类型萃取器`Iterator_traits`，并将你想要的类型绑定到`value_type`中。

## STL 中的 iterator_traits
事实上，在STL中类型萃取不只是用在定义返回类型。为了让算法函数能够兼容任意类型的指针，源码的 iterator_traits 中定义了相关别名。同时，为了兼容你自己定义的迭代器，只要在你的迭代器中 typedef 了规定的别名即可，这些别名必须与STL的 iterator_traits 中的别名相同。

下面是`iterator_traits`的定义，定义了5个别名，当算法函数需要知道一些关于你的迭代器的某些信息时，就会调用 iterator_traits 来获取相关信息。
```cpp
template<typename _Iterator>
struct iterator_traits<_Iterator> {
    typedef typename _Iterator::iterator_category iterator_category;
    typedef typename _Iterator::value_type        value_type;
    typedef typename _Iterator::difference_type   difference_type;
    typedef typename _Iterator::pointer           pointer;
    typedef typename _Iterator::reference         reference;
};
```
并不是所有别名都会被用到，主要看算法需要哪些别名。如果你全都定义了，那你的迭代器将兼容所有STL算法函数。

- value_type 即你的迭代器的内嵌类型。
- pointer 即内嵌类型的指针。
- reference 即内嵌类型的引用。
- difference_type 虽然名字起得奇怪，但它的功能是用来统计迭代器之间的距离的，比如指针的 difference_type 采用的类型是C++自带的`ptrdiff_t`，用于表示两个指针相减的值。而比如 count 函数的返回值也是 difference_type，因为 ptrdiff_t 也可以看做是 long 类型。
```cpp
int* a = new int[5]{1, 2, 3, 1, 5};
ptrdiff_t n = count(a, a + 5, 1);
cout << n << endl; // 2
```
- iterator_category 即迭代器的类型。C++会根据不同迭代器类型重载不同的计算函数。分别可以定义为以下几种：
```cpp
struct input_iterator_tag { };
struct output_iterator_tag { };
struct forward_iterator_tag : public input_iterator_tag { };
struct bidirectional_iterator_tag : public forward_iterator_tag { };
struct random_access_iterator_tag : public bidirectional_iterator_tag { };
```

## traits 的意义和作用
traits 这东西其实包含了接口的特性而又高于接口。使用 trait 的一个目的为了各种实现提供一个统一的接口和属性，这和声明实现某个接口，然后实现这个接口的方法和属性是一个道理。而高于接口的原因是，它在接口的外面又套了一层类，通过特化这个类能达到拓展接口的目的，一个明显的例子是，如果没有traits，STL的算法函数是不可能兼容指针的。
