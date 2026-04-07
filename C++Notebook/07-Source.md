# C++ 资源管理完全指南

***

## 核心速查

| 概念 | 作用 | 使用场景 |
|------|------|---------|
| RAII | 资源自动释放 | 任何资源管理类 |
| `unique_ptr` | 独占所有权 | 明确单一所有者 |
| `shared_ptr` | 共享所有权 | 多对象共享资源 |
| `weak_ptr` | 弱引用 | 打破循环引用 |
| `move` | 转换为右值 | 转移所有权 |
| `forward` | 保持左右值 | 完美转发 |

***

## 一、RAII（资源获取即初始化）

**核心思想：资源在构造函数中获取，在析构函数中释放。**

```cpp
// ❌ 坏：手动管理资源（容易忘记、异常不安全）
void bad() {
    int* p = new int[100];
    process();  // 如果抛出异常，p 永远不会被 delete
    delete[] p;
}

// ✅ 好：RAII 自动管理
class IntArray {
    int* data_;
    size_t size_;
public:
    IntArray(size_t size) : size_(size), data_(new int[size]) {}
    ~IntArray() { delete[] data_; }  // 自动释放
    
    int& operator[](size_t i) { return data_[i]; }
};

void good() {
    IntArray arr(100);
    process();  // 即使抛出异常，析构函数也会自动释放
}
```

**RAII 的价值**：
- 异常安全：无论函数如何退出，资源都会释放
- 代码简洁：无需手动释放
- 避免泄漏：资源生命周期与对象绑定

***

## 二、智能指针

### unique_ptr（独占所有权）

```cpp
#include <memory>

// 创建
unique_ptr<int> p1 = make_unique<int>(42);
unique_ptr<int> p2(new int(42));  // 不推荐

// 不能拷贝，只能移动
// unique_ptr<int> p3 = p1;  // ❌ 错误！
unique_ptr<int> p3 = move(p1);  // ✅ 移动，p1 变为 nullptr

// 访问
int x = *p3;
int* raw = p3.get();

// 重置
p3.reset();          // 释放并置空
p3.reset(new int(100));

// 数组版本
unique_ptr<int[]> arr = make_unique<int[]>(100);
arr[0] = 42;
```

**自定义删除器**：
```cpp
// 文件句柄
auto file_deleter = [](FILE* f) { if (f) fclose(f); };
unique_ptr<FILE, decltype(file_deleter)> file(fopen("test.txt", "r"), file_deleter);

// 动态数组
unique_ptr<int[]> arr(new int[100]);  // 自动调用 delete[]
```

### shared_ptr（共享所有权）

```cpp
// 创建
shared_ptr<int> p1 = make_shared<int>(42);
shared_ptr<int> p2 = p1;  // 引用计数 +1
shared_ptr<int> p3 = p1;  // 引用计数 +1

cout << p1.use_count();  // 3

// 最后一个销毁时释放
p1.reset();  // 引用计数 -1，变为 2
p2.reset();  // 引用计数 -1，变为 1
p3.reset();  // 引用计数 -1，变为 0，释放内存
```

**shared_ptr 的开销**：
- 控制块（引用计数）额外内存
- 原子操作保证线程安全

### weak_ptr（弱引用）

**解决循环引用问题**：
```cpp
// ❌ 问题：循环引用导致内存泄漏
struct BadNode {
    shared_ptr<BadNode> next;
    ~BadNode() { cout << "deleted" << endl; }
};

auto n1 = make_shared<BadNode>();
auto n2 = make_shared<BadNode>();
n1->next = n2;
n2->next = n1;  // 循环！永远不会释放

// ✅ 解决：用 weak_ptr 打破循环
struct GoodNode {
    weak_ptr<GoodNode> next;  // 不增加引用计数
    
    shared_ptr<GoodNode> getNext() {
        return next.lock();  // 获取 shared_ptr，可能为空
    }
    
    ~GoodNode() { cout << "deleted" << endl; }
};
```

**weak_ptr 用法**：
```cpp
shared_ptr<int> sp = make_shared<int>(42);
weak_ptr<int> wp = sp;

// 检查是否过期
if (!wp.expired()) {
    // 获取 shared_ptr
    if (auto p = wp.lock()) {
        cout << *p << endl;
    }
}
```

