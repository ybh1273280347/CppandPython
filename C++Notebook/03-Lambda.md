# C++ Lambda 完全指南

***

## 核心速查

| 语法 | 含义 |
|------|------|
| `[]` | 不捕获 |
| `[=]` | 值捕获所有 |
| `[&]` | 引用捕获所有 |
| `[x]` | 值捕获 x |
| `[&x]` | 引用捕获 x |
| `[=, &x]` | 默认值，x 引用 |
| `[&, x]` | 默认引用，x 值 |
| `[this]` | 捕获 this |
| `mutable` | 允许修改捕获的副本 |

***

## 一、基本语法

```cpp
[capture](parameters) -> return_type { body }

// 省略形式
[] { return 42; }                    // 无参数无返回
[](int x) { return x * 2; }         // 有参数
[](int x) -> int { return x * 2; } // 显式返回类型
```

***

## 二、捕获列表

### 捕获方式

```cpp
int a = 1, b = 2;

// 值捕获（拷贝）
auto f1 = [=] { return a + b; };

// 引用捕获
auto f2 = [&] { return a + b; };

// 混合
auto f3 = [=, &a] { a++; return b; };  // a 引用，b 值
auto f4 = [&, a] { return a + b; };   // a 值，b 引用

// 单独捕获
auto f5 = [a] { return a; };
auto f6 = [&a] { return a; };
```

### 捕获 this

```cpp
class Widget {
    int value = 42;
public:
    void process() {
        auto f = [this] { return value; };
        auto f2 = [*this] { return value; };  // C++17 拷贝
    }
};
```

### 初始化捕获（C++14）

```cpp
// 移动捕获
auto p = make_unique<int>(42);
auto f = [ptr = move(p)] { return *ptr; };

// 计算后捕获
auto f2 = [sum = 10 + 20] { return sum; };
```

***

## 三、mutable

**值捕获默认是 const，用 mutable 允许修改**

```cpp
int x = 10;

auto f1 = [x]() mutable { return x++; };

cout << f1() << endl;  // 10
cout << f1() << endl;  // 11
// 外部 x 仍是 10
```

***

## 四、返回类型推导

```cpp
// 自动推导
auto f1 = [](int x) { return x * 2; };   // int
auto f2 = [](int x) { return x * 2.0; }; // double

// 显式指定
auto f3 = [](int x) -> double { return x * 2; };
```

***

## 五、泛型 Lambda（C++14）

```cpp
// 参数类型自动推导
auto add = [](auto a, auto b) { return a + b; };

add(3, 5);           // int + int
add(3.14, 2.71);     // double + double
add(string("a"), "b"); // string + const char*
```

### 模板 Lambda（C++20）

```cpp
// 显式模板参数
auto add = []<typename T>(T a, T b) { return a + b; };

// 类型约束
auto square = []<std::integral T>(T x) { return x * x; };
```

***

## 六、实际运用

### 算法谓词

```cpp
vector<int> v = {1, 2, 3, 4, 5};

auto it = find_if(v.begin(), v.end(), [](int x) { return x > 3; });
int cnt = count_if(v.begin(), v.end(), [](int x) { return x % 2 == 0; });
sort(v.begin(), v.end(), [](int a, int b) { return a > b; });
```

### 回调

```cpp
setTimeout(1000, [] { cout << "超时" << endl; });

// 带捕获
int id = 42;
setTimeout(1000, [id] { cout << "ID: " << id << endl; });
```

### 立即执行（IIFE）

```cpp
int result = [] {
    int x = 10, y = 20;
    return x + y;
}();  // 30
```

### 递归（C++14）

```cpp
function<int(int)> fact = [&](int n) {
    return n <= 1 ? 1 : n * fact(n - 1);
};
```

***

## 七、与 Iterator.md 的协作

Lambda 主要用于算法，参考 [Iterator.md](Iterator.md) 的组合拳：

| 场景 | Lambda 用法 |
|------|------------|
| 删除偶数 | `v.erase(remove_if(v.begin(), v.end(), [](int x){ return x%2==0; }), v.end())` |
| 过滤复制 | `copy_if(src.begin(), src.end(), back_inserter(dst), pred)` |
| 自定义排序 | `sort(v.begin(), v.end(), [](a,b){ return a > b; })` |

***

## 八、记忆口诀

> **方括号是捕获，值引用于**
> **mutable 去 const，auto 参数**
> **算法谓词最常用，捕获注意生命周期**

***

## 快速参考

| 写法 | 可修改捕获值 | 影响外部变量 |
|------|------------|------------|
| `[x]` | 否 | 否 |
| `[&x]` | 是 | 是 |
| `[=]` | 否 | 否 |
| `[&]` | 是 | 是 |
| `[=, &x]` | x 是 | x 是 |
| `[&, x]` | x 否 | x 否 |
