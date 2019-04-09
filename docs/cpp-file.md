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


## 模式
- ios::in	以输入方式打开文件
- ios::out	以输出方式打开文件（这是默认方式），如果已有此名字的文件，则将其原有内容全部清除
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







