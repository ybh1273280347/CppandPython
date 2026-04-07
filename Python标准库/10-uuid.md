# Python UUID

***

## 核心速查

| 函数 | 作用 | 特点 |
|------|------|------|
| `uuid4()` | 随机 UUID | 最常用，无序 |
| `uuid1()` | 基于时间 | 有序，含 MAC 地址 |
| `uuid3(ns, name)` | 基于名字 MD5 | 相同输入相同输出 |
| `uuid5(ns, name)` | 基于名字 SHA-1 | 相同输入相同输出 |

***

## 一、基础用法

```python
import uuid

# UUID4 - 随机生成（最常用）
id4 = uuid.uuid4()
print(id4)           # a8098c1a-f86e-11da-bd1a-00112444be1e
print(str(id4))      # 转字符串
print(id4.hex)       # 无连字符: a8098c1af86e11dabd1a00112444be1e
print(id4.int)       # 整数形式

# UUID1 - 基于时间和 MAC 地址
id1 = uuid.uuid1()
print(id1)           # 包含时间戳和主机信息

# 从字符串创建
u = uuid.UUID('a8098c1a-f86e-11da-bd1a-00112444be1e')
u = uuid.UUID('a8098c1af86e11dabd1a00112444be1e')  # 无连字符也可以
```

***

## 二、UUID 版本对比

| 版本 | 生成方式 | 优点 | 缺点 | 适用场景 |
|------|----------|------|------|---------|
| 1 | 时间 + MAC | 有序，可追溯 | 暴露硬件信息 | 分布式系统 |
| 3 | MD5 哈希 | 可重现 | 不安全 | 旧系统兼容 |
| 4 | 随机 | 简单，安全 | 无序 | 通用场景 |
| 5 | SHA-1 哈希 | 可重现，较安全 | 无序 | 需要确定性 ID |

### 2.1 UUID1 - 基于时间

```python
import uuid

id1 = uuid.uuid1()
# 格式: time_low-time_mid-time_hi_version-clock_seq-node

# 提取时间信息
from datetime import datetime
timestamp = (id1.time - 0x01b21dd213814000) / 1e7  # 转换为 Unix 时间戳
dt = datetime.fromtimestamp(timestamp)
```

### 2.2 UUID3/5 - 基于名字

```python
import uuid

# 命名空间
NAMESPACE_DNS = uuid.NAMESPACE_DNS    # 域名
NAMESPACE_URL = uuid.NAMESPACE_URL    # URL
NAMESPACE_OID = uuid.NAMESPACE_OID    # OID
NAMESPACE_X500 = uuid.NAMESPACE_X500  # X500 DN

# 相同输入产生相同 UUID
name = 'example.com'

id3_1 = uuid.uuid3(uuid.NAMESPACE_DNS, name)
id3_2 = uuid.uuid3(uuid.NAMESPACE_DNS, name)
print(id3_1 == id3_2)  # True

id5_1 = uuid.uuid5(uuid.NAMESPACE_DNS, name)
id5_2 = uuid.uuid5(uuid.NAMESPACE_DNS, name)
print(id5_1 == id5_2)  # True

# UUID5 比 UUID3 更安全（SHA-1 vs MD5）
```

***

## 三、常用模板

### 3.1 生成唯一 ID

```python
import uuid

def generate_id():
    """生成唯一 ID"""
    return str(uuid.uuid4())

def generate_short_id():
    """生成短 ID（无连字符）"""
    return uuid.uuid4().hex

def generate_id_bytes():
    """生成 16 字节二进制 ID"""
    return uuid.uuid4().bytes

# 使用
generate_id()        # 'a8098c1a-f86e-11da-bd1a-00112444be1e'
generate_short_id()  # 'a8098c1af86e11dabd1a00112444be1e'
```

### 3.2 数据库主键

```python
import uuid
from dataclasses import dataclass

@dataclass
class User:
    id: str
    name: str
    email: str

def create_user(name: str, email: str) -> User:
    return User(
        id=str(uuid.uuid4()),
        name=name,
        email=email
    )
```

### 3.3 文件名

```python
import uuid
from pathlib import Path

def unique_filename(original_name: str) -> str:
    """生成唯一文件名，保留扩展名"""
    ext = Path(original_name).suffix
    return f'{uuid.uuid4().hex}{ext}'

unique_filename('document.pdf')  # 'a8098c1af86e11dabd1a00112444be1e.pdf'
```

### 3.4 请求追踪 ID

```python
import uuid
from contextvars import ContextVar

request_id: ContextVar[str] = ContextVar('request_id')

def set_request_id():
    """设置请求 ID"""
    rid = str(uuid.uuid4())[:8]  # 短版本
    request_id.set(rid)
    return rid

def get_request_id() -> str:
    """获取当前请求 ID"""
    return request_id.get('')

# 使用
set_request_id()  # 'a8098c1a'
```

### 3.5 确定性 ID（幂等）

```python
import uuid

def get_deterministic_id(namespace: str, name: str) -> str:
    """相同输入产生相同 ID"""
    ns = uuid.UUID(namespace) if '-' in namespace else uuid.UUID(hex=namespace)
    return str(uuid.uuid5(ns, name))

# 使用：相同用户名始终产生相同 ID
user_namespace = '6ba7b810-9dad-11d1-80b4-00c04fd430c8'
get_deterministic_id(user_namespace, 'alice')  # 每次都一样
```

***

## 四、最佳实践

| 场景 | 建议 |
|------|------|
| 通用唯一 ID | 用 `uuid4()` |
| 需要有序 | 用 `uuid1()`（注意隐私） |
| 需要可重现 | 用 `uuid5()` |
| 数据库主键 | 用 `uuid4()` 的字符串形式 |
| 文件名 | 用 `uuid4().hex` |
| 追踪 ID | 用 `uuid4()` 前 8 位 |
