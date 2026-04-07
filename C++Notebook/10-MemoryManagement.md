# C++ 内存管理与编译原理深度指南

***

## 核心速查

| 主题 | 关键概念 |
|------|---------|
| 内存模型 | 栈/堆/静态存储/线程局部存储 |
| 生命周期 | 自动/静态/动态/线程 |
| 所有权 | unique_ptr/shared_ptr/weak_ptr |
| RAII | 资源获取即初始化 |
| 编译阶段 | 预处理→编译→汇编→链接 |
| 性能开销 | 分配/释放/缓存/对齐 |

***

## 第一部分：内存管理基础

### 1.1 内存区域划分

C++ 程序的内存布局：

```
高地址
┌─────────────────┐
│   命令行参数     │
├─────────────────┤
│   环境变量       │
├─────────────────┤
│   栈 (Stack)     │  ← 向下增长
│        ↓        │
│                 │
│        ↑        │
│   堆 (Heap)      │  ← 向上增长
├─────────────────┤
│   BSS 段         │  未初始化全局/静态变量
├─────────────────┤
│   数据段         │  已初始化全局/静态变量
├─────────────────┤
│   代码段         │  只读，存放机器指令
└─────────────────┘
低地址
```

#### 栈内存 (Stack)

```cpp
void func() {
    int a = 10;           // 栈上分配，自动生命周期
    int arr[100];         // 栈上分配 400 字节
    std::string s = "hi"; // s 在栈上，数据可能在堆上
}                          // 离开作用域自动销毁
```

**特性**：
- 分配速度：O(1)，仅移动栈指针
- 大小限制：通常 1MB-8MB（系统相关）
- 生命周期：作用域绑定，自动管理
- 内存布局：连续，缓存友好

#### 堆内存 (Heap)

```cpp
void func() {
    int* p = new int(10);        // 堆上分配
    int* arr = new int[100];     // 堆上分配 400 字节
    std::vector<int> v(1000);    // v 在栈上，数据在堆上
    
    delete p;                     // 手动释放
    delete[] arr;                 // 数组需要 delete[]
}                                 // v 析构时自动释放堆数据
```

**特性**：
- 分配速度：O(n) 最坏情况，需要查找空闲块
- 大小限制：受系统虚拟内存限制
- 生命周期：手动控制（或通过 RAII）
- 内存布局：不连续，可能碎片化

#### 静态存储 (Static Storage)

```cpp
int global_var = 10;           // 全局变量，程序启动时初始化

void func() {
    static int count = 0;       // 静态局部变量，首次调用初始化
    count++;
}

namespace {
    int internal_linkage = 5;   // 内部链接，本编译单元可见
}
```

**初始化时机**：
- 全局变量：`main()` 之前
- 静态局部变量：首次执行到声明处
- 动态初始化顺序问题：跨编译单元未定义

#### 线程局部存储 (Thread Local Storage)

```cpp
thread_local int tls_var = 0;   // 每个线程独立副本

void func() {
    thread_local static int count = 0;  // 线程局部静态变量
    count++;
}
```

**特性**：
- 每个线程独立实例
- 生命周期：线程创建到结束
- 性能开销：访问需要间接寻址

### 1.2 存储期与生命周期

```cpp
// 自动存储期
void auto_storage() {
    int a;                    // 作用域开始 → 构造
}                             // 作用域结束 → 析构

// 静态存储期
static int global;            // 程序启动 → 构造
                              // 程序结束 → 析构

// 动态存储期
void dynamic_storage() {
    int* p = new int;         // new → 构造
    delete p;                 // delete → 析构
}

// 线程存储期
thread_local int tls;         // 线程启动 → 构造
                              // 线程结束 → 析构
```

### 1.3 对象生命周期详解

```cpp
class Widget {
public:
    Widget() { std::cout << "构造\n"; }
    ~Widget() { std::cout << "析构\n"; }
};

void lifetime_demo() {
    // 1. 自动对象
    {
        Widget w;             // 进入作用域：构造
    }                         // 离开作用域：析构
    
    // 2. 动态对象
    Widget* p = new Widget;   // 构造
    delete p;                 // 析构
    
    // 3. 异常安全
    Widget* raw = new Widget;
    throw std::runtime_error("error");  // 泄漏！raw 未释放
    
    // 4. RAII 解决
    auto smart = std::make_unique<Widget>();
    throw std::runtime_error("error");  // 安全！smart 自动析构
}
```

