# C++ Python风格工具

***

## 核心速查

| Python | C++ | 说明 |
|--------|-----|------|
| `enumerate(seq)` | `views::enumerate` | 带索引遍历 |
| `zip(a, b)` | `views::zip` | 并行遍历 |
| `reversed(seq)` | `views::reverse` | 反向遍历 |
| `range(n)` | `views::iota` | 数字序列 |
| `map(f, seq)` | `views::transform` | 变换 |
| `filter(f, seq)` | `views::filter` | 过滤 |
| `any(seq)` | `ranges::any_of` | 任一为真 |
| `all(seq)` | `ranges::all_of` | 全部为真 |
| `sum(seq)` | `ranges::fold_left` | 求和 |
| `min/max(seq)` | `ranges::min/max_element` | 最值 |
| `sorted(seq)` | `ranges::sort` | 排序 |
| 列表推导 | `views` 链式 | 组合操作 |

***

## 一、enumerate（带索引遍历）

### C++23 原生

```cpp
#include <ranges>

vector<string> names = {"Alice", "Bob", "Charlie"};

for (auto&& [i, name] : names | views::enumerate) {
    cout << i << ": " << name << endl;
}
```

### C++17 替代

```cpp
for (size_t i = 0; i < names.size(); i++) {
    cout << i << ": " << names[i] << endl;
}
```

***

## 二、zip（并行遍历）

### C++23 原生

```cpp
vector<string> names = {"Alice", "Bob"};
vector<int> scores = {90, 85};

for (auto&& [name, score] : views::zip(names, scores)) {
    cout << name << ": " << score << endl;
}
```

### C++17 替代

```cpp
for (size_t i = 0; i < min(names.size(), scores.size()); i++) {
    cout << names[i] << ": " << scores[i] << endl;
}
```

***

## 三、reversed（反向遍历）

### C++20 原生

```cpp
vector<int> v = {1, 2, 3, 4, 5};

for (int x : v | views::reverse) {
    cout << x << " ";  // 5 4 3 2 1
}
```

### 传统方式

```cpp
for (auto it = v.rbegin(); it != v.rend(); ++it) {
    cout << *it << " ";
}
```

***

## 四、range（数字序列）

### C++20 原生

```cpp
// range(n) - 0 到 n-1
for (int i : views::iota(0, 5)) {
    cout << i << " ";  // 0 1 2 3 4
}

// range(start, end)
for (int i : views::iota(1, 6)) {
    cout << i << " ";  // 1 2 3 4 5
}

// range(start, end, step) - C++23
for (int i : views::iota(0, 10) | views::stride(2)) {
    cout << i << " ";  // 0 2 4 6 8
}
```

### 传统方式

```cpp
for (int i = 0; i < 5; i++) { }
for (int i = 1; i <= 5; i++) { }
for (int i = 0; i < 10; i += 2) { }
```

***

## 五、map/filter（变换与过滤）

### Python 风格

```python
# Python
list(map(lambda x: x*2, filter(lambda x: x>0, nums)))
```

### C++20 原生

```cpp
#include <ranges>

vector<int> nums = {-1, 2, -3, 4, 5};

// filter + transform
auto result = nums 
    | views::filter([](int x) { return x > 0; })
    | views::transform([](int x) { return x * 2; });

for (int x : result) {
    cout << x << " ";  // 4 8 10
}
```

### 传统方式

```cpp
vector<int> result;
for (int x : nums) {
    if (x > 0) result.push_back(x * 2);
}
```

***

## 六、any/all（条件判断）

### Python 风格

```python
# Python
any(x > 0 for x in nums)  # 任一为真
all(x > 0 for x in nums)  # 全部为真
```

### C++20 原生

```cpp
#include <ranges>

vector<int> nums = {1, 2, 3, -4, 5};

bool hasNegative = ranges::any_of(nums, [](int x) { return x < 0; });
bool allPositive = ranges::all_of(nums, [](int x) { return x > 0; });
bool noneZero = ranges::none_of(nums, [](int x) { return x == 0; });
```

### 传统方式

```cpp
bool hasNegative = any_of(nums.begin(), nums.end(), [](int x) { return x < 0; });
bool allPositive = all_of(nums.begin(), nums.end(), [](int x) { return x > 0; });
```

***

## 七、sum/min/max（聚合）

### Python 风格

```python
# Python
sum(nums)
min(nums)
max(nums)
```

### C++20 原生

```cpp
#include <ranges>
#include <numeric>

vector<int> nums = {3, 1, 4, 1, 5, 9};

// 求和
int total = ranges::fold_left(nums, 0, plus{});  // C++23
int total2 = accumulate(nums.begin(), nums.end(), 0);  // 传统

// 最值
int minVal = ranges::min(nums);      // 1
int maxVal = ranges::max(nums);      // 9
auto [minV, maxV] = ranges::minmax(nums);  // 同时获取
```

