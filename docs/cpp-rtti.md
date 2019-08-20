# C++ RTTI
RTTI是运行阶段类型识别（Runtime Type Identification）的简称。

RTTI可以让我们的程序在运行时识别对象的类型。下面举个例子：
```cpp
class Vehicle {
public:
    // 纯虚函数
    virtual void drive() = 0;
};

class Car: public Vehicle {
public:
    void drive() override {
        cout << "dirve car" << endl;
    }
};

class Bus: public Vehicle {
public:
    void drive() override {
        cout << "dirve bus" << endl;
    }
};

void checkType(Vehicle* v){
    // Is v Car or Bus ?
}

int main(){
    Vehicle* v1 = new Car();
    Vehicle* v2 = new Bus();
    checkType(v1);
    checkType(v2);
}
```

我们要想办法在`checkType()`函数中知道v到底是Car还是Bus。

这时候就要用到RTTI了。

## dynamic_cast
dynamic_cast运算符是最常用的RTTI组件，它可以将**基类型**转换为**派生类型**。前提是这个基类必须是抽象的，即带有`virtual`虚函数。

如果发现目标类型确实是其派生类型，则转换成功，并会返回派生类型的指针；若不是其派生类型，则转换失败，返回空指针。

```cpp
void checkType(Vehicle* v){
    Car* c;
    c = dynamic_cast<Car *>(v);
    if(c != nullptr){
        c->drive();
    }else{
        cout << "error" << endl;
    }
}

int main(){
    Vehicle* v1 = new Car();
    Vehicle* v2 = new Bus();
    checkType(v1);  // drive car
    checkType(v2);  // error
}
```

## typeid 和 type_info
typeid() 是一个函数，该函数返回了一个type_info类，通过`==`和`!=`比较两个type_info可以知道两个对象是不是同一类型。

```cpp
void checkType(Vehicle* v){
    if(typeid(*v) == typeid(Car)){
        cout << typeid(*v).name() << endl;  // 3Car
        cout << "It's Car" << endl;
    }
    if(typeid(*v) == typeid(Bus)){
        cout << typeid(*v).name() << endl;  // 3Bus
        cout << "It's Bus" << endl;
    }
}

int main(){
    Vehicle* v1 = new Car();
    Vehicle* v2 = new Bus();
    checkType(v1);  // It's Car
    checkType(v2);  // It's Bus
}
```
其中`name()`函数，会返回一个“名字”，这个名字在不同编译器中可能不同，但一般会包含类型的名称。

## 效率问题
有很多人讨论过dynamic_cast的性能问题，说它的性能很低，[stackoverflow](https://stackoverflow.com/questions/579887/how-expensive-is-rtti)上的有人做了测试，发现使用`typeid()`要比`dynamic_cast`快的多。

如果你真的在乎性能，那么在类型转换的时候你可以考虑这么干，先用`typeid()`判断类型，然后再强转。
```cpp
void checkType(Vehicle* v){
    Car* c;
    Bus* b;
    if(typeid(*v) == typeid(Car)){
         c = (Car*) v;
         c->drive();
    }
    if(typeid(*v) == typeid(Bus)){
        b = (Bus*) v;
        b->drive();
    }
}

int main(){
    Vehicle* v1 = new Car();
    Vehicle* v2 = new Bus();
    checkType(v1);  // drive car
    checkType(v2);  // drive bus
}
```

## 其他类型转换
C++提供了四种类型转换方法。
- dynamic_cast
- const_cast
- static_cast
- reinterpret_cast

dynamic_cast 前面说过了，下面介绍另外三种。

### const_cast
const_cast可以将一个指针类型在const和非const，volatile和非volatile之间转换。
```cpp
class City {
    int code;
public:
    City(int code) : code(code) {}
    void setCode(int code) { City::code = code; }
    int getCode() const { return code; }
};

int main(){
    const City* c1 = new City(1);
    // c1->setCode(3); // invaild

    City* c2 = const_cast<City *>(c1);
    c2->setCode(4);
    cout << c2->getCode() << endl;  // 4

    const City* c3 = const_cast<const City*>(c2);
}
```

注意 const_cast 只能修改指向某个值的**指针**。如果指针所指向的值是const，便**可能**无法修改。
```cpp
const int t = 10;
const int* tpc = &t;
int* tp = const_cast<int*>(tpc);
cout << *tp << endl;    // 10
*tp = 99;
cout << *tp << endl;    // 99
cout << t << endl;      // 10
```
上面的例子中，我们产生通过把const指针转换为非const指针来修改const int中的值。书上以及网上都说这是一种`Undefined Behavior`。我自己跑出来的结果相当诡异，从结果上看当`*tp`被修改后，就不再指向`t`了，因为打印出了不同的结果。不管如何，还是不要这么干。

### static_cast
凡是类型之间能够被隐式转换的都能够使用 static_cast 进行转换。比如基类和派生类指针，以及后面会讲到的引用类型。
```cpp
Car* car = new Car();
Vehicle* ve = new Vehicle();

Vehicle* ve2 = static_cast<Vehicle*>(car);
ve2->drive();   // drive car

Car* car2 = static_cast<Car*>(ve);
car2->drive();  // drive vehicle
```

### reinterpret_cast
reinterpret_cast 专门用来处理一些危险的类型转换操作。比如将一个long类型转换为一个结构体类型，结构体访问成员时将按照字节顺序访问。

## 强制转换
强制转换，是两个没有继承关系又不在继承体系中的两个类对象之间的转换。为什么能够强制转换这样两个完全不同的类型呢？当然，这是有条件的。

首先，在C++中，如果不存在继承关系这种隐式转换，你不能够将一个种类型空间的对象复制出另一种类型空间的对象。举个例子，
```
class A {
public:
    int a_;
    A(int a): a_(a) {};
    void say() {
        cout << "A" << endl;
    }
};

class B {
public:
    int a_;
    int b_;
    B(int a, int b): a_(a), b_(b) {};
    void say() {
        cout << "B" << endl;
    }
};

int main() {
    A a = B(1, 2); // error
    B b(1, 2);
    A a = (A) b; // error
}
```


但是，我们却可以使用引用和指针类型。
```cpp
int main() {
    B b(99, 1);
    
    A* ap = (A*) &b;
    cout << ap->a_ << endl; // 99
    ap->say(); // A
    
    A& a = (A&) b;
    cout << a.a_ << endl; // 99
    a.say(); // A
}
```
上面的例子，我们强转成了其他类型的引用和指针。

在C++看来，对于引用类型和指针类型的强制转换是允许的，或者说在内存模型上是可行的。因为引用和指针变量并没有真正产生一片内存，它们只是引用了或指向了某一块内存空间。上面的例子中，虽然类型不同，但却与原变量使用同一片内存空间。

至于函数还是原类型中的函数，是因为成员函数是通过类型索引的，不存在于变量的内存空间中。

