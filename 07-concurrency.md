# Python 并发编程完全手册

***

## 一、核心速查表

| 场景 | 工具 | 原因 |
|------|------|------|
| 网络请求、数据库 | `asyncio` | IO 密集，等待时切换 |
| 大量计算、加密 | `ProcessPoolExecutor` | CPU 密集，绕过 GIL |
| 简单并发 | `ThreadPoolExecutor` | 简单易用 |
| IO + CPU 混合 | `asyncio` + `run_in_executor` | 各取所长 |

***

## 二、asyncio 异步编程

### 基本用法

```python
import asyncio

async def fetch(url):
    await asyncio.sleep(1)  # 模拟 IO
    return f"data from {url}"

async def main():
    tasks = [asyncio.create_task(fetch(f"url{i}")) for i in range(10)]
    results = await asyncio.gather(*tasks)
    print(results)

asyncio.run(main())
```

### 限制并发数

**价值**：防止请求过多被服务器拒绝

```python
async def main():
    semaphore = asyncio.Semaphore(10)  # 最多 10 个并发
    
    async def bounded_fetch(url):
        async with semaphore:
            return await fetch(url)
    
    tasks = [bounded_fetch(f"url{i}") for i in range(100)]
    results = await asyncio.gather(*tasks)
```

### 超时控制

**价值**：避免单个任务卡住整个程序

```python
async def main():
    try:
        result = await asyncio.wait_for(slow_task(), timeout=5)
    except asyncio.TimeoutError:
        print("超时")
```

### 容错处理

**价值**：一个失败不影响其他任务

```python
async def main():
    tasks = [fetch(f"url{i}") for i in range(10)]
    results = await asyncio.gather(*tasks, return_exceptions=True)
    
    for i, result in enumerate(results):
        if isinstance(result, Exception):
            print(f"url{i} 失败: {result}")
        else:
            print(f"url{i} 成功: {result}")
```

***

## 三、ThreadPoolExecutor 线程池

**价值**：简单并发，适合阻塞 IO（如 `requests`）

```python
from concurrent.futures import ThreadPoolExecutor
import requests

def fetch(url):
    return requests.get(url).status_code

urls = [f"https://httpbin.org/delay/1" for _ in range(10)]

with ThreadPoolExecutor(max_workers=5) as executor:
    results = list(executor.map(fetch, urls))

print(results)  # [200, 200, 200, ...]
```

### 获取结果两种方式

```python
with ThreadPoolExecutor(max_workers=5) as executor:
    # 方式1：map（按顺序返回）
    results = list(executor.map(fetch, urls))
    
    # 方式2：submit + as_completed（完成即返回）
    futures = [executor.submit(fetch, url) for url in urls]
    for future in as_completed(futures):
        print(future.result())
```

***

## 四、ProcessPoolExecutor 进程池

**价值**：绕过 GIL，真正并行处理 CPU 密集任务

```python
from concurrent.futures import ProcessPoolExecutor

def cpu_heavy(n):
    return sum(i ** 2 for i in range(n))

with ProcessPoolExecutor(max_workers=4) as executor:
    results = list(executor.map(cpu_heavy, [10_000_000] * 4))

print(results)
```

### 线程池 vs 进程池

| 特性 | 线程池 | 进程池 |
|------|--------|--------|
| 内存占用 | 低 | 高（每个进程独立） |
| 启动开销 | 小 | 大 |
| 适用场景 | IO 密集 | CPU 密集 |
| 数据共享 | 容易 | 需要序列化 |

***

## 五、混合场景

**价值**：异步处理 IO，线程池处理 CPU

```python
import asyncio
from concurrent.futures import ThreadPoolExecutor

def cpu_task(n):
    return sum(i ** 2 for i in range(n))

async def io_task():
    await asyncio.sleep(1)
    return "io done"

async def main():
    loop = asyncio.get_event_loop()
    
    with ThreadPoolExecutor(max_workers=4) as pool:
        # CPU 任务丢给线程池
        cpu_futures = [
            loop.run_in_executor(pool, cpu_task, 1_000_000)
            for _ in range(4)
        ]
        # IO 任务异步执行
        io_tasks = [asyncio.create_task(io_task()) for _ in range(10)]
        
        cpu_results = await asyncio.gather(*cpu_futures)
        io_results = await asyncio.gather(*io_tasks)

asyncio.run(main())
```

***

## 六、常见陷阱

### 陷阱1：异步函数里用阻塞代码

```python
# ❌ 错误：阻塞整个事件循环
async def bad():
    time.sleep(5)

# ✅ 正确：用异步版本
async def good():
    await asyncio.sleep(5)
```

### 陷阱2：忘记 await

```python
# ❌ 错误：拿到的是协程对象
async def bad():
    result = fetch()  # 没有 await
    print(result)     # <coroutine object>

# ✅ 正确
async def good():
    result = await fetch()
```

### 陷阱3：CPU 任务放异步

```python
# ❌ 错误：阻塞事件循环
async def bad():
    for i in range(10_000_000):
        pass

# ✅ 正确：丢给线程池
async def good():
    loop = asyncio.get_event_loop()
    await loop.run_in_executor(None, cpu_heavy)
```

***

## 七、API 详解

### asyncio.run()