***

## 第二部分：RAII 与智能指针

### 2.1 RAII 原则

**核心思想**：将资源生命周期绑定到对象生命周期

```cpp
// 传统 C 风格：手动管理
void bad_example() {
    FILE* f = fopen("data.txt", "r");
    if (!f) return;
    
    // ... 可能抛异常的代码 ...
    
    fclose(f);  // 可能不会执行！
}

// RAII 风格：自动管理
void good_example() {
    std::ifstream f("data.txt");
    if (!f) return;
    
    // ... 可能抛异常的代码 ...
    
}  // f 析构自动关闭文件
```

### 2.2 手动实现 RAII

```cpp
template<typename T>
class ScopedPtr {
    T* ptr_;
public:
    explicit ScopedPtr(T* p = nullptr) : ptr_(p) {}
    ~ScopedPtr() { delete ptr_; }
    
    ScopedPtr(const ScopedPtr&) = delete;
    ScopedPtr& operator=(const ScopedPtr&) = delete;
    
    ScopedPtr(ScopedPtr&& other) noexcept : ptr_(other.ptr_) {
        other.ptr_ = nullptr;
    }
    
    T* operator->() const { return ptr_; }
    T& operator*() const { return *ptr_; }
    explicit operator bool() const { return ptr_ != nullptr; }
};
```

### 2.3 unique_ptr 深度解析

```cpp
#include <memory>

// 独占所有权
void unique_ptr_demo() {
    // 创建方式
    auto p1 = std::make_unique<int>(42);
    auto p2 = std::unique_ptr<int>(new int(42));
    
    // 移动语义
    auto p3 = std::move(p1);  // p1 变为 nullptr
    
    // 自定义删除器
    auto deleter = [](FILE* f) { 
        std::cout << "Closing file\n";
        fclose(f); 
    };
    std::unique_ptr<FILE, decltype(deleter)> file(fopen("test.txt", "r"), deleter);
    
    // 数组支持
    auto arr = std::make_unique<int[]>(100);
    arr[0] = 1;  // operator[] 支持
}
```

**内存布局**：

```
unique_ptr<int>:
┌─────────────────┐
│   int* ptr_     │  8 字节（64位系统）
└─────────────────┘

unique_ptr<int, Deleter>:
┌─────────────────┬─────────────────┐
│   int* ptr_     │   Deleter       │  取决于 Deleter 大小
└─────────────────┴─────────────────┘
```

**空基类优化 (EBO)**：

```cpp
// 无状态删除器
struct NoOpDeleter {
    void operator()(int* p) const { /* no-op */ }
};

static_assert(sizeof(std::unique_ptr<int, NoOpDeleter>) == sizeof(int*));
// 无状态删除器不占用额外空间
```

### 2.4 shared_ptr 深度解析

```cpp
void shared_ptr_demo() {
    // 创建
    auto p1 = std::make_shared<int>(42);
    auto p2 = p1;  // 引用计数 +1
    
    std::cout << p1.use_count();  // 2
    
    // 弱引用
    std::weak_ptr<int> wp = p1;
    if (auto locked = wp.lock()) {  // 尝试提升
        std::cout << *locked;       // 安全访问
    }
    
    // 循环引用问题
    struct Node {
        std::shared_ptr<Node> next;
        std::shared_ptr<Node> prev;  // 危险！
        // 应该用 weak_ptr<Node> prev;
    };
}
```

**控制块结构**：

```
shared_ptr<int> 内存布局:

栈上:
┌─────────────────┐
│   int* ptr_     │  → 指向对象
│   CtrlBlock*    │  → 指向控制块
└─────────────────┘

堆上（控制块）:
┌─────────────────┐
│   strong_count  │  强引用计数
│   weak_count    │  弱引用计数
│   allocator     │  分配器（可选）
│   deleter       │  删除器（可选）
│   object        │  实际对象（make_shared 时）
└─────────────────┘
```

**make_shared vs new**：

