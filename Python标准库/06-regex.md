# Python 正则表达式

***

## 核心速查

| 函数 | 作用 | 返回值 |
|------|------|--------|
| `re.match(pattern, string)` | 从开头匹配 | Match 或 None |
| `re.search(pattern, string)` | 搜索第一个匹配 | Match 或 None |
| `re.findall(pattern, string)` | 查找所有匹配 | 列表 |
| `re.finditer(pattern, string)` | 迭代所有匹配 | 迭代器 |
| `re.sub(pattern, repl, string)` | 替换 | 新字符串 |
| `re.split(pattern, string)` | 分割 | 列表 |

***

## 一、基础匹配

### 1.1 核心函数

```python
import re

# match - 从字符串开头匹配
result = re.match(r'\d+', '123abc')
if result:
    print(result.group())  # '123'

# search - 搜索第一个匹配
result = re.search(r'\d+', 'abc123def456')
print(result.group())  # '123'

# findall - 查找所有匹配（返回字符串列表）
re.findall(r'\d+', 'abc123def456')  # ['123', '456']

# finditer - 迭代所有匹配（返回 Match 对象）
for m in re.finditer(r'\d+', 'abc123def456'):
    print(m.group(), m.span())  # 123 (3, 6), 456 (9, 12)
```

### 1.2 Match 对象

```python
match = re.search(r'(?P<year>\d{4})-(?P<month>\d{2})', '2024-04-06')

match.group()        # '2024-04'（整个匹配）
match.group(1)       # '2024'（第 1 组）
match.group('year')  # '2024'（命名组）
match.groups()       # ('2024', '04')（所有组）
match.groupdict()    # {'year': '2024', 'month': '04'}

match.start()        # 0（起始位置）
match.end()          # 7（结束位置）
match.span()         # (0, 7)

match.re             # 编译后的正则
match.string         # 原字符串
```

***

## 二、字符类

### 2.1 常用字符类

| 模式 | 含义 | 示例 |
|------|------|------|
| `.` | 任意字符（不含换行） | `r'a.c'` 匹配 'abc' |
| `\d` | 数字 [0-9] | `\d+` 匹配 '123' |
| `\D` | 非数字 | `\D+` 匹配 'abc' |
| `\w` | 字母数字下划线 [a-zA-Z0-9_] | `\w+` 匹配 'hello_123' |
| `\W` | 非字母数字 | `\W+` 匹配 '@#' |
| `\s` | 空白字符 | `\s+` 匹配 ' ' '\n' '\t' |
| `\S` | 非空白 | `\S+` 匹配 'abc' |

### 2.2 自定义字符类

```python
# [...] 匹配其中任意一个字符
[abc]       # a 或 b 或 c
[a-z]       # 小写字母
[A-Z]       # 大写字母
[0-9]       # 数字
[a-zA-Z0-9] # 字母数字

# [^...] 排除
[^abc]      # 非 a、b、c
[^0-9]      # 非数字

# 特殊字符需要转义
[.*+?]      # 匹配 . * + ? 本身
[\[\]]      # 匹配 [ ]
```

### 2.3 锚点

| 模式 | 含义 | 示例 |
|------|------|------|
| `^` | 字符串开头 | `^hello` |
| `$` | 字符串结尾 | `world$` |
| `\b` | 单词边界 | `\bword\b` |
| `\B` | 非单词边界 | `\Bword\B` |
| `\A` | 字符串开头（不受 MULTILINE 影响） | `\Astart` |
| `\Z` | 字符串结尾（不受 MULTILINE 影响） | `end\Z` |

```python
# 单词边界
re.findall(r'\bcat\b', 'cat concat catalog')  # ['cat']

# 行首（MULTILINE 模式）
re.findall(r'^\d+', '1. first\n2. second', re.MULTILINE)  # ['1', '2']
```

***

## 三、量词

### 3.1 基本量词

| 量词 | 含义 | 示例 |
|------|------|------|
| `*` | 0 次或多次 | `a*` 匹配 '' 'a' 'aaa' |
| `+` | 1 次或多次 | `a+` 匹配 'a' 'aaa' |
| `?` | 0 次或 1 次 | `a?` 匹配 '' 'a' |
| `{n}` | 恰好 n 次 | `a{3}` 匹配 'aaa' |
| `{n,}` | 至少 n 次 | `a{2,}` 匹配 'aa' 'aaa' |
| `{n,m}` | n 到 m 次 | `a{2,4}` 匹配 'aa' 'aaa' 'aaaa' |

### 3.2 贪婪 vs 非贪婪

