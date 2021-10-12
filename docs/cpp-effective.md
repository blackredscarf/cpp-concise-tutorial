# 良好的C++代码规范

## 独立的new对象放入智能指针
Effective C++的第17条提到，当我们创建智能指针时，最好独立一行语句来创建。

错误示范：
```cpp
processWidget(share_ptr<Widget>(new Widget), priority());
```
C++编译器对于这行的执行顺序是不定的。它的顺序可能是：

1. new Widget
2. priority()
3. share_ptr()

如果priority()抛出错误，则新建的Widget指针永远无法回收。

正确示范：
```cpp
auto widgetPtr = share_ptr<Widget>(new Widget)
processWidget(widgetPtr, priority());
```

## 单例模式
1. 把构造函数、拷贝构造函数和赋值函数设为私有函数。防止新建实例。
2. 提供一个公有、静态方法，用于获取实例。实例可以通过local static构建，C++11编译器保证静态变量的构造是线程安全的。（Effective C++推荐）
```cpp
class Singleton {
private:
	Singleton() { };
	Singleton(const Singleton&);
	Singleton& operator=(const Singleton&);
public:
	static Singleton& getInstance() {
		static Singleton instance;  // local static (thread-safe)
		return instance;
	}
};
```