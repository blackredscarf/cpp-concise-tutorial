# C++ 同步原语

## mutex
mutex称为互斥量，也叫互斥锁。对于线程间共享数据，可以使用互斥锁来锁住一段代码，线程访问前加锁，其他线程访问时阻塞，只有该线程解锁后，其他线程才会被唤醒去能访问。

先来看看不加锁的情况，我们尝试用两个线程去pop一个栈。如果栈为空时，调用top()会报错。
```cpp
void pop(stack<int>& st) {
    if(!st.empty()) {
        st.top();
        st.pop();
    }
}

int main() {
    stack<int> st = stack<int>();
    st.push(1);
    thread th1([&](){
        pop(st);
    });
    thread th2([&](){
        pop(st);
    });
    while (true) {}
}
```
我们可以假设会报错的时序：

| th1 | th2 |
| --- | --- |
| if(!st.empty()) | |
| | if(!st.empty()) |
| st.top() | |
| st.pop() | |
| | st.top() |

明显，第一个线程已经把栈最后的元素pop出去了，但第二个线程仍然通过top()获取栈顶元素。

使用互斥锁的例子：
```cpp
mutex mu;

void pop(stack<int>& st) {
    mu.lock();
    if(!st.empty()) {
        int a = st.top();
        st.pop();
        cout << a << endl;
    }
    mu.unlock();
}
```

使用互斥锁后的时序，

| th1 | th2 |
| --- | --- |
| mu.lock() | |
| | mu.lock() |
| if(!st.empty()) | |
| st.top() | |
| st.pop() | |
| mu.unlock() | |
| | if(!st.empty()) |

线程1解锁后，会唤醒线程2，线程2才开始加锁并访问。

建议使用下面这种RAII思想的加锁方法：
```cpp
void pop(stack<int>& st) {
    std::lock_guard<mutex> lg(mu);
    if(!st.empty()) {
        int a = st.top();
        st.pop();
        cout << a << endl;
    }
}
```

## 死锁
假设一个数据由两份组成，必须拿到两份完整的数据才能处理，如果线程1拿到第1份，线程2拿到第2份，而两个线程都不打算让出自己那份，这时两个线程都无法进行下去，就导致死锁。

看一个死锁的案例：
```cpp
void do_something1() {
    lock_guard<mutex> lg1(mu1);
    lock_guard<mutex> lg2(mu2);
    // ...
}

void do_something2() {
    lock_guard<mutex> lg2(mu2);
    lock_guard<mutex> lg1(mu1);
    // ...
}
```
假设线程1调用do_something1，线程2调用do_something2，这时线程1给mu1加锁，线程2给mu2加锁，当线程1尝试给mu2加锁时则发现已经被锁，线程2同理。

| thread1 | thread2 |
| --- | --- |
| do_something1 | |
| | do_something2 |
| lock_guard<mutex> lg1(mu1) | |
|  | lock_guard<mutex> lg2(mu2) |

解决办法是确保两个锁加锁的顺序永远保持一致。

还有办法就是一次性给两个锁加锁，用std::lock()即可，为了防止lock_guard给一个锁加两次锁，加上std::adopt_lock参数。
```cpp
void do_something2() {
    std::lock(mu2, mu1);
    lock_guard<mutex> lg2(mu2, std::adopt_lock);
    lock_guard<mutex> lg1(mu1, std::adopt_lock);
    // ...
}
```
C++17提供了一个更好的管理对象scoped_lock，一次性管理多个锁：
```cpp
void do_something2() {
    scoped_lock sl(mu2, mu1);
    // ...
}
```

## unique_lock
unique_lock是一个锁的所有权管理对象，使得锁可以安全地在多个作用域之间转移。

```cpp
unique_lock<mutex> prepare(unique_lock<mutex> ul) {
    cout << "prepare" << endl;
    return ul;
}

unique_lock<mutex> do_something(unique_lock<mutex> ul) {
    cout << "do_something" << endl;
    return ul;
}

int main() {
    thread th1([&](){
        unique_lock<mutex> ul(mu);
        cout << "thread 1:" << endl;
        ul = prepare(move(ul));
        ul = do_something(move(ul));
    });
    thread th2([&](){
        unique_lock<mutex> ul(mu);
        cout << "thread 2:" << endl;
        ul = prepare(move(ul));
        ul = do_something(move(ul));
    });
    while (true) {}
}
/*
thread 1:
prepare
do_something
thread 2:
prepare
do_something
*/
```

unique_lock可以调用lock(), try_lock(), unlock()等函数，较小粒度地控制加锁，但这样会增加代码复杂度，要好好考虑使用场景。

##  call_once
在多线程同时获取一块数据时，数据可能只会在开始经历初始化过程存在数据竞争风险，数据初始化和获取在同一个函数里面做在单例模式下会用得很多，但初始化往往只有一次，每次获取都要加锁检查是否初始化就得不偿失了。

初始化加锁例子：
```cpp
string s;
bool isInit = false;

const string& initData() {
    // ...
    return s;
}

const string& getData() {
    unique_lock<mutex> ul(mu);
    if(!isInit) {
        s = initData();
        isInit = true;
    }
    return s;
}
```
上面意味着每次getData()都要加一次锁。

解决办法是使用call_once函数，call_once确保一个函数只调用一次，内部的实现可以确保线程安全和高效率。虽然我还没仔细研究内部实现，但初步猜测是通过原子操作实现的。
```cpp
string s;
std::once_flag isInit;

const string& initData() {
    // ...
    return s;
}

const string& getData() {
    std::call_once(isInit, initData);
    return s;
}
```
