# C++ 函数

## 内联函数
```cpp
inline double area(double r){
    return r * r * 3.14;
}

double x = 1.5;
double z = area(x);
```
上面使用了内联函数(inline)之后，编译器会把代码转换为：
```cpp
double z = x * x * 3.14;
```
使用内联函数的有优点在于可以省去了从栈中获取函数的开销，频繁调用且简单的函数可以声明为inline，但只能在文件内部使用。注意，你要确保你的函数是可以轻易被编译器转换的，否则可能产生奇怪的问题。

## 函数模板
C++ 的函数模板支持程序员将单个函数进行泛型编程，这一点是很多静态语言都没有的特性。<span id="function-template"></span>

```cpp
//template <class T>
template <typename T>
void swap(T &a, T &b){
    T t;
    t = a;
    a = b;
    b = t;
}
```
```cpp
double a = 1.0;
double b = 2.0;
swap(a, b);
cout << a << " " << b << endl;
int x = 10;
int y = 20;
swap(x, y);
cout << x << " " << y << endl;
```

上面这个例子，我们只定义了一个函数模板，就能传进不同类型的参数。实际上，当编译器发现函数被传入了两个int，编译器生成该函数的int版本。也就是说，用int替换所有的T，即生成一个新的函数。

### 例子
例子：返回一个指定行和列的二维数组。
```cpp
template<typename T>
T** array2d(int row, int col) {
	T** p = new T*[row];
	for (int i = 0; i < row; i++) {
		p[i] = new T[col];
	}
	return p;
}
```
```cpp
int** p = array2d<int>(row, col);
double** p2 = array2d<double>(row, col);
```
为什么这里要指定类型`<>`，是因为前面传入参数的时候编译器能够解析出你想要的类型，而这个例子不行，所以你要指定一个类型。

## 显示和隐式实例化
隐式实例化不需要主动声明类型，可以由编译器自动推导。
```cpp
template<class T>
T max2(T a, T b){
    return a > b ? a : b;
}
```
```cpp
max2(1.0, 2.0); // 隐式实例化
max2<double>(1.0, 2.0); // 显式实例化
```
两者差别在于，隐式实例化不具备自动转换数据类型的能力。
```cpp
max2(1.0, 2);   // error
max2<double>(1.0, 2);   // correct
```

## 参数缺省
当存在多个模板参数，可以缺省编译器可推导的参数。
```cpp
template<class TR, class T>
TR max2(T a, T b){
    return a > b ? a : b;
}
cout << max2<double>(2.2, 1.0) << endl; // 2.2
cout << max2<int>(2.2, 1.0) << endl; // 2
```


## 显式具体化
假设我们往函数模板里面传入一个结构体，我们不想将整个结构体交换，而是**只交换某个变量**呢？这时候我们可以使用显式具体化技术去做一个新的定义。

```cpp
struct Node{
    int key;
    int value;
    friend std::ostream &operator<<(std::ostream &os, const Node &node) {
        os << "key: " << node.key << " value: " << node.value;
        return os;
    }
};

template <typename T>
void swap(T &a, T &b){
    T t;
    t = a;
    a = b;
    b = t;
}
// explict specialization
template <> void swap<Node>(Node &a, Node &b){
    // only swap value
    int t = a.value;
    a.value = b.value;
    b.value = t;
}
```
```cpp
double a = 1.0;
double b = 2.0;
swap(a, b);
cout << a << " " << b << endl;

Node n1 {1, 300};
Node n2 {2, 900};
swap(n1, n2);
cout << n1 << " " << n2 << endl;
```
上面的例子中，我们重载了swap函数，并且函数名后面指定了`<Node>`，表明了对Node类型的swap函数进行具体化。


## decltype
类型获取（decltype）是c++11的新特性，在讲解它之前先看看一段代码：
```cpp
template <class T1, class T2>
void add(T1 a, T2 b){
    ??? c = a + b;
}
```
你可能无法得到a + b返回一个什么类型。当然你可能提出一个想法，就是在传入一个泛型T3作为c的类型，这是一种想法，当然c++还为我们提供了一个很好用的特性。

可以用到类型获取（decltype）了。通过计算a+b得到的值作为c的类型。
```cpp
decltype(a + b) c = a + b;
```
同样道理，你可以更自由的使用它：
```cpp
double x = 5.2;
decltype(x) x2 = x; // double
double* y = &x;
decltype(y) y2 = y; // double* 
double& z = x;
decltype(z) z2 = x; // double&
```
还有一个小语法糖，加入双重括号之后可以得到引用类型。
```cpp
double xx = 4.4;
decltype((xx)) r2 = xx; // r2 is double&
r2 += 1;
cout << xx << endl;  // 5.4
cout << r2 << endl;  // 5.4
```

孜孜不倦的程序员们又提出了一个问题，返回类型咋整呢？
```cpp
template <class T1, class T2>
??? add(T1 a, T2 b){
    return a + b;
}
```
而c++11为解决这个问题，又创造了**后置返回类型**：
```cpp
template <class T1, class T2>
auto add(T1 a, T2 b) -> decltype(a + b) {
    return a + b;
}
```
返回类型先暂时写个auto，然后返回的时候再计算它的类型。

## 栈帧
函数的调用以及自动变量会存到栈中，每一个函数的调用就会产生一个新的栈帧。在x86系统中，栈是从高地址向低地址生长的，所有这个栈的开口是朝下的。一个新函数的调用，栈帧会在下面入栈；同理，分配内存时也会从高地址向低地址扩展，但存储数据又是从低地址向高地址存的。在汇编里面，一般用ebp表示栈底，用eps表示栈顶。

下面我用一个段程序以及它的汇编来解释一下函数调用的过程，

```cpp
int func(int m, int n) {
    int a = m;
    int b = n;
    // ...
    return a;
}

int main() {
    int res = func(10, 20);
}
```
```assembly
__main:
    push ebp            ; 栈底ebp入栈
    push 20             ; 传入参数入栈
    push 10             ; 传入参数入栈
    call func           ; 调用函数（跳转至新栈帧__func）
    sub esp, 4          ; func退出后来到这里，分配4个字节空间
    mov [ebp-4], eax    ; 存放函数返回值

__func:
    push ebp            ; 栈底ebp入栈
    mov ebp, esp        ; 使ebp指向栈顶
    sub esp, 8          ; 栈顶下移，即分配栈空间
    mov [ebp-4], 10     ; 前4个字节存一个整数10
    mov [ebp-8], 20     ; 前8个字节存一个整数20，因为整数只占4个字节，所以不会覆盖前面4个字节
    ...                 ; 其他操作，这里省略
    mov esp, ebp        ; 函数返回时，把栈顶指针移到栈底
    pop ebp             ; 把栈底移除
    ret                 ; 返回
```

![stackframe.jpg](https://i.loli.net/2021/09/21/rzVpcaq7wDJ35KB.jpg)

根据汇编注释和示意图应该可以理解整体流程了。其中示意图里的“返回地址”这个东西是每次调用函数时都会入栈的一块空间。你可以把call指令转换为，
```assembly
push eip + 2 ; 返回地址=当前地址+2
jmp __func
```
即每次调用函数至少都会占用两个字节的空间。

函数的返回值会放入被`eax`指向的内存空间，通过`mov`指令，把eax指向的数据移动到指定的内存空间里。

本节参考：

- [x86 Disassembly/Functions and Stack Frames](https://en.wikibooks.org/wiki/X86_Disassembly/Functions_and_Stack_Frames)
- [x86 Assembly Guide](http://www.cs.virginia.edu/~evans/cs216/guides/x86.html)

