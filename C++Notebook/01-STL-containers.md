# C++ STL 容器速查手册

***

## 核心原则

> **默认用 `vector`，只有特定需求时才换其他容器。**

***

## 一、string - 字符串

```cpp
#include <string>
```

### 常用方法

| 方法 | 作用 | 示例 |
|------|------|------|
| `s[i]` | 访问字符 | `s[0]` |
| `s.at(i)` | 带检查访问 | `s.at(0)` |
| `s.size()` / `s.length()` | 长度 | `s.size()` |
| `s.empty()` | 是否为空 | `s.empty()` |
| `s.clear()` | 清空 | `s.clear()` |
| `s.push_back(c)` | 尾部添加字符 | `s.push_back('a')` |
| `s.pop_back()` | 尾部删除字符 | `s.pop_back()` |
| `s.append(str)` | 追加字符串 | `s.append("world")` |
| `s.insert(pos, str)` | 插入 | `s.insert(0, "hello")` |
| `s.erase(pos, len)` | 删除 | `s.erase(0, 5)` |
| `s.substr(pos, len)` | 子串 | `s.substr(0, 5)` |
| `s.find(str)` | 查找位置 | `s.find("abc")` |
| `s.replace(pos, len, str)` | 替换 | `s.replace(0, 5, "hi")` |

### 示例

```cpp
string s = "hello";
s += " world";          // "hello world"
s.push_back('!');       // "hello world!"
s.pop_back();           // "hello world"

string sub = s.substr(0, 5);  // "hello"
size_t pos = s.find("world"); // 6
if (pos != string::npos) { }  // 找到了

// 遍历
for (char c : s) { }
for (int i = 0; i < s.size(); i++) { }
```

### 常用操作

```cpp
// 数字转字符串
string s = to_string(123);

// 字符串转数字
int n = stoi("123");
double d = stod("3.14");

// 分割（无内置，需手动）
vector<string> split(const string& s, char delim) {
    vector<string> res;
    stringstream ss(s);
    string item;
    while (getline(ss, item, delim)) {
        res.push_back(item);
    }
    return res;
}
```

**典型场景**：文本处理、输入输出、路径操作

***

## 二、vector - 动态数组

```cpp
#include <vector>
```

### 常用方法

| 方法 | 作用 | 示例 |
|------|------|------|
| `push_back(x)` | 尾部添加 | `v.push_back(10)` |
| `pop_back()` | 尾部删除 | `v.pop_back()` |
| `v[i]` | 随机访问 | `v[0]` |
| `at(i)` | 带检查访问 | `v.at(0)` |
| `front()` | 首元素 | `v.front()` |
| `back()` | 尾元素 | `v.back()` |
| `size()` | 元素个数 | `v.size()` |
| `empty()` | 是否为空 | `v.empty()` |
| `clear()` | 清空 | `v.clear()` |
| `resize(n)` | 调整大小 | `v.resize(10)` |
| `reserve(n)` | 预分配空间 | `v.reserve(100)` |

### 示例

```cpp
vector<int> v = {1, 2, 3};
v.push_back(4);     // [1, 2, 3, 4]
v.pop_back();       // [1, 2, 3]
v[0] = 10;          // [10, 2, 3]

// 遍历
for (int x : v) { }
for (int i = 0; i < v.size(); i++) { }
```

**典型场景**：默认容器，大多数情况首选

***

## 三、deque - 双端队列

```cpp
#include <deque>
```

### 常用方法

| 方法 | 作用 | 示例 |
|------|------|------|
| `push_back(x)` | 尾部添加 | `dq.push_back(10)` |
| `push_front(x)` | 头部添加 | `dq.push_front(10)` |
| `pop_back()` | 尾部删除 | `dq.pop_back()` |
| `pop_front()` | 头部删除 | `dq.pop_front()` |
| `front()` | 队首 | `dq.front()` |
| `back()` | 队尾 | `dq.back()` |
| `v[i]` | 随机访问 | `dq[0]` |
| `size()` | 元素个数 | `dq.size()` |
| `empty()` | 是否为空 | `dq.empty()` |

### 示例

```cpp
deque<int> dq;
dq.push_back(1);    // [1]
dq.push_back(2);    // [1, 2]
dq.push_front(0);   // [0, 1, 2]

cout << dq.front(); // 0
cout << dq.back();  // 2
dq.pop_front();     // [1, 2]
dq.pop_back();      // [1]
```

**典型场景**：滑动窗口、单调队列

***

## 四、stack - 后进先出（LIFO）

```cpp
#include <stack>
```

### 常用方法

| 方法 | 作用 | 示例 |
|------|------|------|
| `push(x)` | 压栈 | `st.push(10)` |
| `pop()` | 弹栈顶 | `st.pop()` |
| `top()` | 栈顶 | `st.top()` |
| `empty()` | 是否为空 | `st.empty()` |
| `size()` | 元素个数 | `st.size()` |

### 示例

```cpp
stack<int> st;
st.push(1);
st.push(2);
st.push(3);

cout << st.top();   // 3
st.pop();           // 弹出 3
cout << st.top();   // 2
```

**典型场景**：括号匹配、DFS、撤销操作

***

## 五、queue - 先进先出（FIFO）

```cpp
#include <queue>
```

### 常用方法

