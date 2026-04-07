# Python gzip 压缩文件

***

## 核心速查

| 函数 | 作用 |
|------|------|
| `gzip.open()` | 打开 gzip 文件 |
| `gzip.compress()` | 压缩字节 |
| `gzip.decompress()` | 解压字节 |

***

## 一、文件读写

### 文本模式

```python
import gzip

with gzip.open('file.txt.gz', 'wt', encoding='utf-8') as f:
    f.write('Hello, World!\n')
    f.write('多行文本\n')

with gzip.open('file.txt.gz', 'rt', encoding='utf-8') as f:
    content = f.read()
    print(content)

with gzip.open('file.txt.gz', 'rt', encoding='utf-8') as f:
    for line in f:
        print(line.strip())
```

### 二进制模式

```python
import gzip

with gzip.open('data.bin.gz', 'wb') as f:
    f.write(b'\x00\x01\x02\x03')

with gzip.open('data.bin.gz', 'rb') as f:
    data = f.read()
```

### 压缩级别

```python
import gzip

with gzip.open('file.txt.gz', 'wb', compresslevel=9) as f:
    f.write(b'data')

with gzip.open('file.txt.gz', 'wb', compresslevel=1) as f:
    f.write(b'data')
```

***

## 二、内存压缩

### compress / decompress

```python
import gzip

data = b'Hello, World! ' * 1000

compressed = gzip.compress(data)
print(f'原始: {len(data)}, 压缩后: {len(compressed)}')

decompressed = gzip.decompress(compressed)
assert data == decompressed
```

### 压缩级别

```python
import gzip

data = b'Hello, World! ' * 1000

for level in range(1, 10):
    compressed = gzip.compress(data, compresslevel=level)
    ratio = len(compressed) / len(data) * 100
    print(f'级别 {level}: {len(compressed)} 字节 ({ratio:.1f}%)')
```

***

## 三、样板代码

### 压缩普通文件

```python
import gzip
import shutil
from pathlib import Path

def compress_file(input_path: str, output_path: str = None):
    input_path = Path(input_path)
    output_path = output_path or f'{input_path}.gz'
    
    with open(input_path, 'rb') as f_in:
        with gzip.open(output_path, 'wb') as f_out:
            shutil.copyfileobj(f_in, f_out)
    
    return output_path

compress_file('large_file.txt')
```

### 解压 gzip 文件

```python
import gzip
import shutil
from pathlib import Path

def decompress_file(input_path: str, output_path: str = None):
    input_path = Path(input_path)
    output_path = output_path or str(input_path).rstrip('.gz')
    
    with gzip.open(input_path, 'rb') as f_in:
        with open(output_path, 'wb') as f_out:
            shutil.copyfileobj(f_in, f_out)
    
    return output_path

decompress_file('large_file.txt.gz')
```

### 读取远程 gzip 文件

```python
import gzip
import httpx

def read_remote_gzip(url: str):
    response = httpx.get(url)
    response.raise_for_status()
    
    return gzip.decompress(response.content).decode('utf-8')

content = read_remote_gzip('https://example.com/data.txt.gz')
```

### JSON 数据压缩存储

```python
import gzip
import json

def save_json_gz(data: dict, path: str):
    with gzip.open(path, 'wt', encoding='utf-8') as f:
        json.dump(data, f, ensure_ascii=False, indent=2)

def load_json_gz(path: str) -> dict:
    with gzip.open(path, 'rt', encoding='utf-8') as f:
        return json.load(f)

save_json_gz({'key': 'value' * 100}, 'data.json.gz')
data = load_json_gz('data.json.gz')
```

***

## 四、与 zipfile 对比

| 特性 | gzip | zipfile |
|------|------|---------|
| 压缩对象 | 单个文件 | 多个文件 |
| 归档功能 | 无 | 有 |
| 压缩算法 | DEFLATE | 多种 |
| 使用场景 | 单文件压缩 | 打包多个文件 |

***

## 五、最佳实践

| 场景 | 建议 |
|------|------|
| 文本日志 | 用 `gzip.open('wt')` |
| 大文件 | 用 `shutil.copyfileobj` 流式处理 |
| 内存数据 | 用 `compress/decompress` |
| JSON 存储 | 压缩后体积减少 70%+ |
| 压缩级别 | 默认 9，追求速度用 1-6 |

***

## 六、常见问题

### 中文文件名

```python
import gzip

with gzip.open('文件.txt.gz', 'wt', encoding='utf-8') as f:
    f.write('中文内容')
```

### 追加模式

```python
import gzip
from pathlib import Path

def append_gz(path: str, content: str):
    existing = ''
    if Path(path).exists():
        with gzip.open(path, 'rt', encoding='utf-8') as f:
            existing = f.read()
    
    with gzip.open(path, 'wt', encoding='utf-8') as f:
        f.write(existing + content)
```

### 检查是否为 gzip 文件

```python
import gzip

def is_gzip_file(path: str) -> bool:
    try:
        with gzip.open(path, 'rb') as f:
            f.read(1)
        return True
    except gzip.BadGzipFile:
        return False
```
