# C++ 类

## 1. 构造函数
### 1.1 无参构造
如果你没有自己定义构造函数，就会调用**默认构造函数**。
```cpp
Stock first; //1 隐式
Stock first{}; //2  显示
Stock first = Stock(); //3 显示
```
注意1不要加上空`()`，因为会变成函数。

### 1.2 有参构造
```cpp
class Stock{
    string name;
    int shares;
public:
    Stock(const string &name, int shares) : name(name), shares(shares) {}
};
```
构造函数后面可以用冒号`:`来进行值的初始化。
```cpp
Stock(const string &name, int shares) : name(name), shares(shares) {}
```
隐式调用
```cpp
Stock s{"world cabbage", 250};
Stock s("world cabbage", 250);
```
显式调用
```cpp
Stock s = Stock("world cabbage", 250);
```

### 1.3 拷贝构造函数
拷贝构造函数其实就是传入一个同等类型，然后依次对属性赋值的构造函数，这是C++内部的默认拷贝构造函数的行为。当然你可以自己去定义一个。
```cpp
Stock(const Stock& stock){
    this->name = stock.name;
    this->shares = stock.shares;
}
```
使用：
```cpp
Stock s1{"world cabbage", 250};
Stock sc = s1;
Stock sc2(s1);
```

## 2. 析构函数
对象过期时，程序将自动调用**析构函数 (Destructor)**。

- 如果创建的是自动存储类对象（没有使用new），则其析构函数将在程序执行完代码块时自动被调用。
- 如果对象是通过new创建的，则它将驻留在栈内存或自由存储区中，当使用delete来释放内存时，其析构函数将自动被调用。
- 如果程序员没有提供析构函数，编译器将隐式地声明一个默认析构函数，并在发现导致对象被删除的代码后，提供默认析构函数的定义。

用前缀`~`定义：
```cpp
~Stock() {
    cout << "destructor" << endl;
}
```

## 3. 初始化，赋值和拷贝
先写一个类，以便说明：
```cpp
class Stock{
    string name;
    int shares;
public:
//    Stock(const string &name, int shares) : name(name), shares(shares) {}
    Stock(const string &name, int shares) : name{name}, shares{shares} {}
    Stock(){};
    Stock(const Stock& stock){
        this->name = stock.name;
        this->shares = stock.shares;
    }
    const string &getName() const {
        return name;
    }
    int getShares() const {
        return shares;
    }
    void setName(const string &name) {
        Stock::name = name;
    }
    void setShares(int shares) {
        Stock::shares = shares;
    }
    ~Stock() {
        cout << "destructor" << endl;
    }
};
```

### 3.1 初始化
将一个片空间分配给一个新定义的对象，表示**初始化**。
```cpp
Stock s{"world cabbage", 250};
Stock s("world cabbage", 250);
Stock s = Stock("world cabbage", 250);
Stock s1;   // 隐式构造（调用默认构造函数，即没有参数的构造函数）
// Stock s2(); // 别这么干，会被认为是声明了一个返回Stock类型的函数
```
注意，我上面的对初始化的描述没有说是调用“构造函数”，是因为初始化一个对象并不一定直接通过构造函数。

我们知道，从函数返回的变量将是一个临时变量，而当你用一个新定义的变量来接收时，新变量会把这片临时变量的空间作为它自己的空间，即为**初始化**。
```cpp
Stock getStock(){
    Stock s{"inner", 122};
    return s;
}

int main(){
    Stock ss = getStock();
}
```
事实上，这里的表现和上面直接调用构造函数初始化是统一的，因为调用构造函数本身便产生了一个临时变量。

### 3.2 赋值
将一个已存在的变量赋值给另一个已存在的变量为赋值。s1的数据覆盖了s2，内部调用了“赋值运算符”，即`operator=()`。这个赋值默认是一个浅拷贝，关于浅拷贝和深拷贝，会在移动语义那一节详细讲到。
```cpp
Stock s1{"world cabbage", 250};
Stock s2;
s2 = s1;
s2.setName("world cabbage 2");
cout << s2.getName() << endl; // world cabbage 2
cout << s1.getName() << endl; // world cabbage
```

### 3.3 拷贝
由一个已定义的对象赋值给新定义对象为拷贝。新对象将会自动调用拷贝构函数。
```cpp
Stock s1{"world cabbage", 250};
Stock sc = s1;
Stock sc2(s1);
```
包括你把某个对象传进函数中，也将调用拷贝构造。
```cpp
void display(Stock s){  // a copy of s3
    cout << s.getName() << " " << s.getShares();
}
Stock s3{"outer", 233};
display(s3);
```


特别点出，在已存在对象上赋值一个临时对象，这是很不好的习惯。
```cpp
Stock s1{"world cabbage", 250};
s1 = Stock("world cabbage", 250);
```
先说一下程序执行的流程：
1. Stock 调用构造函数得到一个临时对象
2. s1调用赋值运算符对临时对象进行浅拷贝(shallow copy)
3. 临时对象被销毁

这时候，你设法获取s1的数据时就会可能出错，因为s1只是临时对象的浅拷贝，临时对象销毁后，s1中的指针变量可能指向的是已经被回收的内存空间。

当然，如果是使用new来新建对象不会出现上面的问题，因为动态变量不会被自动销毁。


## 4. const对象
*see [constant](cpp-constant.md)*

如果对一个对象声明为const意味着你不能改变他，而且该对象只能调用声明了const的方法。

