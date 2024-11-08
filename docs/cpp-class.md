# C++ 类

## 1. 构造函数
### 1.1 无参构造
如果你没有自己定义构造函数，就会调用**默认构造函数**。
```cpp
Stock first; //1 隐式
Stock first{}; //2  显式
Stock first = Stock(); //3 显式
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
对象过期时，程序将自动调用**析构函数(Destructor)**。

- 如果创建的是自动存储类对象（没有使用new），则其析构函数将在程序执行完代码块时自动被调用。
- 如果对象是通过new创建的，则它将驻留在栈内存或自由存储区中，当使用delete来释放内存时，其析构函数将自动被调用。
- 如果程序员没有提供析构函数，编译器将隐式地声明一个默认析构函数，并在发现导致对象被删除的代码后，提供默认析构函数的调用。

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

假设我们把一个已有变量通过函数返回出去，从函数返回变量时，将调用该变量的拷贝构造函数，创建出一个临时变量，而当你用一个新定义的变量来接收时，新变量会把这片临时变量的空间作为它自己的空间，即为**初始化**。
```cpp
Stock getStock(){
    Stock s{"inner", 122};
    return s; // 将调用一次拷贝构造函数
}

int main(){
    Stock ss = getStock();
}
```
事实上，这里的表现和上面直接调用构造函数初始化是统一的，因为调用构造函数本身便产生了一个临时变量。

如果不是从一个已有变量返回，而是直接返回临时变量，则可避免调用拷贝构造函数创建一个新的临时变量。
```cpp
Stock getStock(){
    return Stock s{"inner", 122};
}
```
当然，在C++的编程习惯里，为了避免从函数中获取对象时触发一次多余的拷贝，一般会先在外部构造好对象，然后传入内部设值。
```cpp
void getStock(Stock& s){
    s.setName("inner");
    s.setShares(122);
}

Stock s;
getStock(s);
```

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

这时候，你设法获取s1的数据时就会可能出错，因为s1只是临时对象的浅拷贝，等离开作用域使临时对象销毁后，s1中的指针变量可能指向的是已经被回收的内存空间。

当然，如果是使用new来新建对象不会出现上面的问题，因为动态变量不会被自动销毁。

## 4. 枚举类
```cpp
enum class Color { red, blue, green };
enum class Trafflc_light { green, yellow, red };
Color color = Color::red;
```

## 5. 继承
在C++的继承关系中，父类被称为基类，子类被称为派生类。派生类可以继承基类除构造函数和析构函数的所有成员。对于公有继承，在派生类后面加上`: public <Base>`来继承`<Base>`类。
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
- **公有继承**（public）：当一个类派生自公有基类时，基类的公有成员也是派生类的公有成员，基类的保护成员也是派生类的保护成员，基类的私有成员不能直接被派生类访问，但是可以通过调用基类的公有和保护成员来访问。
- **保护继承**（protected）： 当一个类派生自**保护**基类时，基类的**公有**和**保护**成员将成为派生类的**保护**成员。
- **私有继承**（private）：当一个类派生自**私有**基类时，基类的**公有**和**保护**成员将成为派生类的**私有**成员。

总结：基类的私有属性不能被派生类直接访问，除此之外，派生类以什么权限继承基类，那么基类中高于该权限的属性就会被降级到该权限。

![](https://i.loli.net/2019/03/17/5c8deb169789f.png)

(From [here](https://www.geeksforgeeks.org/inheritance-in-c/))

在class name后面标注final关键字，可以使class不可被继承，
```cpp
class Data final {};
```

### 5.1 protected 成员
值得一提的是`protected`保护成员，它的作用只有在继承关系中体现出来。比如上面的例子中，我们把name变为protected:
```cpp
protected:
    string name;
