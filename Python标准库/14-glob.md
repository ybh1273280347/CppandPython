# Python glob 文件模式匹配

***

## 核心速查

| 函数 | 作用 | 返回类型 |
|------|------|---------|
| `glob(pattern)` | 匹配文件路径 | `list` |
| `iglob(pattern)` | 匹配文件路径（迭代器） | `iterator` |
| `glob(pattern, recursive=True)` | 递归匹配 `**` | `list` |

***

## 一、基本用法

```python
import glob

glob.glob('*.py')
glob.glob('data/*.csv')
glob.glob('src/**/*.py', recursive=True)
```

### 通配符说明

| 通配符 | 作用 | 示例 |
|--------|------|------|
| `*` | 匹配任意字符（不含 `/`） | `*.py` |
| `**` | 匹配任意目录（需 `recursive=True`） | `**/*.py` |
| `?` | 匹配单个字符 | `file?.txt` |
| `[abc]` | 匹配指定字符 | `file[123].txt` |
| `[!abc]` | 排除指定字符 | `file[!0-9].txt` |

***

## 二、常用模式

### 当前目录

```python
import glob

py_files = glob.glob('*.py')
all_files = glob.glob('*')
dirs_only = [f for f in glob.glob('*') if Path(f).is_dir()]
```

### 子目录

```python
import glob

csv_files = glob.glob('data/*.csv')
nested = glob.glob('data/*/*.csv')
all_py = glob.glob('src/**/*.py', recursive=True)
```

### 多种扩展名

```python
import glob

images = glob.glob('*.png') + glob.glob('*.jpg') + glob.glob('*.gif')

images = []
for ext in ['png', 'jpg', 'gif']:
    images.extend(glob.glob(f'*.{ext}'))
```

***

## 三、iglob - 迭代器版本

```python
import glob

for f in glob.iglob('**/*.py', recursive=True):
    print(f)

large_files = (f for f in glob.iglob('**/*', recursive=True) 
               if Path(f).stat().st_size > 1024 * 1024)
```

***

## 四、样板代码

### 查找所有 Python 文件

```python
import glob
from pathlib import Path

def find_py_files(root='.'):
    return glob.glob(f'{root}/**/*.py', recursive=True)

for f in find_py_files('src'):
    print(f)
```

### 按修改时间排序

```python
import glob
from pathlib import Path

files = glob.glob('*.log')
files.sort(key=lambda f: Path(f).stat().st_mtime, reverse=True)

latest = files[0] if files else None
```

### 批量处理文件

```python
import glob
import shutil
from pathlib import Path

def archive_logs():
    for log_file in glob.glob('logs/*.log'):
        Path(log_file).rename(f'archive/{Path(log_file).name}')
```

### 查找空目录

```python
import glob
from pathlib import Path

empty_dirs = [d for d in glob.glob('**/', recursive=True) 
              if not any(Path(d).iterdir())]
```

***

## 五、glob vs pathlib.Path.glob

| 特性 | `glob.glob()` | `Path.glob()` |
|------|---------------|---------------|
| 返回类型 | 字符串列表 | Path 迭代器 |
| 递归语法 | `**` + `recursive=True` | `**` 自动递归 |
| 链式操作 | 不支持 | 支持 `.glob().glob()` |

```python
import glob
from pathlib import Path

files1 = glob.glob('src/**/*.py', recursive=True)

files2 = list(Path('src').rglob('*.py'))

for f in Path('src').glob('**/*.py'):
    print(f.read_text())
```

***

## 六、最佳实践

| 场景 | 建议 |
|------|------|
| 简单匹配 | 用 `glob.glob()` |
| 大量文件 | 用 `iglob()` 或 `Path.glob()` |
| 需要路径操作 | 用 `Path.glob()` |
| 复杂过滤 | 用 `Path.glob()` + 列表推导 |
| 性能敏感 | `iglob()` 延迟求值 |

***

## 七、常见陷阱

### 不匹配隐藏文件

```python
import glob

glob.glob('*')

glob.glob('[!.]*')
```

### 路径分隔符

```python
import glob
from pathlib import Path

glob.glob('data/*.csv')

glob.glob('data' + '/' + '*.csv')
```

### 空结果

```python
import glob

files = glob.glob('*.xyz')
if not files:
    print('没有匹配的文件')
```
