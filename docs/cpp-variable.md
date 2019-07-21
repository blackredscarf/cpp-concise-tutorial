# C++ 变量

## 0. 基本数据类型
C++ 基本数据类型如下：

![var.jpg](https://i.loli.net/2019/07/04/5d1df52f7ed3c86799.jpg)

## 1. 初始化
C++11提供了大括号`{}`的初始化方式。这是一种完全通用的初始化方式，你可以用来初始化任何类型，包括自定义对象。还有一个特点就是自带类型检查.以往，你可能很容易不小心将一个int赋值给char类型，编译器还不会报错，但如果用`{}`就没关系了。
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

可以将一个指针指向某个硬件位置，其中包含了来自串行端口的时间或信息。在这种情况下，硬件（而不是程序）可能修改其中的内容。或者两个程序可能互相影响，共享数据。

程序在几条语句中两次使用了某个变量的值，则编译器可能不是让程序查找这个值两次，而是将这个值**缓存到寄存器中**。这种优化假设变量的值在这两次使用之间不会变化。

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
关键字extern表明是引用声明，即声明引用在其他文件定义的变量。比如我在file1.cpp 定义了一个变量`x`，而file2.cpp中想访问到这个变量，我们该怎么办？在python中直接使用import模块即可，但c++中你只能include一个`.h`文件。

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