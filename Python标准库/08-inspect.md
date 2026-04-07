# Python 对象检查

***

## 核心速查

| 函数 | 作用 |
|------|------|
| `inspect.getmembers(obj)` | 获取所有成员 |
| `inspect.signature(func)` | 获取函数签名 |
| `inspect.getsource(func)` | 获取源代码 |
| `inspect.getmodule(obj)` | 获取所在模块 |
| `inspect.isfunction(obj)` | 是否是函数 |
| `inspect.isclass(obj)` | 是否是类 |

***

## 一、基础检查

```python
import inspect

# 获取对象类型
inspect.isfunction(obj)      # 是否是函数
inspect.isclass(obj)         # 是否是类
inspect.ismethod(obj)        # 是否是方法
inspect.ismodule(obj)        # 是否是模块
inspect.isgenerator(obj)     # 是否是生成器
inspect.iscoroutine(obj)     # 是否是协程
inspect.isbuiltin(obj)       # 是否是内置函数
```

### 1.1 获取成员

```python
import inspect

class MyClass:
    def __init__(self):
        self.value = 10
    
    def method(self):
        pass

obj = MyClass()

# 获取所有成员
members = inspect.getmembers(obj)
# [('__class__', <class '__main__.MyClass'>), ('method', <bound method...>), ('value', 10), ...]

# 过滤成员
for name, value in inspect.getmembers(obj):
    if not name.startswith('_'):
        print(name, value)

# 按类型过滤
inspect.getmembers(obj, inspect.ismethod)      # 只取方法
inspect.getmembers(obj, inspect.isfunction)    # 只取函数
inspect.getmembers(obj, lambda x: isinstance(x, int))  # 只取整数
```

***

## 二、函数签名

### 2.1 获取签名

```python
import inspect

def greet(name: str, age: int = 18, *args, **kwargs) -> str:
    return f'{name} is {age}'

sig = inspect.signature(greet)

print(sig)                    # (name: str, age: int = 18, *args, **kwargs) -> str
print(sig.parameters)         # 参数字典
print(sig.return_annotation)  # str
```

### 2.2 参数信息

```python
sig = inspect.signature(greet)

for name, param in sig.parameters.items():
    print(f'{name}:')
    print(f'  默认值: {param.default}')
    print(f'  类型: {param.annotation}')
    print(f'  类型: {param.kind}')

# 参数类型
# POSITIONAL_ONLY        仅位置参数
# POSITIONAL_OR_KEYWORD  位置或关键字参数
# VAR_POSITIONAL         *args
# KEYWORD_ONLY           仅关键字参数
# VAR_KEYWORD            **kwargs
```

### 2.3 绑定参数

```python
def func(a, b, c=10):
    return a + b + c

sig = inspect.signature(func)

# 绑定参数
bound = sig.bind(1, 2)
bound.apply_defaults()
print(bound.arguments)  # {'a': 1, 'b': 2, 'c': 10}

# 验证参数
try:
    sig.bind(1)  # 缺少参数
except TypeError as e:
    print(e)
```

***

## 三、获取源代码

### 3.1 获取源码

```python
import inspect

def my_function():
    """示例函数"""
    return 42

# 获取源代码
source = inspect.getsource(my_function)
print(source)

# 获取源码文件路径
file = inspect.getfile(my_function)

# 获取源码行号
lines = inspect.getsourcelines(my_function)
# (['def my_function():\n', '    return 42\n'], 起始行号)

# 获取模块
module = inspect.getmodule(my_function)
```

### 3.2 获取类信息

```python
import inspect

class Base:
    pass

class Derived(Base):
    """派生类"""
    def method(self):
        pass

# 获取文档字符串
print(inspect.getdoc(Derived))

# 获取注释
print(inspect.getcomments(Derived))

# 获取继承链
print(inspect.getmro(Derived))  # (<class 'Derived'>, <class 'Base'>, <class 'object'>)
```

***

## 四、类型提示

```python
import inspect
from typing import List, Dict, Optional

def process(data: List[int], config: Optional[Dict] = None) -> bool:
    return True

sig = inspect.signature(process)

# 获取参数类型
for name, param in sig.parameters.items():
    if param.annotation != inspect.Parameter.empty:
        print(f'{name}: {param.annotation}')

# 获取返回类型
print(sig.return_annotation)  # bool
```

***

## 五、常用模板

### 5.1 打印对象信息

```python
import inspect

def describe(obj):
    print(f'类型: {type(obj).__name__}')
    print(f'模块: {inspect.getmodule(obj).__name__}')
    
    if inspect.isfunction(obj) or inspect.ismethod(obj):
        sig = inspect.signature(obj)
        print(f'签名: {sig}')
        doc = inspect.getdoc(obj)
        if doc:
            print(f'文档: {doc[:50]}...')

describe(print)
```

### 5.2 获取类属性

```python
import inspect

def get_class_attributes(cls):
    attrs = {}
    for name, value in inspect.getmembers(cls):
        if not name.startswith('_'):
            attrs[name] = value
    return attrs

class Config:
    DEBUG = True
    HOST = 'localhost'
    PORT = 8080

print(get_class_attributes(Config))
```

### 5.3 检查函数参数

```python
import inspect

def validate_args(func, *args, **kwargs):
    sig = inspect.signature(func)
    try:
        bound = sig.bind(*args, **kwargs)
        bound.apply_defaults()
        return True, bound.arguments
    except TypeError as e:
        return False, str(e)

def example(a, b, c=10):
    pass

print(validate_args(example, 1, 2))        # (True, {'a': 1, 'b': 2, 'c': 10})
print(validate_args(example, 1))           # (False, 'missing a required argument: ...')
```

### 5.4 动态调用

```python
import inspect

def safe_call(func, *args, **kwargs):
    sig = inspect.signature(func)
    bound = sig.bind(*args, **kwargs)
    bound.apply_defaults()
    return func(**bound.arguments)

def greet(name, greeting='Hello'):
    return f'{greeting}, {name}!'

print(safe_call(greet, 'Alice'))
print(safe_call(greet, name='Bob', greeting='Hi'))
```

### 5.5 获取调用者信息

```python
import inspect

def who_called_me():
    frame = inspect.currentframe()
    caller = frame.f_back
    print(f'调用者: {caller.f_code.co_name}')
    print(f'文件: {caller.f_code.co_filename}')
    print(f'行号: {caller.f_lineno}')

def test():
    who_called_me()

test()
```

***

## 六、最佳实践

| 场景 | 用法 |
|------|------|
| 调试 | `inspect.getsource()` 查看源码 |
| 文档生成 | `inspect.getdoc()` 获取文档 |
| 参数验证 | `inspect.signature()` 检查参数 |
| 动态调用 | `sig.bind()` 绑定参数 |
| 类型检查 | `param.annotation` 获取类型提示 |
