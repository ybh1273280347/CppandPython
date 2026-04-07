# C++ STL 算法速查手册

***

## 一、algorithm 库

### 1. 排序算法

| 算法 | 作用 | 时间复杂度 | 示例 |
|------|------|-----------|------|
| `sort` | 排序（不稳定） | O(n log n) | `sort(v.begin(), v.end())` |
| `stable_sort` | 稳定排序 | O(n log n) | `stable_sort(v.begin(), v.end())` |
| `partial_sort` | 前k个有序 | O(n log k) | `partial_sort(v.begin(), v.begin()+3, v.end())` |
| `nth_element` | 第k大元素 | O(n) | `nth_element(v.begin(), v.begin()+2, v.end())` |
| `is_sorted` | 检查排序 | O(n) | `is_sorted(v.begin(), v.end())` |

```cpp
vector<int> v = {5, 2, 8, 1, 9, 3};

// 基本排序
sort(v.begin(), v.end());              // 升序
sort(v.begin(), v.end(), greater<int>());  // 降序

// 自定义比较
sort(v.begin(), v.end(), [](int a, int b) {
    return a % 10 < b % 10;  // 按个位数排序
});

// 稳定排序（相等元素保持原顺序）
struct Item { int value; int id; };
vector<Item> items = {{3, 1}, {1, 2}, {3, 3}};
stable_sort(items.begin(), items.end(), 
    [](const Item& a, const Item& b) { return a.value < b.value; });
// id=1 和 id=3 的 value 都是 3，保持原顺序

// 部分排序：只需要前 k 小
partial_sort(v.begin(), v.begin() + 3, v.end());
// 前 3 个是最小的且有序，后面无序

// 快速选择：找第 k 大（O(n)）
nth_element(v.begin(), v.begin() + 2, v.end());
// v[2] 是第 3 小的元素，左边都比它小，右边都比它大
```

**实际运用 - Top K 问题**：
```cpp
vector<int> scores = {95, 87, 92, 78, 88, 91, 85};

// 找前 3 高分
partial_sort(scores.begin(), scores.begin() + 3, scores.end(), greater<int>());
// scores[0..2] 是最高的 3 个分数

// 找中位数
nth_element(scores.begin(), scores.begin() + scores.size() / 2, scores.end());
int median = scores[scores.size() / 2];
```

### 2. 堆操作

| 算法 | 作用 | 时间复杂度 | 示例 |
|------|------|-----------|------|
| `make_heap` | 建堆 | O(n) | `make_heap(v.begin(), v.end())` |
| `push_heap` | 插入堆 | O(log n) | `push_heap(v.begin(), v.end())` |
| `pop_heap` | 弹出堆顶 | O(log n) | `pop_heap(v.begin(), v.end())` |
| `sort_heap` | 堆排序 | O(n log n) | `sort_heap(v.begin(), v.end())` |
| `is_heap` | 检查堆 | O(n) | `is_heap(v.begin(), v.end())` |

```cpp
vector<int> v = {3, 1, 4, 1, 5, 9};

// 建大顶堆
make_heap(v.begin(), v.end());
// v = {9, 5, 4, 1, 1, 3}，堆顶在 v[0]

// 插入元素
v.push_back(6);
push_heap(v.begin(), v.end());
// 6 上浮到合适位置

// 弹出堆顶
pop_heap(v.begin(), v.end());
int top = v.back();  // 获取最大值
v.pop_back();        // 删除

// 堆排序（排完后不再是堆）
sort_heap(v.begin(), v.end());
// v 变为升序
```

**建小顶堆**：
```cpp
vector<int> v = {3, 1, 4, 1, 5, 9};

// 建小顶堆
make_heap(v.begin(), v.end(), greater<int>());
// v = {1, 1, 4, 3, 5, 9}，堆顶是最小值

// 弹出最小值
pop_heap(v.begin(), v.end(), greater<int>());
int min_val = v.back();
v.pop_back();
```

