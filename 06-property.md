# Python property 与 cached_property 完全手册

***

## 一、核心速查表

| 特性 | `@property` | `@cached_property` |
|------|-------------|-------------------|
| 计算次数 | 每次访问都计算 | 只计算一次 |
| 缓存位置 | 不缓存 | 存储在 `__dict__` |
| 可写性 | 可定义 setter | 只读 |
| 适用场景 | 验证、依赖属性 | 昂贵计算、API 调用 |
| 版本要求 | Python 2+ | Python 3.8+ |

***

## 二、@property 完整用法

### 基本结构

```python
class Person:
    def __init__(self, age):
        self._age = age
    
    @property
    def age(self):
        return self._age
    
    @age.setter
    def age(self, value):
        if value < 0:
            raise ValueError("年龄不能为负数")
        self._age = value
    
    @age.deleter
    def age(self):
        del self._age

p = Person(25)
print(p.age)    # 25
p.age = 30      # 触发 setter
del p.age       # 触发 deleter
```

### 只读属性

```python
class Circle:
    def __init__(self, radius):
        self.radius = radius
    
    @property
    def area(self):
        return 3.14 * self.radius ** 2
    
    @property
    def circumference(self):
        return 2 * 3.14 * self.radius

c = Circle(5)
print(c.area)         # 78.5
c.radius = 10
print(c.area)         # 314.0（自动更新）
# c.area = 100        # AttributeError: can't set attribute
```

### 数据验证

```python
class User:
    def __init__(self, email):
        self.email = email
    
    @property
    def email(self):
        return self._email
    
    @email.setter
    def email(self, value):
        if '@' not in value:
            raise ValueError("无效邮箱格式")
        self._email = value.lower()

u = User("Test@Example.COM")
print(u.email)        # test@example.com（自动转小写）
# u.email = "invalid" # ValueError
```

### 延迟初始化

```python
class Database:
    @property
    def connection(self):
        if not hasattr(self, '_connection'):
            print("建立连接...")
            self._connection = "DB Connection"
        return self._connection

db = Database()
print(db.connection)  # 建立连接... DB Connection
print(db.connection)  # DB Connection（复用）
```

***

## 三、@cached_property 用法

### 基本用法

```python
from functools import cached_property

class Analyzer:
    def __init__(self, data):
        self.data = data
    
    @cached_property
    def total(self):
        print("计算中...")
        return sum(self.data)

a = Analyzer([1, 2, 3, 4, 5])
print(a.total)  # 计算中... 15
print(a.total)  # 15（直接返回）
```

### 缓存原理

```python
class Demo:
    @cached_property
    def value(self):
        print("计算...")
        return 42

d = Demo()
print(d.__dict__)  # {}
print(d.value)     # 计算... 42
print(d.__dict__)  # {'value': 42}
```

### 清除缓存

```python
class Data:
    @cached_property
    def score(self):
        print("计算...")
        return 100

d = Data()
print(d.score)  # 计算... 100
del d.__dict__['score']
print(d.score)  # 计算... 100（重新计算）
```

***

## 四、选择指南

| 需求 | 选择 |
|------|------|
| 需要验证输入 | `@property` + setter |
| 依赖其他属性（会变） | `@property` |
| 只读属性 | `@property`（无 setter） |
| 昂贵计算（不变） | `@cached_property` |
| API 调用/数据库查询 | `@cached_property` |

***

## 五、实战案例

### 案例1：温度转换 `@property`

```python
class Temperature:
    def __init__(self, celsius):
        self.celsius = celsius
    
    @property
    def fahrenheit(self):
        return self.celsius * 9 / 5 + 32
    
    @property
    def kelvin(self):
        return self.celsius + 273.15

t = Temperature(25)
print(t.fahrenheit)  # 77.0
t.celsius = 0
print(t.fahrenheit)  # 32.0（自动更新）
```

### 案例2：价格计算 `@property`

```python
class Product:
    def __init__(self, price, tax_rate=0.1):
        self._price = price
        self._tax_rate = tax_rate
    
    @property
    def price(self):
        return self._price
    
    @price.setter
    def price(self, value):
        if value < 0:
            raise ValueError("价格不能为负")
        self._price = value
    
    @property
    def tax(self):
        return self._price * self._tax_rate
    
    @property
    def total(self):
        return self._price + self.tax

p = Product(100)
print(p.tax)     # 10.0
print(p.total)   # 110.0
p.price = 200
print(p.total)   # 220.0
```

### 案例3：API 数据缓存 `@cached_property`

```python
from functools import cached_property

class GitHubUser:
    def __init__(self, username):
        self.username = username
    
    @cached_property
    def profile(self):
        print(f"请求 API: {self.username}")
        return {"name": "Alice", "followers": 100}
    
    @cached_property
    def repos(self):
        print(f"请求仓库列表: {self.username}")
        return ["repo1", "repo2", "repo3"]

user = GitHubUser("alice")
print(user.profile)  # 请求 API: alice
print(user.profile)  # 直接返回
print(user.repos)    # 请求仓库列表: alice
```

### 案例4：配置解析 `@cached_property`

```python
from functools import cached_property

class Config:
    def __init__(self, filename):
        self.filename = filename
    
    @cached_property
    def data(self):
        print(f"加载配置: {self.filename}")
        return {"debug": True, "port": 8080}
    
    @property
    def debug(self):
        return self.data.get("debug", False)
    
    @property
    def port(self):
        return self.data.get("port", 8000)

config = Config("app.yaml")
print(config.debug)  # 加载配置: app.yaml True
print(config.port)   # 8080（不重新加载）
```

### 案例5：懒加载单例 `@cached_property`

```python
from functools import cached_property

class App:
    @cached_property
    def db(self):
        print("初始化数据库连接...")
        return DatabaseConnection()
    
    @cached_property
    def cache(self):
        print("初始化缓存...")
        return CacheSystem()

app = App()
app.db.query("SELECT 1")   # 初始化数据库连接...
app.db.query("SELECT 2")   # 直接使用
```

***

## 六、注意事项

### `@cached_property` 不能用于类方法

```python
class Wrong:
    @cached_property
    @classmethod
    def value(cls):
        return 42

# 报错：cached_property 不能与 classmethod 一起使用
```

### `@cached_property` 不支持继承覆盖

```python
class Base:
    @cached_property
    def value(self):
        return 1

class Child(Base):
    @cached_property
    def value(self):
        return 2

c = Child()
print(c.value)  # 2（正常工作）
# 但如果父类已缓存，子类不会重新计算
```

***

## 七、记忆口诀

> **property 三件套：getter setter deleter**
> **每次访问都计算，验证依赖都用它**
> 
> **cached 只算一次，缓存存在 __dict__**
> **昂贵计算 API 调用，省时省力效率高**
> 
> **只读属性用 property，无需 setter 即可**
> **需要验证加 setter，异常抛出保安全**
