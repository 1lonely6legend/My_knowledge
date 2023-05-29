## virtual关键字

### virtual使用简介

virtual在基类中确定了一个虚拟函数，在派生类中使用重写虚拟函数来实现对基类虚拟函数的覆盖

```cpp
class Base{
	public:
		Base(){};
		virtual void print(){
			cout<<"Base";
	}
};

class Derived : public Base{
	Public:
		void print(){
			cout<<"Derived";
		}
};
```

当类型为基类Base的指针指向了一个Drived型的对象时，调用Print()函数，此时调用的是Drived类中的print函数而不是Base类中的函数。

> 这是面向对象中**多态性**的一种体现。

### 函数重载和覆盖的不同点

- 重载的函数位于同一类中，覆盖(override)的函数应位于有继承关系的不同类中
- 重载的函数要求函数名称相同，但是参数不同，由此编译器在可以通过参数来判断调用的是哪个函数。而覆盖的几个函数必须要求函数名、参数、返回值都相同。
- 重载和关键词virtual无关。覆盖的函数在前面必须加上关键词virtual

### 关于函数覆盖的一些隐藏规则：

1. 如果**派生类的函数与基类的函数同名**，但是参数不同。此时，不论有无virtual关键字，基类的函数将被隐藏（注意别与重载混淆）。
2. 如果派生类的函数与基类的函数同名，并且参数也相同，但是基类函数没有virtual关键字。此时，基类的函数被隐藏（注意别与覆盖混淆）。

### 纯虚函数virtual ··· = 0,可以用来定义接口

当定义纯虚函数时，可以指定让**继承类所实现的接口**。但是基类并不能被基类调用，纯虚函数也不需要实现，只需要声明即可。

纯虚函数的声明如下所示：

```cpp
class Asd{
	public:
		virtual ostream& print(ostream&=cout) = 0;//函数声明后面紧跟赋值为0
};
```
如果一个**类包含纯虚拟函数，那么就是一个抽象基类**，如果你试图创建一个抽象基类的对象会报错

**C++中的接口通常被定义为一个抽象基类**，也就是只包含纯虚函数的类。纯虚函数是一种特殊的虚函数，它没有实现，只有函数原型。

它的作用就是要求实现该接口的具体类必须提供该函数的具体实现。

### 虚析构

如果一个**类用作基类**，我们通常需要**virtual来修饰它的析构函数**，这点很重要。如果基类的析构函数不是虚析构，当我们用delete来释放基类指针(它其实指向的是派生类的对象实例)占用的内存的时候，只有基类的析构函数被调用，而派生类的析构函数不会被调用，这就可能引起内存泄露。**如果基类的析构函数是虚析构，那么在delete基类指针时，继承树上的析构函数会被自低向上依次调用，即最底层派生类的析构函数会被首先调用，然后一层一层向上直到该指针声明的类型**。

使用虚析构是防止只调用了基类的析构函数，而没有调用继承类的析构函数。

### 虚继承 virtual public

举个例子，首先我们定义基类A，再从基类A中派生出两种不同的继承类B和C。

