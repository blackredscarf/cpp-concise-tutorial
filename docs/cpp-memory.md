# C++ 内存管理

## 内存对齐
内存对齐就是把内存分成指定大小的块，如果一个变量的大小大于块的剩余大小，就必须新开一个块来存储。

使用`#pragma pack()`包裹来对一个结构体声明内存对齐。
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


内存对齐的目的是为了减少CPU访问内存的次数。假设默认的内存对齐是2个字节，那么读取一个int值时就需要先访问前两个字节一次，再访问后两个字节一次，然后两部分整合。如果你设置内存对齐为4个字节，那么就能一次性读取出整个int。


如果没有指定对齐字节数目，则编译器会按类或结构中最大类型长度来对齐。这种对齐方式是最快的，结构体中每一个变量都能一次读出来，但也消耗了较多的内存。如果你内存较小，可以把数值调小一点或排列一下属性的顺序。

