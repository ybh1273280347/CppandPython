# Python 并发编程

***

## 核心速查：如何选择

| 场景 | 推荐方案 | 原因 |
|------|----------|------|
| 网络请求、文件 I/O | `asyncio` | 单线程高并发，资源消耗低 |
| CPU 密集计算 | `multiprocessing` | 绕过 GIL，真正并行 |
| 简单并发任务 | `concurrent.futures` | 接口简单，线程/进程池 |
| 共享状态多 | `threading` | 线程间共享内存方便 |
| 批量任务 | `ThreadPoolExecutor` | 控制并发数，复用线程 |

***

## 一、asyncio 异步编程

**适用场景**：网络请求、文件 I/O、数据库操作等 I/O 密集型任务

### 1.1 基础用法

```python
import asyncio

async def main():
    print('开始')
    await asyncio.sleep(1)  # 非阻塞等待
    print('结束')

asyncio.run(main())

# 定义协程
async def fetch(url):
    await asyncio.sleep(0.5)  # 模拟 I/O
    return f'数据: {url}'

# 运行
result = asyncio.run(fetch('https://example.com'))
```

### 1.2 并发执行

```python
import asyncio

async def task(name, delay):
    await asyncio.sleep(delay)
    return f'{name} 完成'

async def main():
    # 并发执行多个任务
    results = await asyncio.gather(
        task('A', 1),
        task('B', 2),
        task('C', 1),
    )
    print(results)  # ['A 完成', 'B 完成', 'C 完成']

asyncio.run(main())

# 创建任务
async def main():
    t1 = asyncio.create_task(task('A', 1))
    t2 = asyncio.create_task(task('B', 2))
    
    r1 = await t1
    r2 = await t2
    print(r1, r2)
```

### 1.3 超时控制

```python
import asyncio

async def slow_operation():
    await asyncio.sleep(5)
    return '完成'

async def main():
    try:
        result = await asyncio.wait_for(slow_operation(), timeout=2.0)
    except asyncio.TimeoutError:
        print('超时')

asyncio.run(main())
```

### 1.4 异步上下文管理器

```python
import asyncio
from contextlib import asynccontextmanager

@asynccontextmanager
async def async_resource():
    print('获取资源')
    await asyncio.sleep(0.1)
    try:
        yield 'resource'
    finally:
        print('释放资源')

async def main():
    async with async_resource() as r:
        print(f'使用: {r}')

asyncio.run(main())
```

***

## 二、threading 多线程

**适用场景**：I/O 密集型、需要共享状态、简单并发

### 2.1 基础用法

```python
import threading

def worker(name):
    print(f'{name} 开始')
    # 执行任务
    print(f'{name} 结束')

# 创建线程
t1 = threading.Thread(target=worker, args=('A',))
t2 = threading.Thread(target=worker, args=('B',))

t1.start()
t2.start()

t1.join()  # 等待完成
t2.join()
```

### 2.2 线程锁

```python
import threading

counter = 0
lock = threading.Lock()

def increment():
    global counter
    for _ in range(10000):
        with lock:  # 加锁
            counter += 1

threads = [threading.Thread(target=increment) for _ in range(10)]
for t in threads:
    t.start()
for t in threads:
    t.join()

print(counter)  # 100000
```

### 2.3 线程安全队列

```python
import threading
import queue

def producer(q):
    for i in range(5):
        q.put(f'item-{i}')
    q.put(None)  # 结束信号

def consumer(q):
    while True:
        item = q.get()
        if item is None:
            break
        print(f'处理: {item}')
        q.task_done()

q = queue.Queue()
t1 = threading.Thread(target=producer, args=(q,))
t2 = threading.Thread(target=consumer, args=(q,))

t1.start()
t2.start()
t1.join()
t2.join()
```

***

## 三、multiprocessing 多进程

**适用场景**：CPU 密集型任务（绕过 GIL）

### 3.1 基础用法

```python
import multiprocessing

def worker(name):
    print(f'{name} 运行在进程 {multiprocessing.current_process().pid}')

if __name__ == '__main__':
    processes = [
        multiprocessing.Process(target=worker, args=(f'P{i}',))
        for i in range(4)
    ]
    
    for p in processes:
        p.start()
    for p in processes:
        p.join()
```

### 3.2 进程池

```python
import multiprocessing

def compute(n):
    return n * n

if __name__ == '__main__':
    with multiprocessing.Pool(4) as pool:
        # map
        results = pool.map(compute, range(10))
        print(results)  # [0, 1, 4, 9, 16, ...]
        
        # 异步
        result = pool.apply_async(compute, (10,))
        print(result.get())  # 100
```

### 3.3 进程间通信