```cpp
class A{
	public:
		fun();
};

class B : public A{```};
class C : public A{```};
```

而当我们使用B和C作为基类派生出D时，就出现了问题。每个D的对象中包含了B 和 C的基类对象数据，即包含两个A对象。

而我们实际上只需要一个A对象，但实际上存储了两个，浪费了存储区。此外在这个过程中，A的构造函数被调用了两次（B和C各一次）

最严重的引起的二义性，当调用D中的A成员时到底是哪个？由此引入了虚拟继承。

我们可以使用关键字Virtual修正，一个基类的声明可以将它指定为被虚拟派生。

```cpp
// 这里关键字 public 和 virtual的顺序不重要
class Bear : public virtual ZooAnimal { ... };
class Raccoon : virtual public ZooAnimal { ... };
```

虚拟派生不是基类本身的一个显式特性，而是它与派生类的关系。如前面所说明的，虚拟继承提供了“按引用组合”。也就是说，对于子对象及其非静态成员的访问是间接进行的。这使得在多继承情况下，把多个虚拟基类子对象组合成派生类中的一个共享实例，从而提供了必要的灵活性。同时，即使一个基类是虚拟的，我们仍然可以通过该基类类型的指针或引用，来操纵派生类的对象。

## override关键字

### override 简介

C++ override从字面意思上，是覆盖的意思，实际上在C++中它是覆盖了一个方法并且对其重写，从而达到不同的作用。override是C++11中的一个继承控制关键字。**override确保在派生类中声明的重载函数跟基类的虚函数有相同的声明。**

override明确地表示一个函数是对基类中一个虚函数的覆盖。更重要的是，它会**检查基类虚函数和派生类中重载函数的签名不匹配问题。如果签名不匹配，编译器会发出错误信息。**

在我们C++编程过程中，最熟悉的就是对接口方法的实现，在接口中一般只是对方法进行了声明，而我们在实现时，就需要实现接口声明的所有方法。还有一个典型应用就是在继承中也可能会在子类覆盖父类的方法。

### 实际在基类和派生类的作用

首先定义一个Person类，含有纯虚函数、虚函数以及普通函数

```cpp
class Person{
public:
	virtual void creat() const = 0;               // 1.纯虚函数
	virtual void say(const std::string& msg);   // 2.普通虚函数
	int name() const;                           // 3.非虚函数
};

class Student : public Person{
	public:
	protected:
	private:
};

class Teacher : public Person{
	public:
	protected:
	private:
};

```

#### 纯虚函数

纯虚函数，**继承的是基类成员函数的接口**，***必须***在派生类中重写该函数的实现：

```cpp
class Student : public Person{
	public: void creat(){
		Person* s1 = new Student; 
	}// calls Student::creat();
	protected:
	private:
};

class Teacher : public Person{
	public: void creat(){
		Person* s1 = new Teacher; 
	}// calls Teacher::eat();
	protected:
	private:
}; 
```
并且基类的eat为纯虚函数不可被调用

#### 普通虚函数

普通虚函数，对应在基类中**定义一个缺省的实现** (default implementation)，表示继承的是**基类成员函数的接口和缺省的实现**，由派生类**自行选择**是否重写该函数。

实际上，允许普通虚函数同时继承接口和缺省实现是危险的。 如下, CarA 和 CarB 是 Car的两种类型，且二者的运行方式完全相同。

```cpp
class Car{
public:
	virtual void run(const Car& destination);

};

class CarA : public Car{
  public:
  protected:
  private:
};

class CarB : public Car{
  public:
  protected:
  private:
};
```
这是典型的面向对象设计，两个类共享一个特性 – run，则 run可在基类中实现，并由两个派生类继承。

---

现增加一个新的飞机型号 CarC，其飞行方式与 CarA，CarB 并不相同，假如不小心忘了在 CarC 中重写新的 fly 函数.

```cpp
class CarC : public Car
{
public:
	... // no fly function is declared
};
```
则调用 CarC 中的 run 函数，就是调用 Car::run，但是 CarC的运行方式和缺省的并不相同

```cpp
Car * car1 = new CarC;
car1->run(China);  // calls Car::run!!
```

这就是前面所说的，普通虚函数同时继承接口和缺省(默认参数)实现是危险的，最好是基类中实现缺省行为 (behavior)，但只有在派生类要求时才提供该缺省行为.

##### 纯虚函数 + 缺省实现

一种方法是 纯虚函数 + 缺省实现，因为是纯虚函数，所以只有接口被继承，其缺省的实现不会被继承。派生类要想使用该缺省的实现，必须显式的调用:

```cpp
class Car
{
public:
	virtual void run(const Car& destination) = 0;//纯虚函数

};

void Car::run(const Car& destination)
{
	// a pure virtual function default code for run a Car to the given destination
}//纯虚函数其实都不需要写实现

class CarA : public Car
{
public:
	virtual void run(const Car& destination)
	{
		Car::run(destination);
	}

};
```

这样在派生类 CarC 中，即使一不小心忘记重写 Run函数，也不会调用 Car的缺省实现.

```cpp
class CarC : public Car
{
public:
	virtual void run(const Car& destination);
};

void CarC::run(const Car& destination)
{
	// code for run a CarC Car to the given destination
}
```

##### 使用override

可以看到，上面问题的关键就在于，一不小心在派生类 CarC中忘记重写 run函数，C++11 中使用关键字 override，可以避免这样的“一不小心”。

非虚函数：
非虚成员函数没有virtual关键字，表示派生类不但继承了接口，而且继承了一个强制实现（mandatory implementation），既然继承了一个强制的实现，则在派生类中，**无须重新定义继承自基类的成员函数**，如下：

使用指针调用 name 函数，则都是调用的 Person::name()
```cpp
Student s1;  // s1 is an object of type Student

Person* p1 = &s1;  // get pointer to s1
p1->name();        //call name() through pointer

Student* s2 = &s1;   // get pointer to s1
s2->name();          // call name() through pointer
```

---

如果在派生类中重新定义了继承自基类的成员函数 name :
```cpp
class Student : public Person
{
public:
   int name() const; // hides Person::name();

};

p1->name();        //call name() through pointer
s2->name();          // call name() through pointer
```
此时，派生类中重新定义的成员函数会 “隐藏” (hide) 继承自基类的成员函数。

>**这是因为非虚函数是 “静态绑定” 的**，p1被声明的是 Person* 类型的指针，则通过 p1调用的非虚函数都是基类中的，即使 指向的是派生类。

>与“静态绑定”相对的是虚函数的“动态绑定”，即无论 p1被声明为 Person* 还是 Student* 类型，其调用的虚函数取决于p1实际指向的对象类型。



### 使用原则

- 基类函数没加virtual，子类有相同函数，实现的是覆盖。用基类指针调用时，调用到的是基类的函数；用子类指针调用时，调用到的是子类的函数。
- 基类函数加了virtual时，实现的是重写。用基类指针或子类指针调用时，调用到的都是子类的函数。
- 函数加上override，强制要求基本相同函数需要是虚函数，否则会编译报错。相当于是一个保险，防止基类没写virtual？
- 子类的virtual可加可不加，建议加override不加virtual。
- C++11 中的 override 关键字，可以**显式的在派生类中声明**，***哪些成员函数需要被重写，如果没被重写，则编译器会报错。***

### 常见用法

- virtual一般用在继承的关系中。
- 如果这个方法无需子类定义，则该方法不用virtual进行修饰。
- 如果这个方法需要子类重写，但有默认实现，则该方法需要virtual进行修饰。
- 如果这个方法只需要子类实现，父类无需处理，则该方法可以定义为纯虚方法：virtual fun()=0
- 在继承的关系中，基类的析构方法，需要定义为虚函数，以避免子类无法析构。
- 构造函数不存在虚函数，因为在创建对象时，需要确切地知道是那个类（静态绑定），而需函数时动态绑定，在构造类时不能确定类的信息是错误的。

---

## final关键字

如果某个类/函数已做好了充分的准备并可供其他类使用的话（即其接口已明确定义且以后不会修改），那么该类/函数就是封闭（你可以称之为完整）的,这时可以使用**final**

### 最重要的两个作用

- 禁止虚函数被重写
- 禁止基类被继承

final:指定**不能在派生类中重写虚函数**或**不能从中继承类**。

在虚函数声明或定义中使用时，final确保函数是虚函数，并指定它不能被派生类重写。否则程序格式错误（生成编译时错误）。

在类定义中使用时，final指定该类不能出现在另一个类定义的基说明符列表中（换句话说，不能从中派生）。否则程序格式错误（生成编译时错误）。final也可以与union定义一起使用，在这种情况下，它没有任何影响（除了std:：is_final的结果），因为联合不能从派生）

final是在成员函数声明或类头中使用时具有特殊含义的标识符。在其他上下文中，它不是保留的，可以用来命名对象和函数。

C++ 11还增加了防止继承类或简单地防止派生类中重写方法的能力。这是用特殊标识符final完成的。

final**阻止类的进一步派生**和**阻止虚函数的进一步重写**

C++ 11关键字的最终目的有两个。它**阻止从类继承，并禁止重写虚函数**。

有时，你不想让派生类重写基类的虚函数。C++ 11允许内置的工具防止使用最终说明符重写虚函数。

C++ 11中的最后关键字可以应用于整个类或方法。当应用于一个类时，它表示该类不允许派生；也就是说，不能创建从最终类派生的类。第二种方法是将final应用于方法时，这样可以防止方法被派生类重写（尽管仍然可以创建子类）。

### 例子

#### 阻止基类函数的覆盖

```cpp
#include <iostream>
 
class Base {//最底层的类
public:
	Base() {
 
	}
public:
	virtual void func()  = 0 ;
	virtual void func2() = 0;
};
 
class Son :public Base{//第二层类
public:
	Son()
	{
 
	}
public:
	void func()
	{
		std::cout << "son func "<< std::endl;
	}
	void func2() final//定义func2 为final 不能被重写覆盖
	{
		std::cout << "son func2 " << std::endl;
 
	}
 
};
 
class Son2 :public Son{//继承自第二层的类，不能重写final函数func2（）
	void func()
	{
		std::cout << "son2 func " << std::endl;
	}
};
 
 
int main()
{
	Base* p = new Son2;
	p->func2();//son func2
}
```

#### 阻止类的继承

```cpp
#include <iostream>
 
class Base final {//final类，不能被继承 断子绝孙类
public:
	Base() {
 
	}
public:
	virtual void func()  = 0 ;
	virtual void func2() = 0;
};

```