```python
asyncio.run(coro, *, debug=False)
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `coro` | 协程 | 要运行的协程对象 |
| `debug` | bool | 是否启用调试模式 |

**功能**：创建事件循环，运行协程，关闭循环。程序入口用。

```python
asyncio.run(main())  # 运行 main 协程
```

### asyncio.create_task()

```python
asyncio.create_task(coro, *, name=None)
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `coro` | 协程 | 要调度的协程 |
| `name` | str | 任务名称（用于调试） |

**返回**：`Task` 对象

**功能**：将协程包装为任务，立即调度执行。

```python
task = asyncio.create_task(fetch(url))  # 立即开始执行
result = await task  # 等待结果
```

### asyncio.gather()

```python
asyncio.gather(*coros, return_exceptions=False)
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `*coros` | 协程/任务 | 要并发执行的对象 |
| `return_exceptions` | bool | `False`：异常向上抛；`True`：异常作为结果返回 |

**返回**：结果列表（按传入顺序）

**功能**：并发执行多个协程，等待全部完成。

```python
results = await asyncio.gather(
    fetch("url1"),
    fetch("url2"),
    return_exceptions=True  # 一个失败不影响其他
)
```

### asyncio.wait_for()

```python
asyncio.wait_for(coro, timeout, *, return_exceptions=False)
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `coro` | 协程 | 要执行的协程 |
| `timeout` | float | 超时秒数 |
| `return_exceptions` | bool | 超时时返回异常而非抛出 |

**功能**：带超时的协程执行。

```python
try:
    result = await asyncio.wait_for(fetch(url), timeout=5)
except asyncio.TimeoutError:
    print("超时")
```

### asyncio.Semaphore()

```python
asyncio.Semaphore(value=1)
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `value` | int | 初始计数（最大并发数） |

**方法**：
- `await sem.acquire()`：获取许可
- `sem.release()`：释放许可
- `async with sem:`：上下文管理（推荐）

```python
sem = asyncio.Semaphore(10)  # 最多 10 个并发

async with sem:
    await fetch(url)  # 获取许可后执行
```

### asyncio.sleep()

```python
asyncio.sleep(delay, result=None)
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `delay` | float | 秒数 |
| `result` | Any | 返回值 |

**功能**：非阻塞睡眠，让出控制权。

```python
await asyncio.sleep(1)  # 睡眠 1 秒，期间其他任务可执行
await asyncio.sleep(0)  # 立即让出控制权
```

### ThreadPoolExecutor / ProcessPoolExecutor

```python
ThreadPoolExecutor(max_workers=None, thread_name_prefix='')
ProcessPoolExecutor(max_workers=None)
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `max_workers` | int | 最大线程/进程数（默认 CPU 核数 × 5） |
| `thread_name_prefix` | str | 线程名前缀（仅线程池） |

```python
with ThreadPoolExecutor(max_workers=10) as executor:
    pass
```

### executor.map()

```python
executor.map(func, *iterables, timeout=None, chunksize=1)
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `func` | callable | 要执行的函数 |
| `*iterables` | iterable | 参数迭代器 |
| `timeout` | float | 超时秒数 |
| `chunksize` | int | 分块大小（仅进程池有效） |

**返回**：结果迭代器（按输入顺序）

```python
results = list(executor.map(fetch, urls))  # 按顺序返回
```

### executor.submit()

```python
executor.submit(func, *args, **kwargs)
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `func` | callable | 要执行的函数 |
| `*args` | Any | 位置参数 |
| `**kwargs` | Any | 关键字参数 |

**返回**：`Future` 对象

```python
future = executor.submit(fetch, url)
result = future.result()  # 阻塞等待结果
result = future.result(timeout=5)  # 带超时
```

### Future 对象方法

| 方法 | 说明 |
|------|------|
| `result(timeout=None)` | 获取结果（阻塞） |
| `exception()` | 获取异常 |
| `done()` | 是否完成 |
| `cancel()` | 取消任务 |
| `cancelled()` | 是否已取消 |

### loop.run_in_executor()

```python
loop.run_in_executor(executor, func, *args)
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `executor` | Executor | 执行器（`None` 用默认线程池） |
| `func` | callable | 要执行的同步函数 |
| `*args` | Any | 函数参数 |

**返回**：协程（可 await）

```python
loop = asyncio.get_event_loop()
result = await loop.run_in_executor(None, cpu_heavy, 1000000)
```

***

## 八、速查表

| 需求 | 代码 |
|------|------|
| 运行协程 | `asyncio.run(main())` |
| 并发多个协程 | `await asyncio.gather(*tasks)` |
| 创建任务 | `asyncio.create_task(coro())` |
| 限制并发 | `asyncio.Semaphore(10)` |
| 超时控制 | `await asyncio.wait_for(coro(), 5)` |
| 线程池 | `ThreadPoolExecutor(max_workers=10)` |
| 进程池 | `ProcessPoolExecutor(max_workers=4)` |
| 异步调用同步函数 | `loop.run_in_executor(pool, func, arg)` |

***

## 九、记忆口诀

> **IO 密集用异步，等待时间不浪费**
> **CPU 密集用进程，绕过 GIL 真并行**
> **简单并发线程池，省心省力够使用**
> **混合场景组合打，异步加执行器**
> 
> **阻塞代码是大忌，事件循环会卡死**
> **await 不能忘，否则拿到协程对象**