```python
text = '<div>content</div>'

# 贪婪（默认）：匹配尽可能多
re.search(r'<.*>', text).group()  # '<div>content</div>'

# 非贪婪：匹配尽可能少
re.search(r'<.*?>', text).group()  # '<div>'

# 非贪婪量词
*?   # 0+，非贪婪
+?   # 1+，非贪婪
??   # 0-1，非贪婪
{n,m}?  # n-m，非贪婪
```

***

## 四、分组与捕获

### 4.1 捕获组

```python
# 普通分组
pattern = r'(\d{4})-(\d{2})-(\d{2})'
match = re.search(pattern, '2024-04-06')

match.group()       # '2024-04-06'（整个匹配）
match.group(1)      # '2024'（第 1 组）
match.group(2)      # '04'
match.group(3)      # '06'
match.groups()      # ('2024', '04', '06')

# findall 返回分组
re.findall(r'(\w+)@(\w+)', 'a@b c@d')  # [('a', 'b'), ('c', 'd')]
```

### 4.2 命名分组

```python
pattern = r'(?P<year>\d{4})-(?P<month>\d{2})-(?P<day>\d{2})'
match = re.search(pattern, '2024-04-06')

match.group('year')   # '2024'
match.group('month')  # '04'
match.group('day')    # '06'
match.groupdict()     # {'year': '2024', 'month': '04', 'day': '06'}
```

### 4.3 非捕获组

```python
# (?:...) 不捕获，仅分组
pattern = r'(?:https?://)?([\w.]+)'
match = re.search(pattern, 'https://example.com')
match.group(1)  # 'example.com'（只有一组）

# 对比捕获组
pattern = r'(https?://)?([\w.]+)'
match = re.search(pattern, 'https://example.com')
match.groups()  # ('https://', 'example.com')
```

### 4.4 分组引用

```python
# 反向引用：匹配重复内容
re.search(r'(\w+)\s+\1', 'hello hello')  # 匹配重复单词
re.findall(r'(["\']).*?\1', '"hello"')   # 匹配引号内容

# 替换时引用
re.sub(r'(\w+) (\w+)', r'\2 \1', 'hello world')  # 'world hello'
re.sub(r'(?P<first>\w+) (?P<last>\w+)', 
       r'\g<last> \g<first>', 'hello world')  # 命名引用
```

***

## 五、替换与分割

### 5.1 替换

```python
text = 'today is 2024-04-06'

# 基础替换
re.sub(r'\d{4}-\d{2}-\d{2}', 'DATE', text)
# 'today is DATE'

# 使用捕获组
re.sub(r'(\d{4})-(\d{2})-(\d{2})', r'\3/\2/\1', text)
# 'today is 06/04/2024'

# 命名分组
re.sub(r'(?P<y>\d{4})-(?P<m>\d{2})-(?P<d>\d{2})', 
       r'\g<d>/\g<m>/\g<y>', text)

# 使用函数
def upper_match(m):
    return m.group().upper()

re.sub(r'\b\w+\b', upper_match, 'hello world')  # 'HELLO WORLD'

# 限制替换次数
re.sub(r'\d', 'X', '12345', count=2)  # 'XX345'

# subn - 返回替换次数
result, count = re.subn(r'\d+', 'X', '123 abc 456')
# result: 'X abc X', count: 2
```

### 5.2 分割

```python
# 基础分割
re.split(r'[,\s]+', 'a,b,c  d  e')  # ['a', 'b', 'c', 'd', 'e']

# 保留分隔符
re.split(r'([,\s])', 'a,b c')  # ['a', ',', 'b', ' ', 'c']

# 限制分割次数
re.split(r'\s+', 'a b c d', maxsplit=2)  # ['a', 'b', 'c d']
```

***

## 六、编译与标志

### 6.1 编译正则

```python
# 编译后复用（多次使用时更高效）
pattern = re.compile(r'\d{4}-\d{2}-\d{2}')

pattern.match('2024-04-06')
pattern.search('date: 2024-04-06')
pattern.findall('2024-04-06 and 2025-01-01')
pattern.sub('DATE', '2024-04-06')
```

### 6.2 常用标志

| 标志 | 作用 | 简写 |
|------|------|------|
| `re.IGNORECASE` | 忽略大小写 | `re.I` |
| `re.MULTILINE` | `^$` 匹配行首行尾 | `re.M` |
| `re.DOTALL` | `.` 匹配换行 | `re.S` |
| `re.VERBOSE` | 允许注释和空白 | `re.X` |
| `re.ASCII` | `\w` 等只匹配 ASCII | `re.A` |