### 智能指针选择

| 场景 | 选择 | 原因 |
|------|------|------|
| 单一所有者 | `unique_ptr` | 零开销，最安全 |
| 共享所有权 | `shared_ptr` | 引用计数自动管理 |
| 观察者模式 | `weak_ptr` | 不影响生命周期 |
| 循环引用 | `weak_ptr` | 打破循环 |

***

## 三、三法则 / 五法则 / 零规则

### 三法则（C++98）

**如果自定义了析构、拷贝构造、拷贝赋值中的任何一个，就需要自定义全部三个。**

```cpp
class RuleOfThree {
    int* data_;
    size_t size_;
public:
    RuleOfThree(size_t size) : size_(size), data_(new int[size]) {}
    
    ~RuleOfThree() { delete[] data_; }
    
    RuleOfThree(const RuleOfThree& other) 
        : size_(other.size_), data_(new int[other.size_]) {
        copy(other.data_, other.data_ + size_, data_);
    }
    
    RuleOfThree& operator=(const RuleOfThree& other) {
        if (this != &other) {
            delete[] data_;
            size_ = other.size_;
            data_ = new int[size_];
            copy(other.data_, other.data_ + size_, data_);
        }
        return *this;
    }
};
```

### 五法则（C++11）

**三法则 + 移动构造 + 移动赋值**

```cpp
class RuleOfFive {
    int* data_;
    size_t size_;
public:
    RuleOfFive(size_t size) : size_(size), data_(new int[size]) {}
    
    ~RuleOfFive() { delete[] data_; }
    
    // 拷贝语义
    RuleOfFive(const RuleOfFive& other) 
        : size_(other.size_), data_(new int[other.size_]) {
        copy(other.data_, other.data_ + size_, data_);
    }
    
    RuleOfFive& operator=(const RuleOfFive& other) {
        if (this != &other) {
            delete[] data_;
            size_ = other.size_;
            data_ = new int[size_];
            copy(other.data_, other.data_ + size_, data_);
        }
        return *this;
    }
    
    // 移动语义
    RuleOfFive(RuleOfFive&& other) noexcept
        : data_(exchange(other.data_, nullptr))
        , size_(exchange(other.size_, 0)) {}
    
    RuleOfFive& operator=(RuleOfFive&& other) noexcept {
        if (this != &other) {
            delete[] data_;
            data_ = exchange(other.data_, nullptr);
            size_ = exchange(other.size_, 0);
        }
        return *this;
    }
};
```

### 零规则（推荐）

**使用 RAII 类型作为成员，让编译器自动生成所有特殊成员函数。**

```cpp
// ✅ 零规则：不需要写任何特殊成员函数
class ZeroRule {
    vector<int> data_;       // 自动管理内存
    string name_;            // 自动管理
    unique_ptr<Logger> log_; // 自动管理
    
    // 编译器自动生成：
    // - 析构函数（调用成员的析构）
    // - 拷贝构造/赋值（调用成员的拷贝）
    // - 移动构造/赋值（调用成员的移动）
};

// unique_ptr 成员会禁止拷贝，但允许移动
ZeroRule a;
ZeroRule b = a;              // ❌ 错误：unique_ptr 不可拷贝
ZeroRule c = move(a);        // ✅ 移动构造
```

***

## 四、移动语义（std::move）

**将对象转换为右值引用，启用移动语义，避免深拷贝。**

```cpp
// move 本质是类型转换
template<typename T>
constexpr remove_reference_t<T>&& move(T&& t) noexcept {
    return static_cast<remove_reference_t<T>&&>(t);
}

// 使用示例
vector<int> v1 = {1, 2, 3, 4, 5};
vector<int> v2 = move(v1);  // 转移内部指针，O(1)

// v1 现在为空（有效但未指定状态）
cout << v1.size();  // 通常为 0

// 可以安全地重新赋值
v1 = {6, 7, 8};  // ✅

// 移动 unique_ptr
unique_ptr<int> p1 = make_unique<int>(42);
unique_ptr<int> p2 = move(p1);  // 转移所有权
// p1 现在为 nullptr
```