```python
import multiprocessing

def worker(q):
    q.put('来自子进程的消息')

if __name__ == '__main__':
    q = multiprocessing.Queue()
    p = multiprocessing.Process(target=worker, args=(q,))
    p.start()
    print(q.get())  # 来自子进程的消息
    p.join()

# 共享内存
def worker2(arr):
    arr[0] = 100

if __name__ == '__main__':
    arr = multiprocessing.Array('i', [0, 0, 0])  # 整数数组
    p = multiprocessing.Process(target=worker2, args=(arr,))
    p.start()
    p.join()
    print(arr[:])  # [100, 0, 0]
```

***

## 四、concurrent.futures 并发池

**适用场景**：批量任务、简化线程/进程管理

### 4.1 ThreadPoolExecutor

```python
from concurrent.futures import ThreadPoolExecutor

def fetch(url):
    import time
    time.sleep(0.5)
    return f'数据: {url}'

urls = ['url1', 'url2', 'url3', 'url4', 'url5']

with ThreadPoolExecutor(max_workers=3) as executor:
    # map 方式
    results = list(executor.map(fetch, urls))
    
    # submit 方式
    futures = [executor.submit(fetch, url) for url in urls]
    for future in futures:
        print(future.result())
```

### 4.2 ProcessPoolExecutor

```python
from concurrent.futures import ProcessPoolExecutor

def compute(n):
    return sum(i * i for i in range(n))

if __name__ == '__main__':
    with ProcessPoolExecutor(max_workers=4) as executor:
        results = list(executor.map(compute, [10**6, 10**6, 10**6]))
        print(results)
```

### 4.3 as_completed

```python
from concurrent.futures import ThreadPoolExecutor, as_completed

def task(n):
    import time
    time.sleep(n)
    return f'任务 {n} 完成'

with ThreadPoolExecutor(max_workers=3) as executor:
    futures = {executor.submit(task, n): n for n in [3, 1, 2]}
    
    for future in as_completed(futures):
        n = futures[future]
        print(f'{n}: {future.result()}')
```

***

## 五、常用模板

### 5.1 异步 HTTP 请求

```python
import asyncio
import httpx

async def fetch(client, url):
    response = await client.get(url)
    return response.text

async def main():
    urls = ['https://example.com'] * 10
    
    async with httpx.AsyncClient() as client:
        tasks = [fetch(client, url) for url in urls]
        results = await asyncio.gather(*tasks)
    
    print(f'获取了 {len(results)} 个页面')

asyncio.run(main())
```

### 5.2 限流并发

```python
import asyncio

async def limited_concurrency(tasks, limit=5):
    semaphore = asyncio.Semaphore(limit)
    
    async def run_with_limit(task):
        async with semaphore:
            return await task
    
    return await asyncio.gather(*[run_with_limit(t) for t in tasks])
```

### 5.3 线程池批量处理

```python
from concurrent.futures import ThreadPoolExecutor

def process_batch(items, func, max_workers=10):
    with ThreadPoolExecutor(max_workers=max_workers) as executor:
        results = list(executor.map(func, items))
    return results

# 使用
def process_item(item):
    return item * 2

results = process_batch(range(100), process_item)
```

### 5.4 进程池 CPU 计算

```python
from concurrent.futures import ProcessPoolExecutor

def parallel_compute(data, func, workers=None):
    if workers is None:
        workers = multiprocessing.cpu_count()
    
    with ProcessPoolExecutor(max_workers=workers) as executor:
        return list(executor.map(func, data))

# 使用
def heavy_compute(n):
    return sum(i ** 2 for i in range(n))

if __name__ == '__main__':
    results = parallel_compute([10**5] * 8, heavy_compute)
```

***

## 六、性能对比

| 方案 | GIL 影响 | 内存消耗 | 切换开销 | 适用场景 |
|------|----------|----------|----------|---------|
| asyncio | 无影响 | 最低 | 最低 | I/O 密集型 |
| threading | 受限 | 低 | 低 | I/O 密集型 + 共享状态 |
| multiprocessing | 无 | 高 | 高 | CPU 密集型 |
| ThreadPoolExecutor | 受限 | 低 | 低 | 批量 I/O 任务 |
| ProcessPoolExecutor | 无 | 高 | 高 | 批量 CPU 任务 |

***

## 七、最佳实践

| 场景 | 建议 |
|------|------|
| 网络爬虫 | `asyncio` + `httpx` |
| 文件批量处理 | `ThreadPoolExecutor` |
| 数据计算 | `ProcessPoolExecutor` |
| 需要共享状态 | `threading` + `Lock` |
| 高并发服务 | `asyncio` |
| 简单并行 | `concurrent.futures` |
