# C++ 变量

## 0. 基本数据类型
C++ 基本数据类型如下：

![var.jpg](https://i.loli.net/2019/07/04/5d1df52f7ed3c86799.jpg)

一个字节的长度是8bit，如果全部用来存数字的无符号类型，则可表示2的8次方即`0~255`，而有符号类型则需要消耗一个比特位，即`-128~127`。如果你只想表示正数，那么无符号位类型会比有符号类型可表示的数大一倍。

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
可以用它来指出，即使结构或类变量为const，其某个成员也可以被修改。
```cpp
struct data {
    mutable int access;
};
const data d {0};
d.access = 1;
```

## 6. extern
关键字extern用于声明一个全局变量。你可以预先声明一个全局变量，相当于作为一个占位符。
```cpp
int main(){
    {
        extern int gg;
        std::cout << gg << std::endl; // 9
    }
}

int gg = 9;
```
上面的例子gg的定义在main函数的后面，理论上，是不能在main里面使用gg的，但我们可以使用通过`extern int gg`声明gg是一个全局变量。除此之外，你还能用extern声明一个全局函数。

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
### 6.1. extern "C"
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


## 7. 链接性和持续性

这里涉及的知识点是链接性(linkage)，当使用extern声明，变量将变为外部链接性。<span id="linkage"></span>

![](https://i.loli.net/2019/03/15/5c8ba1721f258.png)


函数的链接性呢？C++不允许在一个函数中定义另外一个函数，因此所有函数的存储持续性都自动为**静态**的，即在整个程序执行期间都一直存在；并且链接性是**外部**的，即可以在文件间共享，所以上面例子中我们不需要对一个函数声明`extern`。

## 8. static
上面图说明虽然静态变量static在整个程序执行期间都存在，但它意味着在
1. 函数的外面使用关键字static定义的变量的作用域为整个文件，但是不能用于其他文件（内部链接性）。
2. 在代码块中使用关键字static定义的变量被限制在该代码块内（局部作用域、无链接性）。
3. 如果定义在[类](cpp-class.md)内部，则可在同类的多个对象之间实现数据共享。

需要明白，static变量不同于自动变量的是，虽然两者都需要在作用域范围内才能使用，但自动变量脱离作用域后便被回收，而static则是直到程序停止运行后才被回收。

### 8.1 局部静态
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