声明方式是在函数名后面加上const。
```cpp
const string &getName() const {
    return name;
}
```

## 5. 枚举类
```cpp
enum class Color { red, blue, green };
enum class Trafflc_light { green, yellow, red };
Color color = Color::red;
```


## 6. 继承
在C++的继承关系中，父类被称为基类，子类被称为派生类。对于公有继承，在派生类后面加上`: public <Base>`来继承`<Base>`类。
```cpp
#include <iostream>
#include <random>
#include <ctime>
using namespace std;

class Vehicle {
    int vehicleId;
    string name;
public:
    Vehicle(string name): name{name} {
        // 生成随机数
        default_random_engine e((unsigned)time(nullptr));
        vehicleId = e();
        cout << "This is a Vehicle " << vehicleId << endl;
    }
    const int getVehicleId() const {
        return vehicleId;
    }
    const string &getName() const {
        return name;
    }
};

class Car: public Vehicle {
public:
    // 调用基类的构造函数
    Car(const string &name) : Vehicle(name) {}
    void printInfo() {
        // 通过公有方法调用私有成员
        cout << "VehicleId: " << getVehicleId() << endl;
        cout << "VehicleName: " << getName() << endl;
    }
};

int main(){
    Car car = Car("car1");
    car.printInfo();
}
```
输出：
```
This is a Vehicle 1758608530
VehicleId: 1758608530
VehicleName: car1
```
除此之外，还有保护继承和私有继承。
- 公有继承（public）：当一个类派生自公有基类时，基类的公有成员也是派生类的公有成员，基类的保护成员也是派生类的保护成员，基类的私有成员不能直接被派生类访问，但是可以通过调用基类的公有和保护成员来访问。
- 保护继承（protected）： 当一个类派生自**保护**基类时，基类的**公有**和**保护**成员将成为派生类的**保护**成员。
- 私有继承（private）：当一个类派生自**私有**基类时，基类的**公有**和**保护**成员将成为派生类的**私有**成员。

![20190317143706.png](https://i.loli.net/2019/03/17/5c8deb169789f.png)
(From [here](https://www.geeksforgeeks.org/inheritance-in-c/))


值得一提的是`protected`保护成员，它的作用只有在继承关系中体现出来。比如上面的例子中，我们把name变为protected:
```cpp
protected:
    string name;
```
那么在基类中就可以直接访问，而不需要通过公有方法来访问。
```cpp
// cout << "VehicleName: " << getName() << endl;
cout << "VehicleName: " << name << endl;
```


C++ 支持多继承:
```cpp
class Car: public Vehicle, public FourWheeler { 
  
}; 
```

关于继承，还有一些重要的细节问题。你会发现只有构造函数可以带有`:`的参数列表初始化。因为这东西是为继承而准备的。

如果派生类在新建对象时没有在参数列表调用基类的构造函数，那么将自动调用基类的无参构造函数或默认构造函数。
```cpp
class A {
public:
    int a_;
    A(){
        cout << "A" << endl;
    }
    A(int a){
        a_ = a;
    }
};
class B : public A {
public:
    B(){
        cout << "B" << endl;
    }
};
int main(){
    B b = B();  
}
// A
// B
```
上面代码先输出A，再输出B，说明在进入B构造函数体之前，A就已经被初始化了。那么我们在B构造函数体中调用A的构造函数这是没有意义的，这不是在初始化基类A，而是在创建一个A对象。
```cpp
B() {
    A(3);
    cout << a_ << endl; // ???
}
```
那么如果我们想手动初始化A，那么就必须在进入B构造函数体之前，这便是参数列表的作用。
```cpp
B() : A(3) {
    cout << a_ << endl; // 3
}
```
同理，若想在一个构造函数里面调用同类的另一个构造函数，也要在参数列表里面调用。


### 6.1 虚继承
```cpp
class D{......};
class B: public D{......};
class A: public D{......};
class C: public B, public A{.....};
```
上面的继承关系中，C将会继承了两次D。为了排除这种问题，你需要这么干：
```cpp
A: virtual public D
B: virtual public D
```

## 7. 多态
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

## 9. class和struct的区别
- 使用 class 时，类中的成员默认都是 private 属性的；而使用 struct 时，结构体中的成员默认都是 public 属性的。
- class 继承默认是 private 继承，而 struct 继承默认是 public 继承。
- class 可以使用模板，而 struct 不能。


## 10. 类内与内外定义的函数
你可以在内外定义成员函数。
```cpp
class Animal {
public:
    void say();
};

void Animal::say() {
    cout << "hello" << endl;
}
```

那么类内定义和类外定义有什么区别呢？类内定义的函数会被编译器建议性地作为内联函数(inline)。当然，这只是建议性的，要是编译器认为函数的实现无法变为内联，它就会放弃转换。

类内定义的优点：
1. 方便

类外定义的优点：
1. 如果你的软件是闭源的，你可以在头文件中声明类，然后在源文件中实现，最后源文件打包成`.obj`文件，与头文件`.h`一起拿给他人，而他人将无法看到源文件中的实现细节。
2. 同一个声明，可以换上不同的定义（编译时选择不同的文件)。
3. 便于查找使用。在一个项目中，会涉及到许多函数，定义与声明分开可以更快的找到所需要的函数，同时定义可能需要成千上万行代码，而声明只需要几十行，这意味着等待代码载入的时间大大缩短了。




