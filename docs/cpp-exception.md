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
上面的例子中，你会发现当catch一个对象时，我们传入的是引用`&`。但throw语句默认抛出的是一个**自动变量**对象的副本，你去catch一个引用也只是catch了一个副本的引用，本质上还是一个副本。而之所以这么干，是因为我们可以通过引用来**获取其派生类型**。(see [多态](cpp-class.md#polymorphism))

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

## noexcept
C++11 默认情况下会认为你定义的每个函数都可能抛出异常，当你在函数后面加入 noexcept 后，表示告诉编译器该函数不抛出异常，则编译器会做一些优化来提升性能。（如果发生了异常则直接终止程序）
```cpp
RetType function(params) noexcept; //优化最好

RetType function(params) throw();  //没有优化

RetType function(params);          //没有优化
```

## 异常安全
如果一段代码是异常安全的，那么这段代码运行时的失败不会产生有害影响，如内存泄露、存储数据混淆、或无效的输出。

异常中立性：指当你的代码（包括你调用的代码）引发异常时，这个异常能保持原样传递到外层调用代码。

异常安全性：1.抛出异常后，资源不泄露。2.抛出异常后，不会使原有数据恶化（例如正常指针变野指针）

通常有三个异常安全级别：
1. 不抛异常保证（no throw guarantee）：函数不抛出异常。确保办法是只使用内置类型，使用noexcept关键字。
2. 强烈保证（the strong guarantee） ：如果抛出了异常，程序的状态没有发生任何改变。就像没调用这个函数一样。确保办法是 copy and swap 或智能指针。
3. 基本保证（the basic guarantee）：抛出异常后，对象仍然处于合法（valid）的状态。但不确定处于哪个状态。

### 抛出异常
```cpp
class Menu{
    Mutex m;
    Image *bg;
    int changeCount;
public:
    void changeBg(istream& sr);
};
void Menu::changeBg(istream& src){
    lock(&mutex);
    delete bg;
    bg = new Image(src);
    ++changeCount;
    unlock(&mutex);
}
```
上面的代码，万一申请内存失败了，就会抛出"bad alloc"异常。这时mutex资源被泄露了，因为没有被unlock；其次是bg变成了空，原来的数据丢失。

### 使用智能指针的强烈保证
智能指针的reset是用来重置其中的资源的，在其中调用了旧资源的delete。这时如果new Image发生了异常，便不会进入reset函数。
```cpp
class Menu{
    shared_ptr<Image> bg;
    ...
};
void Menu::changeBg(istream& src){
    Lock m1(&m);
    bg.reset(new Image(src));
    ++changeCont;
}
```
事实上，上述代码并不能提供完美的强烈保证，比如Image构造函数中移动了`istream& src`的读指针然后再抛出异常，那么系统还是处于一个被改变的状态。不过这一点可以忽略，我们暂且认为它提供了完美的强烈保证。

### Copy & Swap 的强烈保证
首先需要知道 swap 函数具有noexcept声明，确保不抛出异常。

这时我们可以先把要被改变的那个对象拷贝一份出来，然后去改变这个拷贝对象，如果发生异常，最多只影响拷贝对象，而原本的对象不会改变；如果过改变成功，则把拷贝对象与原对象交换（swap不抛出异常），然后再释放拷贝对象即可。这种方式也能提供强烈保证。
```cpp
class Menu{
    ...
private:
    Mutex m;
    std::shared_ptr<MenuImpl> pImpl;
};
Menu::changeBg(std::istream& src){
    using std::swap;
    Lock m1(&mutex);

    std::shared_ptr<MenuImpl> copy(new MenuImpl(*pImpl));
    copy->bg.reset(new Image(src));
    ++copy->changeCount;

    swap(pImpl, copy);
}
```