```cpp
// 方式 1：两次分配
auto p1 = std::shared_ptr<int>(new int(42));
// 分配 1: int 对象
// 分配 2: 控制块

// 方式 2：一次分配
auto p2 = std::make_shared<int>(42);
// 分配 1: 控制块 + 对象（连续内存）
// 优势：减少内存碎片，提高缓存局部性
// 劣势：对象内存延迟释放（weak_ptr 存在时）
```

### 2.5 weak_ptr 与观察者模式

```cpp
class Observer {
public:
    virtual ~Observer() = default;
    virtual void notify() = 0;
};

class Subject {
    std::vector<std::weak_ptr<Observer>> observers_;
public:
    void attach(std::shared_ptr<Observer> o) {
        observers_.push_back(o);
    }
    
    void notify_all() {
        auto it = observers_.begin();
        while (it != observers_.end()) {
            if (auto obs = it->lock()) {
                obs->notify();
                ++it;
            } else {
                it = observers_.erase(it);  // 清理已销毁的观察者
            }
        }
    }
};
```

***

## 第三部分：编译过程详解

### 3.1 编译四阶段

```
源代码 (.cpp)
    │
    ▼ 预处理器
预处理后代码 (.ii)
    │
    ▼ 编译器
汇编代码 (.s)
    │
    ▼ 汇编器
目标文件 (.o/.obj)
    │
    ▼ 链接器
可执行文件
```

### 3.2 预处理阶段

```cpp
// 示例：test.cpp
#include <iostream>
#define MAX 100
#define SQUARE(x) ((x) * (x))

#ifdef DEBUG
#define LOG(x) std::cout << x << std::endl
#else
#define LOG(x)
#endif

int main() {
    int arr[MAX];
    int result = SQUARE(5);
    LOG("Debug mode");
    return 0;
}
```

**预处理命令**：

```bash
g++ -E test.cpp -o test.ii  # 仅预处理
```

**预处理后结果**：

```cpp
// test.ii（简化）
// #include <iostream> 展开为数千行代码...

int main() {
    int arr[100];
    int result = ((5) * (5));
    // LOG 在非 DEBUG 模式下为空
    return 0;
}
```

**常见预处理陷阱**：

```cpp
// 宏定义陷阱
#define SQUARE(x) x * x
int a = SQUARE(1 + 2);  // 展开为 1 + 2 * 1 + 2 = 5，不是 9

// 正确定义
#define SQUARE(x) ((x) * (x))
int a = SQUARE(1 + 2);  // ((1 + 2) * (1 + 2)) = 9

// 仍有问题：多次求值
#define MAX(a, b) ((a) > (b) ? (a) : (b))
int x = 1, y = 2;
int m = MAX(x++, y++);  // x 或 y 可能自增两次！

// 解决方案：使用内联函数
inline int max(int a, int b) { return a > b ? a : b; }
```

### 3.3 编译阶段

```bash
g++ -S test.cpp -o test.s  # 生成汇编代码
```

**汇编输出示例**：

```asm
; x86-64 汇编
main:
    push    rbp
    mov     rbp, rsp
    sub     rsp, 16
    
    ; int arr[100]
    ; 栈上分配 400 字节（编译时常量）
    
    ; int result = SQUARE(5)
    mov     DWORD PTR [rbp-4], 25
    
    mov     eax, 0
    leave
    ret
```

**编译优化级别**：

```bash
g++ -O0 test.cpp  # 无优化，便于调试
g++ -O1 test.cpp  # 基本优化
g++ -O2 test.cpp  # 标准优化（推荐）
g++ -O3 test.cpp  # 激进优化（可能增大代码体积）
g++ -Os test.cpp  # 优化代码大小
g++ -Ofast test.cpp  # O3 + 快速数学（不严格遵循 IEEE）
```

### 3.4 汇编阶段

```bash
g++ -c test.cpp -o test.o  # 生成目标文件
```

**目标文件结构**：

