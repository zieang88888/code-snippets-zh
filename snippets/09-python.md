# 09 · Python 实用片段

> 文件处理 / 列表推导 / 字典技巧——Python 日常写码高频小件。

### 1. 列表去重（list_unique）
- **作用**：保持顺序去重。
- **代码**：
```python
def unique(lst):
    seen = set()
    out = []
    for x in lst:
        if x not in seen:
            seen.add(x)
            out.append(x)
    return out

unique([1, 2, 2, 3, 1, 4])  # [1, 2, 3, 4]
```
- **说明**：`list(set(lst))` 会乱序，要保序用这个。
- **注意**：元素必须可哈希。

### 2. 字典按 key 排序（sort_dict_by_key）
- **作用**：字典按键排序。
- **代码**：
```python
def sort_by_key(d):
    return dict(sorted(d.items()))

sort_by_key({'b': 2, 'a': 1, 'c': 3})  # {'a':1,'b':2,'c':3}
```
- **说明**：Python 3.7+ dict 保插入序。
- **注意**：按 value 排序把 lambda 换成 `lambda kv: kv[1]`。

### 3. 字典按 value 排序（sort_dict_by_value）
- **作用**：字典按值排序。
- **代码**：
```python
def sort_by_value(d, reverse=False):
    return dict(sorted(d.items(), key=lambda kv: kv[1], reverse=reverse))

sort_by_value({'a': 3, 'b': 1, 'c': 2})  # {'b':1,'c':2,'a':3}
```
- **说明**：TopN 榜单。
- **注意**：返回新 dict，不改原 dict。

### 4. 列表扁平化（flatten）
- **作用**：嵌套列表拍平一层。
- **代码**：
```python
def flatten(lst):
    return [x for sub in lst for x in sub]

flatten([[1, 2], [3, 4]])  # [1,2,3,4]
```
- **说明**：列表推导一行搞定。
- **注意**：只拍一层；深层用递归。

### 5. 深层扁平化（deep_flatten）
- **作用**：任意层数嵌套都拍平。
- **代码**：
```python
def deep_flatten(lst):
    out = []
    for x in lst:
        if isinstance(x, list):
            out.extend(deep_flatten(x))
        else:
            out.append(x)
    return out

deep_flatten([1, [2, [3, 4], 5]])  # [1,2,3,4,5]
```
- **说明**：递归。
- **注意**：别太深，会爆栈。

### 6. 计数器（counter）
- **作用**：统计列表元素频次。
- **代码**：
```python
from collections import Counter

def top_n(lst, n=3):
    return Counter(lst).most_common(n)

top_n(['a', 'b', 'a', 'c', 'a', 'b'])  # [('a',3),('b',2),('c',1)]
```
- **说明**：内置 Counter 一行。
- **注意**：most_common 不传 n 返回全部。

### 7. 分组（group_by）
- **作用**：按 key 函数分组。
- **代码**：
```python
from collections import defaultdict

def group_by(lst, key_fn):
    d = defaultdict(list)
    for x in lst:
        d[key_fn(x)].append(x)
    return dict(d)

group_by([1, 2, 3, 4, 5], lambda x: x % 2)
# {1:[1,3,5], 0:[2,4]}
```
- **说明**：Excel 数据透视思路。
- **注意**：defaultdict 自动建空 list。

### 8. 两字典合并（merge_dicts）
- **作用**：合并两个 dict，重叠 key 后者覆盖。
- **代码**：
```python
def merge(a, b):
    return {**a, **b}

merge({'a': 1, 'b': 2}, {'b': 3, 'c': 4})
# {'a':1,'b':3,'c':4}
```
- **说明**：Python 3.9+ 可直接 `a | b`。
- **注意**：浅合并。

### 9. 字典取值带默认（get_safe）
- **作用**：和 d.get 同义，这里示范嵌套取值。
- **代码**：
```python
def deep_get(d, *keys, default=None):
    cur = d
    for k in keys:
        if isinstance(cur, dict) and k in cur:
            cur = cur[k]
        else:
            return default
    return cur

deep_get({'a': {'b': {'c': 1}}}, 'a', 'b', 'c')  # 1
deep_get({'a': {}}, 'a', 'b', 'x', default=0)  # 0
```
- **说明**：避免嵌套 KeyError。
- **注意**：访问 JSON 接口响应很有用。

