# Python 数据结构扩展

***

## 核心速查

| 数据结构 | 作用 | 典型场景 |
|---------|------|---------|
| `Counter` | 计数器 | 词频统计、投票计数 |
| `defaultdict` | 默认值字典 | 分组聚合、邻接表 |
| `deque` | 双端队列 | 滑动窗口、BFS |
| `heapq` | 堆 | Top K、优先队列 |
| `bisect` | 二分查找 | 有序插入、区间查询 |
| `OrderedDict` | 有序字典 | LRU 缓存 |
| `namedtuple` | 命名元组 | 轻量数据类 |
| `ChainMap` | 链式字典 | 配置层级 |

***

## 一、Counter - 计数器

```python
from collections import Counter
```

### 常用方法

| 方法 | 作用 | 示例 |
|------|------|------|
| `Counter(iterable)` | 创建计数器 | `Counter('aabbc')` |
| `most_common(n)` | 前 n 个最常见 | `c.most_common(2)` |
| `elements()` | 展开元素 | `list(c.elements())` |
| `update(iterable)` | 更新计数 | `c.update('abc')` |
| `subtract(iterable)` | 减少计数 | `c.subtract('a')` |

### 示例

```python
# 词频统计
text = "apple banana apple orange banana apple"
words = text.split()
counter = Counter(words)
# Counter({'apple': 3, 'banana': 2, 'orange': 1})

# 取前 2
counter.most_common(2)  # [('apple', 3), ('banana', 2)]

# 集合运算
c1 = Counter('aabbc')
c2 = Counter('abcc')
c1 + c2  # 加法
c1 - c2  # 减法（负数被忽略）
c1 & c2  # 交集（取最小）
c1 | c2  # 并集（取最大）
```

**典型场景**：词频统计、投票计数、字符分析

***

## 二、defaultdict - 默认值字典

```python
from collections import defaultdict
```

### 常用工厂函数

| 工厂函数 | 默认值 | 示例 |
|---------|-------|------|
| `list` | `[]` | `defaultdict(list)` |
| `set` | `set()` | `defaultdict(set)` |
| `int` | `0` | `defaultdict(int)` |
| `lambda: val` | 自定义 | `defaultdict(lambda: 'N/A')` |

### 示例

```python
# 分组聚合
students = [('A', 90), ('B', 85), ('A', 88), ('B', 92)]
groups = defaultdict(list)
for name, score in students:
    groups[name].append(score)
# {'A': [90, 88], 'B': [85, 92]}

# 计数（无需判断 key 是否存在）
counter = defaultdict(int)
for char in 'aabbc':
    counter[char] += 1
# {'a': 2, 'b': 2, 'c': 1}

# 邻接表
graph = defaultdict(list)
edges = [(1, 2), (1, 3), (2, 4)]
for u, v in edges:
    graph[u].append(v)
```

**典型场景**：分组聚合、邻接表、计数

***

## 三、deque - 双端队列

```python
from collections import deque
```

### 常用方法

| 方法 | 作用 | 时间复杂度 |
|------|------|-----------|
| `append(x)` | 右端添加 | O(1) |
| `appendleft(x)` | 左端添加 | O(1) |
| `pop()` | 右端删除 | O(1) |
| `popleft()` | 左端删除 | O(1) |
| `rotate(n)` | 旋转 | O(k) |
| `maxlen` | 最大长度 | - |

### 示例

```python
# 滑动窗口最大值
from collections import deque

def max_sliding_window(nums, k):
    dq = deque()
    result = []
    for i, num in enumerate(nums):
        while dq and nums[dq[-1]] < num:
            dq.pop()
        dq.append(i)
        if dq[0] <= i - k:
            dq.popleft()
        if i >= k - 1:
            result.append(nums[dq[0]])
    return result

# 固定长度队列（自动丢弃旧元素）
recent = deque(maxlen=5)
for i in range(10):
    recent.append(i)
# deque([5, 6, 7, 8, 9], maxlen=5)

# BFS
queue = deque([start])
while queue:
    node = queue.popleft()
    for neighbor in get_neighbors(node):
        queue.append(neighbor)
```

**典型场景**：滑动窗口、BFS、最近 N 条记录

***

## 四、heapq - 堆操作

```python
import heapq
```

### 常用方法

| 方法 | 作用 | 示例 |
|------|------|------|
| `heapify(list)` | 原地建堆 | `heapq.heapify(lst)` |
| `heappush(heap, x)` | 插入元素 | `heapq.heappush(h, 3)` |
| `heappop(heap)` | 弹出最小 | `heapq.heappop(h)` |
| `heappushpop(heap, x)` | 插入并弹出最小 | `heapq.heappushpop(h, 5)` |
| `heapreplace(heap, x)` | 弹出最小并插入 | `heapq.heapreplace(h, 5)` |
| `nlargest(n, iterable)` | 前 n 大 | `heapq.nlargest(3, lst)` |
| `nsmallest(n, iterable)` | 前 n 小 | `heapq.nsmallest(3, lst)` |