```
ELF 格式（Linux）:
┌─────────────────┐
│   ELF Header    │  文件类型、架构信息
├─────────────────┤
│   .text         │  代码段
├─────────────────┤
│   .rodata       │  只读数据
├─────────────────┤
│   .data         │  已初始化全局变量
├─────────────────┤
│   .bss          │  未初始化全局变量
├─────────────────┤
│   .symtab       │  符号表
├─────────────────┤
│   .rel.text     │  重定位表
├─────────────────┤
│   .strtab       │  字符串表
└─────────────────┘
```

**查看符号表**：

```bash
nm test.o
# 输出:
# 0000000000000000 T main
#                  U _GLOBAL__sub_I_main
#                  U __stack_chk_fail
```

### 3.5 链接阶段

```bash
g++ test.o -o test  # 链接生成可执行文件
```

**链接类型**：

```cpp
// 静态链接
// libmath.a 被完整复制到可执行文件中
g++ main.cpp -L. -lmath -static -o main

// 动态链接
// libmath.so 在运行时加载
g++ main.cpp -L. -lmath -o main
```

**链接过程**：

```
main.o + libmath.a + libc.a
         │
         ▼ 符号解析
    未定义符号 → 查找库文件
         │
         ▼ 重定位
    更新地址引用
         │
         ▼
    可执行文件
```

**常见链接错误**：

```cpp
// undefined reference
void foo();  // 声明
int main() { foo(); }  // 链接错误：foo 未定义

// multiple definition
// a.cpp
int x = 10;  // 定义

// b.cpp
int x = 20;  // 链接错误：重复定义

// 解决方案 1：extern
// a.cpp
int x = 10;  // 定义

// b.cpp
extern int x;  // 声明

// 解决方案 2：inline
// header.h
inline int x = 10;  // C++17 inline 变量

// 解决方案 3：命名空间匿名
// a.cpp
namespace { int x = 10; }  // 内部链接

// b.cpp
namespace { int x = 20; }  // 不同的 x
```

***

## 第四部分：性能分析

### 4.1 内存分配开销

```cpp
#include <chrono>
#include <iostream>

void benchmark_allocation() {
    constexpr int N = 1000000;
    
    // 栈分配
    auto start = std::chrono::high_resolution_clock::now();
    for (int i = 0; i < N; ++i) {
        int arr[10];  // 栈分配
        arr[0] = i;   // 防止优化掉
    }
    auto end = std::chrono::high_resolution_clock::now();
    std::cout << "Stack: " 
              << std::chrono::duration_cast<std::chrono::microseconds>(end - start).count()
              << " us\n";
    
    // 堆分配
    start = std::chrono::high_resolution_clock::now();
    for (int i = 0; i < N; ++i) {
        int* arr = new int[10];  // 堆分配
        arr[0] = i;
        delete[] arr;
    }
    end = std::chrono::high_resolution_clock::now();
    std::cout << "Heap: "
              << std::chrono::duration_cast<std::chrono::microseconds>(end - start).count()
              << " us\n";
}

// 典型结果：
// Stack: ~1000 us
// Heap:  ~50000 us（50 倍差距）
```

### 4.2 智能指针开销

```cpp
void smart_ptr_overhead() {
    constexpr int N = 10000000;
    
    // 原始指针
    auto start = std::chrono::high_resolution_clock::now();
    for (int i = 0; i < N; ++i) {
        int* p = new int(i);
        *p += 1;
        delete p;
    }
    auto end = std::chrono::high_resolution_clock::now();
    std::cout << "Raw: " 
              << std::chrono::duration_cast<std::chrono::milliseconds>(end - start).count()
              << " ms\n";
    
    // unique_ptr
    start = std::chrono::high_resolution_clock::now();
    for (int i = 0; i < N; ++i) {
        auto p = std::make_unique<int>(i);
        *p += 1;
    }
    end = std::chrono::high_resolution_clock::now();
    std::cout << "unique_ptr: "
              << std::chrono::duration_cast<std::chrono::milliseconds>(end - start).count()
              << " ms\n";
    
    // shared_ptr
    start = std::chrono::high_resolution_clock::now();
    for (int i = 0; i < N; ++i) {
        auto p = std::make_shared<int>(i);
        *p += 1;
    }
    end = std::chrono::high_resolution_clock::now();
    std::cout << "shared_ptr: "
              << std::chrono::duration_cast<std::chrono::milliseconds>(end - start).count()
              << " ms\n";
}

// 典型结果（优化开启）：
// Raw:        200 ms
// unique_ptr: 200 ms  （与原始指针相同）
// shared_ptr: 400 ms  （原子操作开销）
```

