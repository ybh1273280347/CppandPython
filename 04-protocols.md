# Python 常用协议速查手册

***

## 一、核心速查表

| 协议 | 必须实现 | 自动获得 |
|------|---------|---------|
| 序列 | `__len__` + `__getitem__` | `len`、`[]`、切片、`for`、`in`、`reversed` |
| 字典 | `__missing__`（继承 dict） | 访问不存在的键时自动处理 |
| 上下文 | `__enter__` + `__exit__` | `with` 语句 |
| 迭代器 | `__iter__` + `__next__` | `for`、`next()` |
| 可调用 | `__call__` | `obj()` |
| 比较 | `__eq__` + `__lt__` | `==`、`!=`、`<`、`<=`、`>`、`>=` |
| 哈希 | `__hash__` + `__eq__` | 字典键、集合成员 |

> **提示**：上下文管理推荐使用 `contextlib` 工具，见第五章

***

## 二、序列协议

### 必须实现

```python
__len__(self)           # 返回长度
__getitem__(self, idx)  # 按索引取值
```

### 自动获得的功能

| 功能                     | 说明         |
| ---------------------- | ---------- |
| `len(obj)`             | 获取长度       |
| `obj[idx]`             | 索引访问（支持负数） |
| `obj[start:stop:step]` | 切片         |
| `for item in obj`      | 迭代         |
| `item in obj`          | 成员判断       |
| `reversed(obj)`        | 反向迭代       |

### 示例：自定义序列

```python
class Deck:
    def __init__(self):
        self.cards = ['A', 'K', 'Q', 'J', '10', '9', '8']
    
    def __len__(self):
        return len(self.cards)
    
    def __getitem__(self, idx):
        return self.cards[idx]

deck = Deck()

print(len(deck))        # 7
print(deck[0])          # A
print(deck[-1])         # 8
print(deck[1:4])        # ['K', 'Q', 'J']
for card in deck:       # 可迭代
    print(card)
print('A' in deck)      # True
print(list(reversed(deck)))  # ['8', '9', '10', 'J', 'Q', 'K', 'A']
```

### 可能的问题: 切片不保持类型
```python
# 解决方法: 重新调用构造函数
def __getitem__(self, key):
    if isinstance(key, slice):
        cls = type(self)
        # 重新调用构造函数
        return cls(self.cards[key])
    index = operator.index(key)
    return self.cards[index]

# slice.indices() 自动处理所有边界情况
def __getitem__(self, key):
    if isinstance(key, slice):
        # 一行搞定所有边界处理
        start, stop, step = key.indices(len(self))
        
        # 提取数据
        items = [self._data[i] for i in range(start, stop, step)]
        
        # 返回同类型实例
        return type(self)(items)
    
    # 普通索引
    import operator
    return self._data[operator.index(key)]
```

### 可变序列

```python
__setitem__(self, idx, value)  # obj[idx] = x
__delitem__(self, idx)         # del obj[idx]
```

***

## 三、字典协议 __missing__

### 作用

当访问不存在的键时自动调用

### 示例：自动创建默认值

**价值**：无需检查键是否存在

```python
class DefaultDict(dict):
    def __missing__(self, key):
        self[key] = []
        return self[key]

d = DefaultDict()
d['fruits'].append('apple')
d['fruits'].append('banana')
print(d)  # {'fruits': ['apple', 'banana']}
```

### 示例：计数器

```python
class Counter(dict):
    def __missing__(self, key):
        return 0

c = Counter()
c['a'] += 1
c['a'] += 1
c['b'] += 1
print(c)  # {'a': 2, 'b': 1}
```

***

## 四、上下文管理协议

### 必须实现

```python
__enter__(self)   # 进入 with 时调用，返回值绑定到 as 变量
__exit__(self, exc_type, exc_val, exc_tb)  # 退出 with 时调用
```

### `__exit__` 参数说明

| 参数 | 类型 | 含义 |
|------|------|------|
| `exc_type` | `type` 或 `None` | 异常类型，无异常时为 `None` |
| `exc_val` | `Exception` 或 `None` | 异常实例，无异常时为 `None` |
| `exc_tb` | `traceback` 或 `None` | 异常追踪信息，无异常时为 `None` |

**返回值**：`True` 抑制异常，`False` 或 `None` 正常传播异常

### 示例：计时器

```python
import time

class Timer:
    def __enter__(self):
        self.start = time.time()
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        self.end = time.time()
        print(f"耗时: {self.end - self.start:.4f}s")
        return False

with Timer():
    time.sleep(0.5)
# 输出: 耗时: 0.5001s
```

### 示例：异常处理

```python
class SafeDivision:
    def __enter__(self):
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        if exc_type is ZeroDivisionError:
            print(f"捕获异常: {exc_val}")
            return True  # 抑制异常
        return False

with SafeDivision():
    result = 1 / 0
print("程序继续执行")
```

***

## 五、contextlib 工具

### 核心速查表

| 工具 | 作用 | 典型场景 |
|------|------|---------|
| `@contextmanager` | 函数变上下文管理器 | 自定义资源管理 |
| `closing` | 自动调用 `close()` | 数据库连接、文件 |
| `suppress` | 忽略指定异常 | 静默处理异常 |
| `ExitStack` | 管理多个资源 | 动态数量资源 |
| `redirect_stdout` | 重定向标准输出 | 捕获打印内容 |
| `nullcontext` | 空上下文管理器 | 可选的上下文 |

