# C++对象的内存模型

## 函数指针
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

## 虚函数表
虽然最终结果相同，但不同编译器对多态的实现可能不同。一般通过虚函数表(virtual function table)实现。只要类中定义了虚函数，编译器自动添加隐藏指针vfptr指向虚函数表。指针vfptr通常在对象内存的首地址，该指针指向一个虚函数表。那么这和多态有什么关系呢？

先下结论：**同一类型的对象实例会共同维护一张虚函数表，这里的对象实例也包括继承中的基类实例**。当存在继承关系，派生类实例的虚函数表是基类实例的拷贝，如果派生类新增虚函数，则在自己的虚函数表中添加一条记录；如果派生类覆盖了基类某个虚函数，则会给自己的虚函数表中对应的虚函数更换一个新地址，而不会影响基类虚函数表上的地址。

上面的理论说明了，尽管你用基类引用一个派生类，虚函数还是会使用派生类的而不是基类的，是因为当调用虚函数时，就会通过对象内存首地址访问虚函数表，并查找你要调用的虚函数。尽管类型声明是基类，但访问的仍然是派生类的虚函数表。而当你通过基类实例调用方法`Derived.Base::function()`，那么就会访问基类的虚函数表，从而调用基类的实现。

类实例与虚函数表之间的关系如图所示，

