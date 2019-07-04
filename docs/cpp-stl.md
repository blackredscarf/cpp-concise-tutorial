# c++ STL

标准模板库 (Standard Template Library, STL) 是C++内置的模板库，包含 Adaptor, Function Object, Iterator, Container, Algorithm 几个模块。

![stl.png](https://i.loli.net/2019/07/04/5d1df5492d1b264877.png)

Adaptor 适配器通过修改其它类的接口，使适配器满足一定需求，可分为容器适配器、迭代器适配器和函数对象适配器三种。

Function Object 函数对象就是定义了操作符operator() 的对象。结合函数模板的使用，函数对象使得STL更加灵活和方便，同时也使得代码更为高效。

Iterator, Container, Algorithm 在下边会重点讲。

## 容器
对于STL容器库，其包含了两类容器，一种为顺序容器（sequence container），另一种为关联容器（associative container）。

### vector
vector是最常用的顺序容器。特点是随机存取任何元素都能在常数时间完成，在**尾端**增删元素具有较佳的性能。

构建，插入，遍历等基本操作如下。注意，插入元素的时候不需要使用new，因为分配空间的事情由vector内部去干，你只需要将数据放到已分配的空间中即可，和指针数组是差不多的。
```cpp
class Node {
    string s;
public:
    Node(string s) : s(s) {}
    friend ostream &operator<<(ostream &os, const Node &node) {
        os << "s: " << node.s;
        return os;
    }
};

// 初始化
vector<Node> v = {
    Node{"hello"},
    Node{"java"},
    Node{"c++"},
    Node{"python"},
};

// 添加元素到结尾
v.push_back(Node{"php"});
v.push_back(Node{"go"});
```

#### 遍历
下面这种是C++11的遍历。如果你想把循环中的修改应用到容器当中，需要加上一个`&`引用。
```cpp
for(Node &n : v) {
    cout << n << '\n';
}
```
vector可以直接通过下标访问，所以还能这么遍历：
```cpp
for(int i = 0; i < v.size(); i++) {
    cout << v[i] << '\n';
}
```

#### 删除
其中`begin()`返回的是vector中开始元素的“指针”。下面代码的意思是需要删除 [v.begin(), v.begin() + 2) 区间内的元素。`erase()`函数将返回被删除的最后一个元素的后一个位置的迭代器。
```cpp
// 删除v.begin()与v.begin()+1指向元素
it = v.erase(v.begin(), v.begin() + 2);
```
`end()`是vector最后一个元素的再后一个位置，那么删除全部就可以这么干：
```cpp
v.erase(v.begin(), v.end());
```
当然，使用`v.clear()`来删除全部会更加方便。

#### 插入元素
vector::insert() 支持在任一位置插入元素，但插入后需要移动其他元素，时间复杂度较高。若是对性能要求较高，则不建议使用。
```cpp
// 在v.begin()插入元素
v.insert(v.begin(), Node{"fgo"});

vector<Node> vnew = {
        Node{"John"},
        Node{"Mary"},
};
// 在v.begin() + 3的位置开始插入新数据
// 新数据是从vnew.begin()到vnew.end()所指向的数据集
v.insert(v.begin() + 3, vnew.begin(), vnew.end());
```
`begin()`返回的是指向数据的指针，但却不是单纯的`Node*`指针，而是一个 `vector<Node>::iterator`，你可以像访问指针那样去访问他。
```cpp
vector<Node>::iterator data = v.begin();
```
因为这个类型写起来真的很长，所以你可以用`auto`来代替，让编译器自动推断类型。顺便介绍第三种变量方式：
```cpp
for(auto data = v.begin(); data != v.end(); data++) {
    cout << *data << '\n';
}
```

值得注意的是，vector使用`insert()`操作是通过移位实现的，复杂度为O(n)，如果需要频繁的插入操作，而较少查询操作，使用list双向链表的效率更高。

#### 倒序遍历
使用`rbegin()`能够得到一个所谓倒序迭代器，该迭代器将指向容器最后元素，而倒序迭代器的递增操作内部实现为递减。
```cpp
// vector<Node>::reverse_iterator data = v.rbegin();
for(auto data = v.rbegin(); data != v.rend(); data++) {
    cout << *data << '\n';
}
```

关于vector的API详见 [vector](https://zh.cppreference.com/w/cpp/container/vector)。

以及下面介绍的容器API都可以在[这里](https://zh.cppreference.com/w/cpp/container)找。

### deque
deque模板类（在deque头文件中声明）表示**双端队列**（double-ended queue）。除了支持随机访问外，deque对象在**开始位置**和**结束位置**插入和删除元素的时间复杂度都是O(1)，而不像vector中的O(n)。但因实现更加复杂，代价就是与vector相同部分的操作要比vector更慢

### list
list内部实现是双向链表，意味这插入或删除操作的时间复杂度是O(1)。缺点就是不能随机访问。

C++11提供了forward_list，即单向链表，只能正向迭代，但实现更简单，效率更高。

### queue
队列。只提供将元素添加到队尾、从队首删除元素、查看队首和队尾的值、检查元素数目和测试队列是否为空等操作。

### priority_queue
优先队列。在priority_queue中，最大的元素被移到队首。你可以在构造函数中指定比较方法，来实现自定义排序。

### stack
栈。允许将压入推到栈顶、从栈顶弹出元素、查看栈顶的值、检查元素数目和测试栈是否为空等操作。

值得一提的是 Stack, Queue, Priority_queue 都不是独立的容器，而是一种**容器适配器**。它提供原容器的一个专用的受限接口。缺省的stack，queue类，是对deque（双端队列）的一种限制。

### array
C++11新增了模板类array，它也位于名称空间std中。与数组一样，array对象的长度也是固定的，也使用栈（静态内存分配），而不是自由存储区，因此其效率与数组相同，但更方便，更安全。

详见 [array](https://zh.cppreference.com/w/cpp/container/array)

### set
集合。set不允许存储多个相同的值。而且默认从小到大排序。你可以在第二个模板参数中指定排序方式。

还有一种multiset容器。它是允许多个值相同的。

### map
键值对容器。
```cpp
map<int, string> m;
m[100] = "C++";
m.insert(pair<int, string>(60, "java"));
for(auto p : m){
    cout << p.first << " ";
    cout << p.second << endl;
}
// 60 java
// 100 C++
```

还有一种multimap容器，它允许一个键对应多个值。

值得一提，set和map都属于关联容器。C++11提供了无序类型的set和map。分别为unordered_set、unordered_multiset、unordered_map和unordered_multimap。关联容器是基于树结构的，而无序关联容器是基于数据结构哈希表的，这旨在提高添加和删除元素的速度以及提高查找算法的效率。

### 各种容器的优缺点
vector 的数据模型就是数组。优点：内存和C完全兼容、高效随机访问和尾端增删。缺点：在内部插入删除元素代价巨大，动态大小可能导致申请大量内存和做大量拷贝。

list 的数据结构模型是链表。优点：任意位置插入删除元素，以及两个容器融合都是常量时间复杂度。缺点：不支持随机访问、比vector占用更多的存储空间。

deque 的数据模型是双端队列。优点：常数时间的随机访问和双端增删。缺点：内存占用比较高。

map、set、multimap、multiset 的数据结构模型是二叉树(红黑树)。优点：元素会按照键值排序，查找是对数时间复杂度，通过键值查元素，map提供了下标访问。

## 迭代器
迭代器是C++ STL中一种独特的概念，你可以理解为访问数据的方式。也许你会觉得有点陌生和繁重，但相信我，它有着非常独特的意义，通过规范数据的访问方式能够带来非常多的好处。

普通指针可以指向内存中的一个地址，而迭代器可以指向容器中的一个位置。使用迭代器后，算法函数可以访问容器中指定位置的元素，而无需关心元素的具体类型。


### 输入迭代器
`InputIterator`输入迭代器有几种特点：
1. 只能递增式访问，即只能用`++`，不能`--`或其他。
2. 只能读取，不能写入。

### 输出迭代器
`OutputIterator`输出迭代器有几种特点：
1. 只能递增式访问，即只能用`++`，不能`--`或其他。
2. 只能写入，不能读取。

### 正向迭代器
`ForwardIterator`正向迭代器有几种特点：
1. 只能递增式访问，即只能用`++`，不能`--`或其他。
2. 可以写入或读取

### 双向迭代器
`BidirectionalIterator`双向迭代器有几种特点：
1. 可以递增式和递减式访问，支持`++`和`--`，其他则不行。
2. 可以写入或读取

### 随机访问迭代器
`RandomAccessIterator`随机访问迭代器有几种特点：
1. 可以进行任何形式的访问，可以直接访问容器中任何一个元素。
2. 可以写入或读取。

### 迭代器功能一览
![container1.png](https://i.loli.net/2019/03/29/5c9e1c0e25716.png)

其中解除引用，即通过`*`号前缀来读取或写入数据。

容器对迭代器的使用：
- map, set, list 类型提供双向迭代器。
- string, vector和deque 容器上定义的迭代器都是随机访问迭代器，用作访问内置数组元素的指针也是随机访问迭代器。
- istream_iterator 是输入迭代器，ostream_iterator 是输出迭代器。

而上面提到的`rbegin()`和`rend()`其实就是迭代器适配器。它通过修改调整原迭代器的接口获得的适配器。

### 迭代器的意义
通过迭代器控制对数据的访问权限有着重要意义，这个意义在于对通用函数接口的限制与复用。

举个例子：`list`是双向链表，只能前后访问，属于双向迭代器，`vector`则允许随机访问，属于随机访问迭代器，当使用`sort()`时，你觉得这个`sort()`能同时应用于`list`和`vector`两种容器吗？明显不能，因为`sort()`函数源码中就规定只能传入`RandomAccessIterator`类型。`sort()`内部实现是快速排序，明显不能用于`list`这样的数据结构。而`list::sort()`中则使用了归并排序这样只需访问相邻元素便可完成排序的算法。这么便对接口起到限制的作用。

还有就是`find()`这个函数，它要求输入的是`InputIterator`，几乎兼容所有容器类型，而且意味着内部实现是顺序遍历的，复杂度为O(n)。这便是对接口的复用。而对于`set`容器，内部是排好序的，用二分查找速度更快，所以可以使用`set::find()`。


## 算法
容器有很多中，比如`vector`, `array`, `set`等等。他们可能有很多共有的操作，比如遍历，排序，交换元素等等。这时候，c++就提供了能够兼容若干种容器的函数。

### for_each
for_each()就是一个遍历函数，前两个参数是容器的指针范围，而第三个参数传进一个函数，范围内的每一个元素都会传进函数中。
```cpp
void show(int a){
    cout << a << endl;
}
for_each(vs.begin(), vs.end(), show);
```
同理，我们也可以使用c++11的lambda表达式：
```cpp
for_each(vs.begin(), vs.end(), [](int a){
    cout << a << endl;
});
```

### sort
排序，默认是递增的。
```cpp
vector<int> vs = {12, 3, 5, 1};
sort(vs.begin(), vs.end());
// 1 3 5 12
```
若是递减，你需要通过一个函数来进行控制。
```cpp
int cmp(int a, int b){
    return a > b;
}
sort(vs.begin(), vs.end(), cmp);
// 12 5 3 1
```
为了方便，库函数中给我们提供了很好用的模板方法。
```cpp
sort(vs.begin(), vs.end(), greater<int>());
```
同理还有`less`, `greater_equal`, `less_equal`等。


进一步，我们需要对自定义对象进行排序呢？
我们需要重载其`<`运算符。
```cpp
bool operator<(const Node &rhs) const {
    return s < rhs.s;
}
sort(v.begin(), v.end());
```
同理，如果你需要递减排序，就要重载大于`>`运算符，并使用`greater()`，大于等于或小于等于同理。

### copy
copy()函数实现将[first, last)区间的元素复制到另一个地方，这个地方的起始位置为DestBeg。
```cpp
OutputIterator copy(InputIterator first, InputIterator last,  OutputIterator DestBeg);
```

### 函数适配器
```cpp
int count1 = count_if(vec.begin(), vec.end(), bind2nd(less_equal<int>(), 10)); 
//求容器中小于等于10的元素个数  
```
上面的`bind2nd()`其实就是一种函数适配器，也叫绑定器。通过修改`less_equal`函数对象，将其第二个参数绑定成10。


还有一种叫取反器 (negator)，目的是为了翻转函数对象的结果。
```cpp
int count2 = count_if(vec.begin(), vec.end(), not1(bind2nd(less_equal<int>(), 10))); 
//求容器中不小于等于10的元素个数，正好是上面结果的取反
```

