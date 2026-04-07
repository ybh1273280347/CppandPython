# Python 高级特性：框架级工具

***

## 一、核心速查表

| 方法 | 作用 | 典型场景 |
|------|------|---------|
| `__new__` | 控制实例创建 | 单例、对象池、继承不可变类型 |
| `__init_subclass__` | 拦截子类定义 | 自动注册、参数验证 |
| `__slots__` | 内存优化 | 百万级实例 |
| 元类 | 拦截类创建 | ORM、框架基类 |

> **注意**：这些特性主要用于框架开发，日常开发很少用到

***

## 二、__new__ - 控制实例创建

### 与 __init__ 的区别

| 方法 | 作用 | 返回值 |
|------|------|--------|
| `__new__` | 创建实例 | 实例对象 |
| `__init__` | 初始化实例 | None |

**调用顺序**：`__new__` → `__init__`

### 场景1：单例模式

**价值**：全局唯一实例

```python
class Singleton:
    _instance = None
    
    def __new__(cls, *args, **kwargs):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

a = Singleton()
b = Singleton()
print(a is b)  # True
```

### 场景2：对象池

**价值**：复用对象，减少创建开销

```python
class Connection:
    _pool = []
    
    def __new__(cls):
        if cls._pool:
            return cls._pool.pop()
        return super().__new__(cls)
    
    def close(self):
        self.__class__._pool.append(self)

c1 = Connection()
c1.close()  # 放回池中
c2 = Connection()  # 从池中取出
```

### 场景3：继承不可变类型

**价值**：修改不可变类型的创建过程

```python
class UpperStr(str):
    def __new__(cls, value):
        return super().__new__(cls, value.upper())

s = UpperStr("hello")
print(s)  # HELLO
```

***

## 三、__init_subclass__ - 拦截子类定义

### 基本用法

**价值**：子类定义时自动执行代码

```python
class PluginBase:
    plugins = []
    
    def __init_subclass__(cls, **kwargs):
        super().__init_subclass__(**kwargs)
        PluginBase.plugins.append(cls)

class PluginA(PluginBase): pass
class PluginB(PluginBase): pass

print([p.__name__ for p in PluginBase.plugins])  # ['PluginA', 'PluginB']
```

### 场景：自动注册 + 参数验证

**价值**：框架自动收集子类，无需手动注册

```python
class Validator:
    validators = {}
    
    def __init_subclass__(cls, name=None, **kwargs):
        super().__init_subclass__(**kwargs)
        cls.name = name or cls.__name__
        Validator.validators[cls.name] = cls

class EmailValidator(Validator, name="email"): pass
class PhoneValidator(Validator, name="phone"): pass

print(Validator.validators.keys())  # dict_keys(['email', 'phone'])
```

***

## 四、__slots__ - 内存优化

### 基本用法

**价值**：百万级实例时节省内存

```python
import sys

class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

class PersonSlots:
    __slots__ = ('name', 'age')
    
    def __init__(self, name, age):
        self.name = name
        self.age = age

p1 = Person("Alice", 25)
p2 = PersonSlots("Alice", 25)

print(sys.getsizeof(p1.__dict__))  # ~72 字节
print(sys.getsizeof(p2))           # ~48 字节（无 __dict__）
```

### 限制

```python
class Person:
    __slots__ = ('name', 'age')

p = Person()
p.name = "Alice"
# p.email = "a@b.com"  # AttributeError: 无法添加新属性
```

### 继承注意事项

```python
class Parent:
    __slots__ = ('x',)

class Child(Parent):
    __slots__ = ('y',)  # 子类也要定义 __slots__

c = Child()
c.x = 1  # 父类的 slot
c.y = 2  # 子类的 slot
```

***

## 五、元类 - 拦截类创建

### 基本结构

```python
class MyMeta(type):
    def __new__(cls, name, bases, attrs):
        # 修改类属性
        attrs['created_by'] = 'MyMeta'
        return super().__new__(cls, name, bases, attrs)

class MyClass(metaclass=MyMeta):
    pass

print(MyClass.created_by)  # MyMeta
```

### 场景：单例元类

**价值**：任何类使用此元类自动变成单例

```python
class SingletonMeta(type):
    _instances = {}
    
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Database(metaclass=SingletonMeta):
    pass

a = Database()
b = Database()
print(a is b)  # True
```

### 场景：ORM 字段收集

**价值**：自动收集字段定义

```python
class Field:
    def __init__(self, type_):
        self.type = type_

class ModelMeta(type):
    def __new__(cls, name, bases, attrs):
        fields = {k: v for k, v in attrs.items() if isinstance(v, Field)}
        attrs['_fields'] = fields
        return super().__new__(cls, name, bases, attrs)

class Model(metaclass=ModelMeta):
    pass

class User(Model):
    name = Field(str)
    age = Field(int)

print(User._fields)  # {'name': Field(str), 'age': Field(int)}
```

***

## 六、选择指南

| 需求 | 工具 |
|------|------|
| 单例模式 | `__new__` 或元类 |
| 对象池 | `__new__` |
| 继承 str/int/tuple | `__new__` |
| 自动注册子类 | `__init_subclass__` |
| 百万级实例内存优化 | `__slots__` |
| ORM 框架 | 元类 |

***

## 七、记忆口诀

> **new 创建 init 初始化，单例池化都用它**
> **init_subclass 拦截子类，自动注册不用怕**
> **slots 省内存，百万实例首选它**
> **元类最强大，类创建时拦截它**