### 10. 列表 chunk（chunk）
- **作用**：把列表切成固定大小的块。
- **代码**：
```python
def chunk(lst, size):
    return [lst[i:i+size] for i in range(0, len(lst), size)]

chunk([1,2,3,4,5], 2)  # [[1,2],[3,4],[5]]
```
- **说明**：分页、批量处理。
- **注意**：最后一块可能不满。

### 11. 去重并保序（unique_keep_order）
- **作用**：和 unique 同义，用 dict.fromkeys 一行。
- **代码**：
```python
def unique_fast(lst):
    return list(dict.fromkeys(lst))

unique_fast([1,2,2,3,1])  # [1,2,3]
```
- **说明**：Python 3.7+ dict 保序，自动去重。
- **注意**：元素必须可哈希。

### 12. 反转字典（invert_dict）
- **作用**：key 和 value 互换。
- **代码**：
```python
def invert(d):
    return {v: k for k, v in d.items()}

invert({'a': 1, 'b': 2})  # {1:'a', 2:'b'}
```
- **说明**：查反查。
- **注意**：value 必须可哈希且唯一，否则后覆盖前。

### 13. 字典过滤（filter_dict）
- **作用**：按条件过滤字典。
- **代码**：
```python
def filter_dict(d, pred):
    return {k: v for k, v in d.items() if pred(k, v)}

filter_dict({'a': 1, 'b': 2, 'c': 3}, lambda k, v: v > 1)
# {'b':2,'c':3}
```
- **说明**：字典推导式。
- **注意**：和列表推导同思想。

### 14. 文件读取为行（read_lines）
- **作用**：读文本文件成行列表。
- **代码**：
```python
def read_lines(path, encoding='utf-8'):
    with open(path, 'r', encoding=encoding) as f:
        return [line.rstrip('\n') for line in f]

read_lines('a.txt')  # ['line1', 'line2', ...]
```
- **说明**：with 自动关闭。
- **注意**：rstrip('\n') 保留行内空格。

### 15. 写入文本（write_text）
- **作用**：写一个字符串到文件。
- **代码**：
```python
def write_text(path, content, encoding='utf-8'):
    with open(path, 'w', encoding=encoding) as f:
        f.write(content)
```
- **说明**：UTF-8 写文件。
- **注意**：w 模式会覆盖；追加用 'a'。

### 16. CSV 读写（read_csv）
- **作用**：读 CSV 为 dict 列表。
- **代码**：
```python
import csv

def read_csv(path):
    with open(path, encoding='utf-8') as f:
        return list(csv.DictReader(f))

# read_csv('data.csv') -> [{'name':'a','age':'1'}, ...]
```
- **说明**：比 pandas 轻。
- **注意**：所有值都是字符串，要自己转 int/float。

### 17. JSON 读写（read_json / write_json）
- **作用**：读写 JSON 文件。
- **代码**：
```python
import json

def read_json(path):
    with open(path, encoding='utf-8') as f:
        return json.load(f)

def write_json(path, obj):
    with open(path, 'w', encoding='utf-8') as f:
        json.dump(obj, f, ensure_ascii=False, indent=2)
```
- **说明**：ensure_ascii=False 保留中文。
- **注意**：indent=2 美化输出。

### 18. 遍历目录（list_files）
- **作用**：列出目录下所有文件。
- **代码**：
```python
from pathlib import Path

def list_files(root, pattern='*'):
    return [p for p in Path(root).rglob(pattern) if p.is_file()]

list_files('./data', '*.csv')  # 所有 csv 路径
```
- **说明**：pathlib 比 os.path 优雅。
- **注意**：rglob 递归。

### 19. 创建目录（ensure_dir）
- **作用**：目录不存在就创建。
- **代码**：
```python
from pathlib import Path

def ensure_dir(path):
    Path(path).mkdir(parents=True, exist_ok=True)
```
- **说明**：写文件前先建目录。
- **注意**：parents=True 建多级。

