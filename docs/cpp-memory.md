# C++ 内存管理

## 内存对齐
内存对齐就是把内存分成指定大小的块。寄存器在从主存读取数据时会按照一定大小的字节数来读取，而不是根据类型大小来读取，这么做是为了充分利用总线的大小，提供数据传输系效率。比如，64位操作系统一次读取的大小是8字。

但这么做会导致一个问题，一个值在结构体中的位置刚好一半在前8字节，一半在后8字节，导致要分开两次读取，如果读第一次后便发生线程或进程切换，该值可能会被修改，线程切换回来后再读第二次时值就变了。

所以才有内存对齐这一应对措施，我们把一个结构体或类的内存按块来分配，块的大小视结构体的最大类型大小而定，假设结构体最大的类型是int，那么块就是4字节；假设最大的是double，那么块的大小是8字节。内存对齐的目的是防止同一个值被分隔到两个块中。当结构体有第一个成员，我们用第一个块来存，如果块剩余大小能容纳第二个成员，就继续存到第一个块里；如果成员变量的大小大于块的剩余大小，就必须新开一个块来存储。

```cpp
struct A {
    char b;     // 1
    short c;    // 2

    double a;   // 8
};

int main() {
    cout << sizeof(A) << endl;  // 16
    return 0;
}
```

我们可以通过一个宏定义来强制规定块的大小。使用`#pragma pack()`包裹来对一个结构体声明内存对齐。
```cpp
#pragma pack(2) // 声明内存对齐的字节数
struct A {
    int a;
    char b;
    short c;
};
#pragma pack()

int main() {
    cout << sizeof(A) << endl; // 8
    return 0;
}
```
内存分布如图：

![mem1.jpg](https://i.loli.net/2019/10/21/YltiHZxAfe5mREu.jpg)

如果把内存对齐的字节数该为4，你会发现结果是一样的。
```cpp
#pragma pack(4) // 声明内存对齐的字节数
struct A {
    int a;
    char b;
    short c;
};
#pragma pack()

int main() {
    cout << sizeof(A) << endl; // 8
    return 0;
}
```
内存分布如图：

![mem2.jpg](https://i.loli.net/2019/10/21/FvkNjL9pn2mWtYe.jpg)

变量在内存中的排列顺序和结构体中声明顺序一样。如果b在a前面，则a必须新开一块内存。
```cpp
#pragma pack(4) // 声明内存对齐的字节数
struct A {
    char b;
    int a;
    short c;
};
#pragma pack()
```
内存分布如图：

![mem3.jpg](https://i.loli.net/2019/10/21/TRGOm8VbcElCXnM.jpg)


内存对齐可以减少CPU访问内存的次数。假设默认的内存对齐是2个字节，那么读取一个int值时就需要先访问前两个字节一次，再访问后两个字节一次，然后两部分整合。如果你设置内存对齐为4个字节，那么就能一次性读取出整个int。

注意，虽然前面说了一些内存对齐带来的好处，说得好像内存对齐是一个可选的优化项，但实际上这已经是一个默认行为了。如果没有指定对齐字节数目，则编译器会按类或结构中最大类型长度来对齐，不会出现一个属性的内存横跨两个对齐段的情况。这种对齐方式是最快的，结构体中每一个变量都能一次读出来。
```cpp
struct A {
	int a;
};

class Ac {
	virtual void test() {};
	int a;
};

int main()
{
	cout << sizeof(A) << endl;  // 4
	cout << sizeof(Ac) << endl;  // 16 (虚函数表指针是8字节，所以按8字节对齐)
}
```
内存对齐可能会消耗了较多的内存。如果想节省，可以把内存对齐数值调小一点，或者让字节数最大的类型放在最前面或最后面，让小字节类型都相邻，尽量使它们都放在同一个对齐段。
```cpp
class A {
	char s; // 1  (对齐之后独占了4字节)
	int a; // 4
	char s2; // 1  (对齐之后独占了4字节)
};

class A2 {
	char s; // 1
	char s2; // 1  (s和s2共占4字节)
	int a; // 4
};


int main()
{
	cout << sizeof(A) << endl;  // 12
	cout << sizeof(A2) << endl;  // 8
}
```
