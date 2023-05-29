# proto文件

>类似Json？ 一种更紧凑的通信定义

## 1 什么是protocol Buffer
>https://blog.csdn.net/ouyang_peng/article/details/126929753

### 1.1 Protocol Buffer 简介
Protocol Buffer是一种免费的开源跨平台的数据格式，用于**序列化结构数据**。

其在开发程序以通过网络互相通信(apollo中就是各个模块)以及存储数据时很有用

该方法设计描述某些数据结构的**接口描述语言**和从该描述生成源代码（.proto）以及生成或解析表示结构化数据的字节流的程序。

该方法具有以下的优点:
	1.语言中立，平台中立，便于扩展与移植。
	2.类似XML但是更快更小更简单。
	3.只需要定义一次数据的结构化方式，然后就可以使用其生成的源代码轻松地对**结构化的数据进行写入和读取**。
### 1.2 Protocol Buffer如何进行定义
后缀名为.proto 的文件就是Protocol Buffer的文本定义文件，我们可以轻松的阅读并理解。

更多关于Protocol Buffer的资料可以参考
· <https://github.com/protocolbuffers/protobuf>
· <https://developers.google.com/protocol-buffers>
#### 1.2.1 什么是.pb.h和.pb.cc文件？
>结论：其实这两种文件格式就是Proto编译器根据.proto文件，编译生成对应的头文件（.h）和实现文件（.cc）

Proto编译器为每个源代码文件创建一个头文件和一个实现文件

- 对于头文件或者实现文件，扩展名由proto换成.pb.h和.pb.cc
- proto路径（由`--proto_path =`或者`-l`命令行来进行指定）被替换为输出路径（由`--cpp_ou`标志指定）
因此如果按照如下的方式调用编译器：

>`proto --proto_path=src --cpp_out=build/gen src/foo.proto src/bar/bar.proto`
> 其中指定了proto文件的路径 src         以及输出路径build/gen

编译器将读取文件`src/foo.proto`和`src/bar/bar.proto`，并生成四个输出文件：

+ build/gen/foo.pb.h
+ build/gen/foo.pb.cc
+ build/gen/bar/bar.pb.h
+ build/gen/bar/bar.pb.cc    // 因为bar文件的输入路径有两层

#### 1.2.2 设置一个示例
 定义一个polyline.proto,如下所示：
 ```protobuf
 //polyline.proto
 //相当于提前声明来节省空间
 //我们可以使用我们定义的message来编码和解码对象
 syntax = "proto2";
 
 message Point{	//代表一种信息类型Point
 	required int32 x = 1;
 	required int32 y = 2;
 	optional string label = 3;
 }
 
 message Line{
 	required Point start = 1;
 	required Point end = 2;
 	optional string label = 3;
 }
 
 message Polyline{
 	required Point Point = 1;
 	optional string label = 2;
 }
 
 ```

`Point`消息定义了两个强制性数据`x`和`y`，`label`是可以选的。但是消息中的每一个数据项都有着自己的`tag`，例如`x`的`tag`是`1`。

`Line`和`Polyline`消息都使用了`Point`消息，展示了组合在`Protocol Buffer`中的工作方式。`Polyline`有着一个`Point point`的数据项，类似一个数组了。

随后可以编译此模式以供一种或多种编程语言使用。`Google`提供了一种编译器叫`Protoc`。此编译器可以为`cpp`、`java`以及`python`提供输出。还有其他类型的编译器，总计可以为20多种语言提供相关的输出。

例如，将上述的示例代码进行编译后输出，我们可以使用一段C++代码来使用它。

```c++
// polyline.cpp
#include "polyline.pb.h"  
// polyline.proto编译生成的头文件，头文件中引用了polyline.pb.cc，包含了具体的实现

Line* createNewLine(const std::string& name) {
  // create a line from (10, 20) to (30, 40)
  Line* line = new Line;//给Line类型的数据结构申请一块内存空间，并将地址传递给line类型的指针
  line->mutable_start()->set_x(10);//这具体的函数mutable_start应该就是自动生成的函数吧，我猜的
  line->mutable_start()->set_y(20);
  line->mutable_end()->set_x(30);
  line->mutable_end()->set_y(40);
  line->set_label(name);
  return line;
}

Polyline* createNewPolyline() {
  // create a polyline with points at (10,10) and (20,20)
  Polyline* polyline = new Polyline;
  Point* point1 = polyline->add_point();
  point1->set_x(10);
  point1->set_y(10);
  Point* point2 = polyline->add_point();
  point2->set_x(20);
  point2->set_y(20);
  return polyline;
}
```