### 示例

```python
# 最小堆（默认）
h = [3, 1, 4, 1, 5]
heapq.heapify(h)
heapq.heappush(h, 2)
heapq.heappop(h)  # 1（最小值）

# 最大堆（取负）
h = [-x for x in [3, 1, 4, 1, 5]]
heapq.heapify(h)
-heapq.heappop(h)  # 5（最大值）

# Top K 问题
nums = [1, 5, 2, 8, 3, 9, 4]
heapq.nlargest(3, nums)   # [9, 8, 5]
heapq.nsmallest(3, nums)  # [1, 2, 3]

# 合并 K 个有序列表
lists = [[1, 4, 7], [2, 5, 8], [3, 6, 9]]
merged = list(heapq.merge(*lists))

# 堆元素为元组（按第一个元素排序）
h = []
heapq.heappush(h, (2, 'task2'))
heapq.heappush(h, (1, 'task1'))
heapq.heappop(h)  # (1, 'task1')
```

**典型场景**：Top K、优先队列、合并有序列表

***

## 五、bisect - 二分查找

```python
import bisect
```

### 常用方法

| 方法 | 作用 | 返回值 |
|------|------|-------|
| `bisect_left(a, x)` | 插入位置（左） | 第一个 >= x 的位置 |
| `bisect_right(a, x)` | 插入位置（右） | 第一个 > x 的位置 |
| `insort_left(a, x)` | 插入（左） | 无返回值 |
| `insort_right(a, x)` | 插入（右） | 无返回值 |

### 示例

```python
arr = [1, 2, 4, 4, 4, 6, 7]

# 查找插入位置
bisect.bisect_left(arr, 4)   # 2（第一个 4 的位置）
bisect.bisect_right(arr, 4)  # 5（最后一个 4 之后）

# 有序插入
bisect.insort(arr, 5)  # [1, 2, 4, 4, 4, 5, 6, 7]

# 查找元素是否存在
def binary_search(arr, x):
    i = bisect.bisect_left(arr, x)
    return i < len(arr) and arr[i] == x

# 查找区间内元素个数
def count_range(arr, lo, hi):
    return bisect.bisect_right(arr, hi) - bisect.bisect_left(arr, lo)

# 查找第一个 >= x 的元素
def find_ge(arr, x):
    i = bisect.bisect_left(arr, x)
    return arr[i] if i < len(arr) else None
```

**典型场景**：有序插入、区间查询、二分查找

***

## 六、OrderedDict - 有序字典

```python
from collections import OrderedDict
```

### 常用方法

| 方法 | 作用 |
|------|------|
| `move_to_end(key)` | 移到末尾 |
| `popitem(last=True)` | 弹出首/尾 |

### 示例

```python
# LRU 缓存
class LRUCache:
    def __init__(self, capacity):
        self.cache = OrderedDict()
        self.capacity = capacity
    
    def get(self, key):
        if key not in self.cache:
            return -1
        self.cache.move_to_end(key)
        return self.cache[key]
    
    def put(self, key, value):
        if key in self.cache:
            self.cache.move_to_end(key)
        self.cache[key] = value
        if len(self.cache) > self.capacity:
            self.cache.popitem(last=False)
```

**典型场景**：LRU 缓存、需要保持插入顺序

***

## 七、namedtuple - 命名元组

```python
from collections import namedtuple
```

### 示例

```python
# 创建命名元组
Point = namedtuple('Point', ['x', 'y'])
p = Point(1, 2)
p.x  # 1
p.y  # 2

# 从字典创建
Person = namedtuple('Person', ['name', 'age'])
d = {'name': 'Alice', 'age': 30}
p = Person(**d)

# 替换字段
p = p._replace(age=31)

# 转换为字典
p._asdict()  # {'name': 'Alice', 'age': 30}
```

**典型场景**：轻量数据类、函数返回多值

***

## 八、ChainMap - 链式字典

```python
from collections import ChainMap
```

### 示例

```python
# 配置层级
defaults = {'color': 'blue', 'size': 'M'}
user_config = {'color': 'red'}
config = ChainMap(user_config, defaults)

config['color']  # 'red'（优先 user_config）
config['size']   # 'M'（来自 defaults）

# 修改只影响第一个映射
config['size'] = 'L'  # 修改 user_config
```

**典型场景**：配置层级、作用域链

***

## 九、记忆口诀

> **Counter 计数，defaultdict 分组**
> **deque 双端，滑动窗口 BFS**
> **heapq 堆，Top K 优先队列**
> **bisect 二分，有序插入区间查**
> **OrderedDict 做 LRU，namedtuple 轻量类**
