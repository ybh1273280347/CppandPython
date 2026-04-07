# Python 进阶笔记

Python 高级特性速查手册，按常用程度排序。

***

## 文件索引

| 文件 | 内容 | 常用程度 |
|------|------|---------|
| [01-decorators](01-decorators.md) | 装饰器 | ⭐⭐⭐⭐⭐ |
| [02-functools](02-functools.md) | functools 工具 | ⭐⭐⭐⭐⭐ |
| [03-collections](03-collections.md) | 数据结构扩展 | ⭐⭐⭐⭐⭐ |
| [04-protocols](04-protocols.md) | 常用协议 | ⭐⭐⭐⭐ |
| [05-generators](05-generators.md) | 生成器函数 | ⭐⭐⭐⭐ |
| [06-property](06-property.md) | property 属性 | ⭐⭐⭐⭐ |
| [07-concurrency](07-concurrency.md) | 并发编程 | ⭐⭐⭐ |
| [08-dynamic-attributes](08-dynamic-attributes.md) | 动态属性 | ⭐⭐⭐ |
| [09-itertools](09-itertools.md) | operator & itertools | ⭐⭐⭐ |
| [10-descriptors](10-descriptors.md) | 描述符 | ⭐⭐ |
| [11-advanced-features](11-advanced-features.md) | 高级特性 | ⭐ |

***

## 详细介绍

### 01-decorators.md - 装饰器

**内容**：装饰器模板、无参数/有参数装饰器、类装饰器、常见应用场景

**核心模板**：
```python
def decorator(params):      # 接收装饰器参数
    def wrapper(func):      # 接收被装饰函数
        def inner(*args):   # 接收调用参数
            return func(*args)
        return inner
    return wrapper
```

**价值**：横切关注点分离（日志、计时、权限）

**场景**：日志记录、性能计时、权限检查、缓存、重试

---

### 02-functools.md - functools 工具

**内容**：`@wraps`、`partial`、`@cache`/`@lru_cache`、`@singledispatch`、`@total_ordering`、`@cached_property`、`reduce`

**核心用法**：
```python
@lru_cache(maxsize=128)
def fib(n):
    return n if n < 2 else fib(n-1) + fib(n-2)

partial(int, base=2)  # 冻结参数
```

**价值**：减少重复计算，简化函数调用

**场景**：递归优化、回调函数简化、类型分发

---

### 03-collections.md - 数据结构扩展

**内容**：`Counter`、`defaultdict`、`deque`、`heapq`、`bisect`、`OrderedDict`、`namedtuple`、`ChainMap`

**核心用法**：
```python
Counter('aabbc')                    # 计数
defaultdict(list)                   # 分组聚合
deque(maxlen=5)                     # 滑动窗口
heapq.nlargest(3, nums)             # Top K
bisect.bisect_left(arr, x)          # 二分查找
```

**价值**：解决常见数据结构问题，代码更简洁

**场景**：词频统计、分组聚合、滑动窗口、Top K、优先队列、二分查找

---

### 04-protocols.md - 常用协议

**内容**：序列协议、字典协议、上下文管理、迭代器、可调用、比较、哈希协议

**核心协议**：
```python
# 序列协议：__len__ + __getitem__
# 上下文协议：__enter__ + __exit__
# 迭代器协议：__iter__ + __next__
```

**价值**：让自定义类支持 Python 内置操作

**场景**：自定义容器、资源管理、可迭代对象

---

### 05-generators.md - 生成器函数

**内容**：`chain`、`islice`、`zip_longest`、`product`、`combinations`、`permutations`、`groupby`、`cycle`、`count`、`accumulate`、`pairwise`

**核心用法**：
```python
chain(*iterables)              # 连接
product(*iterables)            # 笛卡尔积
combinations(iterable, r)      # 组合
groupby(iterable, key)         # 分组
```

**价值**：惰性计算，节省内存

**场景**：大数据处理、组合枚举、分组聚合

---

### 06-property.md - property 属性

**内容**：`@property` 完整用法、`@cached_property`、只读属性、验证属性

**核心用法**：
```python
@property
def age(self):
    return self._age

@age.setter
def age(self, value):
    if value < 0:
        raise ValueError("年龄不能为负")
    self._age = value
```

**价值**：属性访问控制，延迟计算

**场景**：数据验证、计算属性、懒加载

---

### 07-concurrency.md - 并发编程

**内容**：`asyncio`、`ThreadPoolExecutor`、`ProcessPoolExecutor`、混合场景

**核心用法**：
```python
# asyncio
async def main():
    tasks = [asyncio.create_task(fetch(url)) for url in urls]
    results = await asyncio.gather(*tasks)

# 线程池
with ThreadPoolExecutor(max_workers=10) as executor:
    results = list(executor.map(func, items))
```

**价值**：IO 密集型并发，CPU 密集型并行

**场景**：网络请求、数据库查询、批量计算

---

### 08-dynamic-attributes.md - 动态属性

**内容**：`getattr`、`setattr`、`hasattr`、`__getattr__`、`__setattr__`、`__dict__`

**核心用法**：
```python
getattr(obj, key, default)     # 运时获取
setattr(obj, key, value)       # 批量设置

def __getattr__(self, name):   # 属性不存在时兜底
    return self._data.get(name)
```

**价值**：运行时动态访问属性

**场景**：配置管理、API 响应解析、命令分发

---

### 09-itertools.md - operator & itertools

**内容**：`itemgetter`、`attrgetter`、`methodcaller`、`groupby`、`reduce`

**核心用法**：
```python
itemgetter('name')(dict_obj)   # 提取字段
attrgetter('name')(obj)        # 提取属性
methodcaller('strip')('  hi ') # 调用方法
```

**价值**：替代 lambda，更简洁

**场景**：排序、分组、数据提取

---

### 10-descriptors.md - 描述符

**内容**：描述符协议、`__get__`、`__set__`、`__set_name__`、特征工厂

**核心结构**：
```python
class Typed:
    def __set_name__(self, owner, name):
        self.name = name
    
    def __get__(self, instance, owner):
        return instance.__dict__.get(self.name)
    
    def __set__(self, instance, value):
        if not isinstance(value, self.type):
            raise TypeError()
        instance.__dict__[self.name] = value
```

**价值**：跨类复用属性逻辑

**场景**：ORM 字段、类型系统、数据验证

---

### 11-advanced-features.md - 高级特性

**内容**：`__new__`、`__init_subclass__`、`__slots__`、元类

**核心用法**：
```python
# 单例
def __new__(cls):
    if cls._instance is None:
        cls._instance = super().__new__(cls)
    return cls._instance

# 内存优化
class Point:
    __slots__ = ('x', 'y')
```

**价值**：控制类/实例创建，内存优化

**场景**：框架开发、ORM 实现、单例模式

***

## 快速查找

| 需求 | 查看文件 |
|------|---------|
| 写装饰器 | 01-decorators.md |
| 缓存函数结果 | 02-functools.md |
| 计数/分组/堆/二分 | 03-collections.md |
| 自定义容器 | 04-protocols.md |
| 处理大数据 | 05-generators.md |
| 属性验证 | 06-property.md |
| 并发请求 | 07-concurrency.md |
| 动态属性访问 | 08-dynamic-attributes.md |
| 排序/分组 | 09-itertools.md |
| 类型检查 | 10-descriptors.md |
| 写框架 | 11-advanced-features.md |
