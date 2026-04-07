# C++ 类完全指南

***

## 核心速查

| 概念 | 要点 |
|------|------|
| 构造函数 | 初始化列表优先、explicit 防隐转 |
| 析构函数 | 基类用 virtual、RAII 管理资源 |
| 继承 | public 继承、虚析构、override |
| 多态 | 虚函数、引用传递防切片 |
| struct | 默认 public，适合 POD 数据 |
| 资源管理 | 详见 [Source.md](Source.md) |

***

## 一、类的基本结构

```cpp
class ClassName {
private:           // 只能被本类访问
    int private_member;
    
protected:         // 能被本类和派生类访问
    int protected_member;
    
public:            // 能被任何代码访问
    int public_member;
    
    ClassName();           // 构造函数
    ~ClassName();          // 析构函数
    void func();           // 成员函数
};
```

***

## 二、构造函数

### 初始化列表（推荐）

```cpp
class Person {
    string name_;
    int age_;
public:
    Person(const string& name, int age) 
        : name_(name), age_(age) {}  // 初始化列表
    
    // 委托构造
    Person(const string& name) : Person(name, 0) {}
    Person() : Person("Unknown", 0) {}
};
```

### 必须用初始化列表的情况

```cpp
class Demo {
    const int id_;       // const 成员
    int& ref_;           // 引用成员
public:
    Demo(int id, int& ref) 
        : id_(id), ref_(ref) {}  // 必须用初始化列表
};
```

### explicit 禁止隐式转换

```cpp
class Integer {
    int value_;
public:
    explicit Integer(int x) : value_(x) {}
};

void func(Integer x) {}

func(42);           // ❌ 错误！不能隐式转换
func(Integer(42));  // ✅ 显式构造
```

***

## 三、析构函数

```cpp
class Resource {
    int* ptr_;
public:
    Resource() : ptr_(new int[100]) {}
    
    ~Resource() {
        delete[] ptr_;  // 释放资源
    }
};

// 基类必须虚析构
class Base {
public:
    virtual ~Base() = default;
};
```

> **资源管理最佳实践**：使用 RAII 类型（智能指针、容器），让编译器自动管理。详见 [Source.md](Source.md)

***

## 四、继承

### 基本继承

```cpp
class Animal {
protected:
    string name_;
public:
    Animal(const string& name) : name_(name) {}
    virtual void speak() const { cout << "???" << endl; }
    virtual ~Animal() = default;  // 虚析构！
};

class Dog : public Animal {
public:
    Dog(const string& name) : Animal(name) {}
    void speak() const override {
        cout << "Woof! I'm " << name_ << endl;
    }
};
```

### 访问控制

| 继承方式 | public 成员 | protected 成员 | private 成员 |
|---------|------------|---------------|-------------|
| public 继承 | public | protected | 不可见 |
| protected 继承 | protected | protected | 不可见 |
| private 继承 | private | private | 不可见 |

### override 和 final

```cpp
class Base {
public:
    virtual void func1();
    virtual void func2();
    virtual ~Base() = default;
};

class Derived : public Base {
public:
    void func1() override;      // 明确表示重写
    void func2() final;         // 禁止派生类再重写
};
```

***

## 五、多态

### 虚函数

```cpp
class Shape {
public:
    virtual double area() const = 0;  // 纯虚函数
    virtual void draw() const { }      // 虚函数
    virtual ~Shape() = default;
};

class Circle : public Shape {
    double radius_;
public:
    Circle(double r) : radius_(r) {}
    double area() const override {
        return 3.14159 * radius_ * radius_;
    }
};

// 多态使用
vector<unique_ptr<Shape>> shapes;
shapes.push_back(make_unique<Circle>(5));

for (const auto& s : shapes) {
    cout << s->area() << endl;
}
```

### 切片问题

```cpp
void bad_func(Base b) { }          // 按值传递 → 切片
void good_func(const Base& b) { }  // 按引用 → 多态正常
```

***

## 六、静态成员

```cpp
class Counter {
    static int count_;           // 静态成员（共享）
    static const int MAX = 100;  // 静态常量
    
public:
    Counter() { count_++; }
    ~Counter() { count_--; }
    
    static int getCount() { return count_; }
};

int Counter::count_ = 0;  // 类外定义
```

***

## 七、友元

```cpp
class Vector {
    int x_, y_;
public:
    Vector(int x, int y) : x_(x), y_(y) {}
    
    friend Vector operator+(const Vector& a, const Vector& b);
    friend class VectorHelper;
};

Vector operator+(const Vector& a, const Vector& b) {
    return Vector(a.x_ + b.x_, a.y_ + b.y_);
}
```

***

## 八、运算符重载

```cpp
class Complex {
    double real_, imag_;
public:
    Complex(double r = 0, double i = 0) : real_(r), imag_(i) {}
    
    Complex operator+(const Complex& other) const {
        return Complex(real_ + other.real_, imag_ + other.imag_);
    }
    
    Complex& operator+=(const Complex& other) {
        real_ += other.real_;
        imag_ += other.imag_;
        return *this;
    }
    
    double& operator[](size_t i) {
        return i == 0 ? real_ : imag_;
    }
    
    explicit operator double() const {
        return sqrt(real_ * real_ + imag_ * imag_);
    }
};
```

