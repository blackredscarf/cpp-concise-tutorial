# C++ 变量

## 0. 基本数据类型
C++ 基本数据类型如下：

![var.jpg](https://i.loli.net/2019/07/04/5d1df52f7ed3c86799.jpg)

一个字节的长度是8bit，如果是全部用来存数字的无符号类型，最大值是8个位全是1，即2的8次方减一，可表示`0~255`。而有符号类型则需要消耗一个比特位，即`-128~127`，至于为什么负数可以表示-128，详见后面章节。如果你只想表示正数，那么无符号位类型会比有符号类型可表示的数大一倍。

## 1. 初始化
C++11提供了大括号`{}`的初始化方式。这是一种完全通用的初始化方式，你可以用来初始化任何类型，包括自定义对象。还有一个特点就是自带类型检查。以往，你可能很容易不小心将一个int赋值给char类型，编译器还不会报错，但如果用`{}`就没关系了。
```cpp
int a {}; // 0
int a1{3};
double b{3.0};
char s{'s'};
char s1{123}; // error
float f[3] {1.0, 2.0, 3.0};
float f1[3] {}; // all elements set to 0
```

对于基本类型，不进行主动初始化是一个未定义行为，不同运行时可能有不同的默认值，主动赋一个默认值是个好习惯。
```cpp 
int a;  // undefine
```

## 2. 字节数组
字节数组形式的字符串：
```cpp
char bird[5] = "bird";  // 预留一个 \0
char bird2[100] = "bird"; // 剩下空间全是 \0
char fish[] = "fish"; // 自动计算
```

## 3. string
string是c++中相当方便的字符串对象。默认采用**动态内存**分配。
```cpp
std::string str1 {"hello"};
std::string str2 {"world"};
std::string str3 = str1 + " " + str2;
int sz = str1.size();
```

### 3.1 raw字符串
为了不解析`\n`, `"`等特殊字符，使用`()`括起来，然后前面加一个`R`即可。
```cpp
std::cout << R"("hello" \n world)" << std::endl;
```
如果字符串中本身就有括号，那么就这么写：
```cpp
std::cout << R"+*(("hello") \n world)+*" << std::endl;
```

但你会发现，这东西没办法传进去一个变量。从某种意义上讲，关于raw字符串的处理还不是很完善，除此之外的方法就是遍历字符串，然后对所有特殊字符都加上一个`\`，比如`\\n`。

## 4. volatile
关键字volatile表明，即使程序代码没有对内存单元进行修改，其值也可能发生变化。你会觉得奇怪，问什么程序没有修改它，他还会变？

可以将一个指针指向某个硬件位置，其中包含了来自串行端口的时间或信息。在这种情况下，硬件（而不是程序）可能修改其中的内容。或者两个程序可能互相影响，共享数据。[[ref]](https://www.runoob.com/w3cnote/c-volatile-keyword.html)

编译器通常可能对某个变量进行优化，比如变量值**缓存在寄存器中**，程序在几条语句中多次使用了某个变量的值，则可能会直接使用寄存器中的值。这种优化假设变量的值不会被当前程序之外的程序进行修改。

正如前面所说的，改变这个值的可能不是程序本身，所以可以将变量声明为volatile，相当于告诉编译器，不要进行这种优化。

## 5. mutable
可以用它来指出，即使结构或类变量为const，其某个成员也可以被修改。编译器可以在实际定义之后进行再链接
```cpp
struct data {
    mutable int access;
};
const data d {0};
d.access = 1;
```

## 6. 链接性和持续性

这里涉及的知识点是链接性(linkage)，当使用extern声明，变量将变为外部链接性。<span id="linkage"></span>

![](https://i.loli.net/2019/03/15/5c8ba1721f258.png)


函数的链接性呢？C++不允许在一个函数中定义另外一个函数，因此所有函数的存储持续性都自动为**静态**的，即在整个程序执行期间都一直存在；并且链接性是**外部**的，即可以在文件间共享，所以上面例子中我们不需要对一个函数声明`extern`。

## 7. static
上面图说明虽然静态变量static在整个程序执行期间都存在，但它意味着在
1. 函数的外面使用关键字static定义的变量的作用域为整个文件，但是不能用于其他文件（内部链接性）。
2. 在代码块中使用关键字static定义的变量被限制在该代码块内（局部作用域、无链接性）。
3. 如果定义在[类](cpp-class.md)内部，则可在同类的多个对象之间实现数据共享。

需要明白，static变量不同于自动变量的是，虽然两者都需要在作用域范围内才能使用，但自动变量脱离作用域后便被回收，而static则是直到程序停止运行后才被回收。

### 7.1 局部静态
局部静态(local static)具有两个特性：
1. 只执行一次
2. 线程安全(C++11)

```cpp
struct Data { int x; };

void incrementAndPrint(){
    static Data data{ 1 };  // 只执行一次
    data.x += 1;
    std::cout << data.x << '\n';
}   // data不会销毁，但作用域仍然在函数内

int main() {
    incrementAndPrint();    // 2
    incrementAndPrint();    // 3
    incrementAndPrint();    // 4
    return 0;
}
```

静态变量会存到全局存储区，既不是栈也不是堆。据此，你可以推理出静态变量不会随离开函数作用域而被销毁，并且可以通过引用传递到作用域外部访问。
```cpp
static int& test() {
    static int s = 2;
    cout << s << endl;
    return s;
}

int main() {
    static int& v = test(); // 2
    v++;
    test(); // 3
    return 0;
}
```

## 8. extern
关键字extern用于声明一个全局变量。你可以预先声明一个全局变量，相当于作为一个占位符，它允许你延迟定义一个变量。
```cpp
int main(){
    {
        extern int gg;
        std::cout << gg << std::endl; // 9
    }
}

int gg = 9;
```
上面的例子gg的定义在main函数的后面，理论上，是不能在main里面使用gg的，但我们可以使用通过`extern int gg`声明gg是一个全局变量。除此之外，你还能用extern预声明一个全局函数。
```cpp
int main() {
    extern int test();
    std::cout << test() << std::endl;
    return 0;
}

int test() {
    return 1;
}
```

再举个例子，我在file1.cpp 定义了一个变量`x`，而file2.cpp中想访问到这个变量，我们该怎么办？在python中直接使用import模块即可，但c++中你只能include一个`.h`文件。

所以我们需要以`.h`文件作为中间人，在其中用`extern`**声明**该变量，使得它成为全局变量。注意这里仅仅是声明，而不是定义，说明并未分配存储空间。
```cpp
#ifndef HELLOWORLD_CH06_H
#define HELLOWORLD_CH06_H

extern int global_x;
void get_global_x();

#endif //HELLOWORLD_CH06_H
```
然后在`.cpp`中**定义**该变量。
```cpp
#include "ch06.h"
#include <iostream>

// 定义global_x
int global_x = 6;

void get_global_x() {
    std::cout << global_x << std::endl;
}
```
在第二个`.cpp`中使用该变量：
```cpp
#include <iostream>
#include "ch06.h"
// 声明global_x （这一步可以省略）
extern int global_x;

int main(){
    std::cout << global_x << std::endl; // 6
    global_x = 10;
    get_global_x(); // 10
}
```
你可以直接修改该全局变量，程序任意一个地方都能访问到修改后的值。全局变量存储在独立的内存空间中，既不在堆也不在栈。

### 8.1 全局变量
一般来说，如果不使用extern，我们不应该直接在头文件里定义全局变量。正常来说，函数不写函数体就相当于声明，但不用extern时，变量会默认进行的定义和初始化。其实extern就是为声明变量服务的。
```cpp
// def.h
#ifndef DEF
#define DEF

int a = 1;  // don't do this

#endif
```
```cpp
// def.cpp
#include "def.h"    // 引入时定义了一次
```
```cpp
// main.cpp
#include "def.h"    // 引入时又定义了一次
// error: multiple definition of `a'
```
头文件里定义全局变量或函数（inline除外），一旦头文件被导入不同的cpp文件里，则会导致重复重复定义的问题。而使用extern只是声明，编译器允许重复声明。

如果想在头文件定义全局变量或函数，建议使用static，static具有只定义一次的特点，目前还未清楚编译器是如何处理的，但经过测试可知不会发生重复定义问题。
```cpp
// def.h
#ifndef DEF
#define DEF

static int a = 1;  // static global

static void test() {    // static global
}

#endif
```

### 8.2 extern "C"
extern还有一个作用通过声明`extern "C" { }`来把函数编译和链接成C语言格式。比如一个C++函数`void foo( int x, int y );`会被编译成`_foo_int_int`之类的名字，因为C++允许函数重载改变函数参数，而C则不能重载，所以会编译成类似于`_foo`。如果你尝试在C++里面调用C语言实现的函数，则会编译出错，因为C++编译器找不到函数的链接，如果你用`extern "C"`包裹的C的定义话则不会用问题。

这里举一个C++调用C的例子：
```cpp
// cmax.h
#ifndef C_MAX_H
#define C_MAX_H
extern int max(int x, int y);
#endif

// cmax.c
#include "cmax.h"

int max(int x, int y) {
    return x > y ? x : y;
}

// cpp_main.cpp
extern "C" {
#include "cmax.h"
}

int main() {
    cout << max(2, 3) << endl; // 3
}
```
我们用extern "C"包裹一个C实现的头文件，编译器会把其接口以C的方式连接。

再看看C调用C++的例子：
```cpp

// cppmax.h
#ifndef CPP_MAX_H
#define CPP_MAX_H

#ifdef __cplusplus
extern "C" {
#endif
    int max(int x, int y);

#ifdef __cplusplus
}
#endif

#endif

// cppmax.cpp
#include "cppmax.h"
int max(int x, int y) {
    return x > y ? x : y;
}

// c_main.c
#include "cppmax.h"
#include <stdio.h>

int main() {
    printf("%d\n", max(4, 2)); // 4
}
```
虽然叫我们在C里面调用C++，但本质是你还是必须使用C++编译器去编译C，因为C不认得extern "C"，可以通过`#ifdef __cplusplus`来定义是否使用C++编译器来编译。


## 9. 命名空间
一个名称空间中的名称不会与另外一个名称空间的相同名称发生冲突，同时允许程序的其他部分使用该名称空间中声明的东西。使用关键字namespace创建命名空间。
```cpp
#include <iostream>
using namespace std;
namespace Jill{
    double fetch = 0;
}
double fetch = 5.5;
int main(){
    using Jill::fetch;
    cout << fetch << endl;  // 0

    fetch = 10.5;
    cout << fetch << endl;  // 10.5

    cout << ::fetch << endl;    // 5.5
}
```
上面这个例子中，我们在main函数中使用了`using`把对应变量加入到当前作用域中，我们可以直接使用该变量。若当前文件中存在一个同名的全局变量，我们可以使用`::`前缀来告诉编译器，这是一个全局变量。

我们还可以把整个命名空间导入到当前作用域。
```cpp
using namespace Jill;
```

### 9.1 使用命名空间的完整例子
按照惯例，现在头文件中定义命名空间。同时对于命名空间中的变量，若想产生外部链接，需要声明为`extern`。
```cpp
#ifndef HELLOWORLD_CH07_H
#define HELLOWORLD_CH07_H

namespace pers{
    extern int age;
    int getAge();
}

#endif //HELLOWORLD_CH07_H
```
在源文件中实现该命名空间。(注释部分是另一种实现方式)
```cpp
#include <iostream>
#include "ch07.h"

//int pers::age = 10;
//int pers::getAge() {
//    return age;
//}

namespace pers {
    int age = 10;
    int getAge() {
        return age;
    }
}
```
使用命名空间：
```cpp
#include <iostream>
#include "ch07.h"
int main(){
    using pers::getAge;
    std::cout << getAge() << std::endl;
    return 0;
}
```

### 10. 负数的二进制表示
二进制表示负数的方法是用一个最高位的符号位来表示正数和负数，用0表示正数，用1表示负数，所以有符号类型表示数字的位要比无符号类型少一位。

但实际表示负数时并不是简单的把符号位变成1即可，而是把原码（原来的值）转换成补码。补码的计算过程是：若发现是负数，则对除了符号位的所有位反转（0变1，1变0）得到其反码，然后反码加一。你会发现这个过程有点绕，为什么要算反码？为什么反码要加一？在回答这两个问题之前，先搞清楚一个基本问题，计算机如何做减法？

事实上计算机不会做减法，它只会对二进制做加法，即两个二进制的或运算。做减法时会想办法把减法变成加法去算。**减去一个数等于加上其补数**。所谓补数，其实就是最大值减去该数后得到的值。假设最大值是12，目标值是3，那么其补数就是12-3=9。假设我们用5减去3，则相当于 5-3 = (5+9)%12 = 2，符合“减去一个数等于加上其补数”这个理论。一个比较实际的例子就是我们的时钟，从5点调到2点，我们可以逆时针调3个小时，也可以顺时钟调9个小时。根据计算机理论，我们一般把最大值加1的数称为模数，比如32位整型的模就是2^32，任意一个数对其取模，都是0 ~ 2^32-1以内的数。时钟的模数是12，因为我们把12点看作是0点。

那么二进制是如何算补数（补码）的呢？假设现在对4位整数进行减法计算，我们要算7-3，那么根据理论，我们可以计算7+(16-3)，补数是16-3，虽然计算机不会对二进制做减法运算，但我们现在先手动做减法运算还是可以的，
```shell
16 - 3
10000 - 0011
# 为了确保相同位数之间做减法，我们把溢出位拆出来
(1 + 1111) - 0011
1 + (1111 - 0011)
1 + 1100
1101
```
算出来的1101恰好是13。根据上面的计算过程，可以回答上面提出的两个问题了，为什么要算反码？为什么反码要加一？对于第一个问题，你会发现1111 - 0011 = 1100，1100恰好是0011的反码，即用一个全1位的数减去任意数，都能得到其反码。因为计算机不会做减法，所以直接就通过反转来算反码。第二个问题，我们为了确保全1位以及位数与目标数相同，把模数减了1，所以后面要把1加回来。

继续把7-3算完，
```shell
7 + (16-3)
0111 + 1101
10100
# 截断溢出位
0100
```
10100即20，但位数超过了4位，所以截断溢出位，最后0100=4，符合7-3=4。你会发现截断溢出位的过程本质上就是十进制里的取模（20%16=4）。

同理，对一个负数做加法运算时，本质上就是减去这个数，所以我们可以干脆利落的把这个负数表示为其补码。按照上面的例子就是，-3 => (16-3) => 1101。其实存储时可以不存储成补码，只在运算时才计算出其补码，但意味着每次运算都要算一次补码，这个会造成多余的性能损耗，所以干脆存储时直接存储补码。

还有一个问题是，有符号类型里，为什么负数能表示的数比正数多1，比如char类型的范围是`-128~127`，这是因为表示0的时候，符号位的存在会导致可以表示出+0和-0，可以省掉其中一个，这里省掉-0，而-0就被默认看作是-128了。