### 20. 删除文件（safe_remove）
- **作用**：文件不存在不报错。
- **代码**：
```python
from pathlib import Path

def safe_remove(path):
    p = Path(path)
    if p.exists():
        p.unlink()
```
- **说明**：清理临时文件。
- **注意**：目录用 shutil.rmtree。

### 21. 计时（timer）
- **作用**：测一段代码耗时。
- **代码**：
```python
from time import perf_counter

def timer(fn, *args):
    start = perf_counter()
    r = fn(*args)
    return r, perf_counter() - start

result, secs = timer(sum, range(10_000_000))
```
- **说明**：perf_counter 比 time.time 精度高。
- **注意**：要装饰器版用 contextlib。

### 22. 重试（retry_py）
- **作用**：失败自动重试。
- **代码**：
```python
import time

def retry(fn, times=3, delay=0.5):
    last = None
    for i in range(times):
        try:
            return fn()
        except Exception as e:
            last = e
            time.sleep(delay)
    raise last
```
- **说明**：网络请求重试。
- **注意**：要指数退避把 delay *= 2。

### 23. 记忆化（memoize_py）
- **作用**：lru_cache 一行。
- **代码**：
```python
from functools import lru_cache

@lru_cache(maxsize=None)
def fib(n):
    return n if n < 2 else fib(n - 1) + fib(n - 2)

fib(100)  # 瞬间
```
- **说明**：递归 DP 神器。
- **注意**：参数必须可哈希。

### 24. 拼路径（join_path）
- **作用**：跨平台拼路径。
- **代码**：
```python
from pathlib import Path

p = Path('data') / 'sub' / 'a.csv'
# Windows: data\sub\a.csv
# Linux:   data/sub/a.csv
```
- **说明**：别手写斜杠。
- **注意**：Path 对象可直接传给 open()。

### 25. 文件大小（file_size）
- **作用**：拿文件字节数。
- **代码**：
```python
from pathlib import Path

def file_size(path):
    return Path(path).stat().st_size

file_size('a.txt')  # 1024
```
- **说明**：stat 一次拿齐元数据。
- **注意**：目录的 st_size 没意义。

### 26. 唯一 ID（uuid4）
- **作用**：生成 UUID 字符串。
- **代码**：
```python
import uuid

def new_id():
    return uuid.uuid4().hex

new_id()  # 'a1b2c3...' 32 位
```
- **说明**：订单号、临时文件名。
- **注意**：hex 不带横杠；要横杠用 str(uuid.uuid4())。

### 27. 字符串模板（template）
- **作用**：简单变量替换。
- **代码**：
```python
from string import Template

s = Template('Hello, $name! You have $count msgs.')
s.substitute(name='Alice', count=3)
# 'Hello, Alice! You have 3 msgs.'
```
- **说明**：比 f-string 适合外部模板。
- **注意**：缺变量报 KeyError；safe_substitute 不报错。

### 28. 列表推导平方（square_list）
- **作用**：基础列表推导。
- **代码**：
```python
def squares(n):
    return [x * x for x in range(1, n + 1)]

squares(5)  # [1, 4, 9, 16, 25]
```
- **说明**：比 for append 快。
- **注意**：要带条件 `[x*x for x in range(10) if x % 2 == 0]`。

### 29. 字典推导（dict_comprehension）
- **作用**：从两个列表建字典。
- **代码**：
```python
def zip_to_dict(keys, values):
    return dict(zip(keys, values))

zip_to_dict(['a', 'b'], [1, 2])  # {'a':1,'b':2}
```
- **说明**：zip + dict 一行。
- **注意**：长度不一致按短的截断。

### 30. 矩阵转置（transpose）
- **作用**：行列互换。
- **代码**：
```python
def transpose(matrix):
    return [list(row) for row in zip(*matrix)]

transpose([[1,2,3],[4,5,6]])  # [[1,4],[2,5],[3,6]]
```
- **说明**：zip(*matrix) 是 Python 惯用法。
- **注意**：zip 返回 tuple，外面 list 包一下。

