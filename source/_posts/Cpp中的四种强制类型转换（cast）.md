---
title: C++中的四种强制类型转换（cast）
date: 2020-08-01 13:32:13
tags: [C++,强制类型转换]
---

# C++中的四种强制类型转换（cast）

## 1.C中的强制类型转换

​	在C语言中，如果要将一个变量的类型强制转化为另一种类型，可以使用以下形式：

> (new_type) var

​	在C++中拓展了一种新的写法，又被称为函数式写法（functional notation），更加符合我们的书写习惯，形式如下：

> new_type(var)

​	例如：

```cpp
char a='A';
int newa=int(a);
cout<<newa<<endl;//output:65
```

​	C中的强制类型转换是不受任何约束的，它可以将任意的指针类型转换为任意其它类型的指针，甚至可以将基本数据类型（`char`）转换为用户自定义指针类型（`myclass *`）。

​	例如：

```cpp
#include <bits/stdc++.h>
using namespace std; 
class myclass
{
	int i,j;
};
int main(int argc, char** argv) 
{

	char a='1';
	myclass *A=(myclass *) a;//将char类型转换为指向myclass类型的指针，编译能够通过
	cout<<A<<endl;
	return 0;
}
```

​	这种“无拘无束“的转换被认为非常不安全，C++的作者也建议程序员尽量不要使用C风格的强制类型转换，而是使用标准C++的类型转换符：`static_cast`、`dynamic_cast`、`const_cast`和`reinterpret_cast`。



## 2.static_cast

> 用法：static_cast <type-id> (expression)

​	注意：`expression`两边的圆括号()不要遗漏！

​	说明：该运算符把`expression`转换为`type-id`类型，在编译时会进行类型检查,**但没有运行时类型检查来保证转换的安全性**。

它主要有如下4种用法：

1. **用于类层次结构中基类和子类之间指针或引用的转换**。进行上行转换（把子类的指针或引用转换成基类表示）是安全的；进行下行转换（把基类指针或引用转换成子类指针或引用）时，**由于没有动态类型检查，所以是不安全的**。
2. **用于基本数据类型之间的转换**，如把`int`转换成`char`，把`int`转换成`enum`。这种转换的安全性也要开发人员来保证。
3. 把`void`指针转换成目标类型的指针。
4. 把任何类型的表达式转换成`void`类型。



​	除了以上4种用法，它杜绝了：

1. ​	基本数据类型（如`int`等）与类指针之间的转换
2. ​	没有“继承与被继承关系“的类指针之间的转换

### 2.1 第一个例子

```cpp
#include <bits/stdc++.h>
using namespace std; 
class myclass
{
	int i,j;
};

class myclass1
{
	int i,j;
	virtual func(){
	}
};
int main(int argc, char** argv) 
{

	myclass1 * a=new myclass1;
	cout<<static_cast <myclass *> (a)<<endl;//编译报错！因为myclass与myclass1之间没有继承关系
	return 0;
}
```

​	C风格的强制类型转换是允许以上列举的两种操作的。

> 注意：`static_cast`不能转换掉`expression`的`const`、`volitale`、或者`__unaligned`属性。



## 3.dynamic_cast

> 用法：dynamic_cast <type-id> (expression)

​	说明：该运算符把`expression`转换成`new_type`类型的对象。**`Type-id`必须是类的指针、类的引用或者`void *`**；如果`type-id`是类指针类型，那么`expression`也必须是一个指针，如果`type-id`是一个引用，那么`expression`也必须是一个引用。

1. 若对指针进行`dynamic_cast`，失败返回`null`，成功返回正常cast后的对象指针；
2. 若对引用进行`dynamic_cast`，失败抛出一个`异常(exception)`，成功返回正常cast后的对象引用。



​	通过说明我们可以看出，`dynamic_cast`必须用于类指针之间的强制类型转换，而且一般来说是针对基类和派生类之间的指针或引用转换，而且它是**安全**的。

### 	3.1 上行转换（upcast）与下行转换（downcast）