**实际运用 - Top K 大元素**：
```cpp
vector<int> nums = {3, 1, 4, 1, 5, 9, 2, 6};
int k = 3;

// 方法1：小顶堆维护前 K 大
vector<int> minHeap(nums.begin(), nums.begin() + k);
make_heap(minHeap.begin(), minHeap.end(), greater<int>());

for (int i = k; i < nums.size(); i++) {
    if (nums[i] > minHeap[0]) {
        pop_heap(minHeap.begin(), minHeap.end(), greater<int>());
        minHeap.back() = nums[i];
        push_heap(minHeap.begin(), minHeap.end(), greater<int>());
    }
}
// minHeap 包含最大的 3 个元素

// 方法2：partial_sort（更简单）
partial_sort(nums.begin(), nums.begin() + k, nums.end(), greater<int>());
// nums[0..k-1] 是最大的 k 个
```

**实际运用 - 合并 K 个有序数组**：
```cpp
vector<vector<int>> arrays = {{1, 4, 7}, {2, 5, 8}, {3, 6, 9}};

// 用小顶堆合并
using Node = tuple<int, int, int>;  // (值, 数组索引, 元素索引)
vector<Node> heap;

for (int i = 0; i < arrays.size(); i++) {
    if (!arrays[i].empty()) {
        heap.emplace_back(arrays[i][0], i, 0);
    }
}
make_heap(heap.begin(), heap.end(), greater<Node>());

vector<int> result;
while (!heap.empty()) {
    pop_heap(heap.begin(), heap.end(), greater<Node>());
    auto [val, arr_idx, elem_idx] = heap.back();
    heap.pop_back();
    
    result.push_back(val);
    
    if (elem_idx + 1 < arrays[arr_idx].size()) {
        heap.emplace_back(arrays[arr_idx][elem_idx + 1], arr_idx, elem_idx + 1);
        push_heap(heap.begin(), heap.end(), greater<Node>());
    }
}
```

**priority_queue 封装**：
```cpp
// 大顶堆（默认）
priority_queue<int> maxHeap;
maxHeap.push(3);
maxHeap.push(1);
maxHeap.push(4);
int top = maxHeap.top();  // 4
maxHeap.pop();

// 小顶堆
priority_queue<int, vector<int>, greater<int>> minHeap;

// 自定义比较
struct Node { int val; int priority; };
auto cmp = [](const Node& a, const Node& b) { return a.priority < b.priority; };
priority_queue<Node, vector<Node>, decltype(cmp)> pq(cmp);
```

### 3. 查找算法

| 算法 | 作用 | 时间复杂度 | 示例 |
|------|------|-----------|------|
| `find` | 查找值 | O(n) | `find(v.begin(), v.end(), 42)` |
| `find_if` | 条件查找 | O(n) | `find_if(v.begin(), v.end(), pred)` |
| `binary_search` | 二分查找 | O(log n) | `binary_search(v.begin(), v.end(), 42)` |
| `lower_bound` | 第一个 >= | O(log n) | `lower_bound(v.begin(), v.end(), 42)` |
| `upper_bound` | 第一个 > | O(log n) | `upper_bound(v.begin(), v.end(), 42)` |
| `equal_range` | 等值范围 | O(log n) | `equal_range(v.begin(), v.end(), 42)` |

```cpp
vector<int> v = {1, 3, 5, 7, 9, 11};

// 线性查找
auto it = find(v.begin(), v.end(), 5);
if (it != v.end()) {
    cout << "找到: " << *it << endl;
}

// 条件查找
auto it2 = find_if(v.begin(), v.end(), [](int x) { return x > 6; });
// it2 指向 7

// 二分查找（必须有序）
bool found = binary_search(v.begin(), v.end(), 5);  // true

// lower_bound：第一个 >= x 的位置
auto lb = lower_bound(v.begin(), v.end(), 5);  // 指向 5
int pos = lb - v.begin();  // 2

// upper_bound：第一个 > x 的位置
auto ub = upper_bound(v.begin(), v.end(), 5);  // 指向 7

// equal_range：等于 x 的范围
auto [first, last] = equal_range(v.begin(), v.end(), 5);
int count = last - first;  // 等于 5 的元素个数
```

**实际运用 - 有序插入位置**：
```cpp
vector<int> sorted = {1, 3, 5, 7, 9};

// 找插入位置（保持有序）
int x = 6;
auto pos = lower_bound(sorted.begin(), sorted.end(), x);
sorted.insert(pos, x);  // {1, 3, 5, 6, 7, 9}

// 统计区间内元素个数
int count_in_range = lower_bound(sorted.begin(), sorted.end(), 8) 
                   - lower_bound(sorted.begin(), sorted.end(), 3);
// [3, 8) 范围内的元素个数
```

