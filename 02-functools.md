# Python functools 完全手册

---

## 一、核心速查表

| 工具 | 作用 | 一句话记忆 |
|------|------|-----------|
| `@wraps` | 保留函数元信息 | 写装饰器必加 |
| `partial` | 冻结函数参数 | 减少参数数量 |
| `@cache` | 无限缓存 | 递归神器 |
| `@lru_cache` | 有限缓存 | 可限制大小 |
| `@singledispatch` | 类型分发 | 一个函数多种实现 |
| `@total_ordering` | 补全比较 | 只定义 `__eq__` 和一个比较方法 |
| `@cached_property` | 缓存实例属性 | 类中只算一次 |
| `reduce` | 累积计算 | 集合运算、聚合 |

---

## 二、wraps - 保留函数元信息

```python
from functools import wraps

# 问题：装饰器会丢失函数名、文档
def bad_decorator(func):
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

@bad_decorator
def hello():
    """说你好"""
    pass

print(hello.__name__)  # wrapper（丢失）

# 解决：使用 @wraps
def good_decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

@good_decorator
def hello():
    """说你好"""
    pass

print(hello.__name__)  # hello ✅
print(hello.__doc__)   # 说你好 ✅
```

---

## 三、partial - 冻结函数参数

```python
from functools import partial

def power(base, exp):
    return base ** exp

# 冻结参数
square = partial(power, exp=2)
cube = partial(power, exp=3)

print(square(5))  # 25
print(cube(5))    # 125

# 冻结多个参数
def request(method, url, timeout=30):
    return f"{method} {url}"

api_get = partial(request, "GET", timeout=10)
print(api_get("https://api.example.com/users"))
```

---

## 四、cache / lru_cache - 缓存函数结果

```python
from functools import cache, lru_cache

# @cache - 无大小限制（Python 3.9+）
@cache
def fib(n):
    if n < 2:
        return n
    return fib(n-1) + fib(n-2)

print(fib(100))  # 瞬间返回

# @lru_cache - 有大小限制
@lru_cache(maxsize=128)
def compute(x, y):
    return x ** y

# 查看缓存统计
print(compute.cache_info())
# 清除缓存
compute.cache_clear()
```

**性能对比**

```python
# 无缓存：O(2^n)
def fib_slow(n):
    if n < 2:
        return n
    return fib_slow(n-1) + fib_slow(n-2)

# 有缓存：O(n)
@lru_cache(maxsize=None)
def fib_fast(n):
    if n < 2:
        return n
    return fib_fast(n-1) + fib_fast(n-2)
```

---

## 五、singledispatch - 类型分发

```python
from functools import singledispatch

@singledispatch
def process(data):
    raise TypeError(f"不支持的类型: {type(data)}")

@process.register(int)
def _(data):
    return f"整数: {data * 2}"

@process.register(str)
def _(data):
    return f"字符串: {data.upper()}"

@process.register(list)
@process.register(tuple)
def _(data):
    return f"序列长度: {len(data)}"

print(process(10))       # 整数: 20
print(process("hi"))     # 字符串: HI
print(process([1,2,3]))  # 序列长度: 3
```

---

## 六、total_ordering - 补全比较方法

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
    # __le__, __gt__, __ge__ 自动生成

p1 = Person("Alice", 25)
p2 = Person("Bob", 30)

print(p1 < p2)   # True
print(p1 >= p2)  # False（自动生成）
```

---

## 七、cached_property - 缓存实例属性

```python
from functools import cached_property

class DataAnalyzer:
    def __init__(self, data):
        self.data = data
    
    @cached_property
    def average(self):
        print("计算中...")
        return sum(self.data) / len(self.data)

analyzer = DataAnalyzer([1, 2, 3, 4, 5])

print(analyzer.average)  # 计算中... 3.0
print(analyzer.average)  # 3.0（直接返回）
```

---

## 八、reduce - 累积计算

```python
from functools import reduce
from operator import add, mul

nums = [1, 2, 3, 4, 5]

total = reduce(add, nums)      # 15
product = reduce(mul, nums)    # 120

# 带初始值
total = reduce(add, [], 0)     # 0
product = reduce(mul, [], 1)   # 1
```

---

## 九、记忆口诀

> **写装饰器用 wraps，元信息不丢失**
> **参数太多 partial，冻结参数变简单**
> **重复计算用 cache，递归速度飞起来**
> **类型分发 singledispatch，参数类型不同办**
> **比较方法 total_ordering，只需定义两个半**
> **实例属性 cached_property，只算一次省时间**
> **累积计算用 reduce，集合运算很方便**