### 传统方式

```cpp
int total = accumulate(nums.begin(), nums.end(), 0);
int minVal = *min_element(nums.begin(), nums.end());
int maxVal = *max_element(nums.begin(), nums.end());
```

***

## 八、sorted（排序）

### Python 风格

```python
# Python
sorted(nums)                    # 升序
sorted(nums, reverse=True)      # 降序
sorted(nums, key=lambda x: abs(x))  # 按绝对值
```

### C++20 原生

```cpp
#include <ranges>

vector<int> nums = {3, 1, 4, 1, 5};

// 升序（原地）
ranges::sort(nums);

// 降序
ranges::sort(nums, greater{});

// 按条件
ranges::sort(nums, {}, [](int x) { return abs(x); });  // 按绝对值

// 不修改原数组
auto sorted = nums;
ranges::sort(sorted);
```

> 更多排序用法见 [Algorithm.md](Algorithm.md)

***

## 九、列表推导式风格

### Python 风格

```python
# Python
[x*x for x in nums if x > 0]           # 过滤+变换
[(i, x) for i, x in enumerate(nums)]   # 带索引
[x for x in nums if x not in exclude]  # 集合差
```

### C++20 原生

```cpp
#include <ranges>

vector<int> nums = {-1, 2, -3, 4, 5};

// 过滤+变换
vector<int> result;
ranges::copy(
    nums | views::filter([](int x) { return x > 0; })
         | views::transform([](int x) { return x * x; }),
    back_inserter(result)
);
// result = {4, 16, 25}

// 带索引（C++23）
for (auto&& [i, x] : nums | views::enumerate) {
    cout << i << ": " << x << endl;
}

// 集合差
set<int> exclude = {2, 4};
for (int x : nums | views::filter([&](int x) { return !exclude.count(x); })) {
    cout << x << " ";
}
```

***

## 十、字典操作（map）

### Python 风格

```python
# Python
d = {"a": 1, "b": 2}
d["c"] = 3
d.get("d", 0)      # 不存在返回默认值
"d" in d           # 是否存在
d.keys()           # 所有键
d.values()         # 所有值
d.items()          # 键值对
```

### C++ 对应

```cpp
#include <map>
#include <unordered_map>

map<string, int> d = {{"a", 1}, {"b", 2}};

// 插入
d["c"] = 3;
d.insert({"d", 4});
d.emplace("e", 5);

// 安全访问（不存在返回默认值）
int val = d.count("d") ? d["d"] : 0;
int val2 = d.contains("d") ? d["d"] : 0;  // C++20

// 是否存在
bool exists = d.count("d") > 0;
bool exists2 = d.contains("d");  // C++20

// 遍历键
for (auto& [key, _] : d) { cout << key << " "; }

// 遍历值
for (auto& [_, value] : d) { cout << value << " "; }

// 遍历键值对
for (auto& [key, value] : d) {
    cout << key << ": " << value << endl;
}
```

> 更多容器操作见 [STL-containers.md](STL-containers.md)

***

## 十一、集合操作（set）

### Python 风格

```python
# Python
s = {1, 2, 3}
s.add(4)
s.discard(2)       # 删除，不存在不报错
2 in s             # 是否存在
s1 & s2            # 交集
s1 | s2            # 并集
s1 - s2            # 差集
```

### C++ 对应

```cpp
#include <set>
#include <unordered_set>

set<int> s = {1, 2, 3};

// 插入
s.insert(4);

// 删除
s.erase(2);

// 是否存在
bool exists = s.count(2) > 0;
bool exists2 = s.contains(2);  // C++20

// 交集
set<int> s1 = {1, 2, 3}, s2 = {2, 3, 4};
vector<int> intersect;
ranges::set_intersection(s1, s2, back_inserter(intersect));
// intersect = {2, 3}

// 并集
vector<int> unite;
ranges::set_union(s1, s2, back_inserter(unite));
// unite = {1, 2, 3, 4}

// 差集
vector<int> diff;
ranges::set_difference(s1, s2, back_inserter(diff));
// diff = {1}
```

> 更多容器操作见 [STL-containers.md](STL-containers.md)

***

## 十二、列表操作（vector）

### Python 风格

```python
# Python
lst = [1, 2, 3]
lst.append(4)          # 追加
lst.extend([5, 6])     # 扩展
lst.insert(0, 0)       # 插入
lst.pop()              # 弹出末尾
lst.pop(0)             # 弹出指定位置
lst.remove(2)          # 删除值
2 in lst               # 是否存在
lst.count(2)           # 计数
lst.index(2)           # 索引
lst[::-1]              # 反转
lst[1:3]               # 切片
lst + [4, 5]           # 拼接
lst * 2                # 重复
```

