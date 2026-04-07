# Python 随机数

***

## 核心速查

| 函数 | 作用 |
|------|------|
| `random.random()` | [0, 1) 随机浮点数 |
| `random.randint(a, b)` | [a, b] 随机整数 |
| `random.choice(seq)` | 随机选择一个元素 |
| `random.choices(seq, k=n)` | 随机选择 n 个（可重复） |
| `random.sample(seq, k=n)` | 随机选择 n 个（不重复） |
| `random.shuffle(seq)` | 原地打乱顺序 |

***

## 一、基础随机数

```python
import random

# 随机浮点数 [0, 1)
random.random()          # 0.37444887175646646

# 指定范围 [a, b]
random.uniform(1, 10)    # 7.180414911282462

# 随机整数
random.randint(1, 100)   # [1, 100] 闭区间
random.randrange(1, 100) # [1, 100) 半开区间
random.randrange(0, 100, 5)  # [0, 100) 步长 5

# 设置种子（可重现）
random.seed(42)
print(random.random())   # 每次运行相同
```

***

## 二、序列操作

### 2.1 选择元素

```python
import random

items = ['a', 'b', 'c', 'd', 'e']

# 随机选择一个
random.choice(items)     # 'c'

# 随机选择多个（可重复）
random.choices(items, k=3)  # ['b', 'e', 'b']

# 带权重
random.choices(items, weights=[1, 2, 3, 2, 1], k=3)

# 随机选择多个（不重复）
random.sample(items, k=3)   # ['d', 'a', 'e']

# 打乱顺序（原地修改）
random.shuffle(items)
print(items)  # ['e', 'b', 'd', 'a', 'c']
```

### 2.2 生成随机序列

```python
import random
import string

# 随机字符串
def random_string(length=8):
    chars = string.ascii_letters + string.digits
    return ''.join(random.choices(chars, k=length))

random_string(10)  # 'aB3xK9mN2p'

# 随机数字字符串
def random_digits(length=6):
    return ''.join(random.choices(string.digits, k=length))

random_digits(6)  # '384729'

# 随机密码
def random_password(length=12):
    chars = string.ascii_letters + string.digits + '!@#$%^&*'
    return ''.join(random.choices(chars, k=length))
```

***

## 三、常用分布

```python
import random

# 正态分布
random.gauss(mu=0, sigma=1)      # 均值 0，标准差 1
random.normalvariate(mu=0, sigma=1)

# 均匀分布
random.uniform(a, b)

# 指数分布
random.expovariate(lambd=1)

# 三角分布
random.triangular(low=0, high=1, mode=0.5)

# 贝塔分布
random.betavariate(alpha=1, beta=1)

# 伽马分布
random.gammavariate(alpha=1, beta=1)
```

***

## 四、secrets 安全随机

**用于密码、Token 等安全场景**

```python
import secrets

# 安全随机整数
secrets.randbelow(100)      # [0, 100)
secrets.randbits(256)       # 256 位随机数

# 安全随机字节
secrets.token_bytes(16)     # b'\x12\x34...'

# 安全随机字符串
secrets.token_hex(16)       # 'f9bf78b9a18ce6d46a0cd2b0b86df9da'
secrets.token_urlsafe(16)   # 'Drmhze6EPcv0fN_81Bj-nA'

# 安全密码
import string
def secure_password(length=16):
    alphabet = string.ascii_letters + string.digits + string.punctuation
    return ''.join(secrets.choice(alphabet) for _ in range(length))

# 安全 Token
def generate_token():
    return secrets.token_urlsafe(32)
```

***

## 五、常用模板

### 5.1 随机验证码

```python
import random

def verification_code(length=6):
    """数字验证码"""
    return ''.join(random.choices('0123456789', k=length))

def mixed_code(length=4):
    """数字+字母验证码"""
    chars = '0123456789ABCDEFGHJKLMNPQRSTUVWXYZ'  # 排除易混淆字符
    return ''.join(random.choices(chars, k=length))

verification_code()  # '384729'
mixed_code()         # 'A3K9'
```

### 5.2 随机采样

```python
import random

def sample_data(data, ratio=0.1):
    """按比例采样"""
    k = int(len(data) * ratio)
    return random.sample(data, k)

def stratified_sample(data, key_func, n_per_group):
    """分层采样"""
    groups = {}
    for item in data:
        key = key_func(item)
        groups.setdefault(key, []).append(item)
    
    result = []
    for group in groups.values():
        result.extend(random.sample(group, min(n_per_group, len(group))))
    return result
```

### 5.3 随机打乱数据

```python
import random

def shuffle_data(data, seed=None):
    """打乱数据（可重现）"""
    data = data.copy()
    if seed is not None:
        random.seed(seed)
    random.shuffle(data)
    return data

def train_test_split(data, test_ratio=0.2, seed=None):
    """划分训练测试集"""
    data = shuffle_data(data, seed)
    k = int(len(data) * test_ratio)
    return data[k:], data[:k]
```

### 5.4 加权随机

```python
import random

def weighted_choice(items, weights):
    """加权随机选择"""
    total = sum(weights)
    r = random.uniform(0, total)
    cumulative = 0
    for item, weight in zip(items, weights):
        cumulative += weight
        if r <= cumulative:
            return item

# 使用
items = ['普通', '稀有', '史诗', '传说']
weights = [70, 20, 8, 2]
result = weighted_choice(items, weights)
```

### 5.5 随机颜色

```python
import random

def random_color():
    """随机 RGB 颜色"""
    return (random.randint(0, 255), random.randint(0, 255), random.randint(0, 255))

def random_hex_color():
    """随机十六进制颜色"""
    return '#{:06x}'.format(random.randint(0, 0xFFFFFF))

def random_pastel_color():
    """随机柔和颜色"""
    r = random.randint(128, 255)
    g = random.randint(128, 255)
    b = random.randint(128, 255)
    return (r, g, b)
```

***

## 六、random vs secrets

| 特性 | random | secrets |
|------|--------|---------|
| 安全性 | 伪随机 | 加密安全 |
| 性能 | 快 | 较慢 |
| 可重现 | 可设置种子 | 不可重现 |
| 用途 | 模拟、游戏、测试 | 密码、Token、密钥 |

***

## 七、最佳实践

| 场景 | 建议 |
|------|------|
| 模拟、测试 | `random` |
| 游戏、抽奖 | `random` |
| 密码、Token | `secrets` |
| 验证码 | `random`（非安全场景）或 `secrets`（安全场景） |
| 需要重现 | `random.seed()` |
| 统计采样 | `random.sample()` |
