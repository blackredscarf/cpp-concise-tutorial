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
