# C++ 笔记

C++ 核心知识速查手册，按常用程度排序。

***

## 文件索引

| 序号 | 文件 | 内容 | 常用程度 |
|------|------|------|---------|
| 01 | [STL-containers](01-STL-containers.md) | STL 容器速查 | ⭐⭐⭐⭐⭐ |
| 02 | [Iterator](02-Iterator.md) | 迭代器组合拳 | ⭐⭐⭐⭐⭐ |
| 03 | [Lambda](03-Lambda.md) | Lambda 表达式 | ⭐⭐⭐⭐⭐ |
| 04 | [Algorithm](04-Algorithm.md) | STL 算法 | ⭐⭐⭐⭐⭐ |
| 05 | [Class](05-Class.md) | 类与设计模式 | ⭐⭐⭐⭐ |
| 06 | [Template](06-Template.md) | 模板编程 | ⭐⭐⭐⭐ |
| 07 | [Source](07-Source.md) | 资源管理 | ⭐⭐⭐⭐ |
| 08 | [ModernCpp](08-ModernCpp.md) | 现代特性 | ⭐⭐⭐ |
| 09 | [PythonStyle](09-PythonStyle.md) | Python 风格工具 | ⭐⭐⭐ |
| 10 | [MemoryManagement](10-MemoryManagement.md) | 内存管理与编译原理 | ⭐⭐⭐⭐ |

***

## 详细介绍

### 01-STL-containers.md - STL 容器速查

**内容**：`string`、`vector`、`deque`、`list`、`set`、`map`、`unordered_set`、`unordered_map`、`stack`、`queue`、`priority_queue`

**核心用法**：
```cpp
// string
s.substr(0, 5); s.find("abc"); s += " world";

// vector
v.push_back(x); v.pop_back(); v[i]; v.size();

// map
m[key] = value; m.count(key); m.contains(key);  // C++20

// set
s.insert(x); s.erase(x); s.count(x);

// unordered_*（哈希实现，O(1) 查找）
unordered_map<string, int> um;
unordered_set<int> us;
```

**价值**：容器选择决策，方法速查

**场景**：日常开发，容器选择，方法查阅

---

### 02-Iterator.md - 迭代器组合拳

**内容**：迭代器基础、遍历方式、删除/查找/复制组合拳、插入迭代器、流迭代器

**核心用法**：
```cpp
// 删除特定值
v.erase(remove(v.begin(), v.end(), val), v.end());

// 条件删除
v.erase(remove_if(v.begin(), v.end(), pred), v.end());

// 去重
sort(v.begin(), v.end());
v.erase(unique(v.begin(), v.end()), v.end());

// 过滤复制
copy_if(src.begin(), src.end(), back_inserter(dst), pred);

// 读文件
vector<int> v{istream_iterator<int>(fin), istream_iterator<int>()};
```

**价值**：算法与迭代器的经典组合模式

**场景**：删除元素、去重、过滤、文件读写

---

### 03-Lambda.md - Lambda 表达式

**内容**：基本语法、捕获列表、mutable、返回类型、泛型 Lambda、模板 Lambda（C++20）

**核心用法**：
```cpp
// 基本形式
[](int x) { return x * 2; }

// 捕获
[=] { return a + b; };      // 值捕获所有
[&] { return a + b; };      // 引用捕获所有
[=, &a] { a++; };           // 混合捕获
[ptr = move(p)] { };        // 初始化捕获（C++14）

// 泛型 Lambda（C++14）
[](auto a, auto b) { return a + b; }

// 算法谓词
sort(v.begin(), v.end(), [](int a, int b) { return a > b; });
```

**价值**：就地定义匿名函数，算法谓词

**场景**：排序比较、过滤条件、回调函数、延迟执行

---

### 04-Algorithm.md - STL 算法

**内容**：`algorithm` 库（排序、查找、变换、复制）、`numeric` 库、`ranges` 库（C++20）、`utility` 库

