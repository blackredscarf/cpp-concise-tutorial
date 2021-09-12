## 输入输出

## 流
C++程序把输入和输出看作字节流。输入时，程序从输入流中抽取字节；输出时，程序将字节插入到输出流中。

## 缓冲
缓冲方法则从磁盘上读取大量信息，将这些信息存储在缓冲区中，然后每次从缓冲区里读取下一个字节。因为从内存中读取单个字节的速度比从硬盘上读取快很多，所以这种方法更快，也更方便。到达缓冲区尾部后，程序将从磁盘上读取另一块数据。

输出时，程序首先填满缓冲区，然后把**整块数据传输给硬盘**，并**清空缓冲区**，以备下一批输出使用。这被称为**刷新缓冲区**。


## 输入输出对象
下面这些都是对象，通过使用`>>`或`<<`方法来获取你要输入或输出的数据。

cin 在默认情况下，这个流被关联到标准输入设备（通常为键盘）。

cout 在默认情况下，这个流被关联到标准输出设备。

cerr 可用于显示错误消息。在默认情况下，这个流被关联到标准输出设备。这个流没有被缓冲，这意味着信息将被直接发送给屏幕，而不会等到缓冲区填满或新的换行符。

clog 在默认情况下，这个流被关联到标准输出设备。与cerr相似，不同的是这个流会被缓冲。

```cpp
cerr << "errer." << endl;
clog << "log." << endl;
```

## 简单输入
cin >> a 将以空白字符为结束的标志。但空白字符会留在输入队列。
```cpp
char a[10];
cin >> a;
```

cin.getline(char*, size) 读取一行到字符串中。把回车符换成\0。
```cpp
char name[10];
std::cin.getline(name, 10);
```

