# C++ 异常处理

## 简单try-catch
```cpp
double cal(double a, double b){
    return a * b / (a + b);
}
```
上面这个函数，如果 a == -b 的话，分母就变成了0，虽然不会报错，但得到的结果是一个-inf，仍然不是我们期望的。当我们认为这是一个异常时，我们需要抛出并处理。
```cpp
double cal(double a, double b){
    if(a == -b){
        throw "error";
    }
    return a * b / (a + b);
}

int main(){
    try {
        double res = cal(3, -3);
    }catch(const char* e){
        cout << e << endl;  // error
    };
}
```


## 异常对象
上面的例子中抛出了一个字符串，其实还可以抛出一个自定义对象。
```cpp
class CalException{
    double a, b;
public:
    CalException(double a, double b) : a(a), b(b) {}
    inline void msg(){
        cout << "cal(" << a << ", " << b << ") ";
        cout << "invalid arguments: a = -b" << endl;
    }
};

double cal(double a, double b){
    if(a == -b){
        throw CalException(a, b);
    }
    return a * b / (a + b);
}

int main(){
    try {
        double res = cal(3, -3);
    }catch(CalException &e){
        e.msg();
    };
}
```

## 为什么要catch一个引用
上面的例子中，你会发现当catch一个对象时，我们传入的是引用`&`。但throw语句默认抛出的是一个**自动变量**对象的副本，你去catch一个引用也只是catch了一个副本的引用，本质上还是一个副本。

之所以这么干，是因为我们可以通过引用来**获取其派生类型**。(see [多态](cpp-class.md#polymorphism))

```cpp
class Exception {
public:
    virtual void msg() = 0;
};

class TooBigException: public Exception {
    inline void msg() override {
        cout << "argument is too big" << endl;
    }
};

class CalException: public Exception {
    double a, b;
public:
    CalException(double a, double b) : a(a), b(b) {}
    inline void msg() override {
        cout << "cal(" << a << ", " << b << ") ";
        cout << "invalid arguments: a = -b" << endl;
    }
};

double cal(double a, double b){
    if(a > 100 || b > 100){
        throw TooBigException();
    }
    if(a == -b){
        throw CalException(a, b);
    }
    return a * b / (a + b);
}

int main(){
    try {
        double res = cal(3, -3);
    }catch(Exception &e){
        // 基类可以捕获其所有派生类
        e.msg();
    };
}
```

当要捕获多个异常时，try-catch是按顺序捕获的，你要优先捕获其派生类。
```cpp
try {
    double res = cal(3, -3);
}catch (CalException &e){ // 优先捕获派生类
    e.msg();
}catch (Exception &e){ // 基类可以捕获其所有派生类
    e.msg();
};
```

## 内建异常类
C++库定义了很多基于`std::exception`的异常类型。头文件stdexcept定义了其他几个异常类。首先，该文件定义了`logic_error`和`runtime_error`类，它们都是以公有方式从exception派生而来的。

其中logic_error定义的是逻辑异常，包含：
- domain_error 数学中的定义域错误
- invalid_argument 参数传递不如预期
- length_error 超出长度
- out_of_bounds 下标越界



而runtime_error定义的是运行时异常，包含：
- overflow_error 上溢，数值超出范围
- underflow_error 下溢，浮点数比规定最小非零数值还小（小数点超过多少位）。
- range_error 超出允许范围，太大或太小，但又没有发生上溢或下溢。

捕获后调用`what()`函数打印错误消息。
```cpp
try {
    double res = cal(3, -3);
}catch (out_of_range &e){
    e.what();
}catch (underflow_error &e){
    e.what();
}catch (runtime_error &e){
    e.what();
}catch (logic_error &e){
    e.what();
}catch (exception &e){
    e.what();
};
```

## 栈解退

假设我们嵌套调用函数，main -> b1() -> b2() -> b3()，当b3()里面抛出异常，那么程序会直接返回main中的catch块，这个过程叫**栈解退**（unwinding the stack）。

而程序会把从try到throw之间的所有自动对象的析构函数将被调用，注意这里的表达是调用析构函数，而不是回收内存，是否回收了所有内存要看析构函数了。

那么问题就是，如果是非自动变量，比如数组这种直接申请内存，又没有析构函数的怎么办？

```cpp
void func(int x){
    char* arr = new char[1024];
    string s("hello world");
    if (x) 
        throw std::runtime_error("boom");

    delete [] arr;    // 未被执行
}

int main(){
    try{
        func(10);
    }catch (const exception& e){
        return 1;
    }
}
```

上面的例子中，字符串`s`的内部虽然动态内存分配，但对象本身是自动变量，其析构函数将被调用。而数组`arr`的回收操作不会执行，造成内存泄漏。

傻一点的解决办法就是捕获它然后回收内存，然后再抛出去。
```cpp
void func(int x){
    char* arr = new char[1024];
    string s("hello world");
    try{
        if (x)
            throw runtime_error( "boom");
    }catch (exception &e){
        delete [] arr;
        throw ;
    }
    delete [] arr;
}
```
