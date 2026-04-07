# C++ 现代特性速查

***

## 核心速查

| 特性 | 版本 | 用途 |
|------|------|------|
| `initializer_list` | C++11 | 统一初始化语法 |
| `auto` 类型推导 | C++11 | 简化类型声明 |
| `decltype` | C++11 | 获取表达式类型 |
| 范围 for | C++11 | 简化遍历 |
| `nullptr` | C++11 | 空指针常量 |
| `constexpr` | C++11 | 编译时计算 |
| `std::tuple` | C++11 | 元组 |
| 结构化绑定 | C++17 | 解包 tuple/pair |
| `std::optional` | C++17 | 可选值 |
| `std::variant` | C++17 | 类型安全联合体 |
| `std::string_view` | C++17 | 字符串视图 |
| `std::span` | C++20 | 数组视图 |
| 折叠表达式 | C++17 | 可变参数展开 |
| `std::format` | C++20 | 格式化（C++23） |

***

## 一、initializer_list（初始化列表）

```cpp
#include <initializer_list>

// 自定义类支持列表初始化
class Vector {
    vector<int> data;
public:
    Vector(initializer_list<int> list) : data(list) {}
    
    void push(initializer_list<int> list) {
        data.insert(data.end(), list);
    }
};

Vector v = {1, 2, 3, 4, 5};
v.push({6, 7, 8});

// 函数参数
void print(initializer_list<int> list) {
    for (int x : list) cout << x << " ";
}
print({1, 2, 3, 4, 5});
```

**实际运用**：
```cpp
// 构造函数
map<string, int> m = {{"a", 1}, {"b", 2}};
vector<int> v = {1, 2, 3, 4, 5};

// 自定义容器
class Config {
    map<string, string> data;
public:
    Config(initializer_list<pair<string, string>> list) {
        for (auto& p : list) data[p.first] = p.second;
    }
};

Config cfg = {
    {"host", "localhost"},
    {"port", "8080"}
};
```

***

## 二、auto 与 decltype

### auto（类型推导）

```cpp
// 基本用法
auto x = 42;              // int
auto d = 3.14;            // double
auto s = "hello";         // const char*
auto str = string("hi");  // string

// 迭代器
map<string, vector<int>> m;
auto it = m.begin();      // 简化复杂类型

// 范围 for
for (auto& x : v) { x *= 2; }

// 函数返回类型（C++14）
auto add(int a, int b) {
    return a + b;
}
```

### decltype（类型获取）

```cpp
int x = 10;
decltype(x) y = 20;       // int y = 20

// 返回类型推导
template<typename T, typename U>
auto add(T a, U b) -> decltype(a + b) {
    return a + b;
}

// C++14 简化
template<typename T, typename U>
auto add(T a, U b) {
    return a + b;
}
```

***

## 三、nullptr（空指针）

```cpp
// 旧方式
int* p1 = NULL;           // 宏，可能有问题
int* p2 = 0;              // 整数，不安全

// C++11
int* p3 = nullptr;        // 类型安全

// 函数重载
void f(int);
void f(int*);

f(NULL);      // 调用 f(int)，可能不是预期
f(nullptr);   // 调用 f(int*)，正确
```

***

## 四、constexpr（编译时计算）

```cpp
// 编译时常量
constexpr int MAX = 100;
constexpr double PI = 3.14159;

// 编译时函数
constexpr int square(int n) {
    return n * n;
}

int arr[square(5)];       // 编译时计算，数组大小 25

// 运行时也可用
int x = 5;
int y = square(x);        // 运行时计算

// C++14：允许更多语句
constexpr int factorial(int n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);
}

// C++17：if constexpr
template<typename T>
auto get_value(T t) {
    if constexpr (is_integral_v<T>) {
        return t * 2;
    } else {
        return t;
    }
}
```

***

## 五、std::tuple（元组）

```cpp
#include <tuple>

// 创建
tuple<int, double, string> t1(1, 3.14, "hello");
auto t2 = make_tuple(1, 3.14, "hello");

// 访问
cout << get<0>(t1) << endl;  // 1
cout << get<1>(t1) << endl;  // 3.14
cout << get<2>(t1) << endl;  // "hello"

// 结构化绑定（C++17）
auto [a, b, c] = t1;
cout << a << " " << b << " " << c << endl;

// tie 解包
int x;
double y;
string z;
tie(x, y, z) = t1;

// 忽略某些值
tie(x, ignore, z) = t1;
```

**实际运用**：
```cpp
// 多返回值
tuple<int, string, double> parse(const string& s) {
    return {1, "result", 3.14};
}

auto [id, name, value] = parse("input");

// 字典序比较
tuple<int, int> p1 = {1, 2};
tuple<int, int> p2 = {1, 3};
bool less = p1 < p2;  // true
```

***

## 六、结构化绑定（C++17）

```cpp
// pair
auto p = make_pair(1, 2.0);
auto [a, b] = p;

// tuple
auto t = make_tuple(1, 2.0, "hello");
auto [x, y, z] = t;

// 数组
int arr[3] = {1, 2, 3};
auto [a, b, c] = arr;

// 结构体
struct Point { int x, y; };
Point pt{1, 2};
auto [px, py] = pt;

// map 遍历
map<string, int> m = {{"a", 1}, {"b", 2}};
for (auto& [key, value] : m) {
    cout << key << ": " << value << endl;
    value++;  // 可以修改
}
```

***

## 七、折叠表达式（C++17）

