# c++ 容器

## vector
vector是所有容器中最常用的，所以会详细讲讲。

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

遍历。下面这种是C++11的遍历。如果你想把循环中的修改应用到容器当中，需要加上一个`&`引用。
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
<br>

删除操作。其中`begin()`返回的是vector中开始元素的“指针”。下面代码的意思是需要删除 [v.begin(), v.begin() + 2) 区间内的元素。
```cpp
// 删除v.begin()与v.begin()+1指向元素
v.erase(v.begin(), v.begin() + 2);
```
`end()`是vector最后一个元素的再后一个位置，那么删除全部就可以这么干：
```cpp
v.erase(v.begin(), v.end());
```
当然，使用`v.clear()`来删除全部会更加方便。

<br>

插入元素：
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
<br>

倒序遍历。使用`rbegin()`能够得到一个所谓倒序迭代器，该迭代器将指向容器最后元素，而倒序迭代器的递增操作内部实现为递减。
```cpp
// vector<Node>::reverse_iterator data = v.rbegin();
for(auto data = v.rbegin(); data != v.rend(); data++) {
    cout << *data << '\n';
}
```

关于vector的API详见 [vector](https://zh.cppreference.com/w/cpp/container/vector)

以及下面介绍的容器API都可以在[这里](https://zh.cppreference.com/w/cpp/container)找。

## deque
deque模板类（在deque头文件中声明）表示双端队列（double-ended queue）。除了支持随机访问外，deque对象的开始位置插入和删除元素的时间复杂度都是O(1)，而不像vector中的O(n)。但因实现更加复杂，代价就是与vector相同部分的操作要比vector更慢

## list
list内部实现是双向链表，意味这插入或删除操作的时间复杂度是O(1)。缺点就是不能随机访问。

C++11提供了forward_list，即单向链表，只能正向迭代，但实现更简单，效率更高。

## queue
队列。只提供将元素添加到队尾、从队首删除元素、查看队首和队尾的值、检查元素数目和测试队列是否为空等操作。

## priority_queue
优先队列。在priority_queue中，最大的元素被移到队首。你可以在构造函数中指定比较方法，来实现自定义排序。

## stack
栈。允许将压入推到栈顶、从栈顶弹出元素、查看栈顶的值、检查元素数目和测试栈是否为空等操作。

## array
C++11新增了模板类array，它也位于名称空间std中。与数组一样，array对象的长度也是固定的，也使用栈（静态内存分配），而不是自由存储区，因此其效率与数组相同，但更方便，更安全。

详见 [array](https://zh.cppreference.com/w/cpp/container/array)

## set
集合。set不允许存储多个相同的值。而且默认从小到大排序。你可以在第二个模板参数中指定排序方式。

还有一种multiset容器。它是允许多个值相同的。

## map
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


## 通用函数
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

<br>

进一步，我们需要对自定义对象进行排序呢？
我们需要重载其`<`运算符。
```cpp
bool operator<(const Node &rhs) const {
    return s < rhs.s;
}
sort(v.begin(), v.end());
```
同理，如果你需要递减排序，就要重载大于`>`运算符，并使用`greater()`，大于等于或小于等于同理。

## 迭代器
迭代器是C++中一种独特的概念，你可以理解为访问数据的方式。也许你会觉得有点陌生和繁重，但相信我，它有着非常独特的意义，通过规范数据的访问方式能够带来非常多的好处。

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

### 迭代器的意义
通过迭代器控制对数据的访问权限有着重要意义，这个意义在于对通用函数接口的限制与复用。

举个例子：`list`是双向链表，只能前后访问，属于双向迭代器，`vector`则允许随机访问，属于随机访问迭代器，当使用`sort()`时，你觉得这个`sort()`能同时应用于`list`和`vector`两种容器吗？明显不能，因为`sort()`函数源码中就规定只能传入`RandomAccessIterator`类型。`sort()`内部实现是快速排序，明显不能用于`list`这样的数据结构。而`list::sort()`中则使用了归并排序这样只需访问相邻元素便可完成排序的算法。这么便对接口起到限制的作用。

还有就是`find()`这个函数，它要求输入的是`InputIterator`，几乎兼容所有容器类型，而且意味着内部实现是顺序遍历的，复杂度为O(n)。这便是对接口的复用。而对于`set`容器，内部是排好序的，用二分查找速度更快，所以可以使用`set::find()`。



