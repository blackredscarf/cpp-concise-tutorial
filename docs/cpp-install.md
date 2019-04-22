# C++ 安装

## windows

首先下载并安装 [Visual Studio](https://visualstudio.microsoft.com)，安装时选择“C++桌面开发”模块即可，大概6G左右。虽然windows上开发不一定要用VS，但VS和windows都是微软做的，VS提供了一系列辅助工具，以后在windows平台上开发总是会用到的。

VS内置了C++编译器可供使用。但从我个人角度而言，为了尽量与Linux上保持一致的开发体验。我建议安装MinGW作为编译器，MinGW是GCC的移植版本。

下载 [MinGW](https://sourceforge.net/projects/mingw-w64/files/mingw-w64/)，该网站下面有一系列版本提供下载。如果你不知道该下哪个，可以下载和我相同的版本 [GCC-8.1.0 x86_64-posix-seh](https://sourceforge.net/projects/mingw-w64/files/Toolchains%20targetting%20Win64/Personal%20Builds/mingw-builds/8.1.0/threads-posix/seh/x86_64-8.1.0-release-posix-seh-rt_v6-rev0.7z)，解压到你电脑上某个存放库文件的地方。把bin目录写入环境变量。

打开CMD，执行以下命令可以看到版本。
```
g++ --version
```

下载[CMake](https://cmake.org/download/)并安装。
把安装目录下的bin目录加到环境变量中。

打开CMD，执行以下命令可以看到版本。
```
cmake --version
```


## 编辑器与配置
推荐使用 CLion 或 Visiual Studio 。

### CLion 
CLion 内置了一个CMake，为了统一，你可以设置自己的CMake。设置方式：File -> settings -> Build, ... -> Toolchains 指定自己的CMake。