**核心用法**：
```cpp
// 排序
sort(v.begin(), v.end());
stable_sort(v.begin(), v.end());
partial_sort(v.begin(), v.begin() + 3, v.end());

// 查找
auto it = find(v.begin(), v.end(), val);
bool found = binary_search(v.begin(), v.end(), val);

// 变换
transform(src.begin(), src.end(), dst.begin(), f);

// 聚合
int sum = accumulate(v.begin(), v.end(), 0);

// ranges（C++20）
auto result = v | views::filter(pred) | views::transform(f);
```

**价值**：避免重复造轮子，代码更简洁

**场景**：排序、查找、变换、聚合、链式操作

---

### 05-Class.md - 类与设计模式

**内容**：类基本结构、构造/析构函数、继承与多态、struct 对比、常用设计模式（Pimpl、工厂方法、CRTP）

**核心用法**：
```cpp
// 构造函数初始化列表
class Widget {
    int x_, y_;
public:
    Widget(int x, int y) : x_(x), y_(y) { }
};

// 虚析构函数
class Base {
public:
    virtual ~Base() = default;
};

// override 明确重写
class Derived : public Base {
public:
    void foo() override { }
};

// struct vs class
struct Point { int x, y; };  // 默认 public
class Data { int x; };       // 默认 private
```

**价值**：OOP 基础，设计模式实践

**场景**：类设计、继承体系、封装、设计模式

---

### 06-Template.md - 模板编程

**内容**：函数模板、类模板、全特化、偏特化、可变参数模板、非类型参数、Concept（C++20）、CRTP

**核心用法**：
```cpp
// 函数模板
template<typename T>
T max(T a, T b) { return a > b ? a : b; }

// 类模板
template<typename T>
class Box { T data; };

// 可变参数
template<typename... Args>
void print(Args... args) { (cout << ... << args) << endl; }

// Concept（C++20）
template<std::integral T>
T square(T x) { return x * x; }

// CRTP 静态多态
template<typename Derived>
class Base { };
```

**价值**：泛型编程基础，类型安全约束

**场景**：通用函数、通用类、类型约束、静态多态

---

### 07-Source.md - 资源管理

**内容**：RAII、`unique_ptr`、`shared_ptr`、`weak_ptr`、移动语义、完美转发

**核心用法**：
```cpp
// RAII
class FileHandle {
    FILE* file_;
public:
    FileHandle(const char* path) : file_(fopen(path, "r")) { }
    ~FileHandle() { if (file_) fclose(file_); }
};

// unique_ptr（独占）
auto p = make_unique<int>(42);

// shared_ptr（共享）
auto p1 = make_shared<int>(42);
auto p2 = p1;

// weak_ptr（打破循环引用）
weak_ptr<int> wp = p1;

// 移动语义
vector<string> v;
v.push_back(move(str));  // 转移所有权

// 完美转发
template<typename T>
void wrapper(T&& arg) { func(forward<T>(arg)); }
```

**价值**：自动资源管理，避免内存泄漏

**场景**：动态内存、文件句柄、锁管理、所有权转移

---

### 08-ModernCpp.md - 现代特性

**内容**：`initializer_list`、`auto`/`decltype`、`nullptr`、`constexpr`、`tuple`、结构化绑定、折叠表达式、`optional`、`variant`、`string_view`、`span`、`format`

**核心用法**：
```cpp
// initializer_list
Vector v = {1, 2, 3, 4, 5};

// auto + 结构化绑定
auto [key, value] = *m.begin();

// constexpr
constexpr int MAX = 100;
constexpr int square(int n) { return n * n; }

// tuple 多返回值
auto [id, name, score] = make_tuple(1, "Alice", 90);

// 折叠表达式
template<typename... Args>
auto sum(Args... args) { return (args + ...); }

// optional 可选值
optional<int> findValue(int key);

// string_view / span（零拷贝）
void process(string_view sv);
void process(span<int> data);
```

