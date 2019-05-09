# C++ 多态

## 多态
多态允许派生类重载基类的方法。只要对基类方法加上关键字`virtual`即可。<span id="polymorphism"></span>
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
        // 调用基类函数
        Vehicle::start();
        cout << "car start" << endl;
    }
    void drive() override {
        cout << "drive car" << endl;
    }
};

class Bus: public Vehicle {
public:
    void start() override {
        Vehicle::start();
        cout << "bus start" << endl;
    }
    void drive() override {
        cout << "drive bus" << endl;
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
因为C++支持多继承，所以没有像java那样的super关键字，若要调用基类方法，可以使用上面例子的方法`Vehicle::start()`。

C++中的自动变量不能像java那样在`Vehicle bus = Bus()`直接用基类型来接收其派生类，必须是实例化一个派生类对象，然后再用基类对象去引用。

## 8. 抽象类和接口
C++中没有关键字声明抽象类和接口，是通过一些特征来区分的。

抽象类：只要存在一个虚函数，就是抽象类。

接口：
1. 类中没有定义任何成员变量 
2. 类中所有成员函数都是公有且都是纯虚函数

