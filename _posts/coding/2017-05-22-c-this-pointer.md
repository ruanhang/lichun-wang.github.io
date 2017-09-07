---
layout: post
title:  "Ｃ++--this指针"
category: coding
tags: [ｃ++]
description: c++
---  

[参考：http://c.biancheng.net/cpp/biancheng/view/201.html](http://c.biancheng.net/cpp/biancheng/view/201.html)  
this 是 C++ 中的一个关键字，也是一个 const 指针，它指向当前对象，通过它可以访问当前对象的所有成员。  
所谓当前对象，是指正在使用的对象。例如对于stu.show();，stu 就是当前对象，this 就指向 stu。  
下面是使用 this 的一个完整示例：
```c++
#include <iostream>
using namespace std;
class Student{
public:
    void setname(char *name);
    void setage(int age);
    void setscore(float score);
    void show();
private:
    char *name;
    int age;
    float score;
};
void Student::setname(char *name){
    this->name = name;
}
void Student::setage(int age){
    this->age = age;
}
void Student::setscore(float score){
    this->score = score;
}
void Student::show(){
    cout<<this->name<<"的年龄是"<<this->age<<"，成绩是"<<this->score<<endl;
}
int main(){
    Student *pstu = new Student;
    pstu -> setname("李华");
    pstu -> setage(16);
    pstu -> setscore(96.5);
    pstu -> show();
    return 0;
}
```
运行结果：  
李华的年龄是16，成绩是96.5  
this 只能用在类的内部，通过 this 可以访问类的所有成员，包括 private、protected、public 属性的。  

本例中成员函数的参数和成员变量重名，只能通过 this 区分。以成员函数setname(char *name)为例，它的形参是name，和成员变量name重名，如果写作name = name;这样的语句，就是给形参name赋值，而不是给成员变量name赋值。而写作this -> name = name;后，=左边的name就是成员变量，右边的name就是形参，一目了然。  

**注意，this 是一个指针，要用->来访问成员变量或成员函数。**

this 虽然用在类的内部，但是只有在对象被创建以后才会给 this 赋值，并且这个赋值的过程是编译器自动完成的，不需要用户干预，用户也不能显式地给 this 赋值。本例中，this 的值和 pstu 的值是相同的。  

我们不妨来证明一下，给 Student 类添加一个成员函数printThis()，专门用来输出 this 的值，如下所示：  
```
void Student::printThis(){
    cout<<this<<endl;
}
```
然后在 main() 函数中创建对象并调用 printThis()：  
```
Student *pstu1 = new Student; 
pstu1 -> printThis();
cout<<pstu1<<endl;
Student *pstu2 = new Student;
pstu2 -> printThis();
cout<<pstu2<<endl;
```
运行结果：  
0x7b17d8  
0x7b17d8  
0x7b17f0  
0x7b17f0  

可以发现，this 确实指向了当前对象，而且对于不同的对象，this 的值也不一样。  

几点注意：  

* this 是 const 指针，它的值是不能被修改的，一切企图修改该指针的操作，如赋值、递增、递减等都是不允许的。
* this 只能在成员函数内部使用，用在其他地方没有意义，也是非法的。  
* 只有当对象被创建后 this 才有意义，因此不能在 static 成员函数中使用（后续会讲到 static 成员）。  
### this 到底是什么
this 实际上是成员函数的一个形参，在调用成员函数时将对象的地址作为实参传递给 this。不过 this 这个形参是隐式的，它并不出现在代码中，而是在编译阶段由编译器默默地将它添加到参数列表中。  

this 作为隐式形参，本质上是成员函数的局部变量，所以只能用在成员函数的内部，并且只有在通过对象调用成员函数时才给 this 赋值。  

在《C++函数编译原理和成员函数的实现》一节中讲到，成员函数最终被编译成与对象无关的普通函数，除了成员变量，会丢失所有信息，所以编译时要在成员函数中添加一个额外的参数，把当前对象的首地址传入，以此来关联成员函数和成员变量。这个额外的参数，实际上就是 this，它是成员函数和成员变量关联的桥梁。  


