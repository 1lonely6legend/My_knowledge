# explicit与implicit（显示转换与隐式转换）

在C++中，我们有时可以将**构造函数**用作**自动类型转换函数**。但这种自动特性并非总是合乎要求的，**有时会导致意外的类型转换**，因此，C++新增了关键字explicit，用于关闭这种自动特性。即:

> 被explicit关键字修饰的**类构造函数**，**不能进行自动地隐式类型转换**，只能显式地进行类型转换。

首先, C++中的explicit关键字**只能用于修饰只有一个参数或只有一个未提供默认值的类构造函数**, 它的作用是表明该构造函数是显示的, 而非隐式的, 跟它相对应的**另一个关键字是implicit, 意思是隐藏的,类构造函数默认情况下即声明为implicit(隐式)**.

注意：只有一个参数的构造函数，或者构造函数有n个参数，但有n-1个参数提供了默认值，这样的情况才能进行类型转换。

>  **注意是构造函数、构造函数、构造函数**

```cpp
class Demo{
public:
    Demo(){};//没有参数，无法进行隐示类型转换
    Demo(int a){};//有唯一参数，可以进行隐示类型转换
    Demo(int a,int b){};//多个参数且无默认值，无法进行隐示类型转换
    Demo(int a=0,int b=0,int c,int d=0){};//只有一个参数无默认值，可以进行隐示类型转换   
};
```

下面的代码中, “Demo de1=10;” 这句为什么是可以的呢? 在C++中, **如果的构造函数只有一个参数时, 那么在编译的时候就会有一个缺省的转换操作**:将该构造函数对应数据类型的数据转换为该类对象. 也就是说 “Demo de1=10;” 这段代码, 编译器自动将整型转换为Demo类对象, 实际上**等同于下面的操作:Demo de1=Demo(10)**。


```cpp
#include <iostream>
using namespace std;

class Demo{
public:
    Demo(){};//没有参数，无法进行隐示类型转换
    Demo(int a){};//有唯一参数，可以进行隐示类型转换
    Demo(int a,int b){};//多个参数且无默认值，无法进行隐示类型转换
		//Demo(int a=0,int b=0,int c,int d=0){};//只有一个参数c无默认值，可以进行隐示类型转换
};
int main()
{
    Demo de=Demo(10);//可以
    Demo de1=10;//可以
    
    return 0;
}

```

---

explicit（明确的）关键字的作用就是防止类构造函数的隐式自动转换.

```cpp
#include <iostream>
using namespace std;

class Demo{
public:
    Demo(){};//没有参数，无法进行隐示类型转换
    explicit Demo(int a){};//有唯一参数，可以进行隐示类型转换，但加了explicit关键字
    Demo(int a,int b){};//多个参数且无默认值，无法进行隐示类型转换
		//Demo(int a=0,int b=0,int c,int d=0){};//只有一个参数无默认值，可以进行隐示类型转换

};
int main()
{
    Demo de=Demo(10);//可以
    Demo de1=10;//不可以，构造函数加了explicit关键字，不能进行隐式转换

    return 0;
}

```

---

举一个实际上使用explicit的例子

```cpp
#include <iostream>
using namespace std;

class A{
	public:
		A(int x){//A的一个构造函数
		cout<<"我被用了"<<endl;
	}
};

void f(A a){//本意函数的参数是一个类型A的对象a
}
int main( ){
	f(1);// 被隐式转换为f(A(1)) ，本来是1却被自动调用了A(1)这就是拷贝初始化
	//输出："我被调用了"
	return 0;
}
```

