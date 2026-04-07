# Python JSON 处理

***

## 核心速查

| 函数 | 作用 |
|------|------|
| `json.dumps(obj)` | Python 对象 → JSON 字符串 |
| `json.loads(s)` | JSON 字符串 → Python 对象 |
| `json.dump(obj, f)` | Python 对象 → JSON 文件 |
| `json.load(f)` | JSON 文件 → Python 对象 |

***

## 一、基础操作

### 1.1 序列化（Python → JSON）

```python
import json

data = {
    'name': 'Alice',
    'age': 30,
    'skills': ['Python', 'C++'],
    'active': True,
    'balance': None
}

# 转为字符串
json_str = json.dumps(data)
# '{"name": "Alice", "age": 30, "skills": ["Python", "C++"], "active": true, "balance": null}'

# 格式化输出
json_str = json.dumps(data, indent=2)
# {
#   "name": "Alice",
#   "age": 30,
#   ...
# }

# 保留中文
json_str = json.dumps(data, ensure_ascii=False)
# '{"name": "张三", "city": "北京"}'

# 键排序
json_str = json.dumps(data, sort_keys=True)

# 紧凑输出
json_str = json.dumps(data, separators=(',', ':'))
```

### 1.2 反序列化（JSON → Python）

```python
# 从字符串解析
json_str = '{"name": "Alice", "age": 30}'
data = json.loads(json_str)
# {'name': 'Alice', 'age': 30}

# 从字节解析
json_bytes = b'{"name": "Alice"}'
data = json.loads(json_bytes)

# 从文件读取
with open('data.json', 'r', encoding='utf-8') as f:
    data = json.load(f)
```

### 1.3 文件操作

```python
# 写入文件
data = {'name': '张三', 'age': 30}
with open('data.json', 'w', encoding='utf-8') as f:
    json.dump(data, f, ensure_ascii=False, indent=2)

# 读取文件
with open('data.json', 'r', encoding='utf-8') as f:
    data = json.load(f)
```

***

## 二、常用参数

### 2.1 dumps 参数

| 参数 | 作用 | 示例 |
|------|------|------|
| `indent` | 格式化缩进 | `indent=2` |
| `ensure_ascii` | 是否转义非 ASCII | `ensure_ascii=False` |
| `sort_keys` | 键排序 | `sort_keys=True` |
| `separators` | 分隔符 | `separators=(',', ':')` |
| `skipkeys` | 跳过非字符串键 | `skipkeys=True` |
| `default` | 自定义序列化 | `default=str` |

```python
# 完整参数示例
json.dumps(
    data,
    indent=2,              # 缩进 2 空格
    ensure_ascii=False,    # 保留中文
    sort_keys=True,        # 键排序
    separators=(',', ': ') # 自定义分隔符
)
```

### 2.2 loads 参数

| 参数 | 作用 | 示例 |
|------|------|------|
| `object_hook` | 对象转换函数 | `object_hook=as_date` |
| `object_pairs_hook` | 有序键值对转换 | `object_pairs_hook=OrderedDict` |
| `parse_float` | 浮点数解析 | `parse_float=Decimal` |
| `parse_int` | 整数解析 | `parse_int=int` |

```python
from collections import OrderedDict
from decimal import Decimal

# 保持键顺序
data = json.loads(json_str, object_pairs_hook=OrderedDict)

# 使用 Decimal 处理精确小数
data = json.loads('{"price": 19.99}', parse_float=Decimal)
# {'price': Decimal('19.99')}
```

***

## 三、类型映射

### 3.1 Python → JSON

| Python 类型 | JSON 类型 |
|------------|-----------|
| `dict` | object |
| `list`, `tuple` | array |
| `str` | string |
| `int`, `float` | number |
| `True` | true |
| `False` | false |
| `None` | null |

### 3.2 JSON → Python

| JSON 类型 | Python 类型 |
|-----------|------------|
| object | `dict` |
| array | `list` |
| string | `str` |
| number (int) | `int` |
| number (float) | `float` |
| true | `True` |
| false | `False` |
| null | `None` |

***

## 四、自定义序列化

### 4.1 default 参数

```python
from datetime import datetime, date

class User:
    def __init__(self, name, created_at):
        self.name = name
        self.created_at = created_at

def default_handler(obj):
    if isinstance(obj, datetime):
        return obj.isoformat()
    if isinstance(obj, date):
        return obj.isoformat()
    if isinstance(obj, User):
        return obj.__dict__
    raise TypeError(f'Object of type {type(obj)} is not JSON serializable')

user = User('Alice', datetime.now())
json.dumps(user, default=default_handler)
```

### 4.2 JSONEncoder 子类

```python
from datetime import datetime, date
import json

class CustomEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, datetime):
            return obj.isoformat()
        if isinstance(obj, date):
            return obj.isoformat()
        if isinstance(obj, set):
            return list(obj)
        if hasattr(obj, '__dict__'):
            return obj.__dict__
        return super().default(obj)

# 使用
data = {'time': datetime.now(), 'tags': {1, 2, 3}}
json.dumps(data, cls=CustomEncoder)
```

