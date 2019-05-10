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
