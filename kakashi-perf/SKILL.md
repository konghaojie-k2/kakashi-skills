---
name: kakashi-perf
description: 代码性能优化顾问。扫描代码库寻找性能瓶颈，提供向量化、并行处理等优化建议。
metadata: {"openclaw":{"emoji":"⚡","always":false}}
---

# 卡卡西 Perf

代码性能优化顾问。扫描代码库寻找性能瓶颈，提供向量化、并行处理等优化建议。

## 触发条件

当用户请求以下任一操作时使用此技能：
- 性能优化
- 代码效率提升
- 寻找性能瓶颈
- 速度优化
- 向量化
- 并行处理

---

## 工作流程

### 阶段1：扫描性能瓶颈

1. **扫描代码库**
   - 遍历项目代码文件
   - 优先扫描核心业务逻辑

2. **识别瓶颈模式**

| 模式 | 严重程度 | 典型代码 |
|------|----------|----------|
| 嵌套循环 | 🔴 高 | for i in list: for j in list: |
| 同步阻塞 | 🔴 高 | time.sleep(), requests.get() |
| 重复计算 | 🟡 中 | 相同计算多次执行 |
| 字符串拼接 | 🟡 中 | s = ""; for x in list: s += x |
| 低效数据结构 | 🟡 中 | list 查找用 in |
| 未缓存结果 | 🟡 中 | 每次调用都计算 |
| 单线程处理 | 🟢 低 | 顺序处理可并行任务 |

3. **输出扫描报告**

```
【性能扫描报告】

🔴 高优先级瓶颈：
| 文件 | 行号 | 模式 | 建议 |
|------|------|------|------|
| app.py | 45 | 嵌套循环 | 向量化/并行 |
| utils.py | 23 | 同步阻塞 | 异步化 |

🟡 中优先级：
| 文件 | 行号 | 模式 | 建议 |
|------|------|------|------|
| helpers.py | 67 | 字符串拼接 | join() |
| cache.py | 12 | 未缓存 | LRU cache |
```

### 阶段2：优化方案生成

对每个瓶颈，提供以下优化方向：

#### 优化方向1：向量化 (Vectorization)

适用场景：数值计算、批量处理

```python
# ❌ 低效
result = []
for i in range(len(a)):
    result.append(a[i] * 2)

# ✅ 向量化
result = [x * 2 for x in a]  # List comprehension
# 或
result = np.array(a) * 2      # NumPy
```

**判断条件**：
- 是否有大量数值运算
- 是否可以用 NumPy/JAX 替代
- 循环体内是否可以并行

#### 优化方向2：并行处理 (Parallel)

适用场景：独立任务、I/O 密集

```python
# ❌ 顺序执行
results = []
for url in urls:
    results.append(fetch(url))

# ✅ 并行处理
from concurrent.futures import ThreadPoolExecutor
with ThreadPoolExecutor(max_workers=10) as executor:
    results = list(executor.map(fetch, urls))
```

**判断条件**：
- 任务是否相互独立
- 是否有大量 I/O 操作
- CPU 核心数是否够用

#### 优化方向3：异步处理 (Async)

适用场景：I/O 密集、网络请求

```python
# ❌ 同步阻塞
def get_data():
    for url in urls:
        resp = requests.get(url)  # 阻塞

# ✅ 异步
async def get_data():
    async with aiohttp.ClientSession() as session:
        tasks = [session.get(url) for url in urls]
        responses = await asyncio.gather(*tasks)
```

#### 优化方向4：缓存 (Cache)

适用场景：重复计算、频繁调用

```python
# ❌ 重复计算
def fib(n):
    return fib(n-1) + fib(n-2)  # 指数级复杂度

# ✅ 缓存
from functools import lru_cache
@lru_cache(maxsize=128)
def fib(n):
    return fib(n-1) + fib(n-2)
```

### 阶段3：优化建议输出

对每个瓶颈，输出：

```
【优化建议: 文件:行号】

当前代码：
```python
for i in range(len(data)):
    for j in range(len(data[i])):
        result[i][j] = data[i][j] * 2
```

⚡ 建议方案：

1️⃣ 向量化
```python
import numpy as np
result = np.array(data) * 2
```
- 提升：10-100x
- 适用：数值计算

2️⃣ 并行处理
```python
from concurrent.futures import ProcessPoolExecutor
with ProcessPoolExecutor() as executor:
    result = list(executor.map(process_chunk, chunks))
```
- 提升：CPU核心数倍
- 适用：CPU密集型

请选择优化方案，或输入自定义需求。
```

---

## 性能优化原则

| 原则 | 说明 |
|------|------|
| **测量先行** | 先用 profiler 定位瓶颈，不要猜测 |
| **小步迭代** | 一次只改一处，改完就测 |
| **权衡取舍** | 可读性 vs 性能，读写次数 |
| **渐进优化** | 先简单优化，再深入向量化/并行 |
| **保持兼容** | 接口不变，单元测试通过 |

---

## 常用优化工具

| 语言 | 工具 | 用途 |
|------|------|------|
| Python | `cProfile` | CPU 性能分析 |
| Python | `line_profiler` | 行级性能 |
| Python | `memory_profiler` | 内存分析 |
| Python | `numba` | JIT 加速 |
| Python | `numexpr` | 快速数值计算 |
| JS | Chrome DevTools | 性能分析 |
| Go | `pprof` | 性能分析 |

---

## 注意事项

1. **先测量再优化** - 不要凭感觉优化
2. **不要过早优化** - 先保证功能正确
3. **保持代码可读性** - 优化后要能维护
4. **考虑场景** - 离线处理 vs 实时请求
5. **测试验证** - 优化后必须跑通测试