| 方法 | 作用 | 示例 |
|------|------|------|
| `push(x)` | 入队 | `q.push(10)` |
| `pop()` | 出队 | `q.pop()` |
| `front()` | 队首 | `q.front()` |
| `back()` | 队尾 | `q.back()` |
| `empty()` | 是否为空 | `q.empty()` |
| `size()` | 元素个数 | `q.size()` |

### 示例

```cpp
queue<int> q;
q.push(1);
q.push(2);
q.push(3);

cout << q.front();  // 1
cout << q.back();   // 3
q.pop();            // 弹出 1
cout << q.front();  // 2
```

**典型场景**：BFS、消息队列、任务调度

***

## 六、priority_queue - 优先级队列

```cpp
#include <queue>
```

### 常用方法

| 方法 | 作用 | 示例 |
|------|------|------|
| `push(x)` | 插入元素 | `pq.push(10)` |
| `pop()` | 删除堆顶 | `pq.pop()` |
| `top()` | 堆顶 | `pq.top()` |
| `empty()` | 是否为空 | `pq.empty()` |
| `size()` | 元素个数 | `pq.size()` |

### 示例

```cpp
// 最大堆（默认）
priority_queue<int> pq;
pq.push(10);
pq.push(30);
pq.push(20);
cout << pq.top();   // 30（最大值）
pq.pop();

// 最小堆
priority_queue<int, vector<int>, greater<int>> min_pq;
min_pq.push(10);
min_pq.push(30);
min_pq.push(20);
cout << min_pq.top();   // 10（最小值）
```

**典型场景**：Dijkstra、Top K、任务调度

***

## 七、unordered_set - 快速查找（无序）

```cpp
#include <unordered_set>
```

### 常用方法

| 方法 | 作用 | 时间复杂度 | 示例 |
|------|------|-----------|------|
| `insert(x)` | 插入 | O(1) | `s.insert(10)` |
| `erase(x)` | 删除 | O(1) | `s.erase(10)` |
| `find(x)` | 查找 | O(1) | `s.find(10) != s.end()` |
| `contains(x)` | 是否包含 (C++20) | O(1) | `s.contains(10)` |
| `count(x)` | 统计个数 | O(1) | `s.count(10)` |
| `size()` | 元素个数 | O(1) | `s.size()` |
| `empty()` | 是否为空 | O(1) | `s.empty()` |
| `clear()` | 清空 | O(n) | `s.clear()` |

### 示例

```cpp
unordered_set<int> s;
s.insert(10);
s.insert(20);
s.insert(30);

if (s.contains(20)) { }         // C++20 推荐
if (s.find(20) != s.end()) { }  // C++17 前

s.erase(20);
```

**典型场景**：去重、快速成员检测

***

## 八、set - 有序且唯一

```cpp
#include <set>
```

### 常用方法

| 方法 | 作用 | 时间复杂度 | 示例 |
|------|------|-----------|------|
| `insert(x)` | 插入 | O(log n) | `s.insert(10)` |
| `erase(x)` | 删除 | O(log n) | `s.erase(10)` |
| `find(x)` | 查找 | O(log n) | `s.find(10) != s.end()` |
| `contains(x)` | 是否包含 (C++20) | O(log n) | `s.contains(10)` |
| `lower_bound(x)` | 第一个 >= x | O(log n) | `s.lower_bound(10)` |
| `upper_bound(x)` | 第一个 > x | O(log n) | `s.upper_bound(10)` |

### 示例

```cpp
set<int> s;
s.insert(30);
s.insert(10);
s.insert(20);
// 自动排序：10, 20, 30

for (int x : s) { }  // 有序遍历

auto it = s.lower_bound(15);  // 指向 20
```

**典型场景**：需要有序的唯一元素集合

***

## 九、完整对比表

| 容器 | 底层结构 | 特点 | 时间复杂度 | 典型场景 |
|------|---------|------|-----------|---------|
| **string** | 连续数组 | 字符操作 | O(1) 访问 | 文本处理 |
| **vector** | 连续数组 | 随机访问 | O(1) 访问 | 默认选择 |
| **deque** | 分段数组 | 双端操作 | O(1) 双端 | 滑动窗口 |
| **stack** | deque | LIFO | O(1) 操作 | DFS、括号匹配 |
| **queue** | deque | FIFO | O(1) 操作 | BFS、消息队列 |
| **priority_queue** | vector（堆） | 优先级 | O(log n) 插入/删除 | Dijkstra、Top K |
| **unordered_set** | 哈希表 | 无序唯一 | O(1) 查找 | 去重、成员检测 |
| **set** | 红黑树 | 有序唯一 | O(log n) 查找 | 有序集合 |

***

## 十、选择决策树

```
需要什么特性？
│
├─ 处理字符串 → string
│
├─ 后进先出（LIFO）→ stack
│
├─ 先进先出（FIFO）→ queue
│
├─ 按优先级取元素 → priority_queue
│
├─ 需要快速查找？
│   ├─ 需要有序？ → set / map
│   └─ 不需要有序？ → unordered_set / unordered_map
│
├─ 需要在两端操作？ → deque
│
└─ 其他情况 → vector（默认选择）
```

***

## 十一、记忆口诀

> **字符串用 string，默认容器用 vector**
> **LIFO 用 stack，FIFO 用 queue，优先级用 priority_queue**
> **查找用 unordered，有序用 set，双端用 deque**
