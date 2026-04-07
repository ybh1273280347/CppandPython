# Python 枚举类型

***

## 核心速查

| 类 | 作用 |
|-----|------|
| `Enum` | 基础枚举 |
| `IntEnum` | 整数枚举（可比较） |
| `StrEnum` | 字符串枚举（Python 3.11+） |
| `Flag` | 位标志枚举 |
| `IntFlag` | 整数位标志 |

***

## 一、基础用法

```python
from enum import Enum

class Color(Enum):
    RED = 1
    GREEN = 2
    BLUE = 3

# 访问
Color.RED              # <Color.RED: 1>
Color.RED.name         # 'RED'
Color.RED.value        # 1

Color['RED']           # <Color.RED: 1>
Color(1)               # <Color.RED: 1>

# 遍历
for color in Color:
    print(color.name, color.value)

# 成员检查
Color.RED in Color     # True
Color.RED is Color.RED # True
```

### 1.1 自动赋值

```python
from enum import Enum, auto

class Color(Enum):
    RED = auto()    # 1
    GREEN = auto()  # 2
    BLUE = auto()   # 3

class Status(Enum):
    PENDING = auto()
    APPROVED = auto()
    REJECTED = auto()
```

### 1.2 自定义值

```python
class HttpStatus(Enum):
    OK = 200
    NOT_FOUND = 404
    SERVER_ERROR = 500

class Direction(Enum):
    NORTH = 'N'
    SOUTH = 'S'
    EAST = 'E'
    WEST = 'W'

class Priority(Enum):
    LOW = 1
    MEDIUM = 2
    HIGH = 3
    CRITICAL = 4
```

***

## 二、枚举类型

### 2.1 IntEnum

```python
from enum import IntEnum

class Priority(IntEnum):
    LOW = 1
    MEDIUM = 2
    HIGH = 3

# 可以与整数比较
Priority.HIGH > 2      # True
Priority.HIGH == 3     # True

# 可以用于算术运算
Priority.HIGH + 1      # 4
list(range(Priority.LOW, Priority.HIGH))  # [1, 2]
```

### 2.2 StrEnum（Python 3.11+）

```python
from enum import StrEnum

class Color(StrEnum):
    RED = 'red'
    GREEN = 'green'
    BLUE = 'blue'

# 自动是字符串
Color.RED.upper()      # 'RED'
f"Color: {Color.RED}"  # 'Color: red'

# JSON 序列化友好
import json
json.dumps({'color': Color.RED})  # '{"color": "red"}'
```

### 2.3 Flag 和 IntFlag

```python
from enum import Flag, IntFlag, auto

# Flag - 位标志
class Permission(Flag):
    READ = auto()     # 1
    WRITE = auto()    # 2
    EXECUTE = auto()  # 4

# 组合权限
READ_WRITE = Permission.READ | Permission.WRITE
READ_WRITE.value  # 3

# 检查权限
Permission.READ in READ_WRITE  # True
READ_WRITE & Permission.READ   # Permission.READ

# IntFlag - 整数位标志（可比较）
class FileMode(IntFlag):
    READ = 4
    WRITE = 2
    EXECUTE = 1

RW = FileMode.READ | FileMode.WRITE
RW.value  # 6
RW == 6   # True
```

***

## 三、高级用法

### 3.1 方法

```python
from enum import Enum

class Planet(Enum):
    MERCURY = (3.303e+23, 2.4397e6)
    VENUS = (4.869e+24, 6.0518e6)
    EARTH = (5.976e+24, 6.37814e6)
    
    def __init__(self, mass, radius):
        self.mass = mass
        self.radius = radius
    
    @property
    def surface_gravity(self):
        G = 6.67300E-11
        return G * self.mass / (self.radius ** 2)

Planet.EARTH.surface_gravity  # 9.8026...
Planet.EARTH.mass             # 5.976e+24
```

### 3.2 类方法

```python
from enum import Enum

class Color(Enum):
    RED = 1
    GREEN = 2
    BLUE = 3
    
    @classmethod
    def from_name(cls, name):
        return cls[name.upper()]
    
    @classmethod
    def values(cls):
        return [c.value for c in cls]
    
    @classmethod
    def names(cls):
        return [c.name for c in cls]

Color.from_name('red')  # Color.RED
Color.values()          # [1, 2, 3]
Color.names()           # ['RED', 'GREEN', 'BLUE']
```

### 3.3 别名

