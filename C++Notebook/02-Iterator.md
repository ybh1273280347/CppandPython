# C++ 迭代器组合拳

***

## 思路模板

```
要做什么？
│
├─ 删除元素 → erase + remove/remove_if
│
├─ 去重 → sort + unique + erase
│
├─ 查找 → find / find_if / lower_bound
│
├─ 变换 → transform
│
├─ 过滤 → copy_if
│
├─ 统计 → count / count_if
│
├─ 最值 → max_element / min_element
│
└─ 读写文件 → istream_iterator / ostream_iterator
```

***

## 一、迭代器基础

### 迭代器类型

| 类型 | 支持操作 | 容器 |
|------|---------|------|
| 输入迭代器 | `++`, `*`, `==` | istream_iterator |
| 输出迭代器 | `++`, `*` | ostream_iterator |
| 前向迭代器 | `++`, `*`, `==` | forward_list |
| 双向迭代器 | `++`, `--`, `*`, `==` | list, set, map |
| 随机访问迭代器 | `++`, `--`, `+`, `-`, `[]`, `*`, `==` | vector, deque, array |

### 基本操作

```cpp
vector<int> v = {1, 2, 3, 4, 5};

// 获取迭代器
auto it = v.begin();     // 指向第一个元素
auto end = v.end();      // 指向末尾之后

// 访问元素
cout << *it << endl;     // 1（解引用）
cout << it[2] << endl;   // 3（随机访问，仅 vector/deque）

// 移动
++it;                    // 下一个元素
--it;                    // 上一个（双向迭代器）
it += 2;                 // 前进 2（随机访问）

// 比较
if (it != v.end()) { }   // 是否到达末尾
if (it < v.end()) { }    // 位置比较（随机访问）

// 距离
int dist = distance(v.begin(), it);  // 迭代器间距离
```

### 常用函数

```cpp
vector<int> v = {1, 2, 3, 4, 5};

// 首尾元素
auto first = v.begin();      // 第一个
auto last = v.end() - 1;     // 最后一个
auto last2 = prev(v.end());  // 最后一个（通用）

// 前进/后退
advance(it, 3);              // it 前进 3 步
auto it2 = next(it);         // 下一个
auto it3 = prev(it);         // 上一个
```

***

## 二、遍历方式

### 范围 for（推荐）

```cpp
vector<int> v = {1, 2, 3, 4, 5};

// 只读
for (int x : v) {
    cout << x << " ";
}

// 修改
for (int& x : v) {
    x *= 2;
}

// 避免拷贝
for (const auto& x : v) {
    cout << x << " ";
}
```

### 索引遍历

```cpp
// 需要索引时
for (size_t i = 0; i < v.size(); i++) {
    cout << i << ": " << v[i] << endl;
}

// C++20: 范围 for + 初始化
for (size_t i = 0; auto x : v) {
    cout << i++ << ": " << x << endl;
}
```

### 迭代器遍历

```cpp
// 需要迭代器操作时
for (auto it = v.begin(); it != v.end(); ++it) {
    cout << *it << " ";
    if (*it == 3) {
        v.erase(it);  // 删除元素
        break;        // 删除后必须跳出
    }
}
```

### 反向遍历

```cpp
// 反向迭代器
for (auto it = v.rbegin(); it != v.rend(); ++it) {
    cout << *it << " ";  // 5 4 3 2 1
}

// C++20: ranges
for (int x : v | views::reverse) {
    cout << x << " ";
}
```

### 遍历选择

| 场景 | 推荐方式 |
|------|---------|
| 只读遍历 | 范围 for + `const auto&` |
| 修改元素 | 范围 for + `auto&` |
| 需要索引 | 索引遍历 |
| 删除元素 | 迭代器遍历 |
| 反向遍历 | `rbegin()`/`rend()` |

***

## 核心速查

| 组合拳 | 代码 |
|--------|------|
| 删除特定值 | `v.erase(remove(v.begin(), v.end(), val), v.end())` |
| 条件删除 | `v.erase(remove_if(v.begin(), v.end(), pred), v.end())` |
| 去重 | `sort(v.begin(), v.end()); v.erase(unique(v.begin(), v.end()), v.end())` |
| 查找位置 | `distance(v.begin(), find(v.begin(), v.end(), val))` |
| 最值位置 | `distance(v.begin(), max_element(v.begin(), v.end()))` |
| 过滤复制 | `copy_if(src.begin(), src.end(), back_inserter(dst), pred)` |
| 变换复制 | `transform(src.begin(), src.end(), back_inserter(dst), f)` |
| 读文件 | `vector<int> v{istream_iterator<int>(fin), istream_iterator<int>()}` |
| 写文件 | `copy(v.begin(), v.end(), ostream_iterator<int>(fout, "\n"))` |

***

## 三、删除类组合拳

