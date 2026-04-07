# Python sys 系统参数

***

## 核心速查

| 属性/函数 | 作用 |
|-----------|------|
| `sys.argv` | 命令行参数列表 |
| `sys.path` | 模块搜索路径 |
| `sys.version` | Python 版本信息 |
| `sys.exit(code)` | 退出程序 |
| `sys.stdin/stdout/stderr` | 标准输入/输出/错误 |
| `sys.getsizeof(obj)` | 对象内存大小 |
| `sys.modules` | 已加载模块字典 |

***

## 一、命令行参数

### sys.argv

```python
import sys

print(sys.argv[0])
print(sys.argv[1:])

if len(sys.argv) < 2:
    print(f"用法: python {sys.argv[0]} <参数>")
    sys.exit(1)
```

### argparse 替代方案

```python
import argparse

parser = argparse.ArgumentParser(description='处理文件')
parser.add_argument('input', help='输入文件')
parser.add_argument('-o', '--output', help='输出文件')
parser.add_argument('-v', '--verbose', action='store_true')

args = parser.parse_args()
print(args.input, args.output, args.verbose)
```

***

## 二、模块路径

### sys.path

```python
import sys

print(sys.path)

sys.path.insert(0, '/custom/path')

sys.path.append('/another/path')
```

***

## 三、标准流

### 基本用法

```python
import sys

sys.stdout.write('标准输出\n')
sys.stderr.write('错误输出\n')

line = sys.stdin.readline()

for line in sys.stdin:
    print(line.strip())
```

### 重定向

```python
import sys
from io import StringIO

old_stdout = sys.stdout
sys.stdout = StringIO()

print('这行被捕获')

output = sys.stdout.getvalue()
sys.stdout = old_stdout
print(f'捕获的内容: {output}')
```

***

## 四、内存与性能

### getsizeof

```python
import sys

print(sys.getsizeof([]))
print(sys.getsizeof([1, 2, 3]))
print(sys.getsizeof({}))
print(sys.getsizeof('hello'))

def get_size(obj, seen=None):
    seen = seen or set()
    if id(obj) in seen:
        return 0
    seen.add(id(obj))
    size = sys.getsizeof(obj)
    if isinstance(obj, dict):
        size += sum(get_size(k, seen) + get_size(v, seen) for k, v in obj.items())
    elif hasattr(obj, '__iter__') and not isinstance(obj, str):
        size += sum(get_size(item, seen) for item in obj)
    return size
```

***

## 五、样板代码

### 命令行工具模板

```python
import sys
import argparse

def main():
    parser = argparse.ArgumentParser(description='工具描述')
    parser.add_argument('input', help='输入文件')
    parser.add_argument('-o', '--output', default='output.txt', help='输出文件')
    parser.add_argument('-v', '--verbose', action='store_true', help='详细模式')
    
    args = parser.parse_args()
    
    if args.verbose:
        print(f'处理: {args.input} -> {args.output}')
    
    try:
        process(args.input, args.output)
    except FileNotFoundError:
        print(f'错误: 文件不存在 {args.input}', file=sys.stderr)
        sys.exit(1)
    except Exception as e:
        print(f'错误: {e}', file=sys.stderr)
        sys.exit(1)

def process(input_path, output_path):
    pass

if __name__ == '__main__':
    main()
```

### 捕获所有输出

```python
import sys
from io import StringIO
from contextlib import redirect_stdout, redirect_stderr

def capture_output(func, *args, **kwargs):
    stdout = StringIO()
    stderr = StringIO()
    
    with redirect_stdout(stdout), redirect_stderr(stderr):
        try:
            result = func(*args, **kwargs)
            return result, stdout.getvalue(), stderr.getvalue()
        except Exception as e:
            return None, stdout.getvalue(), str(e)

result, out, err = capture_output(print, 'hello')
```

***

## 六、常用属性

| 属性 | 作用 |
|------|------|
| `sys.platform` | 系统平台 (`win32`, `linux`, `darwin`) |
| `sys.executable` | Python 解释器路径 |
| `sys.prefix` | Python 安装路径 |
| `sys.maxsize` | 最大整数值 |
| `sys.float_info` | 浮点数信息 |
| `sys.int_info` | 整数信息 |

```python
import sys

if sys.platform == 'win32':
    print('Windows 系统')
elif sys.platform.startswith('linux'):
    print('Linux 系统')
elif sys.platform == 'darwin':
    print('macOS 系统')

print(sys.executable)
print(sys.prefix)
```

***

## 七、最佳实践

| 场景 | 建议 |
|------|------|
| 命令行参数 | 用 `argparse`，不用 `sys.argv` |
| 退出程序 | `sys.exit(0)` 成功，`sys.exit(1)` 失败 |
| 错误输出 | 用 `print(..., file=sys.stderr)` |
| 模块路径 | 优先用 `sys.path.insert(0, ...)` |
| 内存分析 | `sys.getsizeof()` 只算直接大小 |
