# Python tempfile 临时文件

***

## 核心速查

| 函数 | 作用 | 自动清理 |
|------|------|---------|
| `NamedTemporaryFile` | 创建临时文件 | 关闭时删除 |
| `TemporaryDirectory` | 创建临时目录 | 退出 with 时删除 |
| `mkstemp` | 创建临时文件（低级） | 需手动删除 |
| `mkdtemp` | 创建临时目录（低级） | 需手动删除 |
| `gettempdir` | 获取临时目录路径 | - |

***

## 一、NamedTemporaryFile - 临时文件

### 基本用法

```python
import tempfile

with tempfile.NamedTemporaryFile(mode='w', suffix='.txt', delete=True) as f:
    f.write('临时数据')
    f.flush()
    print(f.name)

with tempfile.NamedTemporaryFile(mode='w+b', suffix='.json', delete=False) as f:
    f.write(b'{"key": "value"}')
    temp_path = f.name

import os
os.unlink(temp_path)
```

### 常用参数

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `mode` | `'w+b'` | 文件模式 |
| `suffix` | `None` | 文件后缀 |
| `prefix` | `None` | 文件前缀 |
| `dir` | `None` | 指定目录 |
| `delete` | `True` | 关闭时是否删除 |

### 注意事项

- Windows 上 `delete=True` 时，文件被占用无法再次打开
- 需要 `delete=False` 才能在 with 外访问文件

***

## 二、TemporaryDirectory - 临时目录

```python
import tempfile
import os

with tempfile.TemporaryDirectory(prefix='myapp_') as tmpdir:
    file_path = os.path.join(tmpdir, 'data.txt')
    with open(file_path, 'w') as f:
        f.write('临时文件内容')
    print(f"临时目录: {tmpdir}")

print(os.path.exists(tmpdir))
```

### 常用参数

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `suffix` | `None` | 目录后缀 |
| `prefix` | `tmp` | 目录前缀 |
| `dir` | `None` | 父目录 |
| `ignore_cleanup_errors` | `False` | 忽略清理错误 |

***

## 三、样板代码

### 测试文件处理函数

```python
import tempfile
import os

def test_file_processor():
    with tempfile.NamedTemporaryFile(mode='w', suffix='.csv', delete=False) as f:
        f.write('name,score\nAlice,90\nBob,85\n')
        temp_path = f.name
    
    try:
        result = process_csv(temp_path)
        assert len(result) == 2
    finally:
        os.unlink(temp_path)

def process_csv(path):
    with open(path) as f:
        return f.readlines()[1:]
```

### 缓存大数据到临时文件

```python
import tempfile
import pickle

def save_to_temp(data):
    with tempfile.NamedTemporaryFile(mode='wb', suffix='.pkl', delete=False) as f:
        pickle.dump(data, f)
        return f.name

def load_from_temp(path):
    with open(path, 'rb') as f:
        data = pickle.load(f)
    os.unlink(path)
    return data

temp_path = save_to_temp({'large': 'data'})
data = load_from_temp(temp_path)
```

### 临时工作目录

```python
import tempfile
import os
from pathlib import Path

def process_files():
    with tempfile.TemporaryDirectory() as tmpdir:
        work_dir = Path(tmpdir)
        
        input_file = work_dir / 'input.txt'
        input_file.write_text('原始数据')
        
        output_file = work_dir / 'output.txt'
        output_file.write_text(input_file.read_text().upper())
        
        return output_file.read_text()
```

### 安全的临时文件操作

```python
import tempfile
import os

def atomic_write(path, content):
    with tempfile.NamedTemporaryFile(
        mode='w',
        dir=os.path.dirname(path),
        prefix='.tmp_',
        delete=False
    ) as f:
        f.write(content)
        temp_path = f.name
    
    os.replace(temp_path, path)

atomic_write('config.json', '{"key": "value"}')
```

***

## 四、最佳实践

| 场景 | 建议 |
|------|------|
| 测试代码 | 用 `TemporaryDirectory` + `with` |
| 大文件缓存 | `delete=False`，用完手动删除 |
| 敏感数据 | 用完立即删除，设置权限 |
| 跨进程共享 | `delete=False`，传递文件路径 |
| 原子写入 | 先写临时文件，再 `os.replace` |

***

## 五、与 pathlib 结合

```python
import tempfile
from pathlib import Path

with tempfile.TemporaryDirectory() as tmpdir:
    tmp = Path(tmpdir)
    
    (tmp / 'data').mkdir()
    (tmp / 'data' / 'file.txt').write_text('内容')
    
    for f in tmp.rglob('*.txt'):
        print(f.read_text())
```
