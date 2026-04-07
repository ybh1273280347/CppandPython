# Python 生成器函数速查手册

***

## 一、核心速查表

| 函数             | 作用        | 典型场景    |
| -------------- | --------- | ------- |
| `chain`        | 连接多个可迭代对象 | 扁平化嵌套列表 |
| `islice`       | 切片迭代器     | 大文件分块读取 |
| `zip_longest`  | 不等长打包     | 缺失值填充   |
| `product`      | 笛卡尔积      | 多层循环展开  |
| `combinations` | 组合        | 选组合、抽样  |
| `permutations` | 排列        | 全排列     |
| `groupby`      | 分组        | 分组聚合    |
| `cycle`        | 无限循环      | 轮询、状态切换 |
| `count`        | 无限计数      | 生成序号    |
| `repeat`       | 重复值       | 批量生成    |
| `accumulate`   | 累积计算      | 累加、累乘   |
| `pairwise`     | 相邻配对      | 计算差值    |
| `compress`     | 布尔掩码过滤    | 条件筛选    |
| `dropwhile`    | 跳过直到不满足   | 跳过头部    |
| `takewhile`    | 取到不满足为止   | 截取头部    |

***

## 二、chain - 连接可迭代对象

```python
from itertools import chain

list(chain([1, 2], [3, 4], [5, 6]))  # [1, 2, 3, 4, 5, 6]

# 展平嵌套数组，注意只能展平最外面一层
nested = [[1, 2], [3, 4], [5, 6]]
list(chain.from_iterable(nested))  # [1, 2, 3, 4, 5, 6]
```

**注意事项**

- `chain(*iterables)` 需要提前解包，`from_iterable` 接受单个可迭代对象
- 比 `sum(lists, [])` 更高效，不会创建中间列表

***

## 三、islice - 切片迭代器

```python
from itertools import islice

list(islice(range(100), 5, 10))  # [5, 6, 7, 8, 9]

with open('big.log') as f:
    for line in islice(f, 10):
        print(line.rstrip())
```

**注意事项**

- 不支持负索引（如 `[-1]`）
- 消费迭代器，不能回退

***

## 四、zip\_longest - 不等长打包

```python
from itertools import zip_longest

names = ['Alice', 'Bob', 'Charlie']
scores = [85, 92]

list(zip_longest(names, scores, fillvalue='缺考'))
# [('Alice', 85), ('Bob', 92), ('Charlie', '缺考')]
```

**注意事项**

- `fillvalue` 默认为 `None`
- 内置 `zip` 以最短为准，`zip_longest` 以最长为准

***

## 五、product - 笛卡尔积

```python
from itertools import product

colors = ['红', '蓝']
sizes = ['S', 'M', 'L']

list(product(colors, sizes))
# [('红', 'S'), ('红', 'M'), ('红', 'L'), ('蓝', 'S'), ('蓝', 'M'), ('蓝', 'L')]

list(product('AB', repeat=2))
# [('A', 'A'), ('A', 'B'), ('B', 'A'), ('B', 'B')]
```

**注意事项**

- `repeat=n` 等价于 `product(iterable, iterable, ...)` n 次
- 结果数量为 `len(iterable) ** n`，注意内存

***

## 六、combinations / permutations - 组合与排列

```python
from itertools import combinations, permutations

list(combinations([1, 2, 3, 4], 2))
# [(1, 2), (1, 3), (1, 4), (2, 3), (2, 4), (3, 4)]

list(permutations([1, 2, 3], 2))
# [(1, 2), (1, 3), (2, 1), (2, 3), (3, 1), (3, 2)]
```

**对比**

| 函数             | 特点        | 公式                            |
| -------------- | --------- | ----------------------------- |
| `combinations` | 不考虑顺序，无重复 | C(n, r) = n! / (r! \* (n-r)!) |
| `permutations` | 考虑顺序，无重复  | P(n, r) = n! / (n-r)!         |

***

## 七、groupby - 分组

```python
from itertools import groupby

data = [('A', 1), ('B', 2), ('A', 3), ('B', 4)]
data.sort(key=lambda x: x[0])

for key, group in groupby(data, key=lambda x: x[0]):
    print(key, list(group))
# A [('A', 1), ('A', 3)]
# B [('B', 2), ('B', 4)]
```

**注意事项**

- **必须先排序**，相同元素必须相邻
- `group` 是迭代器，只能消费一次

***

## 八、cycle / count / repeat - 无限迭代器

```python
from itertools import cycle, count, repeat

# cycle - 无限循环
status = cycle(['运行', '停止', '维护'])
[next(status) for _ in range(5)]  # ['运行', '停止', '维护', '运行', '停止']

# count - 无限计数
list(islice(count(1, 2), 5))  # [1, 3, 5, 7, 9]

# repeat - 重复值
list(repeat('A', 3))  # ['A', 'A', 'A']
```

**注意事项**

- 无限迭代器必须配合 `islice` 或 `break` 使用
- `repeat(obj, times)` 不指定 `times` 则无限重复

***

## 九、accumulate - 累积计算

```python
from itertools import accumulate
import operator

list(accumulate([1, 2, 3, 4, 5]))  # [1, 3, 6, 10, 15] 累加
list(accumulate([1, 2, 3, 4, 5], operator.mul))  # [1, 2, 6, 24, 120] 累乘
list(accumulate([3, 1, 4, 1, 5], max))  # [3, 3, 4, 4, 5] 累积最大值
```

***

