# Python 日志系统

***

## 核心速查

| 方法 | 作用 |
|------|------|
| `logger.debug()` | 调试信息 |
| `logger.info()` | 一般信息 |
| `logger.warning()` | 警告 |
| `logger.error()` | 错误 |
| `logger.exception()` | 异常（含堆栈） |

***

## 一、loguru 基础

```python
from loguru import logger

# 直接使用，无需配置
logger.debug('调试信息')
logger.info('一般信息')
logger.warning('警告信息')
logger.error('错误信息')
logger.success('成功信息')

# 异常记录
try:
    1 / 0
except Exception:
    logger.exception('发生异常')
```

### 1.1 格式化输出

```python
# 字符串格式化
logger.info('用户 {} 登录成功', username)
logger.info('处理了 {} 条记录，耗时 {:.2f}s', count, elapsed)

# f-string
logger.info(f'处理完成: {result}')

# 延迟格式化（性能更好）
logger.opt(lazy=True).info('耗时计算: {}', lambda: expensive_func())
```

### 1.2 输出到文件

```python
from loguru import logger

# 添加文件输出
logger.add('app.log')

# 带配置
logger.add(
    'app.log',
    rotation='10 MB',      # 按大小分割
    retention='7 days',    # 保留时间
    compression='zip',     # 压缩旧日志
    encoding='utf-8',
    level='DEBUG'
)

# 按时间分割
logger.add('app_{time}.log', rotation='00:00')  # 每天午夜
logger.add('app_{time}.log', rotation='1 day')  # 每天

# 多文件输出
logger.add('debug.log', level='DEBUG')
logger.add('error.log', level='ERROR')
```

***

## 二、配置详解

### 2.1 add() 参数

```python
logger.add(
    'app.log',              # 文件路径
    level='DEBUG',          # 最低级别
    format='{time} | {level} | {message}',  # 格式
    filter=lambda record: record['level'].name == 'ERROR',  # 过滤
    rotation='10 MB',       # 分割条件
    retention='7 days',     # 保留时间
    compression='zip',      # 压缩格式
    encoding='utf-8',       # 编码
    enqueue=True,           # 异步写入
    serialize=True,         # JSON 格式
    backtrace=True,         # 完整回溯
    diagnose=True,          # 显示变量值
)
```

### 2.2 rotation 分割条件

| 值 | 含义 |
|----|------|
| `'10 MB'` | 文件达到 10MB |
| `'100 KB'` | 文件达到 100KB |
| `'1 GB'` | 文件达到 1GB |
| `'00:00'` | 每天午夜 |
| `'12:00'` | 每天中午 |
| `'1 day'` | 每天 |
| `'1 week'` | 每周 |
| `'1 month'` | 每月 |
| `lambda msg: 'error' in msg` | 自定义条件 |

### 2.3 格式化占位符

| 占位符 | 含义 |
|--------|------|
| `{time}` | 时间戳 |
| `{time:YYYY-MM-DD}` | 格式化时间 |
| `{level}` | 级别 |
| `{level.icon}` | 级别图标 |
| `{message}` | 消息 |
| `{name}` | 模块名 |
| `{function}` | 函数名 |
| `{line}` | 行号 |
| `{file}` | 文件名 |
| `{module}` | 模块名 |
| `{process}` | 进程 ID |
| `{thread}` | 线程 ID |
| `{extra}` | 额外数据 |

***

## 三、高级用法

### 3.1 自定义格式

```python
from loguru import logger

# 自定义格式
fmt = '<green>{time:YYYY-MM-DD HH:mm:ss}</green> | <level>{level: <8}</level> | <cyan>{name}</cyan>:<cyan>{function}</cyan>:<cyan>{line}</cyan> - <level>{message}</level>'

logger.add('app.log', format=fmt)

# 控制台彩色输出
logger.add(sys.stderr, format=fmt, colorize=True)
```

### 3.2 过滤日志

```python
# 只记录 ERROR 及以上
logger.add('error.log', level='ERROR')

# 自定义过滤
def my_filter(record):
    return record['level'].name in ['ERROR', 'WARNING']

logger.add('warnings.log', filter=my_filter)

# 按模块过滤
logger.add('api.log', filter=lambda r: r['name'].startswith('api.'))
```

### 3.3 绑定额外数据

