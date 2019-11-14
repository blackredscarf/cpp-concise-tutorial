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

typedef int max_func(int& a, int& b);
max_func* max_ptr = (max_func*) &max;

int main() {
    (say_hello_ptr)();
    int a = 10, b = 12;
    cout << (max_ptr)(a, b) << endl; // 12
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
    // 指向虚函数表地址的指针
    long* vt_addr = (long*) *(p_addr + 0);
    // 虚函数地址指针 开头指向 第一个虚函数地址
    void* say_addr = (void*) *(vt_addr + 0);

    // 地址转换为函数指针
    void(*say_ptr)() = (void(*)()) say_addr;
    (*say_ptr)(); // hello!
}
```
使用`long*`是因为64位系统地址需要使用long存储，如果是32位，就用`int*`。上面使用的`void*`可以接收任何类型的指针。

### 虚函数表
每一个具有虚函数的类对象，对象的内存首地址都会维护一个vtptr指针，该指针指向一个虚函数表。那么这和多态有什么关系呢？

先下结论：**同一类型的对象实例会共同维护一张虚函数表，这里的对象实例也包括继承中的基类实例**。当存在继承关系，派生类实例的虚函数表是基类实例的拷贝，如果派生类新增虚函数，则在自己的虚函数表中添加一条记录；如果派生类覆盖了基类某个虚函数，则会给自己的虚函数表中对应的虚函数更换一个新地址，而不会影响基类虚函数表上的地址。

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
	Devired(int mem = 3) : dmen1_(mem) { ; }

	// 覆盖基类的虚函数
	void bfunc2() override { std::cout << "In Devired bfunc2()" << std::endl; }
	// 派生类的虚函数
	virtual void dfunc1() { std::cout << "In Devired dfunc1()" << std::endl; }
	// 非虚函数
	void ndfunc1() { std::cout << "In Devired ndfunc1" << std::endl; }
private:
	int dmen1_;
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
	printf("vtptr: 0x%08x\n", vtptr);

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

	printf("bmem1: %d\nbmem2: %d\ndmem1: %d\n\n", bmem1, bmem2, dmem1);
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
```cpp
vtptr: 0x00179b34
In bfunc1()
In bfunc2()
In bfunc3()

         bfunc1 addr: 0x00171028
         bfunc2 addr: 0x00171366
         bfunc3 addr: 0x00171195

bmem1_: 1
bmen2_: 2

----------------
vtptr: 0x00179b78
In bfunc1()
In Devired bfunc2()
In bfunc3()
In Devired dfunc1()

         bfunc1 addr: 0x00171028
         bfunc2 addr: 0x00171375
         bfunc3 addr: 0x00171195
         dfunc1 addr: 0x001711e5

bmem1: 1
bmem2: 2
dmem1: 3
```
你会发现，派生类没有覆盖`bfunc1`和`bfunc2`虚函数，所以它们打印出的地址是一样的。而派生类覆盖了`bfunc2`，导致派生类的虚函数表的`bfunc2`的地址被更新，所以它与基类中的地址不同。

实验2，我们把main函数改成如下：
```cpp
int main() {
    watch_devired();
    cout << "----------------" << endl;
    watch_devired();
    
    system("pause");
    return 0;
}
```
打印两个派生类的实例，最终结果两者输出完全一样，包括虚函数表的地址。说明，同一类型的不同实例使用的是同一张虚函数表。

实验3，尝试用基类引用派生类。
```cpp
// 观察基类引用派生类
void watch_base_ref_revired() {
    Devired d;
    Base &b = d;
    long *dAddress = (long*)&b;
    
    // ...
}

int main() {
    watch_base_ref_revired();
    cout << "----------------" << endl;
    watch_devired();
    
    system("pause");
    return 0;
}
```
最终结果，两次输出的虚函数表地址相同。说明尽管类型变成了基类，但虚函数表仍然是派生类的。

## 构造函数，析构函数和虚函数

**首先，思考构造函数和析构函数能不能是虚函数？**
构造函数不能是虚函数。有些人会说构造函数调用时，内存中指向虚函数表的指针还没被初始化，这是句话是对的，但不是构造函数不能是虚函数的原因，尽管指向虚函数表的指针还没被初始化，但函数可以作为成员函数来调用，下面会说到这个。正确原因是调用构造函数时，它不知道自己的基类还是派生类，还是继承树中的一个中间节点，它应该调用自己的构造函数还是派生类的构造函数呢？若多个派生类都实现了他的构造函数，他应该调用哪个？会产生冲突，所以不能。[[ref]](
https://www.zhihu.com/question/35632207/answer/63936329)

当类中含有虚函数，则析构函数总应该声明为虚函数，因为当用基类对象接收派生类后，基类对象回收时需要调用派生类实现的析构函数。

**那构造函数和析构函数能不能调用虚函数呢？**
答案是可以调用，但最好不要。构造函数调用时，指向虚函数表的指针还没被初始化，虽然确实可以调用，但只是作为成员函数，如果存在继承关系，这时派生类还没有构造，它不会调用到派生类的实现版本；

而对于析构函数，派生类析构后，基类调用的虚函数到底是自己的还是派生类的呢？若是调用基类实现则与运行时的版本不同，若调用派生类的，可能会访问到已回收的属性，所以最好不要这么干。

## 参考
- [深入分析C++虚函数表](https://jocent.me/2017/08/07/virtual-table.html)