```
那么在派生类中就可以直接访问，而不需要通过公有方法来访问。
```cpp
// cout << "VehicleName: " << getName() << endl;
cout << "VehicleName: " << name << endl;
```

### 5.2 初始化参数列表
关于继承，还有一些重要的细节问题。你会发现只有构造函数可以带有`:`的参数列表初始化。

如果派生类在新建对象时没有在参数列表调用基类的构造函数，那么将自动调用基类的无参构造函数或默认构造函数。
```cpp
class A {
public:
    int a_;
    A(){
        cout << "A" << endl;
    }
    A(int a): a_(a) {
        
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

另外，使用参数列表初始化成员属性还有一个好处就是直接 定义+初始化。
```cpp
int a_;
// #1
A(int a) {
    a_ = a;
}
// #2
A(int a): a_(a) {
}
```
采用`#1`的代码的执行顺序是
```cpp
int a_; a_ = a;
```
采用`#2`的代码的执行顺序是
```cpp
int a_ = a;
```
如果只是基本类型，可能没什么区别，但如果是一个`a`内嵌对象成员，按照`#1`的逻辑，**定义的时候将隐式调用无参构造函数进行初始化**，然后在构造函数中再调用赋值运算符进行赋值。所以应该尽可能使用`#2`的写法。

## 6. 多继承
C++ 支持多继承:
```cpp
class Car: public Vehicle, public FourWheeler { 
  
}; 
```

### 6.1 构造和析构的顺序

对于构造函数：
- 如果发现该类继承了某个基类，一定是优先初始化其基类。
- 如果继承了多个基类，那么执行顺序取决于声明派生类时所指定的各个基类的顺序（从左到右）

而析构函数的执行顺序与构造函数执行顺序相反，即后初始化的一定会先被析构。


## 7. 虚继承
既然存在多继承，那么就可能存在一个类被多个类继承。当继承路径从一个起点类出发，并产生相交于某一个类时，交点类会被初始化两次，即产生两个实例。
```cpp
class A {
    int a_;
public:
    A(int a): a_(a) {}
    int getA(){
        return a_;
    }
};

class B: public A {
    int b_;
public:
    B(int a, int b): A(a), b_(b) {}
};

class C: public A {
    int c_;
public:
    C(int a, int c): A(a), c_(c) {}
};

class D: public B, public C {
    int d_;
public:
    D(int a, int b, int c, int d): 
    // 此处对a值做不同的处理
    B(a + 100, b), C(a - 100, c), d_(d) {}
};


int main(){
    D d = D(1, 2, 3, 4);
//    d.getA();   // error
    cout << d.B::getA() << endl; // 101
    cout << d.C::getA() << endl; // -99
}
```
而且起点类无法调用交点类的方法，两份实例意味着两个同名方法，程序不知道该调用哪个。


这时候使用虚继承可以使得该交点类只保留一个实例，我们称该交点类为**虚基类**。虚继承需要声明在交点类的直接派生类中，虚继承的声明方法是`: virtual public <Base>`。虚继承后，起点类将优先初始化虚基类。
```cpp
class A {
    int a_;
public:
    A(int a): a_(a) {}
    int getA(){
        return a_;
    }
};

class B: virtual public A {
    int b_;
public:
    B(int a, int b): A(a), b_(b) {}
};

class C: virtual public A {
    int c_;
public:
    C(int a, int c): A(a), c_(c) {}
};

class D: public B, public C {
    int d_;
public:
    D(int a, int b, int c, int d):
    // 发现虚基类A，则此处必须先初始化A，否则会报错
    A(a), B(a + 100, b), C(a - 100, c), d_(d) {}
};


int main(){
    D d = D(1, 2, 3, 4);
    cout << d.getA() << endl; // 1
    cout << d.B::getA() << endl; // 1
    cout << d.C::getA() << endl; // 1
}
```

![](img/virtual-inherit.jpg)

## 8. class和struct的区别
- 使用 class 时，类中的成员默认都是 private 属性的；而使用 struct 时，结构体中的成员默认都是 public 属性的。
- class 继承默认是 private 继承，而 struct 继承默认是 public 继承。
- class 可以使用模板，而 struct 不能。


## 9. 类内与内外定义的函数
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


## 10. static 成员
定义静态成员属性的好处是可以在同类的多个对象之间实现数据共享。而静态成员方法的好处是可以直接通过类名调用，不需要实例化。当然，静态成员属性也可以通过类名访问。

静态成员属性**必须**定义在“类声明之后，定义对象之前”，即必须在类外定义，类里面写的static只是一个**声明**而已。
```cpp
class G {
public:
    static int g;
    static int getG(){
        return g;
    }
};

int G::g = 10;

int main(){
    cout << G::getG() << endl; // 10
}
```
```cpp
int G::g; // 只定义不初始化也可以

int main(){
    G g = G();
    G::g = 100;
    cout << G::getG() << endl; // 100
```

## 11 const 对象

### 11.1 const 引用
我们一般很喜欢把函数参数写作引用，来减少拷贝。但你不能传入一个临时值。
```cpp
class A {
};

class C {
public:
    void output(A& v) {
    }
};

int main() {
    C c;
    c.output(A()); // error
}
```

解决办法是有的：
1. 重载函数，函数参数声明为`const`，一个常引用类型可以引用一个常数。
```cpp
void output(const A& v) {
}
```
2. 重载函数，使用[右值引用](cpp-move.md#right)。
```cpp
void output(A&& v) {
}
```

### 11.2 const 函数
我们可以标注一个函数为 const 。
```cpp
void say() const;
```
一个const的成员函数是不能够修改非静态的成员属性的。默认传入函数内部的 this 指针也将变成 const 类型。
```cpp
class A {
public:
    int v1;
    static int v2;
    void say() const {
        v1 = 1; // error
        v2 = 1;
        cout << "hello" << endl;
    }
};
```

一个 const 的对象只能调用其 const 的成员函数。
```cpp
class A {
public:
    void say() const {
        cout << "hello" << endl;
    }
    void eat() {
        cout << "eat" << endl;
    }
};

int main() {
    const A a;
    a.say();
    a.eat(); // error
    return 0;
}
```
这个特性告诉我们，**在写成员函数时最好正确标注 const 函数**，因为我们不知道使用这个类的人会不会去创建一个const实例。

还有另一个问题，那就是函数参数应不应该使用const？
```cpp
class B{
    void sayByA(const A& a) {
        a.say();  // say() 必须是const的
    }
};
```
比如上面这个例子，因为A是我自己写的，我很清楚 a.say() 是const函数，我可以放心的把参数标上const。

但万一有时候我们把外部代码的类作为参数，又要调用其中的某个方法，虽然知道方法确实不会修改成员属性，但我们也不确定作者是否有标注为const函数。

有个比较好的方案是，自己写的代码自己清楚，该标注const都标上。别人的代码不清楚，则看看使用对象时有没有调用函数，若是不需要调用函数，则标上const，若是需要调用函数，则不标注。当然使用IDE的话，可以直接跑到头文件看源码里的声明。

## 12 隐式转换
C++ Class在构造时具有一种隐式转换，如果Class具有单个参数的构造函数，那么构造时你可以通过把该类型的变量赋值给对象的形式进行构造。
```cpp
class Person {
public:
    Person(string name): name(name) {}
    string name;
};

void say(Person p) {
    cout << p.name << endl;
}

int main() {
    Person p = string("John"); // implicit conversion
    say(string("John"));    // John
}
```
这种转换看起来很让人匪夷所思，不利于理解，而且还可能与operator=()造成冲突，如果你定义了同样参数的operator=()，那么总是会使用构造函数。

解决办法是加上在构造函数前加上explicit关键字，告诉编译器这个构造函数不允许隐式转换。
```cpp
class Person {
public:
    explicit Person(string name): name(name) {}
    string name;
};
```