```python
from enum import Enum

class Color(Enum):
    RED = 1
    CRIMSON = 1  # RED 的别名
    GREEN = 2
    BLUE = 3

Color.CRIMSON          # <Color.RED: 1>
Color.CRIMSON is Color.RED  # True

# 获取所有唯一值
list(Color)  # [<Color.RED: 1>, <Color.GREEN: 2>, <Color.BLUE: 3>]

# 禁止别名
from enum import Enum, unique

@unique
class Status(Enum):
    PENDING = 1
    APPROVED = 2
    # APPROVED = 3  # 报错：重复值
```

### 3.4 字典映射

```python
from enum import Enum

class Role(Enum):
    ADMIN = 'admin'
    USER = 'user'
    GUEST = 'guest'

# 枚举到权限的映射
ROLE_PERMISSIONS = {
    Role.ADMIN: ['read', 'write', 'delete'],
    Role.USER: ['read', 'write'],
    Role.GUEST: ['read'],
}

def get_permissions(role: Role) -> list:
    return ROLE_PERMISSIONS.get(role, [])
```

***

## 四、常用模板

### 4.1 状态机

```python
from enum import Enum, auto

class OrderStatus(Enum):
    PENDING = auto()
    PAID = auto()
    SHIPPED = auto()
    DELIVERED = auto()
    CANCELLED = auto()
    
    def can_transition_to(self, next_status):
        transitions = {
            OrderStatus.PENDING: {OrderStatus.PAID, OrderStatus.CANCELLED},
            OrderStatus.PAID: {OrderStatus.SHIPPED, OrderStatus.CANCELLED},
            OrderStatus.SHIPPED: {OrderStatus.DELIVERED},
            OrderStatus.DELIVERED: set(),
            OrderStatus.CANCELLED: set(),
        }
        return next_status in transitions.get(self, set())

OrderStatus.PENDING.can_transition_to(OrderStatus.PAID)      # True
OrderStatus.PENDING.can_transition_to(OrderStatus.SHIPPED)   # False
```

### 4.2 配置选项

```python
from enum import Enum

class LogLevel(Enum):
    DEBUG = 'debug'
    INFO = 'info'
    WARNING = 'warning'
    ERROR = 'error'
    
    @classmethod
    def from_string(cls, value):
        try:
            return cls(value.lower())
        except ValueError:
            return cls.INFO

class Config:
    def __init__(self, log_level: LogLevel = LogLevel.INFO):
        self.log_level = log_level

config = Config(LogLevel.DEBUG)
config = Config(LogLevel.from_string('warning'))
```

### 4.3 类型安全的常量

```python
from enum import Enum

class Environment(Enum):
    DEVELOPMENT = 'dev'
    STAGING = 'staging'
    PRODUCTION = 'prod'
    
    @property
    def is_production(self):
        return self == Environment.PRODUCTION
    
    @property
    def database_url(self):
        urls = {
            Environment.DEVELOPMENT: 'localhost:5432',
            Environment.STAGING: 'staging.db.example.com',
            Environment.PRODUCTION: 'prod.db.example.com',
        }
        return urls[self]

env = Environment.PRODUCTION
if env.is_production:
    print(env.database_url)
```

### 4.4 多字段枚举

```python
from enum import Enum

class HTTPStatus(Enum):
    OK = (200, 'OK')
    CREATED = (201, 'Created')
    BAD_REQUEST = (400, 'Bad Request')
    NOT_FOUND = (404, 'Not Found')
    SERVER_ERROR = (500, 'Internal Server Error')
    
    def __init__(self, code, phrase):
        self.code = code
        self.phrase = phrase
    
    @classmethod
    def from_code(cls, code):
        for status in cls:
            if status.code == code:
                return status
        raise ValueError(f'Unknown status code: {code}')

HTTPStatus.OK.code      # 200
HTTPStatus.OK.phrase    # 'OK'
HTTPStatus.from_code(404)  # HTTPStatus.NOT_FOUND
```

### 4.5 与 Pydantic 配合

```python
from enum import Enum
from pydantic import BaseModel

class Status(str, Enum):
    PENDING = 'pending'
    APPROVED = 'approved'
    REJECTED = 'rejected'

class Item(BaseModel):
    name: str
    status: Status

item = Item(name='test', status='pending')
item.status  # Status.PENDING
item.model_dump()  # {'name': 'test', 'status': 'pending'}
```

***

## 五、最佳实践

| 场景 | 建议 |
|------|------|
| 需要字符串值 | 用 `StrEnum`（3.11+）或继承 `str` |
| 需要比较 | 用 `IntEnum` |
| 位标志 | 用 `IntFlag` |
| 禁止重复值 | 加 `@unique` 装饰器 |
| 自动编号 | 用 `auto()` |
| 类型检查 | 用 `Enum` 作为类型注解 |
