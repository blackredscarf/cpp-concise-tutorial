# C++ lambda 

## 基本形式
lambda 相当于一个匿名函数。允许你往函数中传入一个函数，形式为`[](){ ... }`。
```cpp
srand(time(nullptr));
vector<int> v(10);
generate(v.begin(), v.end(), [](){
    return rand() * 255 % 100;
});
for_each(v.begin(), v.end(), [](int a){
    cout <<  a << " ";
});
```

## 返回类型推断
lambda 函数中不需要声明函数的返回值的类型，它是由decltype自动推断的。上面的例子中，自动推断为`int`。

当然，你也可以主动声明返回类型。
```cpp
[]() -> int {
    return rand() * 255 % 100;
}
```

## 参数传入
在`()`填入需要传入的参数。

## 变量引用

lambda 可以使用作用域下所有变量。只需要在`[]`中声明你要使用的变量即可。
```cpp
int base = 10;
generate(v.begin(), v.end(), [base](){
    return base + rand() * 255 % 100;
});
```
当然，直接填写变量只能获取而不能修改，如果你想在函数中修改外部变量，可以加一个引用。
```cpp
int base = 10;
generate(v.begin(), v.end(), [&base](){
    base = rand() * 255 % 100;
    return base;
});
cout << base << endl;
```
传指针变量也是可以的。
```cpp
int* base = new int(10);
generate(v.begin(), v.end(), [base](){
    *base = rand() * 255 % 100;
    return *base;
});
cout << *base << endl;
```

在`[]`中填写`&`，便可引用当前作用域下的所有变量。
```cpp
int base = 10;
generate(v.begin(), v.end(), [&](){
    return base + rand() * 255 % 100;
});
```

总结lambda表达式的基本语法：
```cpp
[捕获列表](参数列表) mutable(可选) 异常属性 -> 返回类型 {
    // 函数体
}
```

## 命名
你可以对一个lambda函数进行命名。
```cpp
int base = 10;
auto mrand = [&base]() {
    base = rand() * 255 % 100;
    return base;
};
generate(v.begin(), v.end(), mrand);
```
注意要用一个auto来接收lambda，因为类型是不确定的。

## 参数类型推断
C++14开始，lambda的参数允许填写为`auto`，由编译器自动推断类型。
```cpp
for_each(v.begin(), v.end(), [](auto a){
    cout <<  a << " ";
});
```

## function 包装器
function类型的变量可以接收一个函数。
```cpp
int Max(int a, int b){
    if(a < b){
        return b;
    }
    return a;
}

int main(){
    function<int(int, int)> max = Max;
    int ans = max(10, 12);
    cout << ans << endl;
}
```
你可以通过泛型来定义函数的形式，比如`int(int, int)`，就表示接收两个int作为形参，并返回一个int的函数。返回空以及无参数可以写作`void()`。

同理，lambda函数也是可以的。
```cpp
int base = 10;
function<int()> mrand = [&base]() {
    base = rand() * 255 % 100;
    return base;
};
generate(v.begin(), v.end(), mrand);
```

你可以在函数参数中声明function类型的参数，以便于向函数中传递函数。
```cpp
void display(vector<int> &v, function<bool(int)> filter){
    for(int d: v){
        if(filter(d)){
            cout << d << endl;
        }
    }
}

int main(){
    vector<int> v{10, 5, 30, 23, 3, 1};
    display(v, [](int a){
        return a >= 10;
    });
}
```

还有更骚的，返回一个函数。
```cpp
function<int(int)> max(int a){
    function<int(int)> max2 = [a](int b){
        if(a > b){
            return a;
        }
        return b;
    };
    return max2;
}

int main(){
    int mx = max(3)(4);
    cout << mx << endl; // 4
}
```

## 可变参数模板
可变参数模板(variadic template)。你可以在模板中定义一个未知长度的参数。
```cpp
template <typename... Args>
```
那你要怎么使用它们呢？很遗憾，使用可变参数的方式可能会让你感到不适。没错，你得通过递归来获取每一个参数。

下面举个例子：
```cpp
template <typename T>
T sum(T &t){
    return t;
}

template <typename T, typename... Args>
T sum(T first, Args... args){
    return first + sum(args...);
}

int main(){
    int s = sum(1, 2, 3, 4);
    cout << s << endl;  // 10
}
```

这个可变参数的使用方式让人感到非常的违和，但只要你不厌其烦，你可以玩出非常多的花样。可以参考这篇[文章](https://www.cnblogs.com/qicosmos/p/4325949.html)。


