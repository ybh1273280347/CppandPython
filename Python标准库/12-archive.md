# Python 压缩文件

***

## 核心速查

| 格式 | 模块 | 压缩率 | 速度 | 适用场景 |
|------|------|--------|------|---------|
| `.zip` | zipfile | 中 | 快 | 跨平台，兼容性好 |
| `.tar.gz` | tarfile | 高 | 中 | Linux，备份 |
| `.tar.bz2` | tarfile | 较高 | 较慢 | 高压缩比需求 |
| `.tar.xz` | tarfile | 最高 | 慢 | 极小体积需求 |

***

## 一、zipfile 模块

```python
import zipfile
```

### 1.1 读取 ZIP 文件

```python
# 打开 ZIP 文件
with zipfile.ZipFile('archive.zip', 'r') as zf:
    # 列出文件
    print(zf.namelist())  # ['file1.txt', 'dir/file2.txt', ...]
    
    # 获取文件信息
    for info in zf.infolist():
        print(info.filename)       # 文件名
        print(info.file_size)      # 原始大小
        print(info.compress_size)  # 压缩后大小
        print(info.date_time)      # 修改时间 (年, 月, 日, 时, 分, 秒)
    
    # 检查文件是否存在
    zf.namelist()  # 文件列表
    'file.txt' in zf.namelist()  # 检查
    
    # 检查 ZIP 完整性
    bad_file = zf.testzip()  # 返回首个损坏的文件名，或 None
```

### 1.2 读取文件内容

```python
with zipfile.ZipFile('archive.zip', 'r') as zf:
    # 读取文件内容（返回 bytes）
    content = zf.read('file.txt')
    
    # 读取文本文件
    text = zf.read('file.txt').decode('utf-8')
    
    # 使用文件对象读取
    with zf.open('file.txt') as f:
        text = f.read().decode('utf-8')
        # 或逐行读取
        for line in f:
            print(line.decode('utf-8'))
```

### 1.3 解压文件

```python
with zipfile.ZipFile('archive.zip', 'r') as zf:
    # 解压单个文件
    zf.extract('file.txt', 'destination/')
    
    # 解压到指定目录
    zf.extract('file.txt', '/path/to/dest')
    
    # 解压全部
    zf.extractall('destination/')
    
    # 解压时设置密码
    zf.extractall('destination/', pwd=b'password')
```

### 1.4 创建 ZIP 文件

```python
# 创建 ZIP 文件
with zipfile.ZipFile('archive.zip', 'w', zipfile.ZIP_DEFLATED) as zf:
    # 添加文件（保持原路径）
    zf.write('file.txt')
    
    # 添加文件（指定路径）
    zf.write('file.txt', 'renamed.txt')
    zf.write('file.txt', 'subdir/file.txt')
    
    # 添加字符串内容
    zf.writestr('hello.txt', 'Hello World!')
    zf.writestr('data.json', json.dumps(data).encode())
    
    # 添加二进制数据
    zf.writestr('binary.bin', b'\x00\x01\x02\x03')

# 压缩方式
# ZIP_STORED      无压缩
# ZIP_DEFLATED    有压缩（默认，需要 zlib）
# ZIP_BZIP2       bzip2 压缩（需要 bz2）
# ZIP_LZMA        LZMA 压缩（需要 lzma）
```

### 1.5 追加文件

```python
# 追加模式
with zipfile.ZipFile('archive.zip', 'a') as zf:
    zf.write('new_file.txt')
    zf.writestr('new_content.txt', 'New content')
```

### 1.6 压缩级别

```python
# Python 3.7+ 支持压缩级别
with zipfile.ZipFile('archive.zip', 'w', 
                     zipfile.ZIP_DEFLATED, 
                     compresslevel=9) as zf:  # 0-9，9 最高压缩率
    zf.write('large_file.dat')
```

### 1.7 ZipInfo 对象

```python
# 自定义文件信息
info = zipfile.ZipInfo('custom.txt')
info.date_time = (2024, 4, 6, 10, 30, 0)  # 修改时间
info.compress_type = zipfile.ZIP_DEFLATED
info.external_attr = 0o644 << 16  # Unix 权限

with zipfile.ZipFile('archive.zip', 'w') as zf:
    zf.writestr(info, 'Custom content')
```

