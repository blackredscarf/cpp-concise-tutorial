# 智能指针

## 1. 为什么需要智能指针
在异常处理那一章中提到，当在函数中发送异常而中途跳出去时，有些内存可能无法被释放。
```cpp
void remodel(string& s){
    string* ps = new string(s);
    if(something()){
        throw exception();
    }
    s = *ps;
    delete ps;
}
```
当异常抛出时，本地变量都将从栈内存中删除——因此指针ps占据的内存将被释放，但ps所指向的内存不会被释放。

因此，ps的问题在于，它只是一个常规指针，不是有析构函数的类对象。如果它是对象，则可以在对象过期时，让它的析构函数删除指向的内存。

所以我们就需要用到智能指针，即`auto_ptr`，而C++11中又新增了`unique_ptr`和`shared_ptr`。

## 2. auto_ptr
用法：
```cpp
void remodel(string& s){
    auto_ptr<string> ps(new string(s));
    if(something()){
        throw exception();
    }
    s = *ps;
    // delete ps;
}
```
可以发现，实现方式是使用一个`auto_ptr`的对象对你的目标对象做了一个包装，在异常抛出后，发生栈解退，通过调用`auto_ptr`的析构函数进行内存回收。

这时你可能猜到，这肯定是通过模板来实现的，auto_ptr的源码开头大概如下：
```cpp
template<typename _Tp>
class auto_ptr {
private:
    _Tp* _M_ptr;
public:
    /// The pointed-to type.
    typedef _Tp element_type;
    explicit auto_ptr(element_type* __p = 0) throw() : _M_ptr(__p) { }
```

一个比较好的实践就是将智能指针用在代码块里边，这样你就不用去手动delete了。
```cpp
int main(){
    // ...
    {
        auto_ptr<string> ps(new string("hello"));
        cout << *ps << endl;
    }
    
    // ...
}
```

## 3. 智能指针的注意事项
先看一个例子：
```cpp
auto_ptr<string> ps(new string("hello"));
auto_ptr<string> voca;
voca = ps;
```
假设ps和voca两个指针将指向同一个string对象，如果不做任何处理，当离开作用域后，将导致程序对同一个对象删除两次，对同一个对象删除两次在C++中是未定义行为，它产生怎样的结果是不可预测的，所以我们应当尽可能避免未定义行为。

避免的方式有多种：
1. 定义赋值运算符，使之执行**深复制**。这样两个指针将指向不同的对象，其中的一个对象是另一个对象的副本。
2. 建立**所有权**（ownership）概念，**对于特定的对象，只能有一个智能指针可拥有它**，这样只有拥有对象的智能指针的构造函数会删除该对象。然后，让赋值操作转让所有权。这就是用于auto_ptr和unique_ptr的策略，但unique_ptr的策略更严格。
3. 创建智能更高的指针，跟踪引用特定对象的智能指针数。这称为**引用计数**（reference counting）。例如，赋值时，计数将加1，而指针过期时，计数将减1。仅当最后一个指针过期时，才调用delete。这是shared_ptr采用的策略。

尽管，auto_ptr使用所有权机制机制防止对同一个对象删除两次，但仍然存在问题。若当把对象的所有权从A移至B后，所有权已经不再A这里了，这时我再去访问A，程序就会崩溃。
```cpp
auto_ptr<string> ps(new string("hello"));
auto_ptr<string> voca;
voca = ps;
cout << *voca << endl;
cout << *ps << endl;    // error
```
auto_ptr 已经决定被移除，你可以使用后面会讲到的 unique_ptr 。

## 4. shared_ptr
```cpp
shared_ptr<string> ps(new string("hello"));
shared_ptr<string> voca;
voca = ps;
cout << *voca << endl;
cout << *ps << endl;
```
shared_ptr使用引用计数，当voca与ps指向同一个对象，该对象的引用计数为2，当voca被回收时，引用计数变成1，当ps被回收时，引用计数变为0，则释放对象的空间。

源码大概如下：
```cpp
class __shared_ptr {
private:
    element_type*	   _M_ptr;         // Contained pointer.
    __shared_count<_Lp>  _M_refcount;    // Reference counter.
}
```
指针对象和计数对象都用的是指针，当赋值的时候就拷贝两个指针。当一个对象回收时，计数对象指针指向的值减1️，当最后一个对象回收时，发现计数减1后变成了0，就可以`delete _M_ptr`，回收指针对象的内存空间了。

当然，引用计数并非完美，当存在循环引用时就会导致无法回收内存。试想正常的赋值是
```cpp
s2 = s1;
s3 = s2;
```
s1, s2, s3 共同维护的计数是3。s3回收，计数变为2；s2回收，计数变为1；s1回收，计数变为0；

而循环引用则是
```
s1->p = s2;
s2->p = s1;
```
s1 和 s2 维护的是不同的计数，都是2。s2回收，计数变为1；s1回收，计数变为1；都没有到达0，所以无法回收。

下面是具体例子：
```cpp
class TP {
    int id_;
public:
    TP(int id) : id_(id) {}
    shared_ptr<TP> ps;
    ~TP() {
        cout << "free " << id_ << endl;
    }
};

void test1() {
    shared_ptr<TP> t1(new TP(1));
    shared_ptr<TP> t2(new TP(2));

    cout << t1.use_count() << endl; // 1
    cout << t2.use_count() << endl; // 1

    t1->ps = t2;
    t2->ps = t1;

    cout << t1.use_count() << endl; // 2
    cout << t2.use_count() << endl; // 2

    // 回收t2时，引用计数减1，但t1->ps指向了t2，所以引用计数还有1
    // 回收t1时，引用计数减1，但t2->ps指向了t1，所以引用计数还有1
}
```

## 5. unique_ptr
上面提到，unique_ptr和auto_ptr一样都使用所有权机制，但unique_ptr不允许直接通过赋值或拷贝来转移所有权。
```cpp
unique_ptr<int> ps(new int(5));
unique_ptr<int> ps2 = ps; // error
```
你可以通过函数来手动转移所有权。
```cpp
unique_ptr<int> ps(new int(5));
unique_ptr<int> ps2(ps.release()); // ps释放所有权并返回指针
unique_ptr<int> ps3(new int(3));
ps3.reset(ps2.release());  // ps3释放原来的指针，并获取新的指针
```

相比于auto_ptr，unique_ptr还有另一个优点。它有一个可用于数组的变体。使用new分配内存时，才能使用auto_ptr和shared_ptr，使用new [ ]分配内存时，不能使用它们。
```cpp
unique_ptr<double[]> ps(new double(5));
auto_ptr<double[]> ps(new double(5));   // error
shared_ptr<double[]> ps(new double(5)); // error
```

## 6. weak_ptr
weak_ptr 从名字上看，表示“弱”指针，实际作用是指向一个 shared_ptr，但不改变 shared_ptr 的引用计数。既然如此，那么我们就可以使用 weak_ptr 来解决前面提到的循环引用问题了。但 weak_ptr 不维护引用计数，所以很可能其指向的对象已经被回收，所以每次获取对象时都要判断一下。
```cpp
auto p = make_shared<int>(5);
weak_ptr<int> wp(p);
if(shared_ptr<int> np = wp.lock()) {
    cout << *np << endl;
}
```