### 4.3 缓存优化

```cpp
// 缓存不友好：链表
struct Node {
    int data;
    Node* next;
};

void traverse_list(Node* head) {
    while (head) {
        process(head->data);
        head = head->next;  // 每次访问可能缓存未命中
    }
}

// 缓存友好：连续数组
void traverse_array(const std::vector<int>& arr) {
    for (int x : arr) {
        process(x);  // 预取器友好
    }
}
```

**缓存行大小**：

```cpp
#include <new>
constexpr size_t CACHE_LINE_SIZE = 64;  // 典型值

// 避免伪共享
struct alignas(CACHE_LINE_SIZE) PaddedCounter {
    std::atomic<int> value;
    char padding[CACHE_LINE_SIZE - sizeof(std::atomic<int>)];
};

// 多线程访问不同缓存行
PaddedCounter counters[4];  // 每个计数器独占缓存行
```

### 4.4 内存对齐

```cpp
struct Unaligned {
    char a;     // 1 字节
    int b;      // 4 字节
    char c;     // 1 字节
    double d;   // 8 字节
};
// sizeof(Unaligned) = 24（填充 14 字节）

struct Aligned {
    double d;   // 8 字节
    int b;      // 4 字节
    char a;     // 1 字节
    char c;     // 1 字节
};
// sizeof(Aligned) = 16（填充 2 字节）

// 手动对齐
struct alignas(16) AlignedStruct {
    int x, y, z, w;
};
static_assert(alignof(AlignedStruct) == 16);
```

**对齐规则**：
- 基本类型：通常对齐到自身大小
- 结构体：对齐到最大成员的对齐值
- 数组：元素连续，每个元素对齐

***

## 第五部分：内存安全与调试

### 5.1 常见内存错误

```cpp
// 1. 悬空指针
void dangling_pointer() {
    int* p = new int(42);
    delete p;
    std::cout << *p;  // 未定义行为！
}

// 2. 双重释放
void double_free() {
    int* p = new int(42);
    delete p;
    delete p;  // 未定义行为！
}

// 3. 内存泄漏
void memory_leak() {
    int* p = new int(42);
    p = new int(43);  // 第一个分配泄漏
}

// 4. 缓冲区溢出
void buffer_overflow() {
    int arr[10];
    arr[10] = 0;  // 越界写入
}

// 5. 使用未初始化变量
void uninitialized() {
    int x;
    if (x > 0) {  // 未定义行为
        // ...
    }
}

// 6. 返回局部变量地址
int* return_local() {
    int x = 42;
    return &x;  // 返回栈上地址
}
```

### 5.2 调试工具

**AddressSanitizer (ASan)**：

```bash
g++ -fsanitize=address -g test.cpp -o test
./test
```

```
输出示例：
==12345==ERROR: AddressSanitizer: heap-use-after-free
READ of size 4 at 0x602000000010 thread T0
    #0 0x400a12 in dangling_pointer() test.cpp:5
    #1 0x400a56 in main test.cpp:10
```

**Valgrind**：

```bash
valgrind --leak-check=full ./test
```

```
输出示例：
==12345== HEAP SUMMARY:
==12345==     in use at exit: 4 bytes in 1 blocks
==12345==   total heap usage: 2 allocs, 1 frees, 8 bytes allocated
==12345== 
==12345== 4 bytes in 1 blocks are definitely lost
==12345==    at 0x4C2E80F: operator new(unsigned long)
==12345==    by 0x400A12: memory_leak() (test.cpp:3)
```

**静态分析**：

```bash
# Clang Static Analyzer
scan-build g++ test.cpp

# Cppcheck
cppcheck --enable=all test.cpp
```

### 5.3 内存安全最佳实践