**移动语义的实际运用**：
```cpp
// 大对象返回
vector<string> createLargeVector() {
    vector<string> result(10000);
    return result;  // 自动移动，无需显式 move
}

// 容器元素转移
vector<string> src = {"a", "b", "c"};
vector<string> dst;
for (auto& s : src) {
    dst.push_back(move(s));  // 移动而非拷贝
}
```

***

## 五、完美转发（std::forward）

**保持参数的左右值属性，原样传递给另一个函数。**

```cpp
// forward 实现
template<typename T>
constexpr T&& forward(remove_reference_t<T>& t) noexcept {
    return static_cast<T&&>(t);
}

// 典型应用：工厂函数
template<typename T, typename... Args>
unique_ptr<T> create(Args&&... args) {
    return make_unique<T>(forward<Args>(args)...);
}

// 示例
struct Widget {
    Widget(int x, string s) : id(x), name(move(s)) {}
    int id;
    string name;
};

string s = "hello";
auto w1 = create<Widget>(42, s);      // s 是左值，会拷贝
auto w2 = create<Widget>(42, "world"); // "world" 是右值，会移动
```

**完美转发的价值**：
```cpp
// 不用 forward：右值变左值
template<typename T>
void wrapper(T&& arg) {
    process(arg);  // arg 永远是左值
}

// 用 forward：保持原样
template<typename T>
void wrapper(T&& arg) {
    process(forward<T>(arg));  // 保持 arg 的左右值属性
}

void process(int& x) { cout << "lvalue" << endl; }
void process(int&& x) { cout << "rvalue" << endl; }

int val = 10;
wrapper(val);   // lvalue
wrapper(20);    // rvalue
```

***

## 六、实际运用

### 资源管理类

```cpp
// 文件句柄
class FileHandle {
    FILE* file_;
public:
    FileHandle(const char* path, const char* mode) 
        : file_(fopen(path, mode)) {
        if (!file_) throw runtime_error("Cannot open file");
    }
    ~FileHandle() { if (file_) fclose(file_); }
    
    FILE* get() const { return file_; }
    
    // 禁止拷贝
    FileHandle(const FileHandle&) = delete;
    FileHandle& operator=(const FileHandle&) = delete;
    
    // 允许移动
    FileHandle(FileHandle&& other) noexcept : file_(exchange(other.file_, nullptr)) {}
};

// 使用
{
    FileHandle f("data.txt", "r");
    // 使用 f.get()...
}  // 自动关闭
```

### 工厂函数

```cpp
template<typename T, typename... Args>
shared_ptr<T> createShared(Args&&... args) {
    return make_shared<T>(forward<Args>(args)...);
}

template<typename T, typename... Args>
unique_ptr<T> createUnique(Args&&... args) {
    return make_unique<T>(forward<Args>(args)...);
}
```

### 延迟初始化

```cpp
class LazyLoader {
    mutable unique_ptr<Data> data_;
public:
    const Data& get() const {
        if (!data_) {
            data_ = make_unique<Data>(loadData());
        }
        return *data_;
    }
};
```

***

## 七、最佳实践

```cpp
// ✅ 推荐：零规则 + 智能指针
class Modern {
    vector<int> data_;
    string name_;
    unique_ptr<Logger> logger_;
};

// ✅ 推荐：用 make_unique/make_shared
auto p = make_unique<Widget>(42);
auto sp = make_shared<Widget>(42);

// ✅ 推荐：用 move 转移大对象
vector<string> v1(10000);
vector<string> v2 = move(v1);

// ✅ 推荐：完美转发参数
template<typename... Args>
auto create(Args&&... args) {
    return make_unique<Widget>(forward<Args>(args)...);
}

// ❌ 避免：手动 new/delete
// int* p = new int(42);
// delete p;

// ❌ 避免：裸指针持有所有权
// Widget* w = new Widget();

// ❌ 避免：循环引用
// struct Node { shared_ptr<Node> next; };
```

***

## 八、记忆口诀

> **RAII 是基础，构造获取析构放**
> **智能指针三兄弟，独占共享弱引用**
> **三/五法则手动管，零规则是首选**
> **move 转右值，forward 完美传**
