# C++ 安装

## Windows

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

## Linux
Linux默认安装了gcc，你可以通过`gcc -v`查看当前系统的gcc版本，Ubuntu 16的默认版本是5.4，可以全面支持C++14的语法。如果你需要用到C++17的语法，至少需要更新到gcc 7.0 。

这里提供一个gcc 8.0在Ubuntu下的安装方法。
```
sudo apt install software-properties-common
sudo add-apt-repository ppa:ubuntu-toolchain-r/test
```
```
sudo apt install gcc-8 g++-8
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-8 80 --slave /usr/bin/g++ g++ /usr/bin/g++-8 --slave /usr/bin/gcov gcov /usr/bin/gcov-8
```
```
sudo update-alternatives --config gcc
```
```
gcc -v
g++ -v
```

CMake的话，直接apt安装。
```
sudo apt-get install cmake
```
如果觉得apt提供的版本有点旧，建议手动下载然后编译安装。编译安装的话要求你的设备至少大于2G内存，不然会出错。
```
version=3.13
build=2
mkdir ~/temp
cd ~/temp
wget https://cmake.org/files/v$version/cmake-$version.$build.tar.gz
tar -xf cmake-$version.$build.tar.gz
cd cmake-$version.$build/
```
```
./bootstrap
make -j$(nproc)
sudo make install
```
```
cmake --version
```


## 编辑器与配置
推荐使用 CLion 或 Visiual Studio 。

### CLion 
CLion 内置了一个CMake，为了统一，你可以设置自己的CMake。设置方式：File -> settings -> Build, ... -> Toolchains 指定自己的CMake。
