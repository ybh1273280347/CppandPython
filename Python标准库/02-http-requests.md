# Python 网络请求

***

## 核心速查

| 模块 | 作用 | 推荐度 |
|------|------|--------|
| `httpx` | 现代异步 HTTP 客户端 | ⭐⭐⭐⭐⭐ |
| `requests` | 简洁易用的 HTTP 库 | ⭐⭐⭐⭐⭐ |
| `urllib` | 标准库，底层操作 | ⭐⭐⭐ |

***

## 一、httpx 模块

```python
import httpx
```

**优势**：支持同步/异步、HTTP/2、连接池、自动重试

### 1.1 同步请求

```python
# 基础 GET
response = httpx.get('https://api.example.com/data')
print(response.status_code)
print(response.text)
print(response.json())

# 带参数
params = {'key': 'value', 'page': 1}
response = httpx.get('https://api.example.com/search', params=params)

# 带请求头
headers = {'Authorization': 'Bearer token', 'User-Agent': 'MyApp/1.0'}
response = httpx.get(url, headers=headers)

# 超时设置
response = httpx.get(url, timeout=10.0)
response = httpx.get(url, timeout=httpx.Timeout(5.0, read=10.0))

# POST 请求
response = httpx.post(url, json={'name': 'Alice'})
response = httpx.post(url, data={'username': 'admin', 'password': '123'})
response = httpx.post(url, content=b'binary data')

# 上传文件
files = {'file': ('report.pdf', open('report.pdf', 'rb'), 'application/pdf')}
response = httpx.post(url, files=files)
```

### 1.2 响应处理

```python
response = httpx.get(url)

response.status_code      # 状态码
response.reason_phrase    # 状态描述
response.text             # 文本内容
response.content          # 字节内容
response.json()           # 解析 JSON
response.headers          # 响应头
response.cookies          # Cookies
response.url              # 最终 URL
response.encoding         # 编码

# 检查状态
response.is_success       # 200-299
response.is_redirect      # 300-399
response.is_client_error  # 400-499
response.is_server_error  # 500-599

response.raise_for_status()  # 非 2xx 抛异常
```

### 1.3 Client 会话

```python
# 使用 Client（连接池复用，性能更好）
with httpx.Client() as client:
    response = client.get(url)
    response = client.post(url, json=data)

# 带配置
with httpx.Client(
    base_url='https://api.example.com',
    headers={'Authorization': 'Bearer token'},
    timeout=10.0,
    follow_redirects=True
) as client:
    response = client.get('/users')  # 自动拼接 base_url
    response = client.post('/login', json=credentials)

# 代理
proxies = {
    'http://': 'http://proxy.com:8080',
    'https://': 'https://proxy.com:8080'
}
with httpx.Client(proxies=proxies) as client:
    client.get(url)

# 认证
with httpx.Client(auth=('user', 'pass')) as client:  # Basic Auth
    client.get(url)

# HTTP/2
with httpx.Client(http2=True) as client:
    client.get(url)
```

### 1.4 异步请求

```python
import asyncio
import httpx

async def fetch():
    async with httpx.AsyncClient() as client:
        response = await client.get('https://api.example.com/data')
        return response.json()

result = asyncio.run(fetch())

# 并发请求
async def fetch_all(urls):
    async with httpx.AsyncClient() as client:
        tasks = [client.get(url) for url in urls]
        responses = await asyncio.gather(*tasks)
        return [r.json() for r in responses]

# 流式响应
async def stream_download(url, path):
    async with httpx.AsyncClient() as client:
        async with client.stream('GET', url) as response:
            with open(path, 'wb') as f:
                async for chunk in response.aiter_bytes():
                    f.write(chunk)
```

### 1.5 流式处理

```python
# 流式下载
with httpx.stream('GET', url) as response:
    for chunk in response.iter_bytes(chunk_size=8192):
        ...

    for line in response.iter_lines():
        ...

    for text in response.iter_text():
        ...

# 流式上传
def generate_data():
    for i in range(1000):
        yield f'chunk {i}\n'.encode()

with httpx.Client() as client:
    response = client.post(url, content=generate_data())
```

### 1.6 重试机制

