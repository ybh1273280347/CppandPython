# Python 描述符完全手册

***

## 一、核心速查表

| 工具 | 作用 | 典型场景 |
|------|------|---------|
| 描述符 | 可复用的属性代理 | 类型检查、验证、ORM |
| `__get__` | 拦截属性读取 | 懒加载、计算属性 |
| `__set__` | 拦截属性赋值 | 验证、类型检查 |
| `__set_name__` | 获取属性名 | 自动存储属性名 |
| 特征工厂 | 批量生成 property | 多个相似属性 |

***

## 二、描述符协议

### 基本结构

```python
class Descriptor:
    def __set_name__(self, owner, name):
        self.name = name  # 自动获取属性名
    
    def __get__(self, instance, owner):
        return instance.__dict__.get(self.name)
    
    def __set__(self, instance, value):
        instance.__dict__[self.name] = value
```

### 参数说明

| 参数 | 含义 |
|------|------|
| `self` | 描述符实例本身 |
| `instance` | 使用描述符的对象实例（类访问时为 None） |
| `owner` | 使用描述符的类 |
| `value` | 要赋的值 |

### 描述符类型

| 类型 | 实现方法 | 优先级 | 示例 |
|------|---------|--------|------|
| 数据描述符 | `__get__` + `__set__` | 最高 | `property` |
| 非数据描述符 | 只有 `__get__` | 较低 | 方法、`classmethod` |

### 查找优先级

```
数据描述符 > 实例 __dict__ > 非数据描述符 > 类 __dict__ > __getattr__
```

***

## 三、为什么用描述符

### 问题：重复的 property

```python
class Person:
    @property
    def name(self):
        return self._name
    
    @name.setter
    def name(self, value):
        if not isinstance(value, str):
            raise TypeError("name 必须是字符串")
        self._name = value
    
    @property
    def age(self):
        return self._age
    
    @age.setter
    def age(self, value):
        if not isinstance(value, int):
            raise TypeError("age 必须是整数")
        self._age = value
```

### 解决：描述符复用逻辑

```python
class Typed:
    def __init__(self, type_):
        self.type = type_
    
    def __set_name__(self, owner, name):
        self.name = name
    
    def __get__(self, instance, owner):
        return instance.__dict__.get(self.name)
    
    def __set__(self, instance, value):
        if not isinstance(value, self.type):
            raise TypeError(f"{self.name} 必须是 {self.type.__name__}")
        instance.__dict__[self.name] = value

class Person:
    name = Typed(str)
    age = Typed(int)
    email = Typed(str)

p = Person()
p.name = "Alice"
p.age = 25
# p.age = "25"  # TypeError: age 必须是 int
```

***

## 四、典型场景

### 场景1：类型检查 `__set__`

**价值**：一处定义，多处复用

```python
class Typed:
    def __init__(self, type_):
        self.type = type_
    
    def __set_name__(self, owner, name):
        self.name = name
    
    def __get__(self, instance, owner):
        if instance is None:
            return self
        return instance.__dict__.get(self.name)
    
    def __set__(self, instance, value):
        if not isinstance(value, self.type):
            raise TypeError(f"{self.name} 必须是 {self.type.__name__}")
        instance.__dict__[self.name] = value

class User:
    name = Typed(str)
    age = Typed(int)

class Product:
    title = Typed(str)
    price = Typed(float)
```

### 场景2：范围验证 `__set__`

**价值**：统一验证规则

```python
class Range:
    def __init__(self, min_=None, max_=None):
        self.min = min_
        self.max = max_
    
    def __set_name__(self, owner, name):
        self.name = name
    
    def __get__(self, instance, owner):
        return instance.__dict__.get(self.name)
    
    def __set__(self, instance, value):
        if self.min is not None and value < self.min:
            raise ValueError(f"{self.name} 不能小于 {self.min}")
        if self.max is not None and value > self.max:
            raise ValueError(f"{self.name} 不能大于 {self.max}")
        instance.__dict__[self.name] = value

class Order:
    quantity = Range(min_=1, max_=100)
    discount = Range(min_=0, max_=1)

o = Order()
o.quantity = 10
# o.quantity = 0   # ValueError
# o.discount = 2   # ValueError
```

### 场景3：懒加载 `__get__`

**价值**：首次访问时才计算，自动缓存

```python
class Lazy:
    def __init__(self, func):
        self.func = func
    
    def __set_name__(self, owner, name):
        self.name = name
    
    def __get__(self, instance, owner):
        if instance is None:
            return self
        if self.name not in instance.__dict__:
            instance.__dict__[self.name] = self.func(instance)
        return instance.__dict__[self.name]

class Data:
    def __init__(self, items):
        self.items = items
    
    @Lazy
    def total(self):
        print("计算中...")
        return sum(self.items)

d = Data([1, 2, 3, 4, 5])
print(d.total)  # 计算中... 15
print(d.total)  # 15（缓存）
```

### 场景4：只读属性 `__set__`

**价值**：防止意外修改

```python
class ReadOnly:
    def __set_name__(self, owner, name):
        self.name = name
    
    def __get__(self, instance, owner):
        return instance.__dict__.get(self.name)
    
    def __set__(self, instance, value):
        if self.name in instance.__dict__:
            raise AttributeError(f"{self.name} 是只读属性")
        instance.__dict__[self.name] = value

class Config:
    version = ReadOnly()
    debug = ReadOnly()

c = Config()
c.version = "1.0"
# c.version = "2.0"  # AttributeError
```

### 场景5：ORM 字段映射

**价值**：数据库字段与 Python 属性映射

```python
class Column:
    def __init__(self, type_, primary=False):
        self.type = type_
        self.primary = primary
    
    def __set_name__(self, owner, name):
        self.name = name
    
    def __get__(self, instance, owner):
        if instance is None:
            return self
        return instance._data.get(self.name)
    
    def __set__(self, instance, value):
        if not isinstance(value, self.type):
            raise TypeError(f"{self.name} 类型错误")
        instance._data[self.name] = value

class Model:
    def __init__(self):
        self._data = {}

class User(Model):
    id = Column(int, primary=True)
    name = Column(str)
    age = Column(int)

u = User()
u.id = 1
u.name = "Alice"
u.age = 25
```

***

## 五、特征工厂

**价值**：批量生成相似的 property

```python
def field(key):
    def getter(self):
        return self._data.get(key)
    def setter(self, value):
        self._data[key] = value
    return property(getter, setter)

class User:
    def __init__(self):
        self._data = {}
    
    name = field('name')
    age = field('age')
    email = field('email')

u = User()
u.name = "Alice"
print(u.name)  # Alice
```

***

## 六、选择指南

| 需求 | 选择 |
|------|------|
| 单个属性验证 | `@property` |
| 多个相似属性（单类） | 特征工厂 |
| 多个类共享验证逻辑 | 描述符 |
| ORM/类型系统 | 描述符 |
| 懒加载 | `@cached_property` 或描述符 |

***

## 七、记忆口诀

> **描述符是代理器，属性访问它拦截**
> **get 读来 set 写，验证类型都能做**
> **set_name 获取名，自动存储不用愁**
> 
> **数据描述符优先高，实例属性压不了**
> **非数据描述符低，实例属性能覆盖**
> **函数也是描述符，自动绑定传 self**
