# C++ 移动语义
在讲移动语义之前先搞清楚拷贝。

## 浅拷贝
当将你将一个对象浅拷贝给对另一个对象，仅仅拷贝成员属性的指针，两个对象的成员指针变量将指向同一块内存空间。
```cpp
class IntArray {
public:
    int* arr;
    int len;
    IntArray(initializer_list<int> il){
        arr = new int[il.size()];
        len = 0;
        for (int i : il) {
            arr[len++] = i;
        }
    }
    // 浅拷贝
    IntArray(const IntArray &ina) {
        len = ina.len;
        arr = ina.arr;
    }
    void show(){
        for(int i = 0; i < len; i++){
            cout << arr[i] << " ";
        }
        cout << endl;
    }
};

int main() {
    IntArray a1{1, 2, 4, 5};
    IntArray a2(a1);
    a2.arr[0] = 99;
    a2.show();  // 99 2 4 5
    a1.show();  // 99 2 4 5
}
```
（如果不写，默认是浅拷贝）

## 深拷贝
当将你将一个对象深拷贝给对另一个对象，将完整拷贝对象成员的值。对象成员指针变量将指向的是不同的内存空间。
```cpp
class IntArray {
public:
    int* arr;
    int len;
    IntArray(initializer_list<int> il){
        arr = new int[il.size()];
        len = 0;
        for (int i : il) {
            arr[len++] = i;
        }
    }
    // 深拷贝
    IntArray(const IntArray &ina) {
        len = ina.len;
        arr = new int[ina.len];
        for(int i = 0; i < ina.len; i++){
            arr[i] = ina.arr[i];
        }
    }
    void show(){
        for(int i = 0; i < len; i++){
            cout << arr[i] << " ";
        }
        cout << endl;
    }
};

int main() {
    IntArray a1{1, 2, 4, 5};
    IntArray a2(a1);
    a2.arr[0] = 99;
    a2.show();  // 99 2 4 5
    a1.show();  // 1 2 4 5
}
```


## 右值引用
C++中有左值和右值两个概念。左值就是在等号左边定义的变量，而右值就是在等号右边的产生临时变量。
<span id="right"></span>
```cpp
IntArray getArray(){
    IntArray temp = IntArray{99, 188};
    return temp;
}

int g1 = 10;  // g1是左值，10是右值
IntArray a2 = getArray();  // a2是左值，getArray()返回的临时变量是右值
```

左值引用。就是对一个左值起了另一个别名。
```cpp
int g1 = 10; // g1是左值
int &g2 = g1; // g2引用了左值g1
```

<br>

那么C++11新增的右值引用呢？右值引用的出现就是为了解决临时变量带来的开销问题。

为了更好的说明，我们在`IntArray`类里面新增以下代码：
```cpp
public:
    IntArray operator+(IntArray &ina){
        IntArray tmp = IntArray(len + ina.len);
        int top = 0;
        for(int i = 0; i < len; i++){
            tmp.arr[top++] = arr[i];
        }
        for(int i = 0; i < ina.len; i++){
            tmp.arr[top++] = ina.arr[i];
        }
        return tmp;
    }

private:
    IntArray(int n) {
        len = n;
        arr = new int[len];
    }
```
一个函数：
```cpp
void display(IntArray &ina){
    ina.show();
}
```
我们需要显示a1和a2相加后的情况，则需要这么写：
```cpp
IntArray a1{1, 2, 3};
IntArray a2{4, 5, 6};
IntArray a3 = a1 + a2;
display(a3);  // 1 2 3 4 5 6
```
而不能直接这么写：
```cpp
display(a1 + a2);
```
a1 + a2返回一个临时对象，而左值引用是不支持引用临时对象的。

而当我给函数参数在多加一个引号`IntArray && ina`，就变为了右值引用，使其可以引用右值（临时变量）。
```cpp
void display(IntArray && ina){
    ina.show();
}

int main() {
    IntArray a1{1, 2, 3};
    IntArray a2{4, 5, 6};
    display(a1 + a2);  // 1 2 3 4 5 6
}
```
这时`a1 + a2`产生的临时变量将被引用，而不会被立刻删除。

问题又来了，`display`函数的参数声明为`&&`右值引用后，你就没办法传一个左值进去了，如`display(a1)`。解决办法是将传入的变量转换为一个右值引用类型。
```cpp
display(static_cast<IntArray &&>(a1));
```
为了方便，c++提供了一个简单方便的模板函数`move`：
```cpp
display(move(a1));
```

## 移动
移动的意思是“移交主权”。移动的出现是为了提供一种新的传值方式。以前的传值方式有三种，一种是拷贝传值，一种是左值引用，一种是右值引用。

我们想往一个左值引用的参数中传递一个表达式或临时变量是不允许的。而表达式或临时变量都是临时的，我们不想特地用一个左值去接收，因为会造成不必要的开销，这时我们可以用右值引用。

而移动是比右值引用更激进的方式，右值引用传递后还可以正常访问原来的变量，而移动则是直接把变量的主权移交出来，被移动的变量将不能再访问数据。当然，移交主权这种事是靠用户自己决定的，后面的例子将说明这一点。

我们在类中新增一个拷贝构造函数：
```cpp
IntArray(IntArray &&ina) {
    len = ina.len;
    arr = ina.arr;
    ina.arr = nullptr;  // here
    ina.len = 0;    // here
}
```
使用：
```cpp
IntArray a1{1, 2, 3};
IntArray a3(move(a1));
a1.show();  // nothing
a3.show();  // 1 2 3
```
上面的例子中，我们把传进去的`a1`通过move变为右值引用，然后在拷贝构造函数中把`a1`的指针设为空。当尝试在main函数中访问`a1`时，访问得到的是空，因为原本`a1`对数据的访问权转移到了`a3`上了。