### 4.3 自定义反序列化

```python
from datetime import datetime

def object_hook(obj):
    for key, value in obj.items():
        if key.endswith('_at') and isinstance(value, str):
            try:
                obj[key] = datetime.fromisoformat(value)
            except ValueError:
                pass
    return obj

json_str = '{"created_at": "2024-04-06T10:30:00"}'
data = json.loads(json_str, object_hook=object_hook)
# {'created_at': datetime(2024, 4, 6, 10, 30, 0)}
```

***

## 五、特殊场景

### 5.1 中文处理

```python
data = {'name': '张三', 'city': '北京'}

# 正确方式：保留中文
json.dumps(data, ensure_ascii=False)
# '{"name": "张三", "city": "北京"}'

# 错误方式：中文变 Unicode 转义
json.dumps(data)
# '{"name": "\\u5f20\\u4e09", "city": "\\u5317\\u4eac"}'
```

### 5.2 流式处理（大文件）

```python
# 写入 JSONL（每行一个 JSON）
with open('data.jsonl', 'w', encoding='utf-8') as f:
    for item in data_list:
        f.write(json.dumps(item, ensure_ascii=False) + '\n')

# 读取 JSONL
with open('data.jsonl', 'r', encoding='utf-8') as f:
    for line in f:
        item = json.loads(line)
        process(item)

# 流式写入大数组
with open('large.json', 'w') as f:
    f.write('[')
    for i, item in enumerate(data_list):
        if i > 0:
            f.write(',')
        json.dump(item, f)
    f.write(']')
```

### 5.3 美化输出

```python
def pretty_json(data):
    return json.dumps(
        data,
        indent=2,
        ensure_ascii=False,
        sort_keys=True
    )

print(pretty_json({'name': '张三', 'age': 30}))
```

### 5.4 合并 JSON

```python
def merge_json(*dicts):
    result = {}
    for d in dicts:
        result.update(d)
    return result

# 深度合并
def deep_merge(a, b):
    result = a.copy()
    for key, value in b.items():
        if key in result and isinstance(result[key], dict) and isinstance(value, dict):
            result[key] = deep_merge(result[key], value)
        else:
            result[key] = value
    return result
```

***

## 六、常用模板

### 6.1 配置文件读写

```python
from pathlib import Path
import json

class Config:
    def __init__(self, path):
        self.path = Path(path)
        self.data = self.load()
    
    def load(self):
        if self.path.exists():
            return json.loads(self.path.read_text(encoding='utf-8'))
        return {}
    
    def save(self):
        self.path.write_text(
            json.dumps(self.data, ensure_ascii=False, indent=2),
            encoding='utf-8'
        )
    
    def get(self, key, default=None):
        return self.data.get(key, default)
    
    def set(self, key, value):
        self.data[key] = value
        self.save()

# 使用
config = Config('config.json')
config.set('theme', 'dark')
print(config.get('theme'))
```

### 6.2 API 响应处理

```python
def safe_json_parse(response, default=None):
    try:
        return response.json()
    except ValueError:
        return default

def extract_fields(data, *keys, default=None):
    for key in keys:
        if isinstance(data, dict) and key in data:
            data = data[key]
        else:
            return default
    return data

# 使用
response = requests.get('https://api.example.com/data')
data = safe_json_parse(response, {})
name = extract_fields(data, 'user', 'profile', 'name', default='Unknown')
```

### 6.3 JSON 验证

```python
def validate_json(json_str, required_fields):
    try:
        data = json.loads(json_str)
    except json.JSONDecodeError:
        return False, 'Invalid JSON'
    
    missing = [f for f in required_fields if f not in data]
    if missing:
        return False, f'Missing fields: {missing}'
    
    return True, data
```

### 6.4 JSON Diff

```python
def json_diff(a, b, path=''):
    diffs = []
    
    if type(a) != type(b):
        diffs.append(f'{path}: type changed from {type(a).__name__} to {type(b).__name__}')
        return diffs
    
    if isinstance(a, dict):
        all_keys = set(a.keys()) | set(b.keys())
        for key in all_keys:
            new_path = f'{path}.{key}' if path else key
            if key not in a:
                diffs.append(f'{new_path}: added')
            elif key not in b:
                diffs.append(f'{new_path}: removed')
            else:
                diffs.extend(json_diff(a[key], b[key], new_path))
    elif isinstance(a, list):
        if len(a) != len(b):
            diffs.append(f'{path}: list length changed from {len(a)} to {len(b)}')
        else:
            for i, (x, y) in enumerate(zip(a, b)):
                diffs.extend(json_diff(x, y, f'{path}[{i}]'))
    elif a != b:
        diffs.append(f'{path}: changed from {a!r} to {b!r}')
    
    return diffs
```
