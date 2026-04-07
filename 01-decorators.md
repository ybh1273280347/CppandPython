# Python 装饰器完全手册

---

## 一、装饰器模板

### 函数装饰器（三层结构）

```python
def First(params):           # 第1层：接收装饰器参数
    def Second(func):        # 第2层：接收被装饰函数
        def Third(*args, **kwargs):  # 第3层：接收函数调用参数
            return func(*args, **kwargs)  # 执行原函数
        return Third         # 返回第3层
    return Second            # 返回第2层
```

### 类装饰器（两层结构）

```python
class Decorator:
    def __init__(self, params):     # 第1层：接收装饰器参数
        self.params = params
    
    def __call__(self, func):       # 第2层：接收被装饰函数
        def wrapper(*args, **kwargs):
            return func(*args, **kwargs)
        return wrapper
```

### 层次对应表

| 函数装饰器 | 类装饰器 | 作用 |
|-----------|---------|------|
| `First(params)` | `__init__(params)` | 接收装饰器参数 |
| `Second(func)` | `__call__(func)` | 接收被装饰函数 |
| `Third(*args, **kwargs)` | `__call__(*args, **kwargs)` | 接收函数调用参数 |

### 记忆口诀

> **有参数三层，无参数两层**
> **外层收参数，中层收函数，内层收调用**
> **函数三层嵌套，类用 init 和 call**

---

## 二、无参数装饰器

```python
from functools import wraps

def timer(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        import time
        start = time.time()
        result = func(*args, **kwargs)
        print(f"{func.__name__} took {time.time() - start:.4f}s")
        return result
    return wrapper

@timer
def my_func():
    pass
```

---

## 三、参数化装饰器

```python
def retry(max_attempts=3, delay=1):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(1, max_attempts + 1):
                try:
                    return func(*args, **kwargs)
                except Exception:
                    if attempt == max_attempts:
                        raise
                    time.sleep(delay)
        return wrapper
    return decorator

@retry(max_attempts=5, delay=0.5)
def unstable_api():
    pass
```

---

## 四、类装饰器

### 4.1 基本类装饰器

```python
class Timer:
    def __init__(self, func):
        self.func = func
    
    def __call__(self, *args, **kwargs):
        import time
        start = time.perf_counter()
        result = self.func(*args, **kwargs)
        print(f"{self.func.__name__} took {time.perf_counter() - start:.4f}s")
        return result

@Timer
def slow_func():
    import time
    time.sleep(0.5)
```

### 4.2 带状态的类装饰器

```python
class CountCalls:
    def __init__(self, func):
        self.func = func
        self.count = 0
    
    def __call__(self, *args, **kwargs):
        self.count += 1
        print(f"调用第 {self.count} 次")
        return self.func(*args, **kwargs)
    
    def reset(self):
        self.count = 0

@CountCalls
def hello():
    print("hello")

hello()       # 调用第 1 次
hello()       # 调用第 2 次
hello.reset()
hello()       # 调用第 1 次
```

### 4.3 函数装饰器 vs 类装饰器

| 特性 | 函数装饰器 | 类装饰器 |
|------|-----------|---------|
| 结构 | 三层嵌套 | `__init__` + `__call__` |
| 状态存储 | 闭包变量 | `self` 属性 |
| 额外方法 | 不易添加 | 可添加（如 `reset()`） |
| 适用场景 | 简单装饰器 | 需要维护状态 |

---

## 五、组合拳实战

### 5.1 partial + 装饰器工厂

```python
from functools import partial, wraps

def retry(max_attempts=3, delay=1):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for _ in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception:
                    continue
        return wrapper
    return decorator

# 预配置装饰器
retry_5_times = partial(retry, max_attempts=5)
retry_quick = partial(retry, delay=0.1)

@retry_5_times()
def unstable_func():
    pass
```

### 5.2 lru_cache + 自定义 TTL

```python
from functools import lru_cache, wraps
import time

def cached(ttl=60):
    """带过期时间的缓存"""
    def decorator(func):
        cache = {}
        
        @wraps(func)
        def wrapper(*args):
            key = args
            now = time.time()
            
            if key in cache:
                result, timestamp = cache[key]
                if now - timestamp < ttl:
                    return result
            
            result = func(*args)
            cache[key] = (result, now)
            return result
        
        return wrapper
    return decorator

@cached(ttl=60)
def get_weather(city):
    return f"{city}: 晴"
```