## 十、pairwise - 相邻配对

```python
from itertools import pairwise

list(pairwise([1, 2, 3, 4, 5]))
# [(1, 2), (2, 3), (3, 4), (4, 5)]

nums = [10, 15, 12, 20]
[b - a for a, b in pairwise(nums)]  # [5, -3, 8] 相邻差值
```

**注意事项**

- Python 3.10+ 新增
- 长度小于 2 时返回空迭代器

***

## 十一、compress - 布尔掩码过滤

```python
from itertools import compress

data = ['A', 'B', 'C', 'D', 'E']
selector = [1, 0, 1, 0, 1]

list(compress(data, selector))  # ['A', 'C', 'E']
```

**注意事项**

- `selector` 中非零值为真，零值为假
- 以较短序列为准

***

## 十二、dropwhile / takewhile - 条件截取

```python
from itertools import dropwhile, takewhile

# dropwhile：跳过满足条件的，遇到第一个不满足后全要
list(dropwhile(lambda x: x < 5, [1, 3, 5, 2, 7]))
# [5, 2, 7]

# takewhile：取满足条件的，遇到第一个不满足就停
list(takewhile(lambda x: x < 5, [1, 3, 5, 2, 7]))
# [1, 3]
```

**注意事项**

- 只看第一个不满足的位置，后面不再判断
- 与 `filter` 不同：`filter` 检查所有元素

***

## 十三、内置函数

### map - 映射

```python
list(map(int, ['1', '2', '3']))  # [1, 2, 3]
list(map(lambda a, b: a + b, [1, 2, 3], [10, 20, 30]))  # [11, 22, 33]
```

### filter - 过滤

```python
list(filter(lambda x: x > 0, [-2, -1, 0, 1, 2]))  # [1, 2]
list(filter(None, [0, '', None, 'a', 42]))  # ['a', 42]
```

### zip - 并行打包

```python
list(zip(['A', 'B', 'C'], [85, 92, 78]))
# [('A', 85), ('B', 92), ('C', 78)]

matrix = [[1, 2, 3], [4, 5, 6]]
list(zip(*matrix))  # [(1, 4), (2, 5), (3, 6)] 转置
```

**注意事项**

- `zip` 以最短序列为准
- Python 3 中 `zip` 返回迭代器，`list()` 消费

***

## 十四、实战案例

### 案例1：大文件分块处理 `islice`

```python
from itertools import islice

def process_large_file(filename, chunk_size=1000):
    with open(filename) as f:
        while True:
            chunk = list(islice(f, chunk_size))
            if not chunk:
                break
            yield chunk

for chunk in process_large_file('big_data.csv', 100):
    process(chunk)
```

### 案例2：多文件合并去重 `chain`

```python
from itertools import chain

files = ['a.txt', 'b.txt', 'c.txt']
all_lines = chain.from_iterable(open(f) for f in files)
unique_lines = set(line.strip() for line in all_lines)
```

### 案例3：生成测试数据 `product` `combinations`

```python
from itertools import product, combinations

browsers = ['Chrome', 'Firefox', 'Safari']
osystems = ['Windows', 'Mac', 'Linux']
resolutions = ['1920x1080', '1366x768']

test_cases = list(product(browsers, osystems, resolutions))
print(f"共 {len(test_cases)} 个测试用例")

for combo in combinations(browsers, 2):
    print(f"对比测试: {combo}")
```

### 案例4：日志分析统计 `groupby`

```python
from itertools import groupby
from operator import itemgetter

logs = [
    {'level': 'ERROR', 'msg': 'db failed'},
    {'level': 'INFO', 'msg': 'started'},
    {'level': 'ERROR', 'msg': 'timeout'},
    {'level': 'WARN', 'msg': 'slow'},
]

logs.sort(key=itemgetter('level'))

for level, group in groupby(logs, key=itemgetter('level')):
    count = sum(1 for _ in group)
    print(f"{level}: {count} 条")
```

### 案例5：轮询负载均衡 `cycle`

```python
from itertools import cycle

servers = ['server1', 'server2', 'server3']
requests = range(10)

pool = cycle(servers)
for req, server in zip(requests, pool):
    print(f"请求 {req} -> {server}")
```

### 案例6：数据窗口滑动 `tee` `islice`

```python
from itertools import islice, tee

def sliding_window(iterable, n):
    iters = tee(iterable, n)
    for i, it in enumerate(iters):
        next(islice(it, i, i), None)
    return zip(*iters)

data = [1, 2, 3, 4, 5, 6]
for window in sliding_window(data, 3):
    print(window)
# (1, 2, 3)
# (2, 3, 4)
# (3, 4, 5)
# (4, 5, 6)
```

### 案例7：跳过文件头部注释 `dropwhile`

```python
from itertools import dropwhile

with open('config.txt') as f:
    lines = dropwhile(lambda x: x.startswith('#'), f)
    for line in lines:
        print(line.rstrip())
```

### 案例8：生成密码组合 `product`

```python
from itertools import product

digits = '0123456789'
for combo in product(digits, repeat=4):
    password = ''.join(combo)
    print(password)
```

***

## 十五、记忆口诀

```markdown
> **chain 连接 islice 切，zip\_longest 填缺页**
> **product 积 comb 组，permutations 排列数**
> **groupby 分组先排序，cycle 循环 count 数**
> **accumulate 累加乘，pairwise 相邻配成对**
> **compress 掩码 filter 筛，dropwhile 跳 takewhile 取**