```python
from httpx import AsyncHTTPTransport, HTTPTransport

# 同步重试
transport = HTTPTransport(retries=3)
with httpx.Client(transport=transport) as client:
    client.get(url)

# 异步重试
transport = AsyncHTTPTransport(retries=3)
async with httpx.AsyncClient(transport=transport) as client:
    await client.get(url)
```

### 1.7 异常处理

```python
import httpx

try:
    response = httpx.get(url, timeout=10.0)
    response.raise_for_status()
except httpx.TimeoutException:
    print('请求超时')
except httpx.ConnectError:
    print('连接失败')
except httpx.HTTPStatusError as e:
    print(f'HTTP 错误: {e.response.status_code}')
except httpx.RequestError as e:
    print(f'请求异常: {e}')
```

### 1.8 httpx vs requests

| 特性 | httpx | requests |
|------|-------|----------|
| 同步请求 | ✅ | ✅ |
| 异步请求 | ✅ | ❌ |
| HTTP/2 | ✅ | ❌ |
| 连接池 | ✅ 内置 | ✅ Session |
| 自动重试 | ✅ 内置 | ❌ 需配置 |
| 类型提示 | ✅ 完整 | ❌ 部分 |
| API 风格 | 类似 requests | - |

***

## 二、requests 模块

```python
import requests
```

### 2.1 核心 API

| 方法 | 作用 |
|------|------|
| `requests.get(url, params, headers, ...)` | GET 请求 |
| `requests.post(url, data, json, files, ...)` | POST 请求 |
| `requests.put(url, data, ...)` | PUT 请求 |
| `requests.delete(url, ...)` | DELETE 请求 |
| `requests.patch(url, data, ...)` | PATCH 请求 |
| `requests.head(url, ...)` | HEAD 请求 |
| `requests.options(url, ...)` | OPTIONS 请求 |

### 2.2 GET 请求

```python
# 基础 GET
response = requests.get('https://api.example.com/data')
print(response.status_code)   # 200
print(response.text)          # 响应文本
print(response.json())        # 解析 JSON

# 带参数
params = {'key': 'value', 'page': 1}
response = requests.get('https://api.example.com/search', params=params)
# 实际 URL: https://api.example.com/search?key=value&page=1

# 带请求头
headers = {
    'Authorization': 'Bearer token',
    'User-Agent': 'MyApp/1.0',
    'Content-Type': 'application/json'
}
response = requests.get(url, headers=headers)

# 超时设置
response = requests.get(url, timeout=10)  # 10 秒超时
response = requests.get(url, timeout=(3, 10))  # 连接 3s，读取 10s

# 代理
proxies = {
    'http': 'http://proxy.com:8080',
    'https': 'https://proxy.com:8080'
}
response = requests.get(url, proxies=proxies)

# 认证
response = requests.get(url, auth=('user', 'pass'))  # Basic Auth
```

### 2.3 POST 请求

```python
# 表单数据（application/x-www-form-urlencoded）
data = {'username': 'admin', 'password': '123456'}
response = requests.post('https://api.example.com/login', data=data)

# JSON 数据（自动设置 Content-Type: application/json）
json_data = {'name': 'Alice', 'age': 30}
response = requests.post('https://api.example.com/create', json=json_data)

# multipart/form-data 上传文件
files = {'file': open('document.pdf', 'rb')}
response = requests.post('https://api.example.com/upload', files=files)

# 指定文件名和类型
files = {
    'file': ('report.pdf', open('report.pdf', 'rb'), 'application/pdf')
}
response = requests.post(url, files=files)

# 多文件上传
files = [
    ('images', ('foo.png', open('foo.png', 'rb'), 'image/png')),
    ('images', ('bar.png', open('bar.png', 'rb'), 'image/png'))
]
response = requests.post(url, files=files)

# 文件 + 表单数据
files = {'file': open('data.csv', 'rb')}
data = {'description': 'Monthly report'}
response = requests.post(url, files=files, data=data)
```

### 2.4 响应处理