***

## 二、tarfile 模块

```python
import tarfile
```

### 2.1 打开模式

| 模式 | 说明 |
|------|------|
| `'r'` | 读取（自动检测压缩方式） |
| `'r:gz'` | 读取 gzip 压缩 |
| `'r:bz2'` | 读取 bzip2 压缩 |
| `'r:xz'` | 读取 lzma 压缩 |
| `'w'` | 写入（无压缩） |
| `'w:gz'` | 写入 gzip 压缩 |
| `'w:bz2'` | 写入 bzip2 压缩 |
| `'w:xz'` | 写入 lzma 压缩 |
| `'a'` | 追加 |

### 2.2 读取 TAR 文件

```python
# 打开 TAR 文件
with tarfile.open('archive.tar.gz', 'r:gz') as tf:
    # 列出文件
    print(tf.getnames())  # ['file1.txt', 'dir/file2.txt', ...]
    
    # 获取文件信息
    for member in tf.getmembers():
        print(member.name)      # 文件名
        print(member.size)      # 大小
        print(member.mtime)     # 修改时间
        print(member.mode)      # 权限
        print(member.isdir())   # 是否是目录
        print(member.isfile())  # 是否是文件
```

### 2.3 读取文件内容

```python
with tarfile.open('archive.tar.gz', 'r:gz') as tf:
    # 获取文件对象
    member = tf.getmember('file.txt')
    f = tf.extractfile(member)
    content = f.read()
    
    # 直接通过文件名
    f = tf.extractfile('file.txt')
    if f:  # 目录返回 None
        content = f.read()
```

### 2.4 解压文件

```python
with tarfile.open('archive.tar.gz', 'r:gz') as tf:
    # 解压单个文件
    tf.extract('file.txt', 'destination/')
    
    # 解压全部
    tf.extractall('destination/')
    
    # 过滤解压
    def filter_files(member, path):
        if member.name.endswith('.txt'):
            return member
        return None
    
    tf.extractall('destination/', members=filter(filter_files, tf.getmembers()))
```

### 2.5 创建 TAR 文件

```python
# 创建 tar.gz
with tarfile.open('archive.tar.gz', 'w:gz') as tf:
    # 添加文件
    tf.add('file.txt')
    
    # 添加文件（指定路径）
    tf.add('file.txt', arcname='renamed.txt')
    
    # 添加整个目录
    tf.add('mydir', arcname='mydir')
    
    # 添加目录（排除某些文件）
    def exclude(tarinfo):
        if tarinfo.name.endswith('.pyc'):
            return None
        return tarinfo
    
    tf.add('mydir', filter=exclude)

# 压缩级别
with tarfile.open('archive.tar.gz', 'w:gz', compresslevel=9) as tf:
    tf.add('large_file.dat')
```

### 2.6 添加字符串内容

```python
import io

with tarfile.open('archive.tar.gz', 'w:gz') as tf:
    # 创建 TarInfo
    info = tarfile.TarInfo(name='hello.txt')
    info.size = len(b'Hello World!')
    
    # 添加内容
    tf.addfile(info, io.BytesIO(b'Hello World!'))
```

### 2.7 TarInfo 对象

```python
# 自定义文件信息
info = tarfile.TarInfo('custom.txt')
info.size = 100
info.mtime = time.time()
info.mode = 0o644
info.uid = 1000
info.gid = 1000
info.uname = 'user'
info.gname = 'group'

with tarfile.open('archive.tar', 'w') as tf:
    tf.addfile(info, io.BytesIO(b'x' * 100))
```

***

## 三、shutil 快捷方式

```python
import shutil

# 创建压缩包
shutil.make_archive('archive', 'zip', 'source_dir')     # archive.zip
shutil.make_archive('archive', 'gztar', 'source_dir')   # archive.tar.gz
shutil.make_archive('archive', 'bztar', 'source_dir')   # archive.tar.bz2
shutil.make_archive('archive', 'xztar', 'source_dir')   # archive.tar.xz

# 解压
shutil.unpack_archive('archive.zip', 'dest_dir')
shutil.unpack_archive('archive.tar.gz', 'dest_dir')

# 获取支持的格式
shutil.get_archive_formats()  # [('zip', 'ZIP file'), ('gztar', "gzip'ed tar-file"), ...]
shutil.get_unpack_formats()   # [('zip', ['.zip'], 'ZIP file'), ...]
```

