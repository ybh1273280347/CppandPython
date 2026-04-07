# Python 文件操作

***

## 核心速查

| 模块 | 作用 | 推荐度 |
|------|------|--------|
| `pathlib` | 面向对象路径操作 | ⭐⭐⭐⭐⭐ |
| `os` | 系统级文件操作 | ⭐⭐⭐⭐ |
| `shutil` | 高级文件操作 | ⭐⭐⭐⭐ |

***

## 一、pathlib 模块

```python
from pathlib import Path
```

### 1.1 路径创建

```python
p = Path('a/b/c.txt')            # 相对路径
p = Path('/home/user/file.txt')  # 绝对路径
p = Path('~') / 'file.txt'       # 使用 / 拼接（自动展开 ~）
Path.cwd()                        # 当前工作目录
Path.home()                       # 用户主目录
Path(__file__).parent             # 当前文件所在目录
```

### 1.2 路径组件

| 属性 | 作用 | 示例 |
|------|------|------|
| `p.name` | 文件名 | `'file.txt'` |
| `p.stem` | 不含扩展名 | `'file'` |
| `p.suffix` | 扩展名 | `'.txt'` |
| `p.suffixes` | 多个扩展名 | `['.tar', '.gz']` |
| `p.parent` | 父目录 | `Path('/home/user')` |
| `p.parts` | 路径组件元组 | `('/', 'home', 'user', 'file.txt')` |

```python
p = Path('/home/user/doc/file.txt')

p.parts        # ('/', 'home', 'user', 'doc', 'file.txt')
p.name         # 'file.txt'
p.stem         # 'file'
p.suffix       # '.txt'
p.parent       # Path('/home/user/doc')
p.parents[0]   # Path('/home/user/doc')
p.parents[1]   # Path('/home/user')
```

### 1.3 路径判断

```python
p.exists()       # 路径是否存在
p.is_file()      # 是否是文件
p.is_dir()       # 是否是目录
p.is_symlink()   # 是否是符号链接
p.is_absolute()  # 是否是绝对路径
```

### 1.4 文件操作

```python
# 读写
p.read_text()              # 读取文本
p.write_text('content')    # 写入文本
p.read_bytes()             # 读取二进制
p.write_bytes(data)        # 写入二进制

# 创建/删除
p.mkdir(parents=True, exist_ok=True)  # 创建目录
p.touch()                              # 创建空文件/更新 mtime
p.unlink()                             # 删除文件
p.rename('new.txt')                    # 重命名
p.replace('new.txt')                   # 替换（原子操作）

# 属性
p.stat()       # 文件信息（size, mtime 等）
p.stat().st_size    # 文件大小
p.stat().st_mtime   # 修改时间
```

### 1.5 目录遍历

```python
# 列出子项
list(p.iterdir())           # 迭代目录内容

# glob 模式匹配
list(p.glob('*.txt'))       # 当前目录下的 .txt 文件
list(p.rglob('*.txt'))      # 递归匹配（等价于 **/*.txt）
list(p.glob('**/*.py'))     # 所有 Python 文件

# 过滤文件
[x for x in p.iterdir() if x.is_file()]
[x for x in p.iterdir() if x.suffix == '.py']
```

### 1.6 路径转换

```python
p = Path('~/doc/file.txt')

p.expanduser()    # 展开用户目录
p.resolve()       # 解析为绝对路径（解析符号链接）
p.absolute()      # 绝对路径（不解析符号链接）

# 相对路径
p = Path('/home/user/doc')
q = Path('/home/user/doc/file.txt')
q.relative_to(p)  # Path('file.txt')
```

### 1.7 常用模板

```python
# 遍历所有 Python 文件
for py_file in Path('.').rglob('*.py'):
    print(py_file)

# 安全创建目录
Path('a/b/c').mkdir(parents=True, exist_ok=True)

# 读取配置文件
config_path = Path(__file__).parent / 'config.json'
config = json.loads(config_path.read_text())

# 批量重命名
for f in Path('.').glob('*.txt'):
    f.rename(f.with_suffix('.md'))

# 计算目录大小
def dir_size(path):
    return sum(f.stat().st_size for f in Path(path).rglob('*') if f.is_file())
```

***

## 二、os 模块

```python
import os
```

### 2.1 路径操作

| 函数 | 作用 | 示例 |
|------|------|------|
| `os.getcwd()` | 当前工作目录 | `os.getcwd()` |
| `os.chdir(path)` | 切换目录 | `os.chdir('/tmp')` |
| `os.path.abspath(path)` | 绝对路径 | `os.path.abspath('.')` |
| `os.path.exists(path)` | 路径是否存在 | `os.path.exists('file.txt')` |
| `os.path.isfile(path)` | 是否是文件 | `os.path.isfile('file.txt')` |
| `os.path.isdir(path)` | 是否是目录 | `os.path.isdir('folder')` |
| `os.path.join(*paths)` | 拼接路径 | `os.path.join('a', 'b', 'c')` |
| `os.path.basename(path)` | 获取文件名 | `os.path.basename('/a/b.txt')` → `'b.txt'` |
| `os.path.dirname(path)` | 获取目录名 | `os.path.dirname('/a/b.txt')` → `'/a'` |
| `os.path.splitext(path)` | 分离扩展名 | `os.path.splitext('a.txt')` → `('a', '.txt')` |
| `os.path.split(path)` | 分离目录和文件名 | `os.path.split('/a/b.txt')` → `('/a', 'b.txt')` |

### 2.2 文件操作

