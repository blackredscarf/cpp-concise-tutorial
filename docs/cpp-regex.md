# c++ 正则表达式
C++11 正式将正则表达式的的处理方法纳入标准库的行列，从语言级上提供了标准的支持，不再依赖第三方。

## 匹配存在
`std::regex_match`匹配后将返回一个说明是否匹配成功bool值。
```cpp
#include <regex>
#include <iostream>
using namespace std;

int main(){
    regex txt_regex("[a-z]+\\.txt");
    bool b = regex_match("hello.txt", txt_regex);
    cout << b << endl;  // 1
}
```

## 匹配查找
`std::smatch`是用来存储匹配结果的对象。
```cpp
regex filename_pattern("(\\w+)\\.txt");
smatch matcher;
string file = "hello.txt";
if(regex_match(file, matcher, filename_pattern)){
    if(matcher.size() == 2){
        cout << matcher[0].str() << endl;   // hello.txt
        cout << matcher[1].str() << endl;   // hello
    }
}
```


