# c++ 类运算符与友元

c++可重载的运算符有：

![20190306172336.png](https://i.loli.net/2019/03/06/5c7f919a673c6.png)


## 加法重载
创建前缀为`operator`的函数，如：
```cpp
class MyTime{
    int hour;
    int mins;
public:
    MyTime(int hour, int mins) : hour(hour), mins(mins) {}
    MyTime() {}
    MyTime operator+(const MyTime &t) const {
        MyTime sum;
        sum.mins = mins + t.mins;
        sum.hour = hour + t.hour + sum.mins / 60;
        sum.mins %= 60;
        return sum;
    }
};


int main(){
    MyTime t1 {10, 35};
    MyTime t2 {11, 50};
    MyTime t3 = t1 + t2;
    return 0;
}
```

## 函数对象
重载`operator()`运算符。
```cpp
class Linear {
    double slope;
    double y0;
public:
    Linear(double s1 = 1, double y = 1): slope(s1), y0(y) {}
    double operator()(double x) { return y0 + slope * x; }
};

int main(){
    Linear f;
    double y = f(3);  // 使用()
    cout << y << endl;
}
```


## 友元函数
所谓友元函数，即通过外部函数访问对象中的成员。
```cpp
class Box{
   double width;
public:
   friend void printWidth(Box box);
   void setWidth(double wid);
};

// 成员函数定义
void Box::setWidth( double wid ){
    width = wid;
}

// printWidth() 不是任何类的成员函数
void printWidth(Box box){
   // 友元可以直接访问该类的任何成员，包括私有成员
   cout << "Width of box : " << box.width << endl;
}
// 程序的主函数
int main( ){
   Box box;
   // 使用成员函数设置宽度
   box.setWidth(10.0);
   // 使用友元函数输出宽度
   printWidth(box);
   return 0;
}
```

### 如何理解友元
友元可以告诉你该外部函数与该类是有关系的，他们是“好朋友”，好处就是能够访问对象中的成员。友元是一种相当奇怪的特性，是反OOP的，但它的存在仍然是有用的。

### 友元与运算符重载
比如我想实现一种需求：
```cpp
MyTime B {10, 35};
MyTime A = B * 2;
MyTime A = 2 * B;
```
第一种好处理，只要把函数参数改成int即可。那第二种呢？
你会发现，面向对象无法解决第二种需求，你必须把运算符函数从对象中解脱出来，作为一个独立的函数或表达式才行。而c++的友元能解决这个问题。

直接在类中实现这一个函数即可：
```cpp
friend MyTime operator*(const double a, const MyTime &t) {
    MyTime res;
    long total = t.hour * a * 60 + t.mins * a;
    res.hour = total / 60;
    res.mins = total % 60;
    return res;
}
```
当运算符函数被标上了friend，它就不再是这个对象的成员函数了，**相当于在类之外实现了该函数。**<br>
比如例子中我们定义了`operator*`函数，那么我们可以这么用：
```cpp
A = operator*(2, B)
```
也可以这么用：
```
A = 2 * B
```
当然，上面这种写法只有运算符能这么干，其他函数不行。

c++中，如果你要用cout输出一个对象，也是通过重载运算符实现的。
```cpp
friend std::ostream &operator<<(std::ostream &os, const MyTime &time) {
    os << "hour: " << time.hour << " mins: " << time.mins;
    return os;
}
```
调用的话就像这样，
```cpp
operator<<(std::cout, A);
std::cout << A;
```


## 友元类
当在一个类中声明了一个友元类“好朋友”后，那么这个“好朋友”就可以使用该类的所有成员。

```cpp
class S1{
private:
    int s1_data;
public:
    S1(int s1_data): s1_data{s1_data} {}
    
    // 声明S2自己的“好朋友”，他可以使用自己的数据
    friend class S2;
};

class S2{
public:
    void getS1Data(S1 s1){
        // 可以直接访问S1的所有成员，包括私有成员
        cout << s1.s1_data << endl;
    }
};

int main(){
    S1 s1 = S1{10};
    S2 s2;
    s2.getS1Data(s1);   // 10
}
```
当然，虽然S1说S2是好朋友，但也只是单方面的，S2并不一定认为S1是他的好朋友，所以S1不能访问S2的成员。


## 总结
友元函数的最大用处就是用在重载运算符上。而友元类则更加匪夷所思，割裂了面向对象思想，而且还会增加代码复杂度。所以友元函数和友元类还是尽量少用。