```python
os.remove(path)       # 删除文件
os.rmdir(path)        # 删除空目录
os.mkdir(path)        # 创建目录
os.makedirs(path)     # 递归创建目录
os.rename(src, dst)   # 重命名/移动
os.replace(src, dst)  # 替换（原子操作）
```

### 2.3 目录遍历

```python
# 列出目录内容
os.listdir('.')  # ['file1.txt', 'dir1', ...]

# 递归遍历
for root, dirs, files in os.walk('/path/to/dir'):
    for f in files:
        print(os.path.join(root, f))
    for d in dirs:
        print(os.path.join(root, d))

# 筛选
[x for x in os.listdir('.') if os.path.isfile(x)]
[x for x in os.listdir('.') if x.endswith('.py')]
```

### 2.4 环境变量

```python
os.environ               # 所有环境变量
os.environ.get('PATH')   # 获取指定环境变量
os.getenv('HOME', default)  # 带默认值
os.environ['MY_VAR'] = 'value'  # 设置环境变量
```

### 2.5 文件属性

```python
os.path.getsize('file.txt')    # 文件大小（字节）
os.path.getmtime('file.txt')   # 修改时间（时间戳）
os.path.getatime('file.txt')   # 访问时间
os.path.getctime('file.txt')   # 创建时间（Windows）
```

### 2.6 os vs pathlib 对比

| 操作 | os | pathlib |
|------|-----|---------|
| 当前目录 | `os.getcwd()` | `Path.cwd()` |
| 用户目录 | `os.path.expanduser('~')` | `Path.home()` |
| 路径拼接 | `os.path.join('a', 'b')` | `Path('a') / 'b'` |
| 文件名 | `os.path.basename(p)` | `Path(p).name` |
| 扩展名 | `os.path.splitext(p)[1]` | `Path(p).suffix` |
| 是否存在 | `os.path.exists(p)` | `Path(p).exists()` |
| 遍历目录 | `os.listdir(p)` | `Path(p).iterdir()` |
| glob | `glob.glob(p)` | `Path(p).glob('*')` |

**推荐**：新代码优先使用 `pathlib`，更直观、更安全。

***

## 三、shutil 模块

```python
import shutil
```

### 3.1 文件复制

```python
shutil.copy('src.txt', 'dst.txt')        # 复制文件内容+权限
shutil.copy2('src.txt', 'dst.txt')       # 复制文件内容+权限+元数据
shutil.copyfile('src.txt', 'dst.txt')    # 仅复制内容
shutil.copymode('src.txt', 'dst.txt')    # 仅复制权限
shutil.copystat('src.txt', 'dst.txt')    # 仅复制元数据
```

### 3.2 目录操作

```python
shutil.copytree('src_dir', 'dst_dir')    # 递归复制目录
shutil.rmtree('dir')                     # 递归删除目录
shutil.move('src', 'dst')                # 移动/重命名
```

### 3.3 压缩文件

```python
# 创建压缩包
shutil.make_archive('archive', 'zip', 'source_dir')     # .zip
shutil.make_archive('archive', 'gztar', 'source_dir')   # .tar.gz
shutil.make_archive('archive', 'bztar', 'source_dir')   # .tar.bz2
shutil.make_archive('archive', 'xztar', 'source_dir')   # .tar.xz

# 解压
shutil.unpack_archive('archive.zip', 'dest_dir')
shutil.unpack_archive('archive.tar.gz', 'dest_dir')
```

### 3.4 磁盘操作

```python
shutil.disk_usage('/')     # 磁盘使用情况
# usage(total=107374182400, used=53687091200, free=53687091200)

shutil.which('python')     # 查找可执行文件路径
```

***

## 四、常用模板

### 4.1 安全读写文件

```python
def safe_write(path, content):
    p = Path(path)
    p.parent.mkdir(parents=True, exist_ok=True)
    p.write_text(content, encoding='utf-8')

def safe_read(path, default=''):
    p = Path(path)
    return p.read_text(encoding='utf-8') if p.exists() else default
```

### 4.2 遍历文件

```python
def find_files(directory, pattern='*'):
    return list(Path(directory).rglob(pattern))

def find_by_ext(directory, extensions):
    exts = set(extensions)
    return [f for f in Path(directory).rglob('*') 
            if f.suffix in exts]
```

### 4.3 清理临时文件

```python
def clean_temp(directory, days=7):
    from datetime import datetime, timedelta
    cutoff = datetime.now() - timedelta(days=days)
    
    for f in Path(directory).rglob('*'):
        if f.is_file():
            mtime = datetime.fromtimestamp(f.stat().st_mtime)
            if mtime < cutoff:
                f.unlink()
```

### 4.4 目录大小统计

```python
def get_dir_size(path):
    return sum(f.stat().st_size 
               for f in Path(path).rglob('*') 
               if f.is_file())

def format_size(size):
    for unit in ['B', 'KB', 'MB', 'GB', 'TB']:
        if size < 1024:
            return f'{size:.1f} {unit}'
        size /= 1024
    return f'{size:.1f} PB'
```

### 4.5 批量文件操作

```python
def batch_rename(directory, old_ext, new_ext):
    for f in Path(directory).glob(f'*{old_ext}'):
        f.rename(f.with_suffix(new_ext))

def copy_with_structure(src, dst, pattern='*'):
    src, dst = Path(src), Path(dst)
    for f in src.rglob(pattern):
        rel = f.relative_to(src)
        dest = dst / rel
        dest.parent.mkdir(parents=True, exist_ok=True)
        shutil.copy2(f, dest)
```