### 4. 修改算法

| 算法 | 作用 | 示例 |
|------|------|------|
| `copy` | 复制 | `copy(src.begin(), src.end(), dst.begin())` |
| `copy_if` | 条件复制 | `copy_if(src.begin(), src.end(), dst.begin(), pred)` |
| `transform` | 变换 | `transform(v.begin(), v.end(), out.begin(), f)` |
| `replace` | 替换值 | `replace(v.begin(), v.end(), old, new)` |
| `replace_if` | 条件替换 | `replace_if(v.begin(), v.end(), pred, new)` |
| `fill` | 填充 | `fill(v.begin(), v.end(), 42)` |
| `reverse` | 反转 | `reverse(v.begin(), v.end())` |
| `rotate` | 旋转 | `rotate(v.begin(), v.begin()+k, v.end())` |

```cpp
vector<int> src = {1, 2, 3, 4, 5};
vector<int> dst(5);

// 复制
copy(src.begin(), src.end(), dst.begin());

// 条件复制
vector<int> evens;
copy_if(src.begin(), src.end(), back_inserter(evens), 
        [](int x) { return x % 2 == 0; });
// evens = {2, 4}

// 变换
transform(src.begin(), src.end(), src.begin(), 
          [](int x) { return x * x; });  // 原地平方

// 替换
replace(src.begin(), src.end(), 4, 100);  // 把 4 替换成 100

// 反转
reverse(src.begin(), src.end());

// 旋转：把前 k 个元素移到末尾
vector<int> v = {1, 2, 3, 4, 5};
rotate(v.begin(), v.begin() + 2, v.end());
// v = {3, 4, 5, 1, 2}
```

**实际运用 - 字符串处理**：
```cpp
string s = "Hello World";

// 反转字符串
reverse(s.begin(), s.end());  // "dlroW olleH"

// 反转单词
reverse(s.begin(), s.end());
// 再逐个单词反转...

// 大小写转换
transform(s.begin(), s.end(), s.begin(), ::tolower);
```

### 5. 删除算法

| 算法 | 作用 | 示例 |
|------|------|------|
| `remove` | 移动元素 | `remove(v.begin(), v.end(), 5)` |
| `remove_if` | 条件移动 | `remove_if(v.begin(), v.end(), pred)` |
| `unique` | 去重移动 | `unique(v.begin(), v.end())` |

**删除组合拳**：
```cpp
vector<int> v = {1, 2, 3, 2, 4, 2, 5};

// 删除特定值
v.erase(remove(v.begin(), v.end(), 2), v.end());
// v = {1, 3, 4, 5}

// 条件删除
v.erase(remove_if(v.begin(), v.end(), 
        [](int x) { return x % 2 == 0; }), v.end());

// 去重
sort(v.begin(), v.end());
v.erase(unique(v.begin(), v.end()), v.end());
```

**实际运用 - 数据清洗**：
```cpp
vector<string> words = {"hello", "", "world", "", "cpp", "hello"};

// 删除空字符串
words.erase(remove(words.begin(), words.end(), ""), words.end());

// 去重
sort(words.begin(), words.end());
words.erase(unique(words.begin(), words.end()), words.end());
// words = {"cpp", "hello", "world"}
```

### 6. 条件算法

| 算法 | 作用 | 示例 |
|------|------|------|
| `all_of` | 全部满足 | `all_of(v.begin(), v.end(), pred)` |
| `any_of` | 任一满足 | `any_of(v.begin(), v.end(), pred)` |
| `none_of` | 全不满足 | `none_of(v.begin(), v.end(), pred)` |
| `count` | 统计值 | `count(v.begin(), v.end(), val)` |
| `count_if` | 条件统计 | `count_if(v.begin(), v.end(), pred)` |

```cpp
vector<int> v = {2, 4, 6, 8, 10};

// 检查是否全为偶数
bool all_even = all_of(v.begin(), v.end(), [](int x) { return x % 2 == 0; });
// true

// 检查是否有负数
bool has_neg = any_of(v.begin(), v.end(), [](int x) { return x < 0; });
// false

// 统计偶数个数
int cnt = count_if(v.begin(), v.end(), [](int x) { return x % 2 == 0; });
```

