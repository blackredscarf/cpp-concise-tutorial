# allocator
allcoator被称为空间分配器。位于头文件`<memory>`中。

## new 和 delete
在说这个东西的作用之前，我们先分析一下 new 和 delete 关键字。
```cpp
A* ap = new A(3);
delete ap;
```
上面是常见的new一个指针对象并将其删除。看上去是两个步骤：1. 初始化一个指针对象。2. 删除该指针对象。

但实质上是四个步骤：
1. malloc() 分配一块内存空间，大小是 sizeof(A) 
2. 在该内存空间上初始化一个A类对象
3. 调用A的析构函数
4. 回收该内存空间

new 负责了前两个步骤，delete 负责了后面两个步骤。那我们能不能在代码中分别控制这四个步骤呢？答案是可以的，
```cpp
#include <new>
#include <iostream>
using namespace std;

class A {
public:
    A(int a) : a_(a) {}
    int a_;
    ~A() {}
};

int main(){
    // 1. 分配空间
    A* ap = (A*) operator new(sizeof(A));
    // 2. 在ap位置上初始化对象
    new(ap) A(1);

    cout << ap->a_ << endl; // 1

    // 3. 调用析构函数
    ap->~A();
    // 4. 回收内存
    operator delete(ap);

    cout << ap->a_ << endl; // ????
}
```
- operator new(size_t) 表示为指针ap指向的地址上分配一块 sizeof(A) 的内存空间。
- new(ap) A(1) 表示在 ap 地址上初始化一个A类对象。[*[wiki]*](https://zh.wikipedia.org/wiki/New_(C%2B%2B))
- operator delete(T*) 用于回收指针所指向的内存。

## allocator
事实上，allocator这东西就是允许我们能够对以上四个步骤进行控制的封装。

我们分别给四个步骤取个名字：
1. allocate
2. construct
3. destroy
4. deallocate

```cpp
#include <memory>
#include <iostream>
using namespace std;
// class A ...
int main() {
    allocator<A> alloc;
    A* ap = alloc.allocate(1);
    alloc.construct(ap, 5);
    cout << ap->a_ << endl;  // 5
    alloc.destroy(ap);
    alloc.deallocate(ap, 1);
    cout << ap->a_ << endl;  // ????
}
```
- allocate(num) 分配 num * sizeof(A) 的空间
- construct(T*, Arg...) 第一个参数是某类型的指针；第二个参数是可变参数，往构造函数输入的参数
- destroy(T*) 调用指针对象的析构函数。但注意，只是针对一个对象调用。
- deallocate(T*, num) 第一个参数是需要回收内存的首地址；第二个参数是需要回收的数量，与 allocate(num) 对应，但在标准库里面这个参数是没有被使用的，因为无论是 delete 还是 operator delete 都能自动确认需要回收多少个元素的空间。

对于指向数组的指针，也可以这么做，
```cpp
allocator<A> alloc;
A* ap = alloc.allocate(2);
alloc.construct(ap, 5);
alloc.construct(ap+1, 6);
alloc.destroy(ap);
alloc.destroy(ap+1);
alloc.deallocate(ap, 2);
```

注意到一件事情，`allocator<T>::construct(U*, ...)` 的第一个参数的类型并不一定是 allocator 的泛型参数，而是函数自己的模板参数，意味着你可以传入任意类型指针。
```cpp
class Data {
public:
    int a_;
    Data() {}
};

int main() {
    allocator<Data> alloc;
    Data* dp = alloc.allocate(1);
    alloc.construct(&dp->a_, 100);
    cout << dp->a_ << endl; // 100

    double* s;
    alloc.construct(&s, new double(88.2));
    cout << *s << endl; // 88.2
}
```

## allocator 实现
这里实现我们自己实现一个 allocator 以便于我们能够对其底层较为理解。
```cpp
#include <new>
#include <iostream>
using namespace std;

namespace my_alloc {
    // allocate的实际实现，简单封装new，当无法获得内存时，报错并退出
    template<class T>
    inline T* _allocate(ptrdiff_t size, T*) {
        set_new_handler(0);
        T* tmp = (T*) (::operator new((size_t) (size * sizeof(T))));
        if (tmp == 0) {
            cerr << "out of memory" << endl;
            exit(1);
        }
        return tmp;
    }

    // deallocate的实际实现，简单封装delete
    template<class T>
    inline void _deallocate(T* buffer) { ::operator delete(buffer); }

    // construct的实际实现，直接调用对象的构造函数
    template<class T1, class T2>
    inline void _construct(T1* p, const T2& value) {
        new(p) T1(value);
    }

    // destroy的实际实现，直接调用对象的析构函数
    template<class T>
    inline void _destroy(T* ptr) { ptr->~T(); }

    template<class T>
    class allocator {
    public:
        typedef T value_type;
        typedef T* pointer;
        typedef const T* const_pointer;
        typedef T& reference;
        typedef const T& const_reference;
        typedef size_t size_type;
        typedef ptrdiff_t difference_type;

        // 构造函数
        allocator() { return; }

        template<class U>
        allocator(const allocator<U>& c) {}

        // rebind allocator of type U
        template<class U>
        struct rebind {
            typedef allocator<U> other;
        };
        // allocate，deallocate，construct和destroy函数均调用上面的实际实现
        // hint used for locality. ref.[Austern],p189
        pointer allocate(size_type n, const void* hint = 0) {
            return _allocate((difference_type) n, (pointer) 0);
        }
        void deallocate(pointer p, size_type n) { _deallocate(p); }
        void construct(pointer p, const T& value) { _construct(p, value); }
        void destroy(pointer p) { _destroy(p); }
        pointer address(reference x) { return (pointer) &x; }
        const_pointer const_address(const_reference x) { return (const_pointer) &x; }
        size_type max_size() const { return size_type(UINT_MAX / sizeof(T)); }
    };
}

int main() {
    my_alloc::allocator<int> alloc;
    int* ip = alloc.allocate(1);
    alloc.construct(ip, 3);
    cout << *ip << endl;    // 3
    alloc.destroy(ip);
    alloc.deallocate(ip, 1);
}
```

## allocator_traits
allocator_traits 即 allocator 的类型萃取器。这东西的出现就是为了能够容纳所有空间分配器的模板分配器。只要把你的分配器定义了规定的别名，就能被它所兼容。
```cpp
// 类型太长，这里定义一个别名
typedef allocator_traits<allocator<A>> alloc_t;
allocator<A> alloc;
// 使用静态方法
A* ap = alloc_t::allocate(alloc, 1);
alloc_t::construct(alloc, ap, 4);
cout << ap->a_ << endl;
alloc_t::destroy(alloc, ap);
alloc_t::deallocate(alloc, ap, 1);
cout << ap->a_ << endl;
```
那为什么不再尝试通过继承 std::allocator 来实现自定义的 allocator 呢？是可以的。但很明显的一个问题是 std::allocator 里的函数实在太冗余了，所以 C++17/20 大刀阔斧地砍了很多函数。[*[ref]*](https://zh.cppreference.com/w/cpp/memory/allocator)

如果你要自定义一个 allocator 并用于 allocator_traits 中 ，那么只要实现 allocate， deallocate 和 max_size 即可。

要定义一个自定义 allocator，只需要下面这样极简版本：
```cpp
template<class T>
class MyAllocator {
public:
    typedef T value_type;
    typedef T* pointer;
    typedef size_t size_type;
    typedef ptrdiff_t difference_type;

    MyAllocator() = default;
    template<class U>
    struct rebind {
        typedef allocator<U> other;
    };
    pointer allocate(size_type n, const void* hint = 0) {
        set_new_handler(0);
        T* tmp = (T*) (::operator new((size_t) (n * sizeof(T))));
        if (tmp == 0) {
            cerr << "out of memory" << endl;
            exit(1);
        }
        return tmp;
    }
    void deallocate(pointer p, size_type n) { ::operator delete(p); }
    size_type max_size() const { return size_type(UINT_MAX / sizeof(T)); }
};

int main() {
    typedef allocator_traits<MyAllocator<A>> alloc_t;
    MyAllocator<A> alloc; // 实例
    A* ap = alloc_t::allocate(alloc, 1);
    alloc_t::construct(alloc, ap, 4);
    cout << ap->a_ << endl;
    alloc_t::destroy(alloc, ap);
    alloc_t::deallocate(alloc, ap, 1);
    cout << ap->a_ << endl;
}
```

## rebind
其中的 rebind 是用来改变泛型类型的。
```cpp
// 等价于 allocator<double> alloc
my_alloc::allocator<int>::rebind<double>::other alloc;
double* ip = alloc.allocate(1);
alloc.construct(ip, 3.2);
cout << *ip << endl;
alloc.destroy(ip);
alloc.deallocate(ip, 1);
```

为什么要有这个设计呢？这个设计在STL里面有得很多，事实上，每个容器的第二个模板参数就是传进一个 allocator，默认是STL自己的 allocator 或者是 std::allocator，那么默认情况下，容器的泛型和 allocator 的泛型是一致的。
```cpp
template<typename _Tp, typename _Alloc = std::allocator<_Tp> >
class vector ...
```
而容器对于你传入的 allocator 类型是不信任的。

STL容器拿到一个 allocator 后一般会做如下几个事情：

1. 不管 allocator 的泛型是什么，一律转为容器泛型
```cpp
typedef typename alloc_traits<_Alloc>::template rebind<_Tp>::other   _Tp_alloc_type;
```
2. 转为 allocator_traits
```cpp
typedef alloc_traits<_Tp_alloc_type>  _Tp_alloc_traits;
```
3. 用 allocator_traits 重新绑定为 node 节点类型的 allocator
```cpp
typedef typename _Tp_alloc_traits::template rebind<_List_node<_Tp> >::other  _Node_alloc_type;
```
4. 把 `allocator<node>` 放入 allocator_traits
```cpp
typedef alloc_traits<_Node_alloc_type> _Node_alloc_traits;
```

## 有状态与无状态
标准库里面的 std::allocator 是无状态的，即不同的实例之间并没有本质区别，有以下特点：
1. 实例的比较是相同的
2. 任一实例分配的内存可以由任一实例回收
```cpp
allocator<A> alloc;         // 实例1
A* ap = alloc.allocate(1);
alloc.construct(ap, 5);

allocator<A> alloc2;        // 实例2
cout << (alloc2 == alloc) << endl; // 1
alloc2.deallocate(ap, 1);
cout << ap->a_ << endl; // ???
```
`==`是怎么判断的？如果你打开源码，你会发现重载的 operator== 总是返回 true。

那么有状态的意思自然就是无状态相反了。C++是支持你自己定义一个有状态的 allocator 的，一个有状态的的 allocator 可以有以下实现方式：
1. 记录所有从该 allocator 中分配的地址，回收时如果不是属于该 allocator 就不回收。
2. 手动维护内存池，不同的 allocator 使用不同的内存空间范围。


