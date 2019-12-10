## 虚函数

### pure virtual

含有纯虚函数的类叫抽象类，抽象类无法实例化。纯虚函数由子类实现，没有实现该函数的子类依然是抽象类，即无法被实例化

```c++
virtual void func1() = 0;
```

### virtual

父类提供默认的实现：子类可以重写函数来覆盖

```c++
virtual void func2()
{
    // do default thing
}
```

### why & how virtual

```c++
class Shape
{
public:
    virtual void virtual_draw() { cout<<"base shape in virtual_draw"<<endl; }
    void virtual_func()
    {
        virtual_draw();
    }
    void normal_draw() { cout<<"base shape in normal_draw"<<endl; }
    void normal_func()
    {
        normal_draw();
    }
};

class Circle : public Shape
{
public:
    void virtual_draw() { cout<<"I'm a circle in virtual_draw"<<endl; }
    void normal_draw() { cout<<"I'm a circle in normal_draw"<<endl; }
};

int main()
{
    Circle shape1;
    shape1.virtual_func(); // I'm a circle in virtual_draw
    shape1.normal_func();  // base shape in normal_draw
}
```

因为在抽象类中，会建立虚函数列表，如果这个类的子类有重写虚函数，则子类的重写会覆盖列表中的原虚函数，所以在列表里查找并执行函数时就会找到子类的方法。

如果没有声明 virtual，就执行类内部的方法

### tips

```c++
class A
{
public:
    virtual void aa(){};
};
class B
{
public:
    void bb(){};
};

cout<<sizeof(A)<<" "<<sizeof(B); // 4 1
// 虚函数列表的指针大小为 4

//-------
class D { char a[0];};				// sizeof = 0

class D_S:public D { };				// sizeof = 0

class D_V:public virtual D { };		// sizeof = 4

class D_V_S:public D_V { };			// sizeof = 4

class D_V_V:public virtual D_V { };	// sizeof = 4

//-------
class D { char a[0];};					// sizeof = 0

class D_S:public D { };					// sizeof = 0

class D_V:public virtual D {char a[0];};// sizeof = 4
class D_V:public virtual D {char b[0];};// sizeof = 4

class D_V_S:public D_V { };				// sizeof = 4

class D_V_V:public virtual D_V { };		// sizeof = 8
```

对于虚继承，如下

```c++
class A public virtual B
```

会在A里多一个指针，指针指向父类表，它（父类表）的意义在于当存在多个父类继承至同一父类时，不会产生二义性，如下图

​      A         class A { public: void func(){ cout<<"A"<<endl; } }

   /      \

 B        C    class B:virtual public A;

   \      /      class C:virtual public A;

​      E          class D:public B,public C;

这个时候 d.func(); 不会有任何问题