**实际运用 - 数据验证**：
```cpp
vector<int> ages = {25, 30, 18, 45, 22};

// 检查是否都是成年人
bool all_adult = all_of(ages.begin(), ages.end(), [](int a) { return a >= 18; });

// 检查是否有未成年人
bool has_minor = any_of(ages.begin(), ages.end(), [](int a) { return a < 18; });
```

### 7. 最值算法

| 算法 | 作用 | 示例 |
|------|------|------|
| `max` | 最大值 | `max(10, 20)` |
| `min` | 最小值 | `min(10, 20)` |
| `max_element` | 最大元素 | `max_element(v.begin(), v.end())` |
| `min_element` | 最小元素 | `min_element(v.begin(), v.end())` |
| `minmax_element` | 最小最大 | `minmax_element(v.begin(), v.end())` |

```cpp
vector<int> v = {5, 2, 8, 1, 9, 3};

// 最值
int m = max(10, 20);  // 20
int n = min(10, 20);  // 10

// 初始化列表
int m2 = max({1, 2, 3, 4, 5});  // 5

// 容器中最值
auto max_it = max_element(v.begin(), v.end());
cout << "最大值: " << *max_it << " 位置: " << max_it - v.begin() << endl;

// 同时获取最小最大
auto [min_it, max_it2] = minmax_element(v.begin(), v.end());
```

**实际运用 - 成绩统计**：
```cpp
struct Student { string name; int score; };
vector<Student> students = {{"Alice", 85}, {"Bob", 92}, {"Carol", 78}};

// 找最高分
auto top = max_element(students.begin(), students.end(),
    [](const Student& a, const Student& b) { return a.score < b.score; });
cout << top->name << ": " << top->score << endl;
```

***

## 二、numeric 库

| 算法 | 作用 | 示例 |
|------|------|------|
| `accumulate` | 求和 | `accumulate(v.begin(), v.end(), 0)` |
| `accumulate` | 累乘 | `accumulate(v.begin(), v.end(), 1, multiplies<int>())` |
| `inner_product` | 内积 | `inner_product(a.begin(), a.end(), b.begin(), 0)` |
| `iota` | 递增序列 | `iota(v.begin(), v.end(), 0)` |
| `partial_sum` | 前缀和 | `partial_sum(v.begin(), v.end(), out.begin())` |
| `adjacent_difference` | 相邻差 | `adjacent_difference(v.begin(), v.end(), out.begin())` |

```cpp
vector<int> v = {1, 2, 3, 4, 5};

// 求和
int sum = accumulate(v.begin(), v.end(), 0);  // 15

// 累乘
int product = accumulate(v.begin(), v.end(), 1, multiplies<int>());  // 120

// 字符串连接
vector<string> words = {"Hello", "World"};
string result = accumulate(words.begin() + 1, words.end(), words[0],
    [](const string& a, const string& b) { return a + " " + b; });
// "Hello World"

// 内积（点积）
vector<int> a = {1, 2, 3};
vector<int> b = {4, 5, 6};
int dot = inner_product(a.begin(), a.end(), b.begin(), 0);  // 1*4 + 2*5 + 3*6 = 32

// 填充递增序列
vector<int> idx(5);
iota(idx.begin(), idx.end(), 0);  // {0, 1, 2, 3, 4}

// 前缀和
vector<int> prefix(5);
partial_sum(v.begin(), v.end(), prefix.begin());  // {1, 3, 6, 10, 15}

// 相邻差
vector<int> diff(5);
adjacent_difference(v.begin(), v.end(), diff.begin());  // {1, 1, 1, 1, 1}
```

**实际运用 - 区间求和**：
```cpp
vector<int> nums = {1, 3, 5, 7, 9, 11};
vector<int> prefix(nums.size() + 1, 0);

// 构建前缀和数组
partial_sum(nums.begin(), nums.end(), prefix.begin() + 1);

// 查询区间 [l, r] 的和
auto range_sum = [&](int l, int r) {
    return prefix[r + 1] - prefix[l];
};
// range_sum(1, 3) = 3 + 5 + 7 = 15
```