![20190306221919.png](https://i.loli.net/2019/03/06/5c7fd6eae83eb.png)

cin.get(char) 读一个字符，而 cin.get(char*, size) 也是读一行，但不读取也不丢弃回车，而是留在输入队列里。由于第一次调用后，换行符将留在输入队列中，因此第二次调用时看到的第一个字符便是换行符。解决办法就是调用一个不带参数的`get()`把回车吃掉。

cin.read(char*, size) 读入标准输入的前count个字符。
```cpp
char data[20];
cin.read(data, 10);
cout << data << endl;
```

## 混合类型输入
比如我们想先输入一个整型，再输入一个字符串。

错误写法：
```cpp
int a;
char line[10];
std::cin >> a;
std::cin.getline(line, 10);
```
为什么会错误？因为cin >> 后留下一个换行符，而getline遇到换行符就会结束。所以正确写法应该是输入a后吃掉换行符。
```cpp
(std::cin >> a).get();
std::cin.getline(line, 10);
```

## string输入
注意 getline() 并非cin的函数。获取cin对象只是为了知道现在读到哪里了而已。
```cpp
std::cin >> str1;
getline(std::cin, str1);
```

## 输出
通过cout输出，如果没有换行符或还没装满缓冲区，在程序执行过程中将暂时不输出，如果你想马上输出，可以使用`flush()`方法。
```cpp
cout << "hello world" << flush;
```
也可以直接调用`flush(cout)`。

## 快速输入输出
很多程序竞赛选手会吐槽C++的IO很慢，但实际上，cin和cout会默认与stdio进行同步，所以才导致了很慢，好处是避免混用printf和cout而造成的输出顺序和代码语句不一致的问题。我们可以关掉同步，但结果是你不能同时使用printf和cout，cin和scanf。

还有就是，默认的情况下cin和cout绑定，每次cin之前都会把输出缓冲区的数据刷新，即调用flush。好处是避免了明明先cout输出，但命令行窗口中cin的读入数据在cout输出数据之前。坏处是影响cin的效率，可以用 cin.tie(0) 关掉绑定。
```cpp
#include <iostream>
int main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    // IO
}
```

## 格式化输出

### 进制
比如，先给cout设置一个要求输出16进制的设置。
```cpp
cout << hex << 16 << " " << 17 << endl; // 10 11
dec(cout);
```
如果你打开`hex`的源码，不会发现不过是在内部为cout对象设置了一些参数。
```cpp
inline ios_base&
hex(ios_base& __base) {
    __base.setf(ios_base::hex, ios_base::basefield);
    return __base;
}
```
八进制同理`oct`。

如果不想影响其他输出，必须最后加上`dec(cout)`来还原设置。

### 宽度
```cpp
for(int i = 1; i <= 1000; i*=10){
    cout.width(5);
    cout << i << ':';
    cout.width(8);
    cout << i * i << endl;
}
/*
    1:       1
   10:     100
  100:   10000
 1000: 1000000
*/
```
你会发现，当设置宽度后，所有输出都向右对齐了。`width()`只影响一行输出。

### 填充
```cpp
cout.fill('*');
cout << "password: ";
cout.width(8);
cout << "1234" << endl;
// password: ****1234
``` 
当时设置宽度后，不足宽度部分会被填充。`fill()`只影响一行输出。

### 精度
所谓精度，就是允许显示多少个数字。
```cpp
// 默认精度
streamsize ss = cout.precision();
cout << ss << endl;  // 6

double a = 2.0 / 6;
cout << a << endl;  // 0.333333

// 设置精度为2
cout.precision(2);  
cout << a << endl;  // 0.33
cout << 88.23 << endl;  // 88

// 恢复精度
cout.precision(ss); 
cout << a << endl;  // 0.333333
cout << 88.23 << endl;  // 88.23
```

### 设置格式
cout内置了名为`setf()`的接口，允许你设置一些格式，包括：

![](https://i.loli.net/2019/03/31/5ca07f82258ab.png)


```cpp
// 设置整数+
cout.setf(ios_base::showpos);
cout << 333 << endl;    // +333

// 恢复
cout.unsetf(ios_base::showpos);
cout << 333 << endl;    // 333
```

一些双参数的设置：

![](https://i.loli.net/2019/03/31/5ca081fde014b.png)


举例：
```cpp
// 科学计数法
ios_base::fmtflags f = cout.setf(ios_base::scientific, ios_base::floatfield);
cout << 1300000.0 << endl;  // 1.300000e+006

// 定点计数法
cout.setf(ios_base::fixed, ios_base::floatfield);
cout << 1300000.0 << endl;  // 1300000.000000

// 中间
cout.setf(ios_base::internal, ios_base::adjustfield);
cout.width(7);
cout << -332 << endl;   // -   332

// 左对齐
cout.setf(ios_base::left, ios_base::adjustfield);
cout.width(7);
cout << -332 << endl;   // -332
```

你会发现，如果你直接设置`cout.setf(ios_base::scientific)`，而不要第二个参数时，其实也可以成功。但当你想设置 `ios_base::fixed`，你会发现无法准确设置。

如果你打开`ios_base::scientific`的源码，你会发现这些格式控制是一堆常数。我的理解是c++是通过这些常数的位运算来控制格式的最终变化。如果少传了第二个参数，就少了一步运算，你想设置时就发生了错误。

那么我想还原成**默认格式**怎么办？在你改变格式之前，先保存一个原始格式，最后再设置回去。
```cpp
ios::fmtflags fo(cout.flags());
// ...
cout.flags(fo);
```

### 简便格式化
正如一开始讨论如何格式化进制那样，同样有很多简便的函数可以使用。

![](https://i.loli.net/2019/03/31/5ca08dc8d5617.png)

```cpp
cout << hex << 16;
```

而头文件`iomanip`中有3个最常用的格式化工具，setprecision( )、setfill( ) 和 setw( )。
```cpp
cout << setw(12) << setfill('*') << setprecision(4) << 12367.12345 << endl;
// **1.237e+004
```
用完`setprecision()`之后要注意还原格式。