为什么要将指针设为空？如果不设为空，当调用两个对象的析构函数式就会发生对一个地址delete两次的未定义行为，而对一个空指针delete是无害的。

注意，`move`这个函数的实现仅是将左值变量转换为右值，本身不存在移走变量的内存使用权这个功能，是否移走，怎么移走要靠你自己去实现。

## 移动成员函数
C++11为了方便程序员使用移动语义，在类中添加了特殊的成员函数。

默认的移动构造函数：
```cpp
IntArray(IntArray &&) = default;
```
默认的移动赋值函数：
```cpp
IntArray &operator=(IntArray &&) = default;
```
当你传入一个右值参数时，就会自动调用移动成员函数。你可以根据需要可以重载它们。

你可能会很好奇，编译器怎么知道我的对象中哪些参数要移动，哪些参数不需要移动。

事实上，默认的移动构造函数和移动赋值运算符的工作方式与复制版本类似：
1. 执行成员初始化并复制内置类型。
2. 如果成员是类对象，将使用对应类的移动构造函数和移动赋值运算符。

为了证明这一点，我写个例子：
```cpp
class A {
    string ad;
public:
    A(string s): ad(s) {}
    A(A &a): ad(a.ad) {}
    A(A &&a): ad(move(a.ad)) {
        cout << "A move ctor" << endl;
    }
    const string &getAd() const { return ad; }
};

class B {
    A a_;
public:
    B(A a): a_(a) {};
};

int main() {
    A a("hello");
    B b(move(a));   // A move ctor
    cout << a.getAd() << endl;  // nothing
}
```
你会发现，当B的默认移动构造函数被调用后，A的重载后的移动构造函数也被调用了。而`a.getAd()`得到空值，是因为string内部的移动构造函数将`a.ad`设为了空。

可以认为，B中生成的移动构造函数是这样的：
```cpp
B(A &&a): a_(move(a)) {

}
```

### 拷贝和移动
还有一个问题是，需要注意编译器在什么情况下会生成这些成员函数。
1. 如果您提供了**析构函数**、**复制构造函数**或**复制赋值运算符**，编译器将不会自动提供**移动构造函数**和**移动赋值运算符**；
2. 如果您提供了**移动构造函数**或**移动赋值运算符**，编译器将不会自动提供**复制构造函数**和**复制赋值运算符**。

移动和拷贝之间，当你手动提供一者，那么另一者将不会被自动提供。

首先一个右值引用可以直接赋给左值引用，那我传一个右值的时候，到底是调用拷贝还是移动呢？这将产生冲突。
```cpp
int &&g = 1;
int &gl = g; // right can pass to left
```
那么为什么手动加入移动函数后，拷贝函数就自动被删掉了呢？看知乎上这个[回答](
https://www.zhihu.com/question/28078756/answer/39301949)。大概意思是标准委员会不太喜欢拷贝构造函数这种东西，所以，你要是提供了移动构造，就可能不需要这东西了。你可以这认为这是标准委员会的任性导致的。如果你确实想使用默认拷贝构造，可以手动声明，如`A(A &a) = defualt`。

那么析构函数呢？事实上，析构函数和移动函数是存在天然的对立的。移动会默认把基本类型进行复制，即指针变量也仅是复制一个指针而已，若失去权力的A被析构了，那么A的指针指向的空间将被回收，而获得权力的B的指针就会访问到一个被回收的空间。


## 例子：交换对象
比如说在交换对象的时候，如果不用移动语义(Move Semantics)，在赋值过程中会发生拷贝。
```cpp
template<class T>
void swap(T& a, T& b) { 
  T tmp { a }; // invokes copy constructor
  a = b; // invokes copy assignment
  b = tmp; // invokes copy assignment
}
```
若使用移动语义，则不需要发生拷贝，而是直接转移对内存空间的管理权。
```cpp
template<class T>
void swap(T& a, T& b) { 
  T tmp { std::move(a) }; // invokes move constructor
  a = std::move(b); // invokes move assignment
  b = std::move(tmp); // invokes move assignment
}
```

## 例子：STL
STL的容器大部分函数都重载了一个可移动版本。
```cpp
vector<string> v;
string data = "hello";
v.push_back(move(data));
cout << v[0] << endl;   // hello
cout << data << endl;   // nothing
```

### 临时变量的常量性
临时变量是不可更改的，或者说右值是不可更改的。所以你可以用一个常量引用接收一个右值，因为你决定不更改右值时，就没必要从右值复制到左值了，而是直接用常量去引用这块临时变量的空间，这样能提高性能。当然，前提是你的常量引用必须总是和临时变量在同一作用域下。
```cpp
Data getDataTemp() {
    return {};
}

void test() {
    const auto& d = getDataTemp();
    auto&& d2 = getDataTemp();
    auto d3 = getDataTemp();  // copy temporary variable
    auto& d4 = getDataTemp();  // error
}
```

## forward
完美转发(perfect forward)。在泛型的情况下，右值引用可以接受右值，左值以及左值引用。为了区分开和转换他们，可以使用`forward`函数。而forward内部将调用`static_cast`进行转换。
```cpp
template <typename T>
void find_ref(T && v){
    cout << "right" << endl;
}

template <typename T>
void find_ref(T & v){
    cout << "left" << endl;
}

template <typename T>
void test(T && var1){   // rvalue reference ?
    find_ref(forward<T>(var1));
}

int main() {
    int v = 1;
    test(v);    // left
    test(3);    // right
}
```
