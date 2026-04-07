# Python operator & itertools 完全手册

---

## 一、核心速查表

| 工具 | 适用对象 | 访问方式 | 典型场景 |
|------|---------|----------|---------|
| `itemgetter` | 字典、列表、元组 | `obj[key]` | 从字典/序列中提取字段 |
| `attrgetter` | 对象、类实例 | `obj.attr` | 从对象中提取属性 |
| `methodcaller` | 任何对象 | `obj.method()` | 批量调用对象方法 |
| `groupby` | 可迭代对象 | 需配合排序 | 连续值分组聚合 |
| `reduce` | 可迭代对象 | 累积计算 | 集合运算、聚合 |

---

## 二、itemgetter - 字典和序列

### 基本用法

```python
from operator import itemgetter

# 字典
data = {'ip': '1.1.1.1', 'status': 200, 'size': 1234}
get_ip = itemgetter('ip')
print(get_ip(data))  # '1.1.1.1'

# 多个字段
get_info = itemgetter('ip', 'status')
print(get_info(data))  # ('1.1.1.1', 200)

# 列表/元组
nums = [1, 2, 3, 4, 5]
get_first = itemgetter(0)
print(get_first(nums))  # 1

# 多个索引
get_ends = itemgetter(0, -1)
print(get_ends(nums))  # (1, 5)

# 切片
get_slice = itemgetter(slice(1, 4))
print(get_slice(nums))  # (2, 3, 4)
```

### 实战：日志处理

```python
logs = [
    {'ip': '1.1.1.1', 'status': 200, 'size': 100},
    {'ip': '1.1.1.2', 'status': 404, 'size': 50},
    {'ip': '1.1.1.1', 'status': 200, 'size': 150},
]

# 提取所有 IP
ips = list(map(itemgetter('ip'), logs))

# 按状态码排序
logs_sorted = sorted(logs, key=itemgetter('status'))

# 多字段排序
logs_sorted = sorted(logs, key=itemgetter('status', 'size'))
```

---

## 三、attrgetter - 对象属性

### 基本用法

```python
from operator import attrgetter

class Log:
    def __init__(self, ip, status, size):
        self.ip = ip
        self.status = status
        self.size = size

logs = [
    Log('1.1.1.1', 200, 100),
    Log('1.1.1.2', 404, 50),
    Log('1.1.1.1', 200, 150),
]

# 单个属性
get_ip = attrgetter('ip')
print(get_ip(logs[0]))  # '1.1.1.1'

# 多个属性
get_info = attrgetter('ip', 'status')
print(get_info(logs[0]))  # ('1.1.1.1', 200)

# 批量提取
ips = list(map(attrgetter('ip'), logs))
```

### 嵌套属性

```python
class Request:
    def __init__(self, path, method):
        self.path = path
        self.method = method

class Log:
    def __init__(self, ip, request):
        self.ip = ip
        self.request = request

log = Log('1.1.1.1', Request('/api/user', 'GET'))

# 链式访问
get_method = attrgetter('request.method')
print(get_method(log))  # 'GET'

# 多层混合
get_info = attrgetter('ip', 'request.path', 'request.method')
print(get_info(log))  # ('1.1.1.1', '/api/user', 'GET')
```

---

## 四、itemgetter vs attrgetter

| 对比项 | itemgetter | attrgetter |
|--------|-----------|------------|
| 适用对象 | 字典、列表、元组 | 对象、类实例 |
| 访问方式 | `obj[key]` | `obj.attr` |
| 嵌套访问 | 不支持 | 支持点号链式 |
| 典型数据 | JSON、API响应 | ORM模型、自定义类 |

```python
# 记忆口诀
# itemgetter → 方括号 [] 访问
# attrgetter → 点号 . 访问

data = {'name': 'Alice'}
get_name = itemgetter('name')  # data['name']

class Person:
    name = 'Alice'
get_name = attrgetter('name')  # p.name
```

---

## 五、methodcaller - 批量调用方法

### 四种方式对比