***

## 四、常用模板

### 4.1 批量压缩

```python
from pathlib import Path

def compress_directory(source_dir, output_file, fmt='zip'):
    shutil.make_archive(
        output_file.rstrip('.zip').rstrip('.tar.gz'),
        fmt,
        source_dir
    )

def compress_files(files, output_file):
    with zipfile.ZipFile(output_file, 'w', zipfile.ZIP_DEFLATED) as zf:
        for f in files:
            path = Path(f)
            zf.write(f, path.name)

def compress_by_pattern(directory, pattern, output_file):
    with zipfile.ZipFile(output_file, 'w', zipfile.ZIP_DEFLATED) as zf:
        for f in Path(directory).rglob(pattern):
            zf.write(f, f.relative_to(directory))
```

### 4.2 批量解压

```python
def extract_all(archive_file, dest_dir):
    if archive_file.endswith('.zip'):
        with zipfile.ZipFile(archive_file, 'r') as zf:
            zf.extractall(dest_dir)
    else:
        with tarfile.open(archive_file, 'r:*') as tf:
            tf.extractall(dest_dir)

def extract_by_pattern(archive_file, dest_dir, pattern):
    with zipfile.ZipFile(archive_file, 'r') as zf:
        for name in zf.namelist():
            if Path(name).match(pattern):
                zf.extract(name, dest_dir)
```

### 4.3 加密 ZIP

```python
# 注意：Python 标准库只支持解密，不支持加密
# 需要使用 pyzipper 库

# pip install pyzipper
import pyzipper

# 创建加密 ZIP
with pyzipper.AESZipFile('secret.zip', 'w', 
                         compression=pyzipper.ZIP_DEFLATED,
                         encryption=pyzipper.WZ_AES) as zf:
    zf.setpassword(b'password')
    zf.write('secret.txt')

# 读取加密 ZIP
with pyzipper.AESZipFile('secret.zip', 'r') as zf:
    zf.setpassword(b'password')
    content = zf.read('secret.txt')
```

### 4.4 增量备份

```python
import os
from datetime import datetime

def incremental_backup(source_dir, backup_dir):
    timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
    backup_file = Path(backup_dir) / f'backup_{timestamp}.zip'
    
    with zipfile.ZipFile(backup_file, 'w', zipfile.ZIP_DEFLATED) as zf:
        for root, dirs, files in os.walk(source_dir):
            for f in files:
                full_path = os.path.join(root, f)
                arcname = os.path.relpath(full_path, source_dir)
                zf.write(full_path, arcname)
    
    return backup_file
```

### 4.5 检查压缩包完整性

```python
def check_zip_integrity(zip_file):
    try:
        with zipfile.ZipFile(zip_file, 'r') as zf:
            bad_file = zf.testzip()
            if bad_file:
                return False, f'Corrupted file: {bad_file}'
            return True, 'OK'
    except zipfile.BadZipFile:
        return False, 'Invalid ZIP file'
    except Exception as e:
        return False, str(e)

def check_tar_integrity(tar_file):
    try:
        with tarfile.open(tar_file, 'r:*') as tf:
            tf.getmembers()
        return True, 'OK'
    except Exception as e:
        return False, str(e)
```

### 4.6 获取压缩包信息

```python
def get_archive_info(archive_file):
    info = {'files': [], 'total_size': 0, 'compressed_size': 0}
    
    if archive_file.endswith('.zip'):
        with zipfile.ZipFile(archive_file, 'r') as zf:
            for f in zf.infolist():
                info['files'].append({
                    'name': f.filename,
                    'size': f.file_size,
                    'compressed': f.compress_size,
                    'date': f.date_time
                })
                info['total_size'] += f.file_size
                info['compressed_size'] += f.compress_size
    else:
        with tarfile.open(archive_file, 'r:*') as tf:
            for m in tf.getmembers():
                info['files'].append({
                    'name': m.name,
                    'size': m.size,
                    'is_dir': m.isdir()
                })
                info['total_size'] += m.size
    
    info['compression_ratio'] = (
        info['compressed_size'] / info['total_size'] 
        if info['total_size'] > 0 else 0
    )
    
    return info
```
