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
    // 虚基类的析构函数必须实现为虚函数
    virtual ~Vehicle(){ 
        cout << "vehicle deconstruct" << endl;
    }
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
    ~Car() override {
        cout << "car deconstruct" << endl;
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
    ~Bus() override {
        cout << "bus deconstruct" << endl;
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

    busv.start(); // bus start
    carv.start(); // car start

    busv.drive(); // dirve bus
    carv.drive(); // dirve car
    
    // car deconstruct
    // vehicle deconstruct
    // bus deconstruct
    // vehicle deconstruct
}
```
因为C++支持多继承，所以没有像java那样的super关键字，若要调用基类方法，可以使用`Vehicle::start()`。

C++中的自动变量不能像java那样在`Vehicle bus = Bus()`直接用基类型来接收其派生类，必须是实例化一个派生类对象，然后再用基类对象去引用。

考虑一个比较偏的情况：如果派生类调用基类方法，而基类方法又调用了虚函数，那么这个虚函数的实现到底是派生类实现，还是基类实现？
```cpp
class Vehicle {
public:
    virtual void start() {
        cout << "vehicle start" << endl;
        // 此处调用虚函数的结果是？
        start2();
        this->start2();
        Vehicle::start2();
    }
    virtual void start2() {
        cout << "vehicle start 2" << endl;
    }
};

class Car: public Vehicle {
public:
    void start() override {
        cout << "car start" << endl;
        Vehicle::start(); // 调用基类start
    }
    void start2() override {
        cout << "car start 2" << endl;
    }
};

int main() {
    Vehicle* v = new Car();
    v->start();
}
```
答案是：
```cpp
start2();       // car start 2
this->start2(); // car start 2
Vehicle::start2(); // vehicle start 2
```
从这个例子可以看出，在一个对象的基类实例中的this指针仍然是派生类的，查找函数会在派生类的虚函数表中查找，所以在基类中调用虚函数，会调用派生类实现。

## 抽象类和接口
C++中没有关键字声明抽象类和接口，是通过一些特征来区分的。

抽象类：
1. 存在一个及以上虚函数，就是抽象类。

接口：
1. 类中没有定义任何成员变量 
2. 类中所有成员函数都是**公有**且都是**纯虚函数**

## 重载，覆盖和隐藏
重载(Overload)：在同一个类中，创建函数名相同，但参数不同的多个函数。

覆盖(Overriding)：在派生类中创建一个与基类中带有virtual关键字的同名同参数函数。

隐藏(Hiding)：当基类具有与派生类同名同参数的非虚函数时，基类函数会被隐藏。当基类具有与派生类同名而参数不同的成员函数时，基类函数会被隐藏。

值得一提，覆盖才是多态的核心功能。多态使得我们可以通过一个类型来接收不同的衍生版本，进而通过一个相同的接口调用不同实现。

## 构造函数，析构函数和虚函数

**首先，思考构造函数和析构函数能不能是虚函数？**
构造函数不能是虚函数。有些人会说构造函数调用时，内存中指向虚函数表的指针还没被初始化，这是句话是对的，但不是构造函数不能是虚函数的原因，尽管指向虚函数表的指针还没被初始化，但函数可以作为成员函数来调用，下面会说到这个。正确原因是调用构造函数时，它不知道自己的基类还是派生类，还是继承树中的一个中间节点，它应该调用自己的构造函数还是派生类的构造函数呢？若多个派生类都实现了他的构造函数，他应该调用哪个？会产生冲突，所以不能。[[ref]](
https://www.zhihu.com/question/35632207/answer/63936329)

当类中含有虚函数，则析构函数总应该声明为虚函数，否则当用基类对象接收派生类，delete这个对象时就只会调用基类的析构函数，而派生类的析构函数则无法调用 [[ref]](cpp-object-model.md)。基类对象回收时需要调用派生类实现的析构函数。

**那构造函数和析构函数能不能调用虚函数呢？**
答案是可以调用，但最好不要。构造函数调用时，指向虚函数表的指针还没被初始化，虽然确实可以调用，但只是作为成员函数，如果存在继承关系，这时派生类还没有构造，它不会调用到派生类的实现版本；

而对于析构函数，派生类析构后，基类调用的虚函数到底是自己的还是派生类的呢？若是调用基类实现则与运行时的版本不同，若调用派生类的，可能会访问到已回收的属性，所以最好不要这么干。

## 参考
- [深入分析C++虚函数表](https://jocent.me/2017/08/07/virtual-table.html)