```cpp
// 1. 使用 RAII
void safe_code() {
    auto p = std::make_unique<int>(42);  // 自动释放
    std::vector<int> v(100);              // 自动释放
    std::string s = "hello";              // 自动释放
}

// 2. 避免裸指针所有权
void bad_ownership(int* p);  // 谁负责删除？
void good_ownership(std::unique_ptr<int> p);  // 明确所有权转移

// 3. 使用 span 避免指针+长度
void old_style(int* arr, size_t n);
void new_style(std::span<int> arr);  // C++20

// 4. 边界检查
void safe_access(std::vector<int>& v, size_t i) {
    if (i < v.size()) {
        v[i] = 0;  // 安全
    }
    // 或使用 at()
    try {
        v.at(i) = 0;  // 抛出异常
    } catch (const std::out_of_range&) {
        // 处理
    }
}

// 5. 初始化所有变量
int x = 0;                    // 直接初始化
int arr[10] = {};             // 零初始化
std::vector<int> v(10, 0);    // 显式初始化
```

***

## 第六部分：高级主题

### 6.1 自定义内存分配器

```cpp
template<typename T, size_t BlockSize = 4096>
class PoolAllocator {
    struct Block {
        Block* next;
        char data[BlockSize];
    };
    
    Block* current_block = nullptr;
    size_t offset = BlockSize;
    
public:
    using value_type = T;
    
    T* allocate(size_t n) {
        size_t bytes = n * sizeof(T);
        if (offset + bytes > BlockSize) {
            Block* new_block = static_cast<Block*>(::operator new(sizeof(Block)));
            new_block->next = current_block;
            current_block = new_block;
            offset = 0;
        }
        T* result = reinterpret_cast<T*>(current_block->data + offset);
        offset += bytes;
        return result;
    }
    
    void deallocate(T*, size_t) {
        // 池分配器通常不单独释放
    }
    
    ~PoolAllocator() {
        while (current_block) {
            Block* next = current_block->next;
            ::operator delete(current_block);
            current_block = next;
        }
    }
};

// 使用
std::vector<int, PoolAllocator<int>> v;
for (int i = 0; i < 10000; ++i) {
    v.push_back(i);  // 从池中分配
}
```

### 6.2 内存映射文件

```cpp
#ifdef _WIN32
#include <windows.h>
#else
#include <sys/mman.h>
#include <fcntl.h>
#include <unistd.h>
#endif

class MemoryMappedFile {
    void* data_ = nullptr;
    size_t size_ = 0;
    
#ifdef _WIN32
    HANDLE file_ = INVALID_HANDLE_VALUE;
    HANDLE mapping_ = nullptr;
#else
    int fd_ = -1;
#endif
    
public:
    MemoryMappedFile(const char* filename) {
#ifdef _WIN32
        file_ = CreateFileA(filename, GENERIC_READ, FILE_SHARE_READ,
                           nullptr, OPEN_EXISTING, 0, nullptr);
        if (file_ == INVALID_HANDLE_VALUE) throw std::runtime_error("Cannot open file");
        
        size_ = GetFileSize(file_, nullptr);
        mapping_ = CreateFileMappingA(file_, nullptr, PAGE_READONLY, 0, 0, nullptr);
        data_ = MapViewOfFile(mapping_, FILE_MAP_READ, 0, 0, 0);
#else
        fd_ = open(filename, O_RDONLY);
        if (fd_ == -1) throw std::runtime_error("Cannot open file");
        
        struct stat st;
        fstat(fd_, &st);
        size_ = st.st_size;
        data_ = mmap(nullptr, size_, PROT_READ, MAP_PRIVATE, fd_, 0);
#endif
    }
    
    ~MemoryMappedFile() {
#ifdef _WIN32
        if (data_) UnmapViewOfFile(data_);
        if (mapping_) CloseHandle(mapping_);
        if (file_ != INVALID_HANDLE_VALUE) CloseHandle(file_);
#else
        if (data_) munmap(data_, size_);
        if (fd_ != -1) close(fd_);
#endif
    }
    
    void* data() { return data_; }
    size_t size() { return size_; }
};
```

### 6.3 原子操作与内存序