```python
from loguru import logger

# 绑定上下文
logger = logger.bind(user_id='123', request_id='abc')

logger.info('用户操作')  # 自动包含 user_id 和 request_id

# 使用 extra
with logger.contextualize(task='import'):
    logger.info('处理中')  # 包含 task='import'
```

### 3.4 异步日志

```python
from loguru import logger

# 异步写入（不阻塞主线程）
logger.add('app.log', enqueue=True)

# 完成后确保写入
import atexit
atexit.register(logger.complete)
```

### 3.5 JSON 格式

```python
from loguru import logger
import json

def json_sink(message):
    record = message.record
    log_entry = {
        'timestamp': record['time'].isoformat(),
        'level': record['level'].name,
        'message': record['message'],
        'module': record['module'],
        'function': record['function'],
        'line': record['line'],
    }
    with open('app.jsonl', 'a') as f:
        f.write(json.dumps(log_entry, ensure_ascii=False) + '\n')

logger.add(json_sink)
```

***

## 四、常用模板

### 4.1 项目配置

```python
from loguru import logger
import sys

def setup_logging():
    # 移除默认处理器
    logger.remove()
    
    # 控制台输出
    logger.add(
        sys.stderr,
        format='<green>{time:HH:mm:ss}</green> | <level>{level: <8}</level> | <level>{message}</level>',
        level='INFO',
        colorize=True
    )
    
    # 文件输出
    logger.add(
        'logs/app_{time:YYYY-MM-DD}.log',
        rotation='00:00',
        retention='7 days',
        compression='zip',
        level='DEBUG',
        encoding='utf-8'
    )
    
    # 错误日志单独记录
    logger.add(
        'logs/error_{time:YYYY-MM-DD}.log',
        rotation='00:00',
        retention='30 days',
        level='ERROR',
        encoding='utf-8'
    )

setup_logging()
```

### 4.2 异常装饰器

```python
from loguru import logger
from functools import wraps

def log_errors(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        try:
            return func(*args, **kwargs)
        except Exception as e:
            logger.exception(f'{func.__name__} 执行失败')
            raise
    return wrapper

@log_errors
def risky_operation():
    1 / 0
```

### 4.3 请求日志中间件

```python
from loguru import logger
from functools import wraps

def log_request(func):
    @wraps(func)
    async def wrapper(*args, **kwargs):
        logger.info(f'请求开始: {func.__name__}')
        try:
            result = await func(*args, **kwargs)
            logger.success(f'请求成功: {func.__name__}')
            return result
        except Exception as e:
            logger.exception(f'请求失败: {func.__name__}')
            raise
    return wrapper
```

### 4.4 敏感信息过滤

```python
from loguru import logger

SENSITIVE = ['password', 'token', 'secret', 'key', 'credit']

def filter_sensitive(message):
    text = str(message)
    for word in SENSITIVE:
        if word in text.lower():
            return False
    return True

logger.add('app.log', filter=filter_sensitive)
```

### 4.5 性能计时

```python
from loguru import logger
from contextlib import contextmanager

@contextmanager
def timer(name):
    logger.info(f'开始: {name}')
    try:
        yield
    finally:
        logger.info(f'完成: {name}')

# 使用
with timer('数据处理'):
    process_data()
```

***

## 五、loguru vs logging

| 特性 | loguru | logging |
|------|--------|---------|
| 开箱即用 | ✅ | ❌ 需配置 |
| 异步支持 | ✅ 内置 | ❌ 需额外配置 |
| 文件分割 | ✅ 简单 | ✅ 复杂 |
| 彩色输出 | ✅ 内置 | ❌ 需第三方 |
| 异常回溯 | ✅ 更友好 | ✅ 标准 |
| JSON 输出 | ✅ 简单 | ❌ 需自定义 |
| 性能 | ✅ enqueue | ✅ QueueHandler |

***

## 六、最佳实践

| 场景 | 建议 |
|------|------|
| 小项目 | 直接用 `logger.info()` |
| 生产环境 | 配置文件分割 + 异步写入 |
| 错误追踪 | 用 `logger.exception()` |
| 敏感信息 | 用 filter 过滤 |
| 性能敏感 | 用 `enqueue=True` |
| 日志格式 | 包含时间、级别、模块、行号 |