```python
from operator import methodcaller

strings = ['hello', 'world', 'python']

# 方式1：列表推导（推荐）
result = [s.upper() for s in strings]

# 方式2：map + lambda
result = list(map(lambda s: s.upper(), strings))

# 方式3：map + methodcaller
result = list(map(methodcaller('upper'), strings))

# 方式4：直接用类方法（最快）
result = list(map(str.upper, strings))
```

### 选择指南

| 场景 | 推荐方式 |
|------|---------|
| 简单转换 | 直接调用 `s.upper()` |
| 列表处理 | 列表推导 `[s.upper() for s in list]` |
| 无参方法传给 map | 类方法 `map(str.upper, list)` |
| 有参方法传给 map | `methodcaller('method', arg1, arg2)` |
| 作为 sorted/groupby 的 key | `methodcaller('get_value')` |

### methodcaller 优势场景

```python
class Person:
    def __init__(self, name):
        self.name = name
    def greet(self, prefix, suffix='!'):
        return f"{prefix}{self.name}{suffix}"

persons = [Person('Alice'), Person('Bob')]

# 有参方法调用
greeter = methodcaller('greet', 'Hello ', '!!!')
result = list(map(greeter, persons))
# ['Hello Alice!!!', 'Hello Bob!!!']

# 关键字参数
greeter = methodcaller('greet', 'Hi ', suffix='?')
result = list(map(greeter, persons))
# ['Hi Alice?', 'Hi Bob?']
```

---

## 六、groupby - 连续分组

### ⚠️ 必须先排序

```python
from itertools import groupby
from operator import itemgetter

logs = [
    {'status': 200, 'ip': '1.1.1.1'},
    {'status': 404, 'ip': '1.1.1.2'},
    {'status': 200, 'ip': '1.1.1.3'},
    {'status': 200, 'ip': '1.1.1.4'},
]

# 先排序，再分组
logs.sort(key=itemgetter('status'))

for status, group in groupby(logs, key=itemgetter('status')):
    print(f"状态码 {status}: {len(list(group))} 条")
# 状态码 200: 3 条
# 状态码 404: 1 条
```

### 对象分组

```python
from operator import attrgetter

class Log:
    def __init__(self, status, ip):
        self.status = status
        self.ip = ip

logs = [Log(200, '1.1.1.1'), Log(404, '1.1.1.2'), Log(200, '1.1.1.3')]
logs.sort(key=attrgetter('status'))

for status, group in groupby(logs, key=attrgetter('status')):
    ips = [log.ip for log in group]
    print(f"状态码 {status}: {ips}")
```

---

## 七、reduce + operator 集合运算

### reduce 原理

```python
from functools import reduce
from operator import add

# reduce(func, [a, b, c, d]) = func(func(func(a, b), c), d)
reduce(add, [1, 2, 3, 4])  # (((1+2)+3)+4) = 10
```

### 集合运算

```python
from functools import reduce
from operator import and_, or_, xor, sub

sets = [{1, 2, 3}, {2, 3, 4}, {3, 4, 5}]

# 并集（所有元素）
union = reduce(or_, sets)           # {1, 2, 3, 4, 5}

# 交集（共同元素）
intersection = reduce(and_, sets)   # {3}

# 对称差（奇数次出现）
sym_diff = reduce(xor, sets)        # {1, 5}

# 差集（第一个有，其他没有）
diff = reduce(sub, sets)            # {1}
```

### 多列表并集

```python
from functools import reduce
from operator import or_

lists = [[1, 2], [2, 3], [3, 4, 5]]

# 拆解过程：
# {1,2} | {2,3} = {1,2,3}
# {1,2,3} | {3,4,5} = {1,2,3,4,5}

union = reduce(or_, map(set, lists))
print(union)  # {1, 2, 3, 4, 5}
```

### 字典合并

```python
from operator import or_

dicts = [{'a': 1}, {'b': 2}, {'c': 3}]
merged = reduce(or_, dicts)  # {'a': 1, 'b': 2, 'c': 3}

# 相同键保留最后一个
dicts = [{'a': 1}, {'a': 2}, {'b': 3}]
merged = reduce(or_, dicts)  # {'a': 2, 'b': 3}
```

### 数值聚合