***

## 三、ranges 库 (C++20)

### 基础对照

| 传统写法 | Ranges 写法 |
|---------|------------|
| `sort(v.begin(), v.end())` | `ranges::sort(v)` |
| `find(v.begin(), v.end(), x)` | `ranges::find(v, x)` |
| `count(v.begin(), v.end(), x)` | `ranges::count(v, x)` |
| `reverse(v.begin(), v.end())` | `ranges::reverse(v)` |
| `for_each(v.begin(), v.end(), f)` | `ranges::for_each(v, f)` |
| `copy(src.begin(), src.end(), dst.begin())` | `ranges::copy(src, dst.begin())` |

### 视图操作

| 视图 | 作用 | 示例 |
|------|------|------|
| `views::filter` | 过滤 | `v \| views::filter(pred)` |
| `views::transform` | 变换 | `v \| views::transform(f)` |
| `views::take` | 取前n个 | `v \| views::take(5)` |
| `views::drop` | 跳过前n个 | `v \| views::drop(5)` |
| `views::reverse` | 反转 | `v \| views::reverse` |
| `views::iota` | 无限序列 | `views::iota(0)` |
| `views::keys` | 取键 | `m \| views::keys` |
| `views::values` | 取值 | `m \| views::values` |
| `views::split` | 分割 | `s \| views::split(',')` |
| `views::join` | 连接 | `v \| views::join` |
| `views::unique` | 去重 | `v \| views::unique` |

```cpp
#include <ranges>

vector<int> v = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

// 基础视图
auto evens = v | views::filter([](int x) { return x % 2 == 0; });
// evens = {2, 4, 6, 8, 10}

auto squares = v | views::transform([](int x) { return x * x; });
// squares = {1, 4, 9, 16, 25, ...}

auto first5 = v | views::take(5);       // {1, 2, 3, 4, 5}
auto skip3 = v | views::drop(3);        // {4, 5, 6, 7, 8, 9, 10}
auto rev = v | views::reverse;          // {10, 9, 8, ...}

// 组合使用
auto result = v 
    | views::filter([](int x) { return x % 2 == 0; })  // 偶数
    | views::transform([](int x) { return x * x; })    // 平方
    | views::take(3);                                   // 前3个
// result = {4, 16, 36}

// 生成序列
auto seq = views::iota(1) | views::take(5);  // {1, 2, 3, 4, 5}
```

### 投影

```cpp
struct Person { string name; int age; };
vector<Person> people = {{"Alice", 30}, {"Bob", 25}, {"Carol", 35}};

// 传统写法
sort(people.begin(), people.end(),
     [](const Person& a, const Person& b) { return a.age < b.age; });

// Ranges 投影（更简洁）
ranges::sort(people, {}, &Person::age);  // 按年龄排序
ranges::sort(people, {}, &Person::name); // 按名字排序

// 查找
auto it = ranges::find(people, 30, &Person::age);  // 按年龄查找

// 统计
int cnt = ranges::count(people, 30, &Person::age);  // 年龄为30的人数
```

### 实际运用

```cpp
// 数据处理管道
struct Order { int id; double amount; string status; };
vector<Order> orders = {
    {1, 100.0, "completed"}, {2, 50.0, "pending"},
    {3, 200.0, "completed"}, {4, 75.0, "cancelled"}
};

// 获取已完成订单的金额，排序取前3
auto topAmounts = orders
    | views::filter([](const Order& o) { return o.status == "completed"; })
    | views::transform(&Order::amount)
    | views::take(3);

// 字符串分割
string text = "hello,world,cpp";
auto parts = text | views::split(',');
// parts = {"hello", "world", "cpp"}

// map 操作
map<string, int> scores = {{"Alice", 90}, {"Bob", 85}, {"Carol", 95}};
auto names = scores | views::keys;    // {"Alice", "Bob", "Carol"}
auto values = scores | views::values; // {90, 85, 95}

// 嵌套容器展平
vector<vector<int>> nested = {{1, 2}, {3, 4}, {5}};
auto flat = nested | views::join;  // {1, 2, 3, 4, 5}
```

***

## 四、utility 库

### swap / move / exchange