![](https://s21.ax1x.com/2024/11/03/pAr88r4.png)

类实例的内存的第一个位置是一个vptr指针，指向虚函数表的第一个虚函数。注意到第一个虚函数的前面还有两个东西：

- offset表示类实例指针与虚函数指针之间的偏移，单继承时总是0，即第一个位置就是虚函数表的指针，而多继承时会有多个虚函数指针，所以会有不同的偏移值。
- type info ptr是一个指向type info数据的指针，一般是用来存类的反射信息。

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

class Derived : public Base {
public:
	// 默认值3
	Derived(int mem = 3) : dmen1_(mem) { ; }

	// 覆盖基类的虚函数
	void bfunc2() override { std::cout << "In Derived bfunc2()" << std::endl; }
	// 派生类的虚函数
	virtual void dfunc1() { std::cout << "In Derived dfunc1()" << std::endl; }
	// 非虚函数
	void ndfunc1() { std::cout << "In Derived ndfunc1" << std::endl; }
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

	// 对象b的Offset和pTypeInfo
	void* pOffset = (void*) *(vtptr - 2);
	void* pTypeInfo = (void*) *(vtptr - 1);

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
void watch_Derived() {
	Derived d;
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
    watch_Derived();

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
In Derived bfunc2()
In bfunc3()
In Derived dfunc1()

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
    watch_Derived();
    cout << "----------------" << endl;
    watch_Derived();
    
    system("pause");
    return 0;
}
```
打印两个派生类的实例，最终结果两者输出完全一样，包括虚函数表的地址。说明，同一类型的不同实例使用的是同一张虚函数表。

实验3，尝试用基类引用派生类。
```cpp
// 观察基类引用派生类
void watch_base_ref_derived() {
    Derived d;
    Base &b = d;
    long *dAddress = (long*)&b;
    
    // ...
}

int main() {
    watch_base_ref_derived();
    cout << "----------------" << endl;
    watch_Derived();
    
    system("pause");
    return 0;
}
```
最终结果，两次输出的虚函数表地址相同。说明尽管类型变成了基类，但虚函数表仍然是派生类的。

## 类的内存模型
结合内存对齐和虚函数表，探查一个类或对象到底会存什么数据到内存里。
```cpp
class B {
public:
    // 8
    char b;   // 2
    short c;  // 2
    int a;    // 4
    static int* p; // 0

    B() {};
    void test0(int x) {};
    virtual void test1() {}     // virtual
    virtual void test2() {}     // virtual
};

int main() {
    cout << sizeof(B) << endl; // 16
    return 0;
}
```
上面的class B会占用16字节的内存。首先是带有虚函数的类的头部会带有一个虚函数指针，在64为系统下占8字节；然后a, b, c三个变量经过内存对齐会占用8字节；剩下的一个静态指针类型不会占用类的内存大小，因为静态变量会存在全局变量区，这也是它能够被所有对象实例共享的理由，静态变量的地址视作全局变量地址维护，类对象不需要花费重复空间来存这个地址，

而非虚函数的成员函数和全局函数都会存放在进程共享的函数表里，不需要放类对象里面；而虚基类对象调用哪个虚函数实现是在运行时才确定的，所以要存到类对象里面去。

不管是虚还是非虚函数，成员函数会被编译成全局函数的模样，成员函数编译成全局函数后，其第一个参数会是该类对应的类型。

比方说，上面的B::test1()函数可能会在预处理阶段生成下面这样的全局函数，
```cpp
void B_test0(B* b, int x) {

}
```
调用部分也会被编译，
```cpp
B* b = new B();
b->test0(1);    // 编译前
B_test0(b, 1);  // 编译后
```

## 命令行打印信息
假设我们的类继承关系是这样的，
```cpp
class MyBase {
public:
    virtual void bfunc1() {}
    virtual void bfunc2() {}
    virtual ~MyBase() {}
};

class MyDerived : public MyBase {
public:
    void bfunc2() override {}
    virtual void dfunc1() {}
};
```
使用命令行，
```
g++ -O0 -std=c++11 -fdump-lang-class -fsanitize=address main.cpp
```
会导出一个文本文件，里面有类的内存模型的分析文本，
```cpp
Vtable for MyBase
MyBase::_ZTV6MyBase: 6 entries
0     (int (*)(...))0
8     (int (*)(...))(& _ZTI6MyBase)
16    (int (*)(...))MyBase::bfunc1
24    (int (*)(...))MyBase::bfunc2
32    (int (*)(...))MyBase::~MyBase
40    (int (*)(...))MyBase::~MyBase

Class MyBase
   size=8 align=8
   base size=8 base align=8
MyBase (0x0x68c4ba0) 0 nearly-empty
    vptr=((& MyBase::_ZTV6MyBase) + 16)
```

- size=8 表示类的大小是8字节，因为它只有一个虚函数指针
- align=8 表示内存对齐的大小
- vptr=(XXX + 16) 这里的+16表示指针偏移16字节，指向虚函数表的第一个虚函数
- Vtable里面帮我们生成了两个析构函数，第一个是手动调用`obj->~MyBase()`时调用的，第二个是调用`delete obj`时调用的

派生类的分析如下，明显看到Vtable里面包含了MyBase类的虚函数，bfunc2指向了MyDerived的实现版本，
```cpp
Vtable for MyDerived
MyDerived::_ZTV9MyDerived: 7 entries
0     (int (*)(...))0
8     (int (*)(...))(& _ZTI9MyDerived)
16    (int (*)(...))MyBase::bfunc1
24    (int (*)(...))MyDerived::bfunc2
32    (int (*)(...))MyDerived::~MyDerived
40    (int (*)(...))MyDerived::~MyDerived
48    (int (*)(...))MyDerived::dfunc1

Class MyDerived
   size=8 align=8
   base size=8 base align=8
MyDerived (0x0x6677f70) 0 nearly-empty
    vptr=((& MyDerived::_ZTV9MyDerived) + 16)
  MyBase (0x0x68ee2a0) 0 nearly-empty
      primary-for MyDerived (0x0x6677f70)
```

## 虚基类的析构函数
通过上面的打印信息发现一个细节，MyDerived明明没有定义析构函数，但还是帮我们生成了虚析构函数，而且打印位置还在`dfunc1`的前面。说明这个析构函数是从基类里面来的，当基类虚函数表复制过来的时候，发现有析构函数，就自动将其替换为了派生类的析构函数。

这可能也与虚基类的析构函数必须是虚函数有关，也与虚函数在机器码中的调用方式有关。虚函数调用被编译为机器码时，是通过对象的地址偏移调用的。当我们有一个基类类型的多态对象被delete时，如果你没有把基类的析构函数定义为虚函数，则会被编译为调用其非虚函数的析构函数，因为这个变量类型是基类类型，而多态是运行时特性，编译器无法判断这个对象是否是多态的，只能根据其类型来判断。

而如果你把析构函数定义为虚函数，则会被编译为通过从地址偏移来调用 [[ref]](https://www.cnblogs.com/chio/archive/2007/09/07/886337.html)。呈现的结果就是从运行时的虚函数表里找到析构函数的地址，然后将其调用。从上面的打印信息可以看出，基类和派生类的虚函数表里呈现覆盖关系的函数的偏移都是相同的，当一个多态的基类类型对象调用析构函数时通过偏移找到的将是派生类的析构函数。

## 多继承

```cpp
class MyBase {
public:
    virtual void bfunc1() {}
    virtual ~MyBase() {}
    int dataBase;  // 这里定义了一个int
};

class MyDerived {
public:
    virtual void dfunc1() {}
    int dataDerived; // 这里定义了另一个int
};

class MySub : public MyBase, MyDerived {
public:
    void bfunc1() override {}
    virtual void sfunc1() {}
};
```
我只贴MySub类的打印信息，
```cpp
Vtable for MySub
MySub::_ZTV5MySub: 9 entries
0     (int (*)(...))0
8     (int (*)(...))(& _ZTI5MySub)
16    (int (*)(...))MySub::bfunc1
24    (int (*)(...))MySub::~MySub
32    (int (*)(...))MySub::~MySub
40    (int (*)(...))MySub::sfunc1
48    (int (*)(...))-16
56    (int (*)(...))(& _ZTI5MySub)
64    (int (*)(...))MyDerived::dfunc1

Class MySub
   size=32 align=8
   base size=28 base align=8
MySub (0x0x720da10) 0
    vptr=((& MySub::_ZTV5MySub) + 16)
  MyBase (0x0x7547300) 0
      primary-for MySub (0x0x720da10)
  MyDerived (0x0x7547360) 16
      vptr=((& MySub::_ZTV5MySub) + 64)
```

- 可以发现有两个vptr，一个偏移16，一个偏移了64。对照Vtable，第一个偏移16就是指向MyBase的第一个虚函数，第二个偏移64指向的是MyDerived的第一个虚函数。
- MyDerived::dfunc1的前面两个东西已经说过了，其中offset=-16，其实是指MyDerived的虚函数表指针相对于MySub对象的指针偏移了16。为什么偏移了16，下面讲。
- size=32，构成是 vptr MyBase（8字节） + MyBase.dataBase（4字节整型，但内存对齐到8字节） + vptr MyDerived（8字节） + MyDerived.dataDerived（对齐到8字节）。其中vptr MyDerived相对于首部的偏移为16。

## 引用

- [C++对象模型](https://miaoerduo.com/2023/01/19/cpp-object-model/)
- 《Inside the C++ Object Model》