```cpp
#include <atomic>

std::atomic<int> counter{0};

void memory_order_demo() {
    // relaxed：无顺序保证
    counter.fetch_add(1, std::memory_order_relaxed);
    
    // acquire：后续读写不能重排到此操作之前
    int x = counter.load(std::memory_order_acquire);
    
    // release：之前的读写不能重排到此操作之后
    counter.store(42, std::memory_order_release);
    
    // seq_cst（默认）：全局顺序一致
    counter.store(42);  // 等价于 memory_order_seq_cst
}

// 双重检查锁定
class Singleton {
    static std::atomic<Singleton*> instance;
    static std::mutex mtx;
    
public:
    static Singleton* get() {
        Singleton* p = instance.load(std::memory_order_acquire);
        if (!p) {
            std::lock_guard<std::mutex> lock(mtx);
            p = instance.load(std::memory_order_relaxed);
            if (!p) {
                p = new Singleton();
                instance.store(p, std::memory_order_release);
            }
        }
        return p;
    }
};
```

### 6.4 Placement New

```cpp
#include <new>

void placement_new_demo() {
    // 在预分配内存上构造对象
    alignas(std::string) char buffer[sizeof(std::string)];
    std::string* s = new (buffer) std::string("hello");
    
    std::cout << *s;  // 正常使用
    
    s->~std::string();  // 手动调用析构函数
    
    // 自定义分配器场景
    class Arena {
        char* ptr_;
    public:
        Arena(size_t size) : ptr_(static_cast<char*>(::operator new(size))) {}
        ~Arena() { ::operator delete(ptr_); }
        
        template<typename T, typename... Args>
        T* create(Args&&... args) {
            return new (ptr_) T(std::forward<Args>(args)...);
            ptr_ += sizeof(T);
        }
    };
}
```

***

## 第七部分：内存模型与优化

### 7.1 C++ 内存模型

```cpp
// 数据竞争：未定义行为
int x = 0;

void thread1() { x = 1; }
void thread2() { std::cout << x; }  // 数据竞争！

// 正确做法：使用原子操作或互斥量
std::atomic<int> x{0};
std::mutex mtx;

void safe_thread1() { x.store(1); }
void safe_thread2() { std::cout << x.load(); }

void safe_thread1_mutex() {
    std::lock_guard<std::mutex> lock(mtx);
    x = 1;
}
```

### 7.2 编译器优化影响

```cpp
// 优化可能改变行为
bool flag = false;

void thread1() {
    // ... 某些操作 ...
    flag = true;
}

void thread2() {
    while (!flag) {
        // 编译器可能优化为：
        // if (!flag) while (true) {}
    }
}

// 解决方案 1：volatile（不推荐）
volatile bool flag = false;  // 禁止某些优化

// 解决方案 2：atomic（推荐）
std::atomic<bool> flag{false};
```

### 7.3 缓存一致性协议

```
MESI 协议状态：
- Modified:  已修改，仅本缓存有
- Exclusive: 独占，与内存一致
- Shared:    共享，多个缓存有
- Invalid:   无效

缓存行状态转换：
CPU1 写入 X
    │
    ▼ 发送 invalidate 消息
其他 CPU 的 X 缓存行变为 Invalid
    │
    ▼ CPU1 独占 X（Modified）
```

***

## 附录：速查表

### 内存分配对比

| 特性 | 栈 | 堆 |
|------|----|----|
| 分配速度 | O(1) | O(n) 最坏 |
| 释放方式 | 自动 | 手动/RAII |
| 大小限制 | 固定（~1-8MB） | 虚拟内存限制 |
| 碎片化 | 无 | 可能 |
| 缓存友好 | 是 | 否 |

### 智能指针选择

| 场景 | 推荐 |
|------|------|
| 独占所有权 | `unique_ptr` |
| 共享所有权 | `shared_ptr` |
| 打破循环引用 | `weak_ptr` |
| 性能关键 | `unique_ptr` 或原始指针（无所有权） |

### 编译命令速查

```bash
# 预处理
g++ -E test.cpp -o test.ii

# 生成汇编
g++ -S test.cpp -o test.s

# 生成目标文件
g++ -c test.cpp -o test.o

# 链接
g++ test.o -o test

# 完整编译
g++ -O2 -std=c++20 test.cpp -o test

# 调试信息
g++ -g test.cpp -o test

# AddressSanitizer
g++ -fsanitize=address -g test.cpp -o test

# 查看符号
nm test.o

# 查看依赖
ldd test  # Linux
```
