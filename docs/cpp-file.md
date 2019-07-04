# C++ 文件

## 写入文件
ofstream类使用被缓冲的输出，因此程序在创建像fout这样的ofstream对象时，将为输出缓冲区分配空间。
```cpp
// ofstream fout;
// fout.open("hello.txt");
ofstream fout("hello.txt");
fout << "hello C++!" << endl;
fout << "hello Java!" << endl;
fout.close();
```
注意，输出的文件可能在一些build的目录下。

## 读取文件
ifstream对象与输出一样，通过缓冲，传输数据的速度比逐字节传输要快得多。
```cpp
ifstream fin("hello.txt");
if(fin.is_open()){
    string data;
    char ch;
    while(fin.get(ch)){
        data += ch;
    }
    cout << data << endl;
}
fin.close();
```
`is_open()`用于检查是否成功打开文件。

### 按行读取
搭配`string::getline`可以读一行到string中。
```cpp
ifstream fis("hello.txt");
if(fis.is_open()){
    string line;
    while(getline(fis, line)){
        cout << line << endl;
    }
}
fis.close();
```

### 读取整个文件
搭配`stringstream`可以读整个文件到string中。
```cpp
ifstream fis("hello.txt");
if(fis.is_open()){
    stringstream ss;
    ss << fis.rdbuf();
    string data = ss.str();
    cout << data << endl;
}
fis.close();
```

## 模式
- ios::in	以输入方式打开文件
- ios::out	以输出方式打开文件（这是默认方式），不会自动创建文件，如果已有此名字的文件，则将其原有内容全部清除
- ios::app	以输出方式打开文件，写入的数据添加在文件末尾
- ios::ate	打开一个已有的文件，文件指针指向文件末尾
- ios::trunc	打开一个文件，如果文件已存在，则删除其中全部数据，如文件不存在，则建立新文件。
- ios:: binary	以二进制方式打开一个文件，如不指定此方式则默认为ASCII方式
- ios::in l ios::out	以输入和输出方式打开文件，文件可读可写
- ios:: out | ios::binary	以二进制方式打开一个输出文件
- ios::in l ios::binar	以二进制方式打开一个输入文件

例如：
```cpp
ofstream f;
f.open("hello.txt", ios::app);
f << "hello python!" << endl;
f.close();
```

旧版本中会有ios::nocreate和ios::noreplace。因为平台不兼容，就删掉了。

## 流的指针位置
通过`seekg()`设置流的指针位置，下次读取时将从这里开始读。`tellg()`可以获取当前指针位置。
```cpp
ifstream fis("hello.txt");
if(fis.is_open()){
    fis.seekg(6);   // 移动到第6个位置
    cout << fis.tellg() << endl; // 查看当前位移
    
    string data;
    getline(fis, data);
    cout << data << endl; // "C++!"

    fis.seekg(-7, ios::end); // 从结束位置往前算7个字符开始读，包括\r\n
    getline(fis, data);
    cout << data << endl; // "Java!"
}
fis.close();
```

### 读取二进制
一件麻烦事是C++17之前没有`byte`这种数据类型，这么多年一直都有`unsigned char`来读，因为刚好占8位，相当于1字节。
```cpp
ifstream fis("test.png", ios::binary);
if(fis.is_open()){
    vector<unsigned char> buffer(std::istreambuf_iterator<char>(fis), {});
    cout << buffer.size() << endl;
}
fis.close();
```

## 状态记录
为什么需要记录状态？比如，当我们在读取或写入发送错误时，函数不会抛出错误，我们需要会通过`fail()`和`bad()`函数判断是否出了问题。

文件状态包含下面几个：
- fail() 检查是否发生了**可恢复**的错误。例如：读入格式错误，当想要读入一个整数，而获得了一个字母的时候。
- bad() 检查是否已发生**不可恢复**的错误。例如：当我们要对一个不是打开为写状态的文件进行写入时，或者我们要写入的设备没有剩余空间的时候。
- good() 检查是否读取或写入正常。一切正常返回true。
- eof() 如果读文件到达文件末尾，返回true。

比如，下面进行错误检查。
```cpp
ifstream fis("hello.txt");
if(fis.is_open()){
    int a;
    fis >> a;
    if(fis.fail()){
        cout << "fail" << endl;
    }
}
fis.close();
```


## 读写
`fstream`支持同时读写，只需在第二个参数注明读写模式即可。注意，`ios::out`是不支持自动创建文件的，你还需要添加上`ios::trunc`。
```cpp
fstream f;
f.open("hero.txt", ios::in | ios::out | ios::trunc);

if(!f.is_open()){
    cout << "error" << endl;
}
f << "iron man" << endl;

char ch;
string data;
while(f.get(ch)) {
    data += ch;
}
cout << data << endl;
f.close();
```
fstream 虽然同时支持读写，但明显没有 iftream 和 ofstream 好用，它并不支持 string::getline。