```cpp
// 一元右折叠：(pack op ...)
template<typename... Args>
auto sum(Args... args) {
    return (args + ...);  // ((a1 + a2) + a3) + ...
}

sum(1, 2, 3, 4, 5);  // 15

// 一元左折叠：(... op pack)
template<typename... Args>
auto sum_left(Args... args) {
    return (... + args);  // 同上
}

// 打印所有参数
template<typename... Args>
void print(Args... args) {
    (cout << ... << args) << endl;
}
print(1, " ", 2, " ", 3);  // 1 2 3

// 带初始值的二元折叠
template<typename... Args>
auto sum_with_init(Args... args) {
    return (0 + ... + args);  // 初始值 0
}

// 全部为真
template<typename... Args>
bool all(Args... args) {
    return (... && args);
}
all(true, true, false);  // false

// 任一为真
template<typename... Args>
bool any(Args... args) {
    return (... || args);
}
any(false, true, false);  // true
```

**实际运用**：
```cpp
// 调用多个函数
template<typename... Funcs>
void call_all(Funcs... funcs) {
    (funcs(), ...);
}

call_all(
    [] { cout << "A\n"; },
    [] { cout << "B\n"; },
    [] { cout << "C\n"; }
);

// 容器大小之和
template<typename... Containers>
size_t total_size(const Containers&... containers) {
    return (containers.size() + ...);
}

total_size(v1, v2, v3);
```

***

## 八、std::optional（可选值）

```cpp
#include <optional>

// 创建
optional<int> o1;              // 空
optional<int> o2 = 42;         // 有值
optional<int> o3 = nullopt;    // 空

// 检查和访问
if (o2.has_value()) {
    cout << o2.value() << endl;
    cout << *o2 << endl;
}

// 默认值
int x = o1.value_or(0);

// 重置
o2.reset();
o2 = 100;
```

**实际运用**：
```cpp
optional<int> findValue(const vector<int>& v, int target) {
    auto it = find(v.begin(), v.end(), target);
    if (it != v.end()) return *it;
    return nullopt;
}

auto result = findValue({1, 2, 3}, 2);
if (result) {
    cout << "Found: " << *result << endl;
}
```

***

## 九、std::variant（类型安全联合体）

```cpp
#include <variant>

// 定义
variant<int, double, string> v;

// 赋值
v = 42;
v = 3.14;
v = "hello";

// 访问
cout << get<int>(v) << endl;      // 类型错误抛异常

// 安全访问
if (holds_alternative<double>(v)) {
    cout << get<double>(v) << endl;
}

// visit 访问
visit([](auto&& arg) {
    cout << arg << endl;
}, v);
```

**实际运用**：
```cpp
variant<int, string> process(int x) {
    if (x >= 0) return x * 2;
    return "negative";
}

auto r = process(5);
visit([](auto&& arg) {
    cout << arg << endl;
}, r);
```

***

## 十、std::string_view（字符串视图）

```cpp
#include <string_view>

// 零拷贝字符串视图
string s = "hello world";
string_view sv = s;

// 操作
cout << sv.substr(0, 5) << endl;
cout << sv.find("world") << endl;

// 函数参数
void process(string_view sv) {
    cout << sv << endl;
}

process("hello");        // const char*
process(s);              // string
process(s.substr(0, 5)); // 子串
```

**注意**：不要返回局部 string 的 string_view！

***

## 十一、std::span（数组视图）

```cpp
#include <span>

// 非拥有的数组视图
int arr[] = {1, 2, 3, 4, 5};
span<int> s(arr);

// 从 vector
vector<int> v = {1, 2, 3, 4, 5};
span<int> sp(v);

// 子视图
span<int> first3 = sp.first(3);
span<int> last2 = sp.last(2);
span<int> mid = sp.subspan(1, 3);

// 函数参数
void process(span<int> data) {
    for (int x : data) { }
}

int arr[] = {1, 2, 3};
vector<int> v = {4, 5, 6};
process(arr);
process(v);
```

***

## 十二、if/switch 初始化（C++17）

```cpp
// if 初始化
if (auto it = m.find(key); it != m.end()) {
    cout << it->second << endl;
}

// 锁作用域
if (auto lock = unique_lock(mtx); lock.owns_lock()) {
    // 临界区
}

// switch 初始化
switch (auto x = getValue(); x) {
    case 1: break;
    case 2: break;
}
```

***

## 十三、内联变量（C++17）

```cpp
// 头文件中定义静态成员
struct Widget {
    static inline int count = 0;
};

// 内联常量
inline constexpr int MAX_SIZE = 100;
```

***

## 十四、std::format（C++20，部分编译器不支持）

```cpp
#include <format>

// 基本格式化
string s1 = format("Hello, {}!", "World");

// 位置参数
string s2 = format("{0} {1} {0}", "a", "b");

// 格式说明
string s3 = format("{:.2f}", 3.14159);  // "3.14"
string s4 = format("{:x}", 255);        // "ff"
```

***

## 十五、指定初始化器（C++20）

```cpp
struct Config {
    string host = "localhost";
    int port = 8080;
    bool debug = false;
};

Config c1 {
    .host = "example.com",
    .port = 443
};
```

***

## 十六、记忆口诀

> **initializer_list 初始化，auto 推导类型**
> **nullptr 空指针，constexpr 编译时**
> **tuple 多返回，结构化绑定解包**
> **折叠表达式展开，optional 可选值**
> **string_view 零拷贝，span 数组视图**

***

## 快速参考

| 需求 | 选择 |
|------|------|
| 列表初始化 | `initializer_list` |
| 简化类型声明 | `auto` |
| 多返回值 | `tuple` + 结构化绑定 |
| 可能为空的值 | `optional<T>` |
| 多种类型 | `variant<T1, T2>` |
| 字符串参数 | `string_view` |
| 数组参数 | `span<T>` |
| 可变参数展开 | 折叠表达式 |
| 编译时计算 | `constexpr` |