**价值**：现代 C++ 必备特性，代码更安全简洁

**场景**：类型推导、编译时计算、多返回值、可选值、零拷贝

---

### 09-PythonStyle.md - Python 风格工具

**内容**：`enumerate`、`zip`、`reversed`、`range`、`map/filter`、`any/all`、`sum/min/max`、`sorted`、列表推导式风格、字典/集合/列表/字符串操作对照

**核心用法**：
```cpp
// enumerate（C++23）
for (auto&& [i, x] : v | views::enumerate) { }

// zip（C++23）
for (auto&& [a, b] : views::zip(v1, v2)) { }

// filter + transform
auto result = nums 
    | views::filter([](int x) { return x > 0; })
    | views::transform([](int x) { return x * 2; });

// any/all
bool hasNeg = ranges::any_of(nums, [](int x) { return x < 0; });

// 字符串分割/连接
auto parts = split("a,b,c", ',');
auto joined = join(parts, ",");
```

**价值**：Python 用户快速上手 C++，代码风格统一

**场景**：Python 转 C++、数据结构对照、链式操作

---

### 10-MemoryManagement.md - 内存管理与编译原理

**内容**：内存区域划分、RAII、智能指针深度解析、编译四阶段、性能分析、内存安全与调试、高级主题（自定义分配器、内存映射、原子操作）

**核心用法**：
```cpp
// 栈 vs 堆
int arr[100];              // 栈：O(1) 分配
int* p = new int[100];     // 堆：O(n) 最坏

// RAII
{
    std::unique_ptr<int> p = std::make_unique<int>(42);
    std::shared_ptr<int> s = std::make_shared<int>(42);
}  // 自动释放

// 编译阶段
// 预处理 → 编译 → 汇编 → 链接
g++ -E test.cpp -o test.ii   // 预处理
g++ -S test.cpp -o test.s    // 编译
g++ -c test.cpp -o test.o    // 汇编
g++ test.o -o test           // 链接

// 调试
g++ -fsanitize=address -g test.cpp -o test  // ASan
valgrind --leak-check=full ./test           // Valgrind
```

**价值**：深入理解 C++ 底层机制，写出高效安全代码

**场景**：性能优化、内存调试、理解编译错误、底层开发

***

## 快速查找

| 需求 | 查看文件 |
|------|---------|
| 选择容器 | 01-STL-containers.md |
| 删除/去重元素 | 02-Iterator.md |
| 写 Lambda | 03-Lambda.md |
| 排序/查找/变换 | 04-Algorithm.md |
| 设计类 | 05-Class.md |
| 写模板 | 06-Template.md |
| 智能指针/RAII | 07-Source.md |
| auto/constexpr/tuple | 08-ModernCpp.md |
| Python 风格写法 | 09-PythonStyle.md |
| 内存管理/编译原理 | 10-MemoryManagement.md |

***

## 学习路径

```
基础 → 进阶 → 高级 → 扩展
  │       │       │       │
  ├─ 01-STL-containers   ├─ 05-Class       ├─ 06-Template       ├─ 08-ModernCpp
  ├─ 02-Iterator         ├─ 06-Template    ├─ 07-Source         ├─ 09-PythonStyle
  ├─ 03-Lambda           ├─ 10-Memory      └─ 10-MemoryManagement
  └─ 04-Algorithm        └─ 07-Source
```

**建议顺序**：
1. **01-STL-containers** → 熟悉常用容器
2. **02-Iterator** → 掌握迭代器操作
3. **03-Lambda** → 学会匿名函数
4. **04-Algorithm** → 掌握常用算法
5. **05-Class** → OOP 与设计模式
6. **06-Template** → 泛型编程
7. **07-Source** → 资源管理
8. **10-MemoryManagement** → 内存管理与编译原理（深入）
9. **08-ModernCpp** → 现代特性（宽泛）
10. **09-PythonStyle** → Python 对照（宽泛）