```python
response = requests.get(url)

# 状态码
response.status_code              # 状态码（如 200, 404, 500）
response.ok                       # True if 200 <= status < 400
response.reason                   # 状态描述（如 'OK', 'Not Found'）
response.raise_for_status()       # 状态码非 2xx 时抛出异常

# 内容
response.text                     # 文本（自动解码）
response.content                  # 原始字节
response.json()                   # 解析 JSON（失败抛异常）
response.raw                      # 原始 urllib3 响应

# 编码
response.encoding                 # 响应编码
response.apparent_encoding        # 检测的编码
response.content.decode('utf-8')  # 手动解码

# 头信息
response.headers                  # 响应头字典
response.headers['Content-Type']  # 获取指定头
response.cookies                  # Cookies
response.url                      # 最终 URL（可能被重定向）
response.history                  # 重定向历史
```

### 2.5 Session 会话

```python
session = requests.Session()

# 保持 Cookie
session.cookies.set('name', 'value')

# 统一设置
session.headers.update({'User-Agent': 'MyApp/1.0'})
session.auth = ('user', 'pass')

# 复用连接（性能更好）
response = session.get('https://api.example.com/data')
response = session.post('https://api.example.com/login', data=credentials)
response = session.get('https://api.example.com/profile')

# 关闭
session.close()

# 或使用上下文管理器
with requests.Session() as session:
    session.get(url)
```

### 2.6 高级用法

```python
# 流式下载（大文件）
with requests.get(url, stream=True) as r:
    r.raise_for_status()
    with open('large_file.zip', 'wb') as f:
        for chunk in r.iter_content(chunk_size=8192):
            f.write(chunk)

# 流式上传
with open('large_file.bin', 'rb') as f:
    requests.post(url, data=f)

# 进度条下载
from tqdm import tqdm

response = requests.get(url, stream=True)
total = int(response.headers.get('content-length', 0))

with open('file.zip', 'wb') as f:
    with tqdm(total=total, unit='B', unit_scale=True) as pbar:
        for chunk in response.iter_content(chunk_size=8192):
            f.write(chunk)
            pbar.update(len(chunk))

# 证书验证
response = requests.get(url, verify='/path/to/cert.pem')  # 指定证书
response = requests.get(url, verify=False)  # 忽略证书（不推荐）

# 忽略警告
import urllib3
urllib3.disable_warnings()

# 重试机制
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

session = requests.Session()
retries = Retry(
    total=5,
    backoff_factor=1,
    status_forcelist=[500, 502, 503, 504]
)
session.mount('http://', HTTPAdapter(max_retries=retries))
session.mount('https://', HTTPAdapter(max_retries=retries))

# 超时重试
retries = Retry(total=3, connect=3, read=3, redirect=3)
```

### 2.7 异常处理

```python
from requests.exceptions import RequestException, Timeout, ConnectionError, HTTPError

try:
    response = requests.get(url, timeout=10)
    response.raise_for_status()
except Timeout:
    print('请求超时')
except ConnectionError:
    print('连接失败')
except HTTPError as e:
    print(f'HTTP 错误: {e.response.status_code}')
except RequestException as e:
    print(f'请求异常: {e}')
```

***

## 三、urllib 模块

```python
from urllib.request import urlopen, Request
from urllib.parse import urlencode, quote, urljoin
from urllib.error import URLError, HTTPError
```

### 3.1 发送请求

```python
# 基础 GET
response = urlopen('https://api.example.com/data')
data = response.read().decode('utf-8')

# 带参数
params = urlencode({'key': 'value', 'page': 1})
response = urlopen(f'https://api.example.com/search?{params}')

# POST 请求
data = urlencode({'username': 'admin', 'password': '123'}).encode('utf-8')
response = urlopen('https://api.example.com/login', data=data)

# 带请求头
req = Request(
    url,
    headers={'User-Agent': 'MyApp/1.0', 'Authorization': 'Bearer token'}
)
response = urlopen(req)

# POST + 请求头
req = Request(
    url,
    data=json.dumps({'name': 'Alice'}).encode('utf-8'),
    headers={'Content-Type': 'application/json'}
)
response = urlopen(req)
```

### 3.2 响应处理

```python
response = urlopen(url)

response.read()        # 读取全部内容
response.readline()    # 读取一行
response.read(1024)    # 读取指定字节数

response.status        # 状态码
response.reason        # 状态描述
response.headers       # 响应头
response.geturl()      # 最终 URL
```

### 3.3 URL 解析

