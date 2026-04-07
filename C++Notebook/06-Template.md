# C++ 模板编程完全指南

***

## 核心速查

| 特性 | 语法 | 用途 |
|------|------|------|
| 函数模板 | `template<typename T> T func(T a)` | 通用函数 |
| 类模板 | `template<typename T> class Box { T data; };` | 通用类 |
| 全特化 | `template<> class Box<int> { ... };` | 特定类型 |
| 偏特化 | `template<typename T> class Box<T*> { ... };` | 类型模式 |
| 可变参数 | `template<typename... Args>` | 任意数量参数 |
| 非类型参数 | `template<size_t N>` | 编译时常量 |
| Concept | `template<Integral T>` | 类型约束 (C++20) |

***

## 一、函数模板

### 基本语法

```cpp
template<typename T>
T max(T a, T b) {
    return a > b ? a : b;
}

// 使用
int x = max(3, 5);           // T 推导为 int
double y = max(3.14, 2.71);  // T 推导为 double

// 显式指定
auto z = max<double>(3, 2.71);  // 强制 T = double
```

### 多类型参数

```cpp
template<typename T, typename U>
auto add(T a, U b) -> decltype(a + b) {
    return a + b;
}

// C++14 更简洁
template<typename T, typename U>
auto add(T a, U b) {
    return a + b;
}

add(3, 4.5);       // int + double → double
add(string("a"), string("b"));  // string + string → string
```

### 类型推导规则

```cpp
template<typename T>
void func(T param);

int x = 10;
const int cx = x;
const int& rx = x;

func(x);    // T = int, param = int
func(cx);   // T = int, param = int（const 被忽略）
func(rx);   // T = int, param = int（引用和 const 都被忽略）

// 保留 const 和引用
template<typename T>
void func_ref(T& param);

func_ref(cx);  // T = const int, param = const int&
```

**实际运用 - 通用打印函数**：
```cpp
template<typename T>
void print(const T& value) {
    cout << value << endl;
}

template<typename T, typename... Args>
void print(const T& first, const Args&... rest) {
    cout << first << " ";
    print(rest...);
}

print(1, 2.5, "hello", 'A');  // 1 2.5 hello A
```

***

## 二、类模板

### 基本语法

```cpp
template<typename T>
class Box {
    T value_;
public:
    Box(const T& v) : value_(v) {}
    T get() const { return value_; }
    void set(const T& v) { value_ = v; }
};

Box<int> intBox(42);
Box<string> strBox("hello");
```

### 多类型参数与默认值

```cpp
template<typename K, typename V, typename Compare = less<K>>
class Dictionary {
    map<K, V, Compare> data_;
public:
    void insert(const K& k, const V& v) { data_[k] = v; }
    V& operator[](const K& k) { return data_[k]; }
};

Dictionary<string, int> d1;                    // 默认 less
Dictionary<string, int, greater<string>> d2;   // 降序
```

### 非类型模板参数

```cpp
template<typename T, size_t N>
class Array {
    T data_[N];
public:
    size_t size() const { return N; }
    T& operator[](size_t i) { return data_[i]; }
    const T& operator[](size_t i) const { return data_[i]; }
    
    T* begin() { return data_; }
    T* end() { return data_ + N; }
};

Array<int, 5> arr = {1, 2, 3, 4, 5};
for (int x : arr) { cout << x << " "; }
```

**实际运用 - 固定大小矩阵**：
```cpp
template<typename T, size_t Rows, size_t Cols>
class Matrix {
    T data_[Rows][Cols];
public:
    T& at(size_t r, size_t c) { return data_[r][c]; }
    
    template<size_t R2, size_t C2>
    Matrix<T, Rows, C2> operator*(const Matrix<T, R2, C2>& other);
};

Matrix<double, 3, 4> mat1;
Matrix<double, 4, 5> mat2;
auto result = mat1 * mat2;  // Matrix<double, 3, 5>
```

***

## 三、模板特化

### 全特化

```cpp
// 通用模板
template<typename T>
class TypeInfo {
public:
    static void print() { cout << "Unknown type" << endl; }
};

// 针对 int 的特化
template<>
class TypeInfo<int> {
public:
    static void print() { cout << "int: 4 bytes" << endl; }
};

// 针对 double 的特化
template<>
class TypeInfo<double> {
public:
    static void print() { cout << "double: 8 bytes" << endl; }
};

TypeInfo<int>::print();     // int: 4 bytes
TypeInfo<double>::print();  // double: 8 bytes
TypeInfo<char>::print();    // Unknown type
```

