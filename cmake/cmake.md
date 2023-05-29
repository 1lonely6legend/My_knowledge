# cmake

首先在使用cmake编译的项目中,根目录和每个目标目录中都应该有一个CMakeLists.txt文件,

在编译时 我们需要在根目录下创建一个build文件

```
mkdir build
cd build
cmake -S .. -B .
```

最后一条命令是将上级目录进行编译,并将构建的输出写入当前目录(build)

然后cmake会在build目录中创建一个makefile文件供我们运行

> 要使用gcc编译器才会生成makefile文件,调用默认的vscv编译器不会生成makefile

## c++编译,可执行文件和库

c++项目中每个文件夹都应该是库或者是可执行程序. 每个文件夹中的CMakeLists.txt文件都规定了这个文件夹中的库或者可执行文件.



## 常见的C++库的结构

一个库将包含一个“src”文件夹、一个“tests”文件夹和一个“include”文件夹。

1. “src”文件夹将包含“.cc”文件形式的 C++ 功能。此文件夹包含一个库。

2. “tests”文件夹将包含测试“src”文件夹中库功能的可执行文件。

3. “include”文件夹将包含“.h”文件，其中包含“src”文件夹中的库声明。

所以我们需要在**src文件夹中包含一个“CMakeLists.txt”文件来定义库**，“**tests”文件夹应该包含一个“CMakeLists.txt”文件来定义用于测试“src”库功能的可执行文件.**

### 根目录的CMakeLists.txt

该文件将包含一些 CMake 设置调用。该文件还将引用其他库和可执行目录，以便它们的 CMake 文件也被编译。要将“src”和“tests”“CMakeLists.txt”文件添加到构建中，

请使用“add_subdirectory”命令

```
add_subdirectory(src)
add_subdirectory(tests)
```

### src库中的CMakeLists.txt

由于“src”文件夹包含我们的库代码，我们需要使用“src”目录中的“CMakeLists.txt”文件来定义库。为此，我们首先执行“add_library”命令。然后，我们需要包含“include”目录，以便我们的函数声明对库文件可见：

- `add_library(my_library file1.cc file2.cc)`

这将库命名为“my_library”并包含“file1.cc”和“file2.cc”文件以进行编译。

- `target_include_directories(my_library PUBLIC ../include)`

### tests中的CMakeLists.txt

要在“tests”中构建可执行文件，我们需要使用“tests”文件夹中的“CMakeLists.txt”文件。对于“tests”文件夹中的每个可执行文件，我们需要使用“add_executable”命令添加可执行文件，并将“src”文件夹中的库链接到可执行文件。这些命令总结如下：

- `add_executable(test main.cc helper.cc helper.h)`
- `target_link_libraries(test PUBLIC my_library)`

再次，“PUBLIC”和“PRIVATE”确定包含此可执行文件的目标是否可以访问“my_library”库。

### target_link_libraries中public和private的区别

![img](https://miro.medium.com/max/868/1*TzL7ZFcJhgL8dZ318MieVQ.png)

在此图中，“库 1”已公开包含“public_includes/”目录。因此，当“库 2”将“库 1”链接到自身时，“库 2”可以访问“public_includes/”目录（以及该目录中的所有头文件）。它还可以从“库 1”目标访问已编译的“.a”文件。但是，它不能访问“private_includes/”目录，因为它是私人包含的。

> 相当于继承类中的public和private