​	那么什么叫“安全”呢？

​	在C++中，类之间的转换被分为“上行转换”（`upcast`）和“下行转换”（`downcast`）

​	上行转换指的是将派生类指针（也被称为子类）转换为基类指针（也被称为父类）；下行转换指的是将基类指针转换为派生类指针。

​	在上行转换中，`dynamic_cast`和`static_class`的效果是一样的，返回`upcast`后的对象指针。

​	而下行转换往往被认为是“不安全的”，`dynamic_cast`会进行类型检查，如果发现了下行转换，则返回`null`。

### 	3.2 第一个例子

```cpp
//Example 1.
#include <bits/stdc++.h>
using namespace std; 
class myclass
{
	int i,j;
};

class myclass1
{
	int i,j;
};
int main(int argc, char** argv) 
{

	myclass1 a;
	cout<<dynamic_cast <myclass *> (&a)<<endl;
    //编译错[Error] cannot dynamic_cast '& a' (of type 'class myclass1*') to type 'class myclass*' 
    //(source type is not polymorphic) 源类不是多态的
	return 0;
}
```

​	**注意：`myclass`中必须要有一个虚函数！**

​	

### 	3.3 第二个例子

```cpp
#include <iostream>
#include <exception>
using namespace std;

class Base { virtual void dummy() {} };//必须要有virtual函数
class Derived: public Base { int a; };

int main () {
  try {
    Base * pba = new Derived;
    Base * pbb = new Base;
    Derived * pd;

    pd = dynamic_cast<Derived*>(pba);
    if (pd==0) cout << "Null pointer on first type-cast.\n";//正常

    pd = dynamic_cast<Derived*>(pbb);//下行转换，将Base指针cast为Derived指针，返回null
    if (pd==0) cout << "Null pointer on second type-cast.\n";//输出！

  } catch (exception& e) {cout << "Exception: " << e.what();}//但是没有exception
  return 0;
}
```



### 	3.4 第三个例子

​	两个类没有继承与被继承关系，编译可以通过，但是转换是不成功的。

```cpp
#include <bits/stdc++.h>
using namespace std; 
class myclass
{
	int i,j;
};

class myclass1
{
	int i,j;
	virtual func(){
	}
};
int main(int argc, char** argv) 
{
	myclass1 a;
	cout<<dynamic_cast <myclass *> (&a)<<endl;// output:0
	return 0;
}
```



## 4.const_cast

> 用法：**const_cast**<type-id> (expression)

​	说明：该运算符用来修改类型的`const`或`volatile`属性。除了`const` 或`volatile`修饰之外， `type_id`和`expression`的类型是一样的。



​	说人话，**`const_cast`并不能从本质上将`const`类型的变量转化为`non-const`类型**，也就是说，你依然不可以对转化后的变量进行任何修改。

​	唯一的优点在于，通过`const_cast`后可以方便你进行处理，避免编译器报错，相当于去掉了`const`修饰符。

### 4.1 第一个例子

```cpp
#include <iostream>
using namespace std;

void print (char * str)
{
  //str[0]='S'                     //如果加上这条语句，编译可以通过，但会出现运行错误
                                   //因为str仍旧是一个const类型，只是它表面上是non-const
  cout << str << '\n';
}

int main () {
  const char * c = "sample text";
  print ( const_cast<char *> (c) );//output: sample text
  return 0;
}
```



## 5. reinpreter_cast

> 用法：**reinpreter_cast**<type-id> (expression)

​	说明：`type-id`必须是一个指针、引用、算术类型、函数指针或者成员指针。它可以把一个指针转换成一个整数，也可以把一个整数转换成一个指针（先把一个指针转换成一个整数，在把该整数转换成原类型的指针，还可以得到原先的指针值）。

​	实际上任何类型的指针它都可以相互转换，操作结果只是简单的从一个指针到别的指针的值的二进制拷贝。在类型之间指向的内容不做任何类型的检查和转换。`reinpreter_cast`是特意用于底层的强制转型，导致实现依赖（就是说，不可移植）的结果。