### `@contextmanager` - 最常用

```python
from contextlib import contextmanager

@contextmanager
def timer(name):
    import time
    start = time.time()
    try:
        yield
    finally:
        print(f"{name}: {time.time() - start:.2f}s")

with timer("下载"):
    time.sleep(1)
# 下载: 1.00s
```

**模板**

```python
@contextmanager
def my_context():
    # 进入时做的事
    try:
        yield value  # 返回给 as 变量
    finally:
        # 退出时做的事（一定会执行）
```

### `closing` - 自动关闭

```python
from contextlib import closing
import sqlite3

with closing(sqlite3.connect('db.db')) as conn:
    conn.execute('SELECT 1')
# 自动调用 conn.close()
```

### `suppress` - 忽略异常

```python
from contextlib import suppress

with suppress(FileNotFoundError):
    os.remove('不存在的文件.txt')
# 文件不存在时静默跳过，不会报错
```

### `ExitStack` - 管理多个资源

```python
from contextlib import ExitStack

with ExitStack() as stack:
    files = [stack.enter_context(open(f)) for f in ['a.txt', 'b.txt', 'c.txt']]
    # 所有文件会在退出时自动关闭
```


### 使用场景

| 需求 | 工具 |
|------|------|
| 自定义进入/退出逻辑 | `@contextmanager` |
| 对象有 `close()` 但不支持 `with` | `closing` |
| 静默忽略某个异常 | `suppress` |
| 动态数量的资源管理 | `ExitStack` |

**90% 的情况用 `@contextmanager` 就够了**

***

## 六、迭代器协议

### 必须实现

```python
__iter__(self)   # 返回迭代器对象（通常返回 self）
__next__(self)   # 返回下一个值，结束时抛出 StopIteration
```

### 示例：计数器

```python
class Counter:
    def __init__(self, start, end):
        self.current = start
        self.end = end
    
    def __iter__(self):
        return self
    
    def __next__(self):
        if self.current >= self.end:
            raise StopIteration
        value = self.current
        self.current += 1
        return value

for i in Counter(1, 5):
    print(i)  # 1, 2, 3, 4

print(next(Counter(10, 13)))  # 10
```

### 示例：斐波那契迭代器

```python
class Fibonacci:
    def __init__(self, n):
        self.n = n
        self.a, self.b = 0, 1
        self.count = 0
    
    def __iter__(self):
        return self
    
    def __next__(self):
        if self.count >= self.n:
            raise StopIteration
        value = self.a
        self.a, self.b = self.b, self.a + self.b
        self.count += 1
        return value

for fib in Fibonacci(10):
    print(fib, end=' ')  # 0 1 1 2 3 5 8 13 21 34
```

***

## 七、可调用协议

### 必须实现

```python
__call__(self, *args, **kwargs)
```

### 示例：函数对象

```python
class Multiplier:
    def __init__(self, factor):
        self.factor = factor
    
    def __call__(self, x):
        return x * self.factor

double = Multiplier(2)
triple = Multiplier(3)

print(double(5))   # 10
print(triple(5))   # 15
print(callable(double))  # True
```

### 示例：计数器函数

```python
class Counter:
    def __init__(self):
        self.count = 0
    
    def __call__(self):
        self.count += 1
        return self.count

counter = Counter()
print(counter())  # 1
print(counter())  # 2
print(counter())  # 3
```

***

## 八、比较协议

### 必须实现

```python
__eq__(self, other)  # 相等判断
__lt__(self, other)  # 小于判断
```

### 配合 @total\_ordering

```python
from functools import total_ordering

@total_ordering
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age
    
    def __eq__(self, other):
        return self.age == other.age
    
    def __lt__(self, other):
        return self.age < other.age

p1 = Person("Alice", 25)
p2 = Person("Bob", 30)
p3 = Person("Charlie", 25)

print(p1 < p2)    # True
print(p1 >= p2)   # False（自动生成）
print(p1 == p3)   # True
print(p1 != p2)   # True（自动生成）

people = [p2, p1, p3]
print([p.name for p in sorted(people)])  # ['Alice', 'Charlie', 'Bob']
```

***

## 九、哈希协议

### 必须实现

```python
__hash__(self)   # 返回整数哈希值
__eq__(self, other)  # 相等判断
```

### 示例：可哈希的点

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def __eq__(self, other):
        return self.x == other.x and self.y == other.y
    
    def __hash__(self):
        return hash((self.x, self.y))

p1 = Point(1, 2)
p2 = Point(1, 2)
p3 = Point(3, 4)

# 可作为字典键
d = {p1: "A", p3: "B"}
print(d[p2])  # "A"（p1 和 p2 哈希相同且相等）

# 可作为集合元素
s = {p1, p2, p3}
print(len(s))  # 2（p1 和 p2 视为相同）
```

***

## 十、记忆口诀

> **序列：len + getitem，列表功能全都有**
> **字典：missing 缺键时，默认值自动有**
> **上下文：enter + exit，with 自动管资源**
> **contextmanager 装饰器，yield 分隔进与出**
> **迭代器：iter + next，for 循环遍历用**
> **可调用：call 一个，对象当函数用**
> **比较：eq + lt 两个，排序比较全都有**
> **哈希：hash + eq 配对，字典集合都能当键**

