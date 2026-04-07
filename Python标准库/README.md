# Python 标准库笔记

Python 标准库与常用第三方库速查手册。

***

## 文件索引

| 序号 | 文件 | 内容 | 常用程度 |
|------|------|------|---------|
| 01 | [file-operations](01-file-operations.md) | os/pathlib/shutil 文件操作 | ⭐⭐⭐⭐⭐ |
| 02 | [http-requests](02-http-requests.md) | httpx/requests/urllib 网络请求 | ⭐⭐⭐⭐⭐ |
| 03 | [json](03-json.md) | JSON 序列化与处理 | ⭐⭐⭐⭐⭐ |
| 04 | [logging](04-logging.md) | loguru 日志系统 | ⭐⭐⭐⭐⭐ |
| 05 | [concurrency](05-concurrency.md) | asyncio/threading/multiprocessing 并发编程 | ⭐⭐⭐⭐⭐ |
| 06 | [regex](06-regex.md) | re 正则表达式 | ⭐⭐⭐⭐ |
| 07 | [enum](07-enum.md) | 枚举类型 | ⭐⭐⭐⭐ |
| 08 | [inspect](08-inspect.md) | 对象检查 | ⭐⭐⭐⭐ |
| 09 | [abc](09-abc.md) | 抽象基类 | ⭐⭐⭐⭐ |
| 10 | [uuid](10-uuid.md) | UUID 生成 | ⭐⭐⭐⭐ |
| 11 | [random](11-random.md) | random/secrets 随机数 | ⭐⭐⭐⭐ |
| 12 | [archive](12-archive.md) | zipfile/tarfile 压缩文件 | ⭐⭐⭐ |

***

## 详细介绍

### 01-file-operations.md - 文件操作

**内容**：`pathlib`（推荐）、`os`、`shutil`

**核心用法**：
```python
from pathlib import Path

p = Path('data/file.txt')
p.read_text()           # 读取
p.write_text('content') # 写入
```

---

### 02-http-requests.md - 网络请求

**内容**：`httpx`（推荐）、`requests`、`urllib`

**核心用法**：
```python
import httpx

response = httpx.get(url)

async with httpx.AsyncClient() as client:
    response = await client.get(url)
```

---

### 03-json.md - JSON 处理

**内容**：`json` 模块完整指南

**核心用法**：
```python
import json

json.dumps(data, ensure_ascii=False, indent=2)
data = json.loads(json_str)
```

---

### 04-logging.md - 日志系统

**内容**：`loguru`（推荐，比 logging 简单）

**核心用法**：
```python
from loguru import logger

logger.info('操作完成')
logger.add('app.log', rotation='10 MB')
```

---

### 05-concurrency.md - 并发编程

**内容**：`asyncio`、`threading`、`multiprocessing`、`concurrent.futures`

**如何选择**：
| 场景 | 方案 |
|------|------|
| 网络/I/O 密集 | `asyncio` |
| CPU 密集 | `multiprocessing` |
| 简单并发 | `concurrent.futures` |

**核心用法**：
```python
import asyncio

async def main():
    await asyncio.gather(task1(), task2())

asyncio.run(main())
```

---

### 06-regex.md - 正则表达式

**内容**：`re` 模块、常用模式、分组捕获

**核心用法**：
```python
import re

re.findall(r'\d+', text)
re.sub(r'\s+', ' ', text)
```

---

### 07-enum.md - 枚举类型

**内容**：`Enum`、`IntEnum`、`StrEnum`、`Flag`

**核心用法**：
```python
from enum import Enum, auto

class Status(Enum):
    PENDING = auto()
    APPROVED = auto()
```

---

### 08-inspect.md - 对象检查

**内容**：获取对象信息、函数签名、源代码

**核心用法**：
```python
import inspect

inspect.getmembers(obj)      # 获取所有成员
inspect.signature(func)      # 获取函数签名
inspect.getsource(func)      # 获取源代码
```

---

### 09-abc.md - 抽象基类

**内容**：`ABC`、`@abstractmethod`、抽象属性

**核心用法**：
```python
from abc import ABC, abstractmethod

class Plugin(ABC):
    @abstractmethod
    def run(self):
        pass
```

---

### 10-uuid.md - UUID 生成

**内容**：`uuid4`、`uuid1`、`uuid5`、`secrets`

**核心用法**：
```python
import uuid

uuid.uuid4()      # 随机 UUID
uuid.uuid4().hex  # 无连字符
```

---

### 11-random.md - 随机数

**内容**：`random`、`secrets`、常用分布

**核心用法**：
```python
import random
import secrets

random.randint(1, 100)     # 随机整数
random.choice(items)       # 随机选择
secrets.token_hex(16)      # 安全 Token
```

---

### 12-archive.md - 压缩文件

**内容**：`zipfile`、`tarfile`、`shutil`

**核心用法**：
```python
import zipfile

with zipfile.ZipFile('archive.zip', 'w') as zf:
    zf.write('file.txt')
```

***

## 快速查找

| 需求 | 查看文件 |
|------|---------|
| 路径操作 | 01-file-operations |
| HTTP 请求 | 02-http-requests |
| JSON 读写 | 03-json |
| 日志记录 | 04-logging |
| 并发编程 | 05-concurrency |
| 正则匹配 | 06-regex |
| 常量枚举 | 07-enum |
| 对象检查 | 08-inspect |
| 接口定义 | 09-abc |
| 唯一 ID | 10-uuid |
| 随机数 | 11-random |
| ZIP 压缩 | 12-archive |

***

## 学习建议

1. **文件操作** → 优先 `pathlib`
2. **网络请求** → 新项目用 `httpx`
3. **日志** → 用 `loguru`
4. **并发** → I/O 密集用 `asyncio`，CPU 密集用 `multiprocessing`
5. **唯一 ID** → 用 `uuid4()`
6. **安全随机** → 用 `secrets`