***

## 九、const 成员函数

```cpp
class Data {
    int value_;
    mutable int cache_;  // mutable 允许在 const 中修改
public:
    int get() const {
        cache_ = 42;      // ✅ mutable 可以修改
        return value_;
    }
    
    void set(int v) { value_ = v; }
};

// const 重载
class Array {
    int data[10];
public:
    int& operator[](int i) { return data[i]; }
    const int& operator[](int i) const { return data[i]; }
};
```

***

## 十、struct 与 class 对比

### 唯一区别：默认访问权限

```cpp
class MyClass {
    int x;      // 默认 private
};

struct MyStruct {
    int x;      // 默认 public
};

// 继承时也一样
class Derived : Base {};      // 默认 private 继承
struct Derived : Base {};     // 默认 public 继承
```

### 完整对比表

| 特性 | class | struct |
|------|-------|--------|
| 成员默认权限 | private | public |
| 继承默认方式 | private | public |
| 能否有构造函数 | ✅ | ✅ |
| 能否有析构函数 | ✅ | ✅ |
| 能否有虚函数 | ✅ | ✅ |
| 能否继承 | ✅ | ✅ |

### 使用场景

```cpp
// struct：纯数据、POD、公开成员
struct Point {
    int x, y;
};

struct Config {
    string host;
    int port;
    bool debug;
};

// class：封装、不变量、私有成员
class BankAccount {
    string owner_;
    double balance_;
public:
    void deposit(double amount);
    void withdraw(double amount);
    double getBalance() const;
};
```

### 习惯约定

| 场景 | 选择 | 原因 |
|------|------|------|
| 纯数据结构 | struct | 成员公开，无需封装 |
| 配置/参数 | struct | 直接访问字段 |
| 需要不变量 | class | 保护数据完整性 |
| 有行为的方法 | class | 封装实现细节 |
| C 兼容接口 | struct | 与 C struct 兼容 |

### 聚合初始化

```cpp
// struct 支持聚合初始化（无用户定义构造函数时）
struct Point {
    int x, y;
};

Point p1 = {1, 2};      // 聚合初始化
Point p2{1, 2};         // 统一初始化

// class 也可以（成员都是 public 且无构造函数）
class Vec3 {
public:
    float x, y, z;
};

Vec3 v = {1.0f, 2.0f, 3.0f};
```

***

## 十一、常用设计模式

### Pimpl（隐藏实现）

```cpp
// widget.h
class Widget {
    struct Impl;
    unique_ptr<Impl> pImpl_;
public:
    Widget();
    ~Widget();
    void doSomething();
};

// widget.cpp
struct Widget::Impl {
    string name_;
    vector<int> data_;
};

Widget::Widget() : pImpl_(make_unique<Impl>()) {}
Widget::~Widget() = default;
```

**价值**：减少头文件依赖、加快编译、稳定 ABI

### 工厂方法

```cpp
class Shape {
public:
    virtual ~Shape() = default;
    virtual void draw() = 0;
    
    static unique_ptr<Shape> create(const string& type);
};

class Circle : public Shape {
public:
    void draw() override { cout << "Circle" << endl; }
};

unique_ptr<Shape> Shape::create(const string& type) {
    if (type == "circle") return make_unique<Circle>();
    return nullptr;
}
```

**价值**：解耦对象创建和使用

### CRTP（奇异递归模板模式）

```cpp
template<typename Derived>
class Comparable {
public:
    bool operator!=(const Derived& other) const {
        return !(static_cast<const Derived&>(*this) == other);
    }
    bool operator>(const Derived& other) const {
        return other < static_cast<const Derived&>(*this);
    }
};

class Point : public Comparable<Point> {
    int x, y;
public:
    bool operator==(const Point& other) const {
        return x == other.x && y == other.y;
    }
    bool operator<(const Point& other) const {
        return x < other.x || (x == other.x && y < other.y);
    }
};

Point p1(1, 2), p2(3, 4);
p1 != p2;  // 自动生成
```

**价值**：静态多态，编译时生成比较运算符

***

## 十二、记忆口诀

> **初始化列表优先，explicit 防隐转**
> **虚析构不能忘，override 保重写**
> **引用传递防切片，const 正确是习惯**
> **struct 纯数据，class 要封装**
> **Pimpl 藏实现，工厂解耦创建**

***

## 十三、检查清单

```cpp
class MyClass {
    vector<int> data_;      // ✅ RAII 类型
    
public:
    MyClass() = default;
    MyClass(int x) : data_(x) {}  // ✅ 初始化列表
    
    ~MyClass() = default;         // ✅ 零规则
    
    int get() const { return data_[0]; }  // ✅ const 正确
    
    virtual ~MyClass() = default;  // ✅ 虚析构（基类）
};
```

***

## 相关文档

- **资源管理**（RAII、智能指针、移动语义）：[Source.md](Source.md)
- **模板编程**：[Template.md](Template.md)
- **STL 算法**：[Algorithm.md](Algorithm.md)
