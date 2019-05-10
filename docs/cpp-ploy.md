# C++ 多态

## 虚函数
多态允许派生类覆盖(override)基类的方法。只要对基类方法加上关键字`virtual`即可。<span id="polymorphism"></span>
```cpp
#include <iostream>
using namespace std;

class Vehicle {
public:
    // 虚函数
    virtual void start() {
        cout << "vehicle start" << endl;
    }
    // 纯虚函数（不实现）
    virtual void drive() = 0;
};

class Car: public Vehicle {
public:
    // 重载基类虚函数
    void start() override {
        cout << "car start" << endl;
    }
    void drive() override {
        cout << "dirve car" << endl;
    }
};

class Bus: public Vehicle {
public:
    void start() override {
        cout << "bus start" << endl;
    }
    void drive() override {
        cout << "dirve bus" << endl;
    }
};

int main(){

    Bus bus = Bus();
    Car car = Car();

    // 用基类引用来获得其派生类对象
    Vehicle& busv = bus;
    Vehicle& carv = car;
    // 或者使用指针类型
    // Vehicle* busv = new Bus();
    // Vehicle* carv = new Car();

    busv.start();
    carv.start();

    busv.drive();
    carv.drive();
}
```
输出：
```
vehicle start
bus start
vehicle start
car start
drive bus
drive car
```
因为C++支持多继承，所以没有像java那样的super关键字，若要调用基类方法，可以使用`Vehicle::start()`。

C++中的自动变量不能像java那样在`Vehicle bus = Bus()`直接用基类型来接收其派生类，必须是实例化一个派生类对象，然后再用基类对象去引用。

## 抽象类和接口
C++中没有关键字声明抽象类和接口，是通过一些特征来区分的。

抽象类：只要存在一个虚函数，就是抽象类。

接口：
1. 类中没有定义任何成员变量 
2. 类中所有成员函数都是公有且都是纯虚函数

## 重载，覆盖和隐藏
重载(Overload)：在同一个类中，创建函数名相同，但参数不同的多个函数。

覆盖(Overriding)：在派生类中创建一个与基类中带有virtual关键字的同名同参数函数。

隐藏(Hiding)：当基类和派生类具有同名函数，而参数不同，基类函数会被隐藏；同样，当参数相同但基类函数没有virtual，则基类函数也会被隐藏。

值得一提，覆盖才是多态的核心功能。多态使得我们可以通过一个类型来接收不同的衍生版本，进而通过一个相同的接口调用不同实现。

## 内存模型
虽然最终结果相同，但不同编译器对多态的实现可能不同。一般通过虚函数表(virtual function table)实现。只要类中定义了虚函数，编译器自动添加隐藏指针vfptr指向虚函数表。指针vfptr通常在对象内存的首地址。

### 函数指针
在详细解析虚函数表的内存模型之前，先认识一下函数指针。事实上，c++中的函数是通过地址来索引的，只要找到该函数的地址，就能够调用该函数。

全局函数的函数指针定义形式：
> 返回类型 (*函数指针名称)(参数列表) = &函数名

调用形式为：
> (*函数指针名称)()

如：
```cpp
int max(int &a, int &b){
    if(a > b){
        return a;
    }
    return b;
}
int (*max_ptr)(int &a, int &b) = &max;

void say_hello(){
    cout << "hello!" << endl;
}
void (*say_hello_ptr)() = &say_hello;

int main() {
    int a = 10, b = 12;
    cout << (*max_ptr)(a, b) << endl; // 12
    (*say_hello_ptr)(); // hello!
}
```
你还可以使用`typedef`来给函数指针定义一个别名。
```cpp
typedef void void_func();
void_func* say_hello_ptr = (void_func*) &say_hello;
int main() {
    (say_hello_ptr)();
}
```


成员函数的函数指针定义形式：
> 返回类型 (类名::*函数指针名称)(参数列表) = &类名::函数名

调用形式为：
> (类实例.*函数指针名称)()

```cpp
class Person {
public:
    void say() { cout << "hello!" << endl; }
};

void (Person::*person_say_ptr)() = &Person::say;

int main() {
    Person p;
    (p.*person_say_ptr)(); // hello!
}
```

对于**成员虚函数**，还有另一种办法可以获取其地址，并转换为函数指针调用。前面提到，凡是具有虚函数的对象都会维护一个**虚函数表**来存储虚函数的地址，只要我们访问这个表，并获取地址即可。

```cpp
class Person {
public:
    virtual void say() { cout << "hello!" << endl; }
};

int main() {
    Person p;
    long* p_addr = (long*) &p;
    // 虚函数表地址指针
    long* vt_addr = (long*) *(p_addr + 0);
    // 虚函数地址指针 开头指向 第一个虚函数地址
    void* say_addr = (void*) *(vt_addr + 0);

    // 地址转换为函数指针
    void(*say_ptr)() = (void(*)()) say_addr;
    (*say_ptr)(); // hello!
}
```
使用`long*`是因为64位系统地址需要使用long存储，如果是32位，就用`int*`。上面的使用的`void*`可以接收任何类型的指针。