```python
from urllib.parse import urlparse, parse_qs, urljoin

# 解析 URL
result = urlparse('https://user:pass@example.com:8080/path;params?query=1#fragment')
# ParseResult(
#     scheme='https',
#     netloc='user:pass@example.com:8080',
#     path='/path;params',
#     params='params',
#     query='query=1',
#     fragment='fragment'
# )

# 解析查询参数
query = 'a=1&b=2&b=3'
parse_qs(query)  # {'a': ['1'], 'b': ['2', '3']}

# URL 编码
quote('hello world')         # 'hello%20world'
quote('/path/file name')     # '/path/file%20name'
quote('中文')                # '%E4%B8%AD%E6%96%87'
quote('中文', safe='')       # '%E4%B8%AD%E6%96%87'

# 编码字典
urlencode({'a': 'b c', 'd': '中文'})  # 'a=b+c&d=%E4%B8%AD%E6%96%87'

# 相对 URL 转绝对
urljoin('https://example.com/dir/', 'file.txt')  # 'https://example.com/dir/file.txt'
urljoin('https://example.com/dir', 'file.txt')   # 'https://example.com/file.txt'
urljoin('https://example.com/dir/', '/absolute') # 'https://example.com/absolute'
```

### 3.4 错误处理

```python
from urllib.error import URLError, HTTPError

try:
    response = urlopen(url, timeout=10)
except HTTPError as e:
    print(f'HTTP Error: {e.code} {e.reason}')
    print(e.headers)
except URLError as e:
    print(f'URL Error: {e.reason}')
```

***

## 四、常用模板

### 4.1 异步并发请求（httpx）

```python
import asyncio
import httpx

async def fetch_all(urls, max_concurrency=10):
    semaphore = asyncio.Semaphore(max_concurrency)
    
    async def fetch(client, url):
        async with semaphore:
            try:
                response = await client.get(url, timeout=10.0)
                return url, response.json()
            except Exception as e:
                return url, e
    
    async with httpx.AsyncClient() as client:
        tasks = [fetch(client, url) for url in urls]
        results = await asyncio.gather(*tasks)
    
    return dict(results)

# 使用
urls = ['https://api.example.com/1', 'https://api.example.com/2']
results = asyncio.run(fetch_all(urls))
```

### 4.2 API 客户端类（httpx）

```python
import httpx

class APIClient:
    def __init__(self, base_url, token=None):
        self.base_url = base_url.rstrip('/')
        self.headers = {'Authorization': f'Bearer {token}'} if token else {}
    
    def get(self, endpoint, **kwargs):
        return self._request('GET', endpoint, **kwargs)
    
    def post(self, endpoint, **kwargs):
        return self._request('POST', endpoint, **kwargs)
    
    def _request(self, method, endpoint, **kwargs):
        url = f'{self.base_url}/{endpoint.lstrip("/")}'
        headers = {**self.headers, **kwargs.pop('headers', {})}
        
        with httpx.Client() as client:
            response = client.request(method, url, headers=headers, timeout=10.0, **kwargs)
            response.raise_for_status()
            return response.json()

# 使用
api = APIClient('https://api.example.com', token='xxx')
data = api.get('/users/1')
```

### 4.3 带重试的请求（requests）

```python
import time
import requests

def request_with_retry(url, max_retries=3, **kwargs):
    for attempt in range(max_retries):
        try:
            response = requests.get(url, timeout=10, **kwargs)
            response.raise_for_status()
            return response
        except requests.exceptions.RequestException as e:
            if attempt == max_retries - 1:
                raise
            time.sleep(2 ** attempt)
```

### 4.4 下载文件

```python
from pathlib import Path

def download_file(url, path, chunk_size=8192):
    response = requests.get(url, stream=True)
    response.raise_for_status()
    
    Path(path).parent.mkdir(parents=True, exist_ok=True)
    
    with open(path, 'wb') as f:
        for chunk in response.iter_content(chunk_size=chunk_size):
            f.write(chunk)
    
    return path
```

### 4.5 解析响应

```python
def safe_json(response, default=None):
    try:
        return response.json()
    except ValueError:
        return default

def get_json_field(response, *keys, default=None):
    data = safe_json(response, {})
    for key in keys:
        if isinstance(data, dict) and key in data:
            data = data[key]
        else:
            return default
    return data
```