```python
from operator import add, mul

nums = [1, 2, 3, 4, 5]

total = reduce(add, nums)       # 15
product = reduce(mul, nums)     # 120

# 带初始值
total = reduce(add, [], 0)      # 0
product = reduce(mul, [], 1)    # 1
```

---

## 八、组合拳实战

### 组合1：统计分组

```python
from operator import itemgetter
from itertools import groupby
from functools import reduce

logs = [
    {'ip': '1.1.1.1', 'size': 100},
    {'ip': '1.1.1.2', 'size': 50},
    {'ip': '1.1.1.1', 'size': 150},
]

# 按 IP 统计总流量
logs.sort(key=itemgetter('ip'))
result = {}
for ip, group in groupby(logs, key=itemgetter('ip')):
    total = sum(item['size'] for item in group)
    result[ip] = total

print(result)  # {'1.1.1.1': 250, '1.1.1.2': 50}
```

### 组合2：对象分组

```python
from operator import attrgetter
from itertools import groupby
from dataclasses import dataclass

@dataclass
class Log:
    ip: str
    status: int

logs = [Log('1.1.1.1', 200), Log('1.1.1.2', 404), Log('1.1.1.1', 500)]
logs.sort(key=attrgetter('ip'))

for ip, group in groupby(logs, key=attrgetter('ip')):
    statuses = [log.status for log in group]
    print(f"{ip}: {statuses}")
```

### 组合3：方法分组

```python
from operator import methodcaller
from itertools import groupby

class Log:
    def __init__(self, path):
        self.path = path
    def get_ext(self):
        return self.path.split('.')[-1]

logs = [Log('a.py'), Log('b.txt'), Log('c.py'), Log('d.txt')]
logs.sort(key=methodcaller('get_ext'))

for ext, group in groupby(logs, key=methodcaller('get_ext')):
    print(f"{ext}: {len(list(group))} files")
```

### 组合4：多数据源分析

```python
from functools import reduce
from operator import or_, and_
from collections import Counter

# 三个数据源的用户
s1 = {'user1', 'user2', 'user3'}
s2 = {'user2', 'user3', 'user4'}
s3 = {'user3', 'user4', 'user5'}
sources = [s1, s2, s3]

# 总用户（并集）
all_users = reduce(or_, sources)
print(all_users)  # {'user1', 'user2', 'user3', 'user4', 'user5'}

# 共同用户（交集）
common = reduce(and_, sources)
print(common)  # {'user3'}

# 出现至少两次的用户
cnt = Counter()
for s in sources:
    cnt.update(s)
result = {u for u, c in cnt.items() if c >= 2}
print(result)  # {'user2', 'user3', 'user4'}
```

---

## 九、常见错误

| 错误 | 原因 | 修正 |
|------|------|------|
| `groupby` 分组不完整 | 未排序 | 先 `sort` 再 `groupby` |
| `reduce(or_, [])` 报错 | 空序列无初始值 | `reduce(or_, [], set())` |
| `itemgetter` 用于对象 | 对象不支持 `[]` | 改用 `attrgetter` |
| `attrgetter` 用于字典 | 字典没有属性 | 改用 `itemgetter` |
| 无参方法用 `methodcaller` | 多此一举 | 直接用 `map(str.upper, list)` |

---

## 十、性能建议

```python
# 简单场景：用内置函数
sum([1, 2, 3])              # 而不是 reduce(add, ...)
[d['key'] for d in data]    # 而不是 map(itemgetter, data)

# 传给高阶函数时：用 operator
sorted(data, key=itemgetter('status'))
groupby(data, key=attrgetter('category'))

# 无参方法：直接用类方法
map(str.upper, strings)     # 比 methodcaller('upper') 快

# 有参方法：用 methodcaller
map(methodcaller('greet', 'Hello'), persons)

# 集合运算：用 reduce + operator
reduce(or_, map(set, lists))
```

---

## 十一、记忆口诀

> **字典列表 itemgetter，方括号里把值拿**
> **对象属性 attrgetter，点号后面跟属性名**
> **批量调用 methodcaller，有参传递最方便**
> **无参方法直接传类，str.upper 就够用**
> **连续分组用 groupby，排序之后才能用**
> **集合运算 reduce 强，or_ and_ 来帮忙**
