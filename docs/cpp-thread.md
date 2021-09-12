# C++ 线程

## 线程构造
调用thread class的构造函数，可以传入一个函数，后面跟着函数的参数。构造之后会马上开启一个线程并执行。
```cpp
#include <iostream>
#include <thread>
using namespace std;

void do_something(int v) {
    std::this_thread::sleep_for(chrono::milliseconds(2000));
    cout << v << endl;
}

int main() {
    thread th(do_something, 3);
    while(true){}
}
```

还有一个种构造方法：传入一个重载了()运算符的对象，thread对象内部会自动执行。
```cpp
class DoSomething {
public:
    DoSomething(int v): _v(v) {};
    void operator()() {     // 重载()
        std::this_thread::sleep_for(chrono::milliseconds(2000));
        cout << _v << endl;
    }
private:
    int _v;
};

int main() {
    thread th(DoSomething(3));
    while(true){}
}
```
传入lambda函数也是可行的。
```cpp
thread th2([](int v) {
    std::this_thread::sleep_for(chrono::milliseconds(2000));
    cout << v << endl;
}, 3);
```

## 线程ID
可以通过 std::this_thread::get_id() 来获取当前线程的ID。
```cpp
void do_something(int v) {
    std::this_thread::sleep_for(chrono::milliseconds(2000));
    cout << "do_something: " << std::this_thread::get_id() << endl;
}

int main() {
    thread th(do_something, 3);
    cout << "main: " << std::this_thread::get_id() << endl;
    while(true){}
}
```
输出：
```cpp
main: 1
do_something: 2
```
可以发现线程ID是从1开始递增的。

但注意get_id()的返回类型不是一个整数，而一个thread::id类型，我们能看到整数是因为它重载了输出运算符。看一下它的构造方法：
```cpp
class id {
    native_handle_type    _M_thread;
public:
    id() noexcept : _M_thread() { }
    explicit id(native_handle_type __id) : _M_thread(__id) { }
// ...
```
内部又构造了native_handle_type这个类型。一直寻找这个类型的最终形态，可以发现其实就是一个uintptr_t类型。
```cpp
typedef uintptr_t pthread_t;
typedef unsigned __int64 uintptr_t;
```
thread对象的内部成员_M_id的类型就是thread::id，当开启一个线程时，_M_id会被赋予一个线程ID。



## 线程结束与销毁
线程所执行的代码块结束后线程就会自动销毁，这个过程由操作系统完成，而用户态程序的thread对象成员_M_id会被重置为thread::id()，即thread::id()的默认构造函数，即_M_thread=0。

如果在OS线程未结束的情况下，销毁了thread对象会发生什么？可以看一下thread的析构函数：
```cpp
~thread() {
    if (joinable())
        std::terminate();
}

bool joinable() const noexcept { 
    return _M_id != id()
}
```
如果joinable()=true，就会调用std::terminate()终止当前进程，注意不是终止线程而是终止整个进程。joinable()其实就是比较了一下线程ID，看看对象里的_M_id是不是被重置过。如果_M_id != id()，则_M_id还没被重置，意味着线程还没结束。如果你要销毁这个对象是不合法行为，则会强制退出进程，控制台会打印 terminate called without an active exception 。
```cpp
void do_something(int v) {
    std::this_thread::sleep_for(chrono::milliseconds(2000));
}

int main() {
    thread* th = new thread(do_something, 3);
    delete th;
}
```

## join
join的意思是连接，调用`thread->join()`意味着一个线程和另一个线程连接在一起，只有线程1执行完后，线程2才继续执行。
```cpp
void do_something(int v) {
    std::this_thread::sleep_for(chrono::milliseconds(2000));
    cout << "thread 1 end" << endl;
}

int main() {
    thread* th = new thread(do_something, 3);

    thread th2([&]() {
        th->join();
        cout << "thread 2 end" << endl;
    });

    while (true) {}
}
```
输出：
```
thread 1 end
thread 2 end
```
我们再回头想一下，为什么析构函数里的判断函数叫joinable()，个人感觉是因为一个结束的进程不可能再join，而未结束进程才能join。但这个并不影响join()函数的调用，因为我实验发现join()函数就算调用多次也不会报错。


## detach
detach意味着脱离，调用detach()后，OS线程不再受thread对象控制，_M_id会被重置，即使你把thread对象销毁，线程仍然运行。但从OS层面上，主线程退出后子线程依旧会被强制退出。
```cpp
void do_something(int v) {
    std::this_thread::sleep_for(chrono::milliseconds(2000));
    cout << "thread 1 end" << endl;
}

int main() {
    thread* th = new thread(do_something, 3);
    th->detach();
    delete th;  // delete thread object
    while (true) {}
}
// thread 1 end
```

## 异常处理
C++的异常处理是个非常棘手的问题，一旦发生异常，函数会被终止并抛出异常，导致原本该执行的代码无法被执行，比如回收内存。对于线程场景下，一旦发生异常，函数终止，导致thread自动对象的析构函数调用，进而导致进程终止。

先看一个错误示例：
```cpp
int main() {
    try {
        thread th = thread(do_something, 3);
        throw runtime_error("test");
    } catch (runtime_error& e) {
    }
    while (true) {}
}
```
throw一个错误后，就马上离开了try代码块，自动变量th调用析构函数，但可能线程还没结束，这时进程会被终止。

解决办法是通过“资源获取即初始化”(RAII，Resource Acquisition Is Initialization)进行规避，
```cpp
class ThreadGuard {
    thread& t_;
public:
    explicit ThreadGuard(thread& t): t_(t) {}
    ~ThreadGuard() {
        if(t_.joinable()) {
            t_.join();
        }
    }
};

int main() {
    try {
        thread th = thread(do_something, 3);
        ThreadGuard tg(th);
        throw runtime_error("test");
    } catch (runtime_error& e) {
    }
    while (true) {}
}
```
我们在自动变量tg的析构函数里面调用thread的join方法，使其在被调用时能够等待线程执行完毕。

## 参数传递

### 引用传递
C++可以传引用类型，但thread的构造函数并不能分辨是否应该传递引用，这时你可以通过构造reference_wrapper来显示声明需要传递引用，
```cpp
void handle(string& s) {
    cout << s << endl;
}

int main() {
    string s = "hello";
    thread th(handle, std::ref(s));
    while (true) {}
}
```

### 类方法传递
可以通过传入 类方法地址+对象地址 的形式把方法传入thread对象。
```cpp
struct Handle {
    void do_something(int v) {
        cout << v << endl;
    }
};

int main() {
    Handle h;
    thread th(&Handle::do_something, &h, 99999); 
    while (true) {}
}
// 99999
```

### 所有权转移
通过move()搭配unique_ptr可以完成所有权转移，同一刻总是只有一个线程拥有数据的所有权，这是一个解决数据竞争的良好解决办法。
```cpp
void handle(unique_ptr<string> s) {
    cout << *s << endl;
}

int main() {
    unique_ptr<string> data(new string("hello"));
    thread th(handle, move(data));
    while (true) {}
}
// hello
```