### 偏特化

```cpp
// 通用模板
template<typename T, typename U>
class Pair {
public:
    void print() { cout << "Generic pair" << endl; }
};

// 偏特化：两个类型相同
template<typename T>
class Pair<T, T> {
public:
    void print() { cout << "Same type pair" << endl; }
};

// 偏特化：第二个是指针
template<typename T, typename U>
class Pair<T, U*> {
public:
    void print() { cout << "Pointer pair" << endl; }
};

Pair<int, double> p1;   // Generic
Pair<int, int> p2;      // Same type
Pair<int, double*> p3;  // Pointer
```

**实际运用 - 类型判断**：
```cpp
template<typename T>
struct IsPointer {
    static constexpr bool value = false;
};

template<typename T>
struct IsPointer<T*> {
    static constexpr bool value = true;
};

// 使用
static_assert(IsPointer<int*>::value);    // 通过
static_assert(!IsPointer<int>::value);    // 通过
```

***

## 四、可变参数模板

### 参数包展开

```cpp
// 递归终止
void print() {}

// 递归展开
template<typename T, typename... Args>
void print(T first, Args... rest) {
    cout << first << " ";
    print(rest...);  // 递归调用
}

print(1, 2.5, "hello", 'A');  // 1 2.5 hello A
```

### 折叠表达式 (C++17)

```cpp
// 求和
template<typename... Args>
auto sum(Args... args) {
    return (args + ...);  // 右折叠
}

// 求积
template<typename... Args>
auto product(Args... args) {
    return (... * args);  // 左折叠
}

// 打印
template<typename... Args>
void printAll(Args... args) {
    ((cout << args << " "), ...);  // 逗号折叠
}

sum(1, 2, 3, 4, 5);        // 15
product(1, 2, 3, 4, 5);    // 120
printAll(1, 2.5, "hi");    // 1 2.5 hi
```

**实际运用 - 工厂函数**：
```cpp
template<typename T, typename... Args>
unique_ptr<T> make(Args&&... args) {
    return make_unique<T>(forward<Args>(args)...);
}

class Widget {
public:
    Widget(int id, string name, double value);
};

auto w = make<Widget>(1, "test", 3.14);
```

### 完美转发

```cpp
template<typename F, typename... Args>
auto invoke(F&& f, Args&&... args) 
    -> decltype(forward<F>(f)(forward<Args>(args)...)) 
{
    return forward<F>(f)(forward<Args>(args)...);
}

void process(int& x) { cout << "lvalue: " << x << endl; }
void process(int&& x) { cout << "rvalue: " << x << endl; }

int val = 10;
invoke(process, val);   // lvalue: 10
invoke(process, 20);    // rvalue: 20
```

***

## 五、Concept (C++20)

### 定义 Concept

```cpp
#include <concepts>

// 基本概念
template<typename T>
concept Integral = is_integral_v<T>;

template<typename T>
concept Numeric = integral<T> || floating_point<T>;

// 复合要求
template<typename T>
concept Addable = requires(T a, T b) {
    { a + b } -> same_as<T>;
};

template<typename T>
concept Container = requires(T t) {
    typename T::value_type;
    { t.begin() } -> input_iterator;
    { t.end() } -> input_iterator;
    { t.size() } -> convertible_to<size_t>;
};
```

### 使用 Concept

```cpp
// 方式1：直接约束
template<Integral T>
T abs(T x) {
    return x < 0 ? -x : x;
}

// 方式2：requires 子句
template<typename T>
    requires Integral<T>
T half(T x) { return x / 2; }

// 方式3：简写
auto half2(Integral auto x) { return x / 2; }

// 方式4：尾置 requires
template<typename T>
T divide(T a, T b) requires Integral<T> {
    return a / b;
}
```

**实际运用 - 约束模板**：
```cpp
template<typename T>
concept Printable = requires(T t, ostream& os) {
    { os << t } -> same_as<ostream&>;
};

template<Printable T>
void log(const T& value) {
    cout << "[LOG] " << value << endl;
}

log(42);           // OK
log(string("hi")); // OK
log(vector<int>{});// 编译错误：不满足 Printable
```