```cpp
// swap：交换两个值
int a = 10, b = 20;
swap(a, b);  // a = 20, b = 10

vector<int> v1 = {1, 2}, v2 = {3, 4};
swap(v1, v2);  // 交换整个容器，O(1)

// move：转移所有权
string s1 = "hello";
string s2 = move(s1);  // s2 = "hello", s1 变为空

vector<int> src = {1, 2, 3};
vector<int> dst = move(src);  // dst = {1, 2, 3}, src 变为空

// exchange：设置新值并返回旧值
int x = 10;
int old = exchange(x, 20);  // old = 10, x = 20

// 常用于移动语义
class Widget {
    string name_;
public:
    void setName(string n) {
        name_ = exchange(n, "");  // 移动 n 到 name_，n 变为空
    }
};
```

### pair / tuple

```cpp
// pair：键值对
pair<int, string> p1(1, "one");
pair<int, string> p2 = {2, "two"};
auto p3 = make_pair(3, "three");

// 访问
cout << p1.first << ": " << p1.second << endl;  // 1: one

// tuple：任意数量元素
tuple<int, string, double> t(1, "hello", 3.14);
cout << get<0>(t) << endl;  // 1
cout << get<string>(t) << endl;  // "hello"（按类型访问）

auto t2 = make_tuple(1, 2.0, "three");
```

### 结构化绑定 (C++17)

```cpp
// pair 解包
pair<int, string> p = {1, "hello"};
auto [id, name] = p;
cout << id << ": " << name << endl;  // 1: hello

// tuple 解包
tuple<int, double, string> t = {1, 3.14, "pi"};
auto [x, y, z] = t;

// map 遍历
map<string, int> m = {{"a", 1}, {"b", 2}};
for (auto [key, value] : m) {
    cout << key << ": " << value << endl;
}

// 数组解包
int arr[3] = {1, 2, 3};
auto [a, b, c] = arr;

// 结构体解包（聚合类型）
struct Point { int x, y; };
Point pt = {10, 20};
auto [px, py] = pt;
```

### 实际运用

```cpp
// 多返回值
pair<bool, int> findValue(const vector<int>& v, int target) {
    auto it = find(v.begin(), v.end(), target);
    if (it != v.end()) {
        return {true, it - v.begin()};
    }
    return {false, -1};
}

auto [found, pos] = findValue({1, 2, 3, 4, 5}, 3);
if (found) { cout << "Found at " << pos << endl; }

// 交换技巧：实现移动语义
template<typename T>
void swap_with_move(T& a, T& b) {
    T temp = move(a);
    a = move(b);
    b = move(temp);
}

// exchange 实现原子更新
class Counter {
    int value_ = 0;
public:
    int getAndSet(int newValue) {
        return exchange(value_, newValue);  // 返回旧值，设置新值
    }
    int increment() {
        return ++value_;
    }
};
```

***

## 五、常用组合拳

| 场景 | 代码 |
|------|------|
| 排序 | `sort(v.begin(), v.end())` |
| 删除值 | `v.erase(remove(v.begin(), v.end(), val), v.end())` |
| 去重 | `sort(v.begin(), v.end()); v.erase(unique(v.begin(), v.end()), v.end())` |
| 最值 | `auto [min_it, max_it] = minmax_element(v.begin(), v.end())` |
| 求和 | `accumulate(v.begin(), v.end(), 0)` |
| 查位置 | `distance(v.begin(), find(v.begin(), v.end(), val))` |
| 过滤 | `copy_if(src.begin(), src.end(), back_inserter(dst), pred)` |
| 变换 | `transform(src.begin(), src.end(), back_inserter(dst), f)` |
| 建堆 | `make_heap(v.begin(), v.end())` |
| 堆排序 | `make_heap(v.begin(), v.end()); sort_heap(v.begin(), v.end())` |
| Top K | `partial_sort(v.begin(), v.begin() + k, v.end(), greater<int>())` |

***

## 六、记忆口诀

> **sort 排序，find 查找，erase-remove 删元素**
> **make_heap 建堆，push_pop 插弹，sort_heap 堆排序**
> **count 计数，copy 复制，transform 来变换**
> **accumulate 求和，iota 填充序列**
> **minmax_element 找最值，all_of any_of 条件判断**
> **Ranges 直接传容器，投影管道更优雅**
