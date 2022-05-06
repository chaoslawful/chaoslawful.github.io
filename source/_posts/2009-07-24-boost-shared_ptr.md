---
title: 如何避免 boost::shared_ptr 管理的共享对象被意外删除
tags:
  - Boost
  - C++
date: 2009-07-24 20:23:00
---

`boost::shared_ptr` （已进入C++ TR1标准）是管理共享对象的好帮手，但由于其 `get()` 方法能获取原对象裸指针，因此存在其管理的对象被人为意外删除的危险。最近看 boost 相关资料时发现一些方法能避免该问题。
<!-- more -->

## 方案1 - 使用包装类

```cpp
class T {
protected:
    ~T() {...}
public:
   ...
};

class TWrapper {
};

boost::shared_ptr<T> p(new TWrapper);
delete p.get(); // Oops...Compile-time error!
```

这是因为 `shared_ptr` 对象内部指向的共享引用计数对象保存的是原始指针（这里就是 `TWrapper*` ），引用计数为 0 时可以用该指针正常调用 `delete` ，而从 `get()` 方法获得的是 `shared_ptr` 对象缓存的指针，类型为 `T*` ，而 `class T` 的析构函数被保护了，故不能显式被 `delete` 。该方案缺点是需要引入一个新类，且当原类构造函数有多种形式时，包装类也要对应提供这些形式的构造函数，二者是紧耦合的；优点是创建对象方式同以前没有变化。

## 方案2 - 使用自定义析构函数

```cpp
class T {
    struct deleter {
        void operator() (T *p) { delete p; }
    };
    friend class deleter;
protected:
    T() { ... }
    ~T() { ... }
public:
    static boost::shared_ptr<T> createObject()
    {
        return boost::shared_ptr<T>(new T(), T::deleter());
    }
};

boost::shared_ptr<T> p = T::createObject();
delete p.get(); // Oops...Compile-time error!
```

该方案利用 `shared_ptr` 的自定义析构函数形式实现，将析构函子类作为私有类封装在 `class T` 内部，以避免被外部直接利用；创建对象时也不再使用标准 `new` 形式，而是封装了对应的静态工厂方法，为了避免绕过工厂方法创建对象，还要将 `class T` 的构造函数也保护起来。该方案的缺点是创建对象方法彻底改变，且工厂方法接口要同原有类构造函数的形式一致；优点是所有的机制都封装在原有类内部，不需要额外引入新类型。

从 `shared_ptr` 内部的共享引用计数对象保存原始指针这一特性可知，若创建 `shared_ptr` 对象时传入的是派生类指针，即使基类没有将析构函数定义为 `virtual` 的，共享对象被删除时也会正常调用派生类的析构函数：

```cpp
class Base {
public:
    ~Base() { cout << "Base dtor\n"; }
};

class Derived : virtual public Base {
public:
    ~Derived() { cout << "Derived dtor\n"; }
};

{
    boost::shared_ptr<Base> p(new Derived);
    // shared_ptr object release managed-object here, will call dtors of class Derived and Base in order
}

Base *q = new Derived;
delete q; // Here only dtor of class Base is called for it's not declared to be virtual
```