***

## 六、CRTP 模式

### 静态多态

```cpp
template<typename Derived>
class Base {
public:
    void interface() {
        static_cast<Derived*>(this)->impl();
    }
    
    void base_method() {
        cout << "Base method" << endl;
    }
};

class Derived : public Base<Derived> {
public:
    void impl() {
        cout << "Derived implementation" << endl;
    }
};

Derived d;
d.interface();  // Derived implementation
```

**实际运用 - 可比较对象**：
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
    bool operator<=(const Derived& other) const {
        return !(static_cast<const Derived&>(*this) > other);
    }
    bool operator>=(const Derived& other) const {
        return !(static_cast<const Derived&>(*this) < other);
    }
};

class Point : public Comparable<Point> {
    int x, y;
public:
    Point(int x, int y) : x(x), y(y) {}
    bool operator==(const Point& other) const {
        return x == other.x && y == other.y;
    }
    bool operator<(const Point& other) const {
        return x < other.x || (x == other.x && y < other.y);
    }
};

Point p1(1, 2), p2(3, 4);
p1 < p2;   // true
p1 != p2;  // true（自动生成）
```

***

## 七、模板元编程

### 编译时计算

```cpp
// 阶乘
template<int N>
struct Factorial {
    static constexpr int value = N * Factorial<N - 1>::value;
};

template<>
struct Factorial<0> {
    static constexpr int value = 1;
};

constexpr int fact5 = Factorial<5>::value;  // 120

// C++17 constexpr 函数更简洁
constexpr int factorial(int n) {
    return n <= 1 ? 1 : n * factorial(n - 1);
}
constexpr int fact6 = factorial(6);  // 720
```

### 类型列表

```cpp
template<typename... Types>
struct TypeList {};

using IntOrDouble = TypeList<int, double>;

// 获取第 N 个类型
template<typename List, size_t N>
struct TypeAt;

template<typename Head, typename... Tail>
struct TypeAt<TypeList<Head, Tail...>, 0> {
    using type = Head;
};

template<typename Head, typename... Tail, size_t N>
struct TypeAt<TypeList<Head, Tail...>, N> {
    using type = typename TypeAt<TypeList<Tail...>, N - 1>::type;
};

using T0 = TypeAt<IntOrDouble, 0>::type;  // int
using T1 = TypeAt<IntOrDouble, 1>::type;  // double
```

***

## 八、常用技巧

### 类型别名

```cpp
template<typename T>
using Vec = vector<T>;

template<typename T>
using Ptr = unique_ptr<T>;

template<typename K, typename V>
using Map = unordered_map<K, V>;

Vec<int> v;           // vector<int>
Ptr<Widget> p;        // unique_ptr<Widget>
Map<string, int> m;   // unordered_map<string, int>
```

### SFINAE 技巧

```cpp
// enable_if
template<typename T>
typename enable_if<is_integral<T>::value, T>::type
abs(T x) {
    return x < 0 ? -x : x;
}

// void_t (C++17)
template<typename T, typename = void>
struct HasSize : false_type {};

template<typename T>
struct HasSize<T, void_t<decltype(declval<T>().size())>> : true_type {};

// 使用
bool b1 = HasSize<vector<int>>::value;   // true
bool b2 = HasSize<int>::value;           // false
```

***

## 九、最佳实践

```cpp
// ✅ 推荐
template<typename T>           // typename 比 class 更清晰
T max(T a, T b) { return a > b ? a : b; }

template<typename T>
using Vec = vector<T>;         // 用别名简化

template<Integral T>           // 用 Concept 约束
T half(T x) { return x / 2; }

// ❌ 避免
template<typename T>
class Widget;                  // 模板声明和定义分离（链接错误）

template<typename T1, typename T2, typename T3, typename T4, typename T5>
class TooMany;                 // 参数太多，考虑封装

template<typename T>
void unused(T x) {}            // 过度模板化，简单函数不需要
```

***

## 十、记忆口诀

> **模板编程泛型化，类型参数代替具体**
> **函数类都可模板，编译时期实例化**
> **特化偏特化灵活，可变参数真强大**
> **Concept 约束类型，CRTP 静态多态**
> **折叠表达式简洁，完美转发保语义**
