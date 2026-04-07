# Python 动态属性完全手册

***

## 一、核心速查表

| 工具 | 作用 | 典型场景 |
|------|------|---------|
| `getattr()` | 字符串获取属性 | 配置读取、命令分发 |
| `setattr()` | 字符串设置属性 | 批量赋值、JSON 转对象 |
| `hasattr()` | 检查属性存在 | 安全访问 |
| `__dict__` | 实例属性字典 | 查看所有属性 |
| `__getattr__` | 属性不存在时调用 | 代理、懒加载、默认值 |
| `__setattr__` | 拦截属性赋值 | 验证、日志 |

***

## 二、内置函数

### getattr - 属性名来自变量

**价值**：当属性名在运行时才能确定时使用

```python
class Config:
    host = "localhost"
    port = 8080

key = input("输入配置项: ")  # 运行时才知道
print(getattr(Config, key, "未设置"))
```

### setattr - 批量设置属性

**价值**：将字典转为对象属性，访问更方便

```python
class User:
    pass

data = {"name": "Alice", "age": 25, "email": "a@b.com"}
user = User()
for k, v in data.items():
    setattr(user, k, v)

print(user.name)  # 比 data["name"] 更简洁
```

***

## 三、魔术方法详解

### `__getattr__` - 属性不存在时的兜底

**触发时机**：访问不存在的属性时调用

**核心价值**：
- 代理模式：转发属性访问
- 懒加载：首次访问时才计算
- 默认值：避免 AttributeError

```python
class LazyLoader:
    def __init__(self):
        self._cache = {}
    
    def __getattr__(self, name):
        if name not in self._cache:
            print(f"加载 {name}...")
            self._cache[name] = f"{name}_data"
        return self._cache[name]

loader = LazyLoader()
print(loader.config)   # 加载 config... config_data
print(loader.config)   # config_data（不重新加载）
```

**陷阱**：在 `__getattr__` 中访问 `self.xxx` 会无限递归

```python
class Wrong:
    def __getattr__(self, name):
        return self.other  # 错误！self.other 不存在，又调用 __getattr__

class Right:
    def __getattr__(self, name):
        return self.__dict__.get(name)  # 正确：直接操作 __dict__
```

### `__setattr__` - 拦截所有赋值

**触发时机**：每次属性赋值都会调用

**核心价值**：
- 数据验证：赋值前检查
- 日志记录：追踪属性变化
- 数据分离：存到指定位置

```python
class Validated:
    def __setattr__(self, name, value):
        if name == "age" and value < 0:
            raise ValueError("年龄不能为负")
        super().__setattr__(name, value)  # 必须调用父类方法

v = Validated()
v.age = 25
# v.age = -5  # ValueError
```

**陷阱**：在 `__setattr__` 中使用 `self.xxx = value` 会无限递归

```python
class Wrong:
    def __setattr__(self, name, value):
        self._data[name] = value  # 错误！self._data 又触发 __setattr__

class Right:
    def __setattr__(self, name, value):
        super().__setattr__('_data', {})  # 正确：用父类方法
        self._data[name] = value
```

### 调用顺序

```
访问 obj.attr
    ↓
__getattribute__（总是先调用）
    ↓
属性存在？ ──是──→ 返回值
    │
    否
    ↓
__getattr__（兜底处理）
```

**`__getattribute__` 慎用**：每次访问都触发，极易无限递归

***

## 四、典型场景

### 场景1：JSON 转对象 `setattr`

**价值**：API 响应直接用点号访问

```python
class APIResponse:
    def __init__(self, data: dict):
        for k, v in data.items():
            setattr(self, k, v)

resp = APIResponse({"status": 200, "user_id": 123})
print(resp.status)   # 200
print(resp.user_id)  # 123
```

### 场景2：命令分发 `getattr` `hasattr`

**价值**：字符串命令映射到方法，避免 if-elif 链

```python
class Handler:
    def cmd_help(self): return "帮助信息"
    def cmd_exit(self): return "退出"
    
    def execute(self, cmd):
        method = getattr(self, f"cmd_{cmd}", None)
        if method:
            return method()
        return f"未知命令: {cmd}"

h = Handler()
print(h.execute("help"))  # 帮助信息
```

### 场景3：代理模式 `__getattr__` `__setattr__`

**价值**：控制对目标对象的访问

```python
class Proxy:
    def __init__(self, target):
        object.__setattr__(self, '_target', target)
    
    def __getattr__(self, name):
        return getattr(self._target, name)
    
    def __setattr__(self, name, value):
        setattr(self._target, name, value)

db = type('DB', (), {'host': 'localhost'})()
proxy = Proxy(db)
print(proxy.host)  # localhost
```

### 场景4：属性验证 `__setattr__`

**价值**：统一验证逻辑，子类只需定义规则

```python
class Validated:
    _validators = {}
    
    def __setattr__(self, name, value):
        if name in self._validators and not self._validators[name](value):
            raise ValueError(f"无效的 {name}")
        super().__setattr__(name, value)

class User(Validated):
    _validators = {
        "age": lambda x: x >= 0,
        "email": lambda x: '@' in x,
    }

u = User()
u.age = 25
# u.age = -1  # ValueError
```

***

## 五、__dict__ 技巧

**价值**：批量操作属性

```python
class Data:
    def __init__(self, **kwargs):
        self.__dict__.update(kwargs)

d = Data(name="Alice", age=25)
print(d.__dict__)  # {'name': 'Alice', 'age': 25}
```

***

## 六、记忆口诀

> **getattr 字串取，属性名运行时定**
> **setattr 批量设，字典转对象最方便**
> 
> **__getattr__ 兜底王，属性没有它来当**
> **__setattr__ 拦截虎，验证日志都能做**
> **两者都要防递归，super 调用保平安**
