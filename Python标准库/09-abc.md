# Python 抽象基类

***

## 核心速查

| 类/装饰器 | 作用 |
|-----------|------|
| `ABC` | 抽象基类基类 |
| `@abstractmethod` | 抽象方法装饰器 |
| `@abstractproperty` | 抽象属性 |
| `@abstractclassmethod` | 抽象类方法 |
| `@abstractstaticmethod` | 抽象静态方法 |

***

## 一、基础用法

```python
from abc import ABC, abstractmethod

class Animal(ABC):
    @abstractmethod
    def speak(self):
        pass
    
    @abstractmethod
    def move(self):
        pass

# 子类必须实现所有抽象方法
class Dog(Animal):
    def speak(self):
        return '汪汪'
    
    def move(self):
        return '跑'

# 实例化
dog = Dog()
dog.speak()  # '汪汪'

# 未实现所有方法会报错
class Cat(Animal):
    def speak(self):
        return '喵喵'

# cat = Cat()  # TypeError: Can't instantiate abstract class Cat with abstract method move
```

### 1.1 默认实现

```python
from abc import ABC, abstractmethod

class Base(ABC):
    @abstractmethod
    def process(self, data):
        """子类必须实现"""
        pass
    
    def validate(self, data):
        """可选方法，有默认实现"""
        return data is not None

class Impl(Base):
    def process(self, data):
        return data * 2

impl = Impl()
impl.process(5)    # 10
impl.validate(5)   # True
```

***

## 二、抽象属性

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @property
    @abstractmethod
    def area(self):
        pass
    
    @property
    @abstractmethod
    def perimeter(self):
        pass

class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height
    
    @property
    def area(self):
        return self.width * self.height
    
    @property
    def perimeter(self):
        return 2 * (self.width + self.height)

rect = Rectangle(3, 4)
rect.area       # 12
rect.perimeter  # 14
```

***

## 三、抽象类方法

```python
from abc import ABC, abstractmethod

class Database(ABC):
    @abstractmethod
    def connect(self):
        pass
    
    @classmethod
    @abstractmethod
    def get_default_config(cls):
        pass
    
    @staticmethod
    @abstractmethod
    def validate_connection_string(conn_str):
        pass

class MySQL(Database):
    def connect(self):
        return 'MySQL connected'
    
    @classmethod
    def get_default_config(cls):
        return {'host': 'localhost', 'port': 3306}
    
    @staticmethod
    def validate_connection_string(conn_str):
        return conn_str.startswith('mysql://')
```

***

## 四、检查类型

```python
from abc import ABC, abstractmethod

class Plugin(ABC):
    @abstractmethod
    def run(self):
        pass

class PluginA(Plugin):
    def run(self):
        return 'A running'

class NotAPlugin:
    pass

# 检查是否是子类
issubclass(PluginA, Plugin)      # True
issubclass(NotAPlugin, Plugin)   # False

# 检查实例
a = PluginA()
isinstance(a, Plugin)            # True
```

***

## 五、常用模板

### 5.1 插件系统

```python
from abc import ABC, abstractmethod

class PluginBase(ABC):
    """插件基类"""
    
    @property
    @abstractmethod
    def name(self) -> str:
        """插件名称"""
        pass
    
    @property
    @abstractmethod
    def version(self) -> str:
        """插件版本"""
        pass
    
    @abstractmethod
    def initialize(self, config: dict):
        """初始化插件"""
        pass
    
    @abstractmethod
    def execute(self, *args, **kwargs):
        """执行插件功能"""
        pass
    
    def shutdown(self):
        """可选：清理资源"""
        pass

class MyPlugin(PluginBase):
    name = 'my_plugin'
    version = '1.0.0'
    
    def initialize(self, config):
        self.config = config
    
    def execute(self, data):
        return f'Processing: {data}'
```

### 5.2 数据源接口

```python
from abc import ABC, abstractmethod
from typing import Iterator, Any

class DataSource(ABC):
    @abstractmethod
    def read(self) -> Iterator[Any]:
        """读取数据"""
        pass
    
    @abstractmethod
    def write(self, data: Any):
        """写入数据"""
        pass
    
    @abstractmethod
    def close(self):
        """关闭连接"""
        pass
    
    def __enter__(self):
        return self
    
    def __exit__(self, *args):
        self.close()

class FileSource(DataSource):
    def __init__(self, path):
        self.path = path
        self.file = None
    
    def read(self):
        with open(self.path) as f:
            for line in f:
                yield line.strip()
    
    def write(self, data):
        with open(self.path, 'a') as f:
            f.write(str(data) + '\n')
    
    def close(self):
        pass
```

### 5.3 策略模式

```python
from abc import ABC, abstractmethod

class SortStrategy(ABC):
    @abstractmethod
    def sort(self, data):
        pass

class QuickSort(SortStrategy):
    def sort(self, data):
        return sorted(data)

class MergeSort(SortStrategy):
    def sort(self, data):
        if len(data) <= 1:
            return data
        mid = len(data) // 2
        left = self.sort(data[:mid])
        right = self.sort(data[mid:])
        return self._merge(left, right)
    
    def _merge(self, left, right):
        result = []
        i = j = 0
        while i < len(left) and j < len(right):
            if left[i] <= right[j]:
                result.append(left[i])
                i += 1
            else:
                result.append(right[j])
                j += 1
        result.extend(left[i:])
        result.extend(right[j:])
        return result

class Sorter:
    def __init__(self, strategy: SortStrategy):
        self.strategy = strategy
    
    def sort(self, data):
        return self.strategy.sort(data)
```

***

## 六、最佳实践

| 场景 | 建议 |
|------|------|
| 定义接口 | 继承 `ABC`，用 `@abstractmethod` |
| 强制实现 | 所有抽象方法必须实现才能实例化 |
| 可选方法 | 不加 `@abstractmethod`，提供默认实现 |
| 类型检查 | 用 `isinstance(obj, ABC类)` |
| 属性 | 用 `@property` + `@abstractmethod` |