### 删除特定值

```cpp
vector<int> v = {1, 2, 3, 2, 4, 2, 5};

v.erase(remove(v.begin(), v.end(), 2), v.end());
// v = {1, 3, 4, 5}
```

**原理**：
```
remove 前：[1, 2, 3, 2, 4, 2, 5]
remove 后：[1, 3, 4, 5, ?, ?, ?]
                  ↑ 返回的迭代器
erase 后：[1, 3, 4, 5]
```

### 条件删除

```cpp
v.erase(remove_if(v.begin(), v.end(), 
        [](int x) { return x % 2 == 0; }), v.end());
```

> Lambda 语法详见 [Lambda.md](Lambda.md)

### 去重

```cpp
vector<int> v = {5, 2, 2, 3, 1, 4, 4, 5, 3};

sort(v.begin(), v.end());                      // 先排序
v.erase(unique(v.begin(), v.end()), v.end());  // 再去重
// v = {1, 2, 3, 4, 5}
```

***

## 四、查找类组合拳

### 查找元素位置

```cpp
vector<int> v = {10, 20, 30, 40, 50};

auto it = find(v.begin(), v.end(), 30);
if (it != v.end()) {
    int pos = distance(v.begin(), it);  // 2
}
```

### 最值及位置

```cpp
vector<int> v = {5, 2, 8, 1, 9, 3};

int max_val = *max_element(v.begin(), v.end());  // 9
int min_val = *min_element(v.begin(), v.end());  // 1

int max_pos = distance(v.begin(), max_element(v.begin(), v.end()));  // 4
```

### 二分查找（有序容器）

```cpp
vector<int> v = {1, 3, 5, 7, 9};

bool found = binary_search(v.begin(), v.end(), 5);  // true

auto lb = lower_bound(v.begin(), v.end(), 5);  // 第一个 >= 5
auto ub = upper_bound(v.begin(), v.end(), 5);  // 第一个 > 5
```

***

## 五、复制类组合拳

### 过滤复制

```cpp
vector<int> src = {1, 2, 3, 4, 5, 6};
vector<int> dst;

copy_if(src.begin(), src.end(), back_inserter(dst), 
        [](int x) { return x % 2 == 0; });
// dst = {2, 4, 6}
```

### 变换复制

```cpp
vector<int> src = {1, 2, 3};
vector<int> dst;

transform(src.begin(), src.end(), back_inserter(dst), 
          [](int x) { return x * x; });
// dst = {1, 4, 9}
```

### 原地变换

```cpp
vector<int> v = {1, 2, 3};

transform(v.begin(), v.end(), v.begin(), 
          [](int x) { return x * 2; });
// v = {2, 4, 6}
```

***

## 六、插入迭代器

### back_inserter

```cpp
vector<int> src = {1, 2, 3};
vector<int> dst;  // 无需预分配

copy(src.begin(), src.end(), back_inserter(dst));
// 自动调用 push_back
```

### 合并容器

```cpp
vector<int> a = {1, 2, 3};
vector<int> b = {4, 5, 6};
vector<int> result;

copy(a.begin(), a.end(), back_inserter(result));
copy(b.begin(), b.end(), back_inserter(result));
```

***

## 七、流迭代器

### 读文件

```cpp
ifstream fin("data.txt");
vector<int> v{istream_iterator<int>(fin), istream_iterator<int>()};
```

### 写文件

```cpp
ofstream fout("output.txt");
copy(v.begin(), v.end(), ostream_iterator<int>(fout, "\n"));
```

### 输出到屏幕

```cpp
copy(v.begin(), v.end(), ostream_iterator<int>(cout, " "));
```

### 处理 CSV

```cpp
string line = "10,20,30,40,50";
replace(line.begin(), line.end(), ',', ' ');

istringstream iss(line);
vector<int> v{istream_iterator<int>(iss), istream_iterator<int>()};
```

***

## 八、容器初始化

```cpp
vector<int> v1 = {1, 2, 3};                     // 列表初始化
vector<int> v2(10, 0);                          // 10 个 0
vector<int> v3(arr, arr + 3);                   // 从数组
vector<int> v4(v1.begin(), v1.end());           // 从迭代器
vector<int> v5{istream_iterator<int>(cin), istream_iterator<int>()};  // 从流
```

***

## 九、常用头文件

```cpp
#include <algorithm>   // sort, find, copy, remove, transform...
#include <numeric>     // accumulate, iota...
#include <iterator>    // back_inserter, istream_iterator...
#include <fstream>     // ifstream, ofstream
```

***

## 十、记忆口诀

> **删除元素用 remove，配合 erase 真正删**
> **去重要先排序后，unique 加 erase**
> **查找元素用 find，位置用 distance**
> **过滤用 copy_if，变换用 transform**
> **back_inserter 自动插，流迭代器读写文件**