### C++ 对应

```cpp
#include <vector>
#include <algorithm>

vector<int> lst = {1, 2, 3};

// 追加
lst.push_back(4);

// 扩展
lst.insert(lst.end(), {5, 6});

// 插入
lst.insert(lst.begin(), 0);

// 弹出末尾
lst.pop_back();

// 弹出指定位置
lst.erase(lst.begin() + 1);

// 删除值
lst.erase(remove(lst.begin(), lst.end(), 2), lst.end());

// 是否存在
bool exists = ranges::find(lst, 2) != lst.end();

// 计数
int cnt = ranges::count(lst, 2);

// 索引
auto it = ranges::find(lst, 2);
int idx = (it != lst.end()) ? distance(lst.begin(), it) : -1;

// 反转
ranges::reverse(lst);

// 切片（子数组）
vector<int> sub(lst.begin() + 1, lst.begin() + 3);

// 拼接
vector<int> other = {4, 5};
lst.insert(lst.end(), other.begin(), other.end());
```

> 更多容器操作见 [STL-containers.md](STL-containers.md)

***

## 十三、字符串操作

### Python 风格

```python
# Python
s = "hello world"
s.split()             # 按空格分割
s.split(",")          # 按逗号分割
",".join(lst)         # 连接
s.strip()             # 去首尾空白
s.lower() / s.upper() # 大小写
s.replace("a", "b")   # 替换
s.startswith("he")    # 前缀
s.endswith("ld")      # 后缀
"a" in s              # 包含
s.find("world")       # 查找
```

### C++ 对应

```cpp
#include <string>
#include <sstream>
#include <algorithm>

string s = "hello world";

// 分割
vector<string> split(const string& str, char delim = ' ') {
    vector<string> result;
    stringstream ss(str);
    string item;
    while (getline(ss, item, delim)) {
        if (!item.empty()) result.push_back(item);
    }
    return result;
}
auto parts = split(s);        // {"hello", "world"}
auto parts2 = split("a,b,c", ',');  // {"a", "b", "c"}

// 连接
string join(const vector<string>& parts, const string& delim) {
    string result;
    for (size_t i = 0; i < parts.size(); i++) {
        result += parts[i];
        if (i + 1 < parts.size()) result += delim;
    }
    return result;
}
auto joined = join({"a", "b", "c"}, ",");  // "a,b,c"

// 去首尾空白
auto strip = [](string s) {
    s.erase(s.begin(), find_if(s.begin(), s.end(), [](char c) { return !isspace(c); }));
    s.erase(find_if(s.rbegin(), s.rend(), [](char c) { return !isspace(c); }).base(), s.end());
    return s;
};

// 大小写
ranges::transform(s, s.begin(), ::tolower);
ranges::transform(s, s.begin(), ::toupper);

// 替换
ranges::replace(s, 'a', 'b');

// 前缀/后缀
bool starts = s.starts_with("he");   // C++20
bool ends = s.ends_with("ld");       // C++20

// 包含
bool contains = s.find("world") != string::npos;
bool contains2 = s.contains("world");  // C++23

// 查找
size_t pos = s.find("world");
```

***

## 十四、实用组合

### 链式调用

```cpp
vector<int> v = {1, 2, 3, 4, 5, 6};

// 过滤偶数 + 平方 + 取前3
for (int x : v 
        | views::filter([](int n) { return n % 2 == 0; })
        | views::transform([](int n) { return n * n; })
        | views::take(3)) {
    cout << x << " ";  // 4 16 36
}

// 反向 + 带索引（C++23）
for (auto&& [i, x] : v | views::reverse | views::enumerate) {
    cout << i << ": " << x << endl;
}
```

### 多容器操作

```cpp
// 并行遍历三个容器
vector<string> names = {"Alice", "Bob"};
vector<int> scores = {90, 85};
vector<char> grades = {'A', 'B'};

// C++23
for (auto&& [name, score, grade] : views::zip(names, scores, grades)) {
    cout << name << ": " << score << " (" << grade << ")" << endl;
}
```

***

## 十五、选择建议

| 场景 | 推荐 |
|------|------|
| C++23 环境 | 原生 `views::*` |
| C++20 环境 | `views::reverse`, `views::iota`, `ranges::*` |
| C++17 环境 | 传统算法 + 结构化绑定 |
| 性能敏感 | 传统迭代器/索引 |
| 代码简洁 | ranges 链式调用 |

***

## 十六、记忆口诀

> **enumerate 带索引，zip 并行遍历**
> **reverse 反向走，iota 数字列**
> **filter 来过滤，transform 变换**
> **any/all 判断，fold_left 求和**
> **ranges 链式调，Python 风格现**