### 虚函数表
每一个具有虚函数的类对象，对象的内存首地址都会维护一个vtptr指针，该指针指向一个虚函数表。那么这和多态有什么关系呢？

先下结论：**每一个对象实例都会维护一个虚函数表，这里的实例也包括继承实例**。当存在继承关系，基类实例会维护一张自己的虚函数表，而派生类会将基类的虚函数表复制一份到自己那里。如果派生类覆盖了基类某个虚函数，则会将自己的虚函数表对应的虚函数更换一个新地址，而不会影响基类虚函数表上的地址。

上面的理论说明了，尽管你用基类引用一个派生类，虚函数还是会使用派生类的而不是基类的，是因为当调用虚函数时，就会通过对象内存首地址访问虚函数表，并查找你要调用的虚函数。尽管类型声明是基类，但访问的仍然是派生类的虚函数表。而当你通过基类实例调用方法`devired.Base::function()`，那么就会访问基类的虚函数表，从而调用基类的实现。


下面是证明代码：
```cpp
class Base {
public:
    // 默认值1和2
    Base(int mem1 = 1, int mem2 = 2) : bmem1_(mem1), bmen2_(mem2) { ; }

    virtual void bfunc1() { std::cout << "In bfunc1()" << std::endl; }
    virtual void bfunc2() { std::cout << "In bfunc2()" << std::endl; }
    virtual void bfunc3() { std::cout << "In bfunc3()" << std::endl; }

private:
    int bmem1_;
    int bmen2_;
};

class Devired : public Base {
public:
    // 默认值3
    Devired(int mem = 3) : dbmem1_(mem) { ; }

    // 覆盖基类的虚函数
    void bfunc2() override { std::cout << "In Devired bfunc2()" << std::endl; }
    // 派生类的虚函数
    virtual void dfunc1() { std::cout << "In Devired dfunc1()" << std::endl; }
    // 非虚函数
    void ndfunc1() { std::cout << "In Devired ndfunc1" << std::endl; }
private:
    int dbmem1_;
};

typedef void func(void);
func* FUNC(void *pf) {
    func* f1 = (func*)pf;
    return f1;
}

// 观察基类
void watch_base() {
    Base b;

    // 对象b的地址
    long* bAddress = (long*) &b;

    // 对象b的vtptr的值
    long* vtptr = (long*) *(bAddress + 0);
    printf("vtptr: 0x%08x\n", vtptr);

    // 对象b的第一个虚函数的地址
    void* pFunc1 = (void*) *(vtptr + 0);
    void* pFunc2 = (void*) *(vtptr + 1);
    void* pFunc3 = (void*) *(vtptr + 2);

    // 调用虚函数
    (FUNC(pFunc1))();
    (FUNC(pFunc2))();
    (FUNC(pFunc3))();

    cout << endl;
    printf("\t bfunc1 addr: 0x%08x \n"
           "\t bfunc2 addr: 0x%08x \n"
           "\t bfunc3 addr: 0x%08x \n",
           pFunc1,
           pFunc2,
           pFunc3);

    // 对象b的两个成员变量的值（用这种方式可轻松突破private不能访问的限制）
    int mem1 = (int) *(bAddress + 1);
    int mem2 = (int) *(bAddress + 2);
    cout << endl;
    printf("bmem1_: %d \nbmen2_: %d \n\n", mem1, mem2);
}

// 观察派生类
void watch_devired() {
    Devired d;
    long *dAddress = (long*) &d;

    // 虚表地址
    long *vtptr = (long*) *(dAddress + 0);

    // 数据成员的地址
    int bmem1 = (int) *(dAddress + 1);
    int bmem2 = (int) *(dAddress + 2);
    int dmem1 = (int) *(dAddress + 3);

    // 函数地址
    void* pFunc1 = (void*) *(vtptr + 0);
    void* pFunc2 = (void*) *(vtptr + 1);
    void* pFunc3 = (void*) *(vtptr + 2);
    void* pdFunc1 = (void*) *(vtptr + 3);

    (FUNC(pFunc1))();
    (FUNC(pFunc2))();
    (FUNC(pFunc3))();
    (FUNC(pdFunc1))();

    cout << endl;
    printf("\t bfunc1 addr: 0x%08x \n"
           "\t bfunc2 addr: 0x%08x \n"
           "\t bfunc3 addr: 0x%08x \n"
           "\t dfunc1 addr: 0x%08x \n\n",
           pFunc1,
           pFunc2,
           pFunc3,
           pdFunc1
    );

    printf("bmem1: %d\nbmem2: %d\ndmem3: %d\n\n\n", bmem1, bmem2, dmem1);
}

int main() {
    watch_base();
    cout << "----------------" << endl;
    watch_devired();

    system("pause");
    return 0;
}
```
最终输出：(代码运行在Visual Studio)

![](https://i.loli.net/2019/05/10/5cd59d39268af.png)

你会发现，派生类没有覆盖`bfunc1`和`bfunc2`虚函数，所以它们打印出的地址是一样的。而派生类覆盖了`bfunc2`，导致派生类的虚函数表的`bfunc2`的地址被更新，所以它与基类中的地址不同。

## 参考
- [深入分析C++虚函数表](https://jocent.me/2017/08/07/virtual-table.html)
