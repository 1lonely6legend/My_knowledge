# C++友元函数和友元类（C++ friend）详解

私有成员只能在类的成员函数内部访问，如果想在别处访问对象的私有成员，只能通过类提供的接口（成员函数）间接地进行。这固然能够带来数据隐藏的好处，利于将来程序的扩充，但也会增加程序书写的麻烦。

[C++](http://c.biancheng.net/cplus/) 是从结构化的C语言发展而来的，需要照顾结构化设计程序员的习惯，所以在对私有成员可访问范围的问题上不可限制太死。

C++ 设计者认为， 如果有的程序员真的非常怕麻烦，就是想在类的成员函数外部直接访问对象的私有成员，那还是做一点妥协以满足他们的愿望为好，这也算是眼前利益和长远利益的折中。因此，C++ 就有了**友元（friend）**的概念。打个比方，这相当于是说：朋友是值得信任的，所以可以对他们公开一些自己的隐私。

友元分为两种：友元函数和友元类。

## 友元函数

在定义一个类的时候，可以把一些函数（包括全局函数和其他类的成员函数）声明为“友元”，这样那些函数就成为该类的友元函数，在友元函数内部就可以访问该类对象的私有成员了。

将全局函数声明为友元的写法如下：

> friend  返回值类型  函数名(参数表);

将其他类的成员函数声明为友元的写法如下：

> friend  返回值类型  其他类的类名::成员函数名(参数表);

但是，不能把其他类的私有成员函数声明为友元。

关于友元，看下面的程序示例。

```c++
#include<iostream>
using namespace std;
class CCar;  //提前声明CCar类，以便后面的CDriver类使用
class CDriver
{
  public:    
  void ModifyCar(CCar* pCar);  //改装汽车
};
class CCar{
  
  private:    
  int price;    
  friend int MostExpensiveCar(CCar cars[], int total);  //声明友元    
  friend void CDriver::ModifyCar(CCar* pCar);  //声明友元
};
void CDriver::ModifyCar(CCar* pCar)
{    
  pCar->price += 1000;  //汽车改装后价值增加
}
  int MostExpensiveCar(CCar cars[], int total)  //求最贵气车的价格
  {    
    int tmpMax = -1;    
    for (int i = 0; i<total; ++i)        
      if (cars[i].price > tmpMax)           
        tmpMax = cars[i].price;    
    return tmpMax;
  }
  int main()
  {    
    return 0;
  }
```

这个程序只是为了展示友元的用法，所以 main 函数什么也不做。

第 3 行声明了 CCar 类，CCar 类的定义在后面。之所以要提前声明，是因为 CDriver 类的定义中用到了 CCar 类型（第7行），而此时 CCar 类还没有定义，编译会报错。

不要第 3 行，而把 CCar 类的定义写在 CDriver 类的前面，是解决不了这个问题的，因为 CCar 类中也用到了 CDriver 类型（第14行），把 CCar 类的定义写在前面会导致第 14 行的 CDriver 因没有定义而报错。C++ 为此提供的解决办法是：可以简单地将一个类的名字提前声明，写法如下：

class 类名;

尽管可以提前声明，但是在一个类的定义出现之前，仍然不能有任何会导致该类对象被生成的语句。但使用该类的[指针](http://c.biancheng.net/c/80/)或引用是没有问题的。

第 13 行将全局函数 MostExpensiveCar 声明为 CCar 类的友元，因此在第 24 行可以访问 cars[i] 的私有成员 price。同理，第 14 行将 CDriver 类的 ModifyCar 成员函数声明为友元，因此在第 18 行可以访问 pCar 指针所指向的对象的私有成员变量 price。

## 友元类

一个类 A 可以将另一个类 B 声明为自己的友元，类 B 的所有成员函数就都可以访问类 A 对象的私有成员。在类定义中声明友元类的写法如下：

> friend class 类名;

来看如下例程：

```c++
class CCar
{
  private:    
  int price;    
  friend class CDriver;  //声明 CDriver 为友元类
};
class CDriver
{
  public:    
  CCar myCar;    
  void ModifyCar()  //改装汽车    
  {        
    myCar.price += 1000; //因CDriver是CCar的友元类，故此处可以访问其私有成员    
  }
};
int main()
{   
  return 0;
}
```

第 5 行将 CDriver 声明为 CCar 的友元类。这条语句本来就是在声明 CDriver 是一个类，所以 CCar 类定义前面就不用声明 CDriver 类了。第 5 行使得 CDriver 类的所有成员函数都能访问 CCar 对象的私有成员。如果没有第 5 行，第 13 行对 myCar 私有成员 price 的访问就会导致编译错误。

一般来说，类 A 将类 B 声明为友元类，则类 B 最好从逻辑上和类 A 有比较接近的关系。例如上面的例子，CDriver 代表司机，CCar 代表车，司机拥有车，所以 CDriver 类和 CCar 类从逻辑上来讲关系比较密切，把 CDriver 类声明为 CCar 类的友元比较合理。

友元关系在类之间不能传递，即类 A 是类 B 的友元，类 B 是类 C 的友元，并不能导出类 A 是类 C 的友元。“咱俩是朋友，所以你的朋友就是我的朋友”这句话在 C++ 的友元关系上 不成立。