### 31. 扁平化字典（flatten_dict）
- **作用**：嵌套字典拍平成 a.b.c 路径。
- **代码**：
```python
def flatten_dict(d, parent_key='', sep='.'):
    items = []
    for k, v in d.items():
        new_key = f'{parent_key}{sep}{k}' if parent_key else k
        if isinstance(v, dict):
            items.extend(flatten_dict(v, new_key, sep).items())
        else:
            items.append((new_key, v))
    return dict(items)

flatten_dict({'a': {'b': 1, 'c': {'d': 2}}})
# {'a.b':1, 'a.c.d':2}
```
- **说明**：日志/上报字段拍平。
- **注意**：数组不拍。

### 32. 合并字典深度合并（deep_merge）
- **作用**：嵌套字典合并。
- **代码**：
```python
def deep_merge(a, b):
    out = dict(a)
    for k, v in b.items():
        if isinstance(v, dict) and k in out and isinstance(out[k], dict):
            out[k] = deep_merge(out[k], v)
        else:
            out[k] = v
    return out

deep_merge({'a': {'x': 1}}, {'a': {'y': 2}})
# {'a': {'x':1, 'y':2}}
```
- **说明**：配置合并。
- **注意**：列表是覆盖而非合并。

### 33. 中位数（median_py）
- **作用**：列表中位数。
- **代码**：
```python
from statistics import median

median([1, 2, 3, 4, 5])  # 3
median([1, 2, 3, 4])     # 2.5
```
- **说明**：标准库自带。
- **注意**：不用手写排序。

### 34. 标准差（std_py）
- **作用**：总体标准差。
- **代码**：
```python
from statistics import pstdev

pstdev([1, 2, 3, 4, 5])  # 1.414...
```
- **说明**：标准库。
- **注意**：样本标准差用 stdev。

### 35. 按条件计数（count_if）
- **作用**：统计满足条件的元素数。
- **代码**：
```python
def count_if(lst, pred):
    return sum(1 for x in lst if pred(x))

count_if([1, 2, 3, 4, 5], lambda x: x > 2)  # 3
```
- **说明**：生成器表达式。
- **注意**：比 filter+len 省内存。

### 36. 取最小/最大 N 个（top_n_py）
- **作用**：列表里最小/最大 n 个。
- **代码**：
```python
import heapq

def smallest_n(lst, n=3):
    return heapq.nsmallest(n, lst)

def largest_n(lst, n=3):
    return heapq.nlargest(n, lst)

smallest_n([5, 1, 4, 2, 3])  # [1, 2, 3]
```
- **说明**：比 sort 后切片省，O(n log k)。
- **注意**：n 远小于 n 时优势明显。

### 37. 日期格式化（now_str）
- **作用**：当前时间字符串。
- **代码**：
```python
from datetime import datetime

def now_str(fmt='%Y-%m-%d %H:%M:%S'):
    return datetime.now().strftime(fmt)

now_str()  # '2026-10-04 14:30:00'
```
- **说明**：日志时间戳。
- **注意**：要 UTC 用 datetime.utcnow()。

### 38. 相对时间（days_ago）
- **作用**：n 天前的日期字符串。
- **代码**：
```python
from datetime import datetime, timedelta

def days_ago(n):
    return (datetime.now() - timedelta(days=n)).strftime('%Y-%m-%d')

days_ago(7)  # '2026-09-27'
```
- **说明**：默认查询范围。
- **注意**：timedelta 支持 days/hours/minutes。

### 39. 去重计数（unique_count）
- **作用**：一行拿去重后数量。
- **代码**：
```python
def unique_count(lst):
    return len(set(lst))

unique_count([1, 2, 2, 3, 3, 3])  # 3
```
- **说明**：要列表就用 unique() 再 len。
- **注意**：元素可哈希。

### 40. 安全除法（safe_div）
- **作用**：除零返回默认。
- **代码**：
```python
def safe_div(a, b, default=0.0):
    return a / b if b else default

safe_div(10, 0)  # 0.0
safe_div(10, 2)  # 5.0
```
- **说明**：统计占比防除零。
- **注意**：default 自己定。