```python
# 忽略大小写
re.search(r'hello', 'HELLO', re.I)  # 匹配

# 多行模式
text = 'first line\nsecond line'
re.findall(r'^\w+', text, re.M)  # ['first', 'second']

# DOTALL 模式
re.search(r'a.*b', 'a\nb', re.S)  # 匹配

# 组合标志
re.search(r'hello', 'HELLO', re.I | re.M)

# VERBOSE 模式（可读性更好）
pattern = re.compile(r'''
    \d{4}      # 年
    -          # 分隔符
    \d{2}      # 月
    -          # 分隔符
    \d{2}      # 日
''', re.VERBOSE)
```

***

## 七、常用正则模板

### 7.1 常用模式

```python
# 手机号（中国）
r'1[3-9]\d{9}'

# 邮箱
r'[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}'

# URL
r'https?://[\w\-.]+(?:/[\w\-./?%&=]*)?'

# IP 地址
r'(?:(?:25[0-5]|2[0-4]\d|[01]?\d\d?)\.){3}(?:25[0-5]|2[0-4]\d|[01]?\d\d?)'

# 日期 YYYY-MM-DD
r'\d{4}-(?:0[1-9]|1[0-2])-(?:0[1-9]|[12]\d|3[01])'

# 时间 HH:MM:SS
r'(?:[01]\d|2[0-3]):[0-5]\d:[0-5]\d'

# 身份证号（中国）
r'[1-9]\d{5}(?:19|20)\d{2}(?:0[1-9]|1[0-2])(?:0[1-9]|[12]\d|3[01])\d{3}[\dXx]'

# 银行卡号
r'\d{16,19}'

# 中文字符
r'[\u4e00-\u9fa5]+'

# HTML 标签
r'<(\w+)[^>]*>.*?</\1>'

# 颜色值
r'#[0-9a-fA-F]{6}|#[0-9a-fA-F]{3}'
```

### 7.2 数据提取

```python
# 提取 URL 参数
def parse_url_params(url):
    params = {}
    for match in re.finditer(r'([^&?]+)=([^&?]*)', url):
        params[match.group(1)] = match.group(2)
    return params

# 提取邮箱
def extract_emails(text):
    return re.findall(r'[\w\.-]+@[\w\.-]+\.\w+', text)

# 提取链接
def extract_links(text):
    return re.findall(r'https?://[^\s<>"{}|\\^`\[\]]+', text)

# 提取手机号
def extract_phones(text):
    return re.findall(r'1[3-9]\d{9}', text)
```

***

## 八、常用模板

### 8.1 验证函数

```python
def is_email(text):
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    return bool(re.match(pattern, text))

def is_phone(text):
    pattern = r'^1[3-9]\d{9}$'
    return bool(re.match(pattern, text))

def is_id_card(text):
    pattern = r'^[1-9]\d{5}(?:19|20)\d{2}(?:0[1-9]|1[0-2])(?:0[1-9]|[12]\d|3[01])\d{3}[\dXx]$'
    return bool(re.match(pattern, text))

def is_url(text):
    pattern = r'^https?://[\w\-.]+(?:/[\w\-./?%&=]*)?$'
    return bool(re.match(pattern, text))
```

### 8.2 文本清洗

```python
def clean_whitespace(text):
    return re.sub(r'\s+', ' ', text).strip()

def remove_html_tags(text):
    return re.sub(r'<[^>]+>', '', text)

def remove_special_chars(text):
    return re.sub(r'[^\w\s]', '', text)

def normalize_phone(phone):
    return re.sub(r'[^\d]', '', phone)
```

### 8.3 高级替换

```python
# 隐藏敏感信息
def mask_phone(phone):
    return re.sub(r'(\d{3})\d{4}(\d{4})', r'\1****\2', phone)

def mask_email(email):
    return re.sub(r'(.{2}).*(@.*)', r'\1***\2', email)

def mask_id_card(id_card):
    return re.sub(r'(.{6}).*(.{4})', r'\1********\2', id_card)
```

### 8.4 解析模板

```python
# 解析日志
def parse_log_line(line):
    pattern = r'^(\S+) \S+ \S+ \[([^\]]+)\] "(\w+) ([^"]+)" (\d+) (\d+)'
    match = re.match(pattern, line)
    if match:
        return {
            'ip': match.group(1),
            'time': match.group(2),
            'method': match.group(3),
            'path': match.group(4),
            'status': match.group(5),
            'size': match.group(6)
        }
    return None

# 解析键值对
def parse_kv_string(text):
    return dict(re.findall(r'(\w+)=([^\s,]+)', text))
```
