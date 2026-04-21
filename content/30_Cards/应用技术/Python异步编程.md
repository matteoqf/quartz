---
type: procedure
tags:
  - Python
  - 异步编程
  - asyncio
  - 并发
创建时间: 2026-04-21
修改时间: 2026-04-21
关联版块:
---

# Python异步编程

## 概述

异步编程是一种并发模型，允许程序在等待I/O操作完成时继续执行其他任务。Python的asyncio模块提供了完整的异步编程支持，通过事件循环、协程和async/await语法实现高效的并发处理。

## 核心概念

### 1. 事件循环（Event Loop）

事件循环是异步编程的核心，负责调度和执行所有协程任务。

```python
import asyncio

# 创建事件循环
loop = asyncio.new_event_loop()
asyncio.set_event_loop(loop)

# 运行协程
loop.run_until_complete(coro())
loop.close()
```

### 2. 协程（Coroutine）

协程是异步编程的基本单位，使用async def定义。

```python
async def hello():
    print("Hello")
    await asyncio.sleep(1)
    print("World")
```

### 3. async/await

- `async`：声明协程函数
- `await`：挂起当前协程，等待另一个协程完成

## 基础用法

### 创建和运行协程

```python
import asyncio

async def main():
    print("Start")
    await asyncio.sleep(1)
    print("End")

# 三种运行方式
asyncio.run(main())  # Python 3.7+
```

### 并发执行多个协程

```python
async def task(id):
    print(f"Task {id} start")
    await asyncio.sleep(1)
    print(f"Task {id} end")

async def main():
    # 并发运行多个协程
    await asyncio.gather(
        task(1),
        task(2),
        task(3)
    )

asyncio.run(main())
```

### 并行任务处理

```python
async def fetch_data(url):
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            return await response.json()

async def main():
    urls = ["url1", "url2", "url3"]
    # 并发获取所有数据
    results = await asyncio.gather(*[fetch_data(url) for url in urls])
```

## 常用API

### asyncio.gather

并发执行多个协程，返回结果列表。

```python
results = await asyncio.gather(coro1(), coro2(), coro3())
```

### asyncio.create_task

创建任务并立即调度执行。

```python
async def main():
    task = asyncio.create_task(coro())
    result = await task
```

### asyncio.wait

等待一组协程完成，可设置超时。

```python
done, pending = await asyncio.wait([coro1(), coro2()], timeout=5)
```

### asyncio.wait_for

等待单个协程，设置超时时间。

```python
await asyncio.wait_for(coro(), timeout=10)
```

### asyncio.sleep

异步睡眠，不阻塞事件循环。

```python
await asyncio.sleep(1)  # 睡眠1秒
```

## 异步上下文管理器

```python
class AsyncResource:
    async def __aenter__(self):
        await self.connect()
        return self
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        await self.close()

async def main():
    async with AsyncResource() as resource:
        await resource.use()
```

## 异步迭代器

```python
class AsyncIterator:
    def __init__(self):
        self.items = [1, 2, 3]
        self.index = 0
    
    def __aiter__(self):
        return self
    
    async def __anext__(self):
        if self.index >= len(self.items):
            raise StopAsyncIteration
        item = self.items[self.index]
        self.index += 1
        return item

async def main():
    async for item in AsyncIterator():
        print(item)
```

## 异步生成器

```python
async def async_generator():
    for i in range(10):
        await asyncio.sleep(0.1)
        yield i

async def main():
    async for value in async_generator():
        print(value)
```

## 实际应用示例

### 异步HTTP请求（使用aiohttp）

```python
import aiohttp
import asyncio

async def fetch_all(urls):
    async with aiohttp.ClientSession() as session:
        tasks = [fetch(session, url) for url in urls]
        return await asyncio.gather(*tasks)

async def fetch(session, url):
    async with session.get(url) as response:
        return await response.json()

async def main():
    urls = [
        "https://api.example.com/data/1",
        "https://api.example.com/data/2"
    ]
    results = await fetch_all(urls)
    print(results)
```

### 异步文件操作（使用aiofiles）

```python
import aiofiles
import asyncio

async def read_files(files):
    async def read_one(filepath):
        async with aiofiles.open(filepath, 'r') as f:
            return await f.read()
    
    return await asyncio.gather(*[read_one(f) for f in files])

async def main():
    contents = await read_files(['file1.txt', 'file2.txt'])
    for content in contents:
        print(content)
```

### 异步数据库操作（使用asyncpg）

```python
import asyncio
import asyncpg

async def query_db():
    conn = await asyncpg.connect(
        host='localhost',
        port=5432,
        user='user',
        password='password',
        database='dbname'
    )
    
    try:
        result = await conn.fetch('SELECT * FROM users')
        return result
    finally:
        await conn.close()

async def main():
    rows = await query_db()
    for row in rows:
        print(dict(row))
```

### 异步Redis操作（使用redis-py）

```python
import asyncio
import redis.asyncio as redis

async def async_redis_example():
    client = redis.Redis(decode_responses=True)
    
    # 设置值
    await client.set('key', 'value')
    
    # 获取值
    value = await client.get('key')
    
    # 哈希操作
    await client.hset('user', mapping={'name': 'John', 'age': '30'})
    user = await client.hgetall('user')
    
    await client.close()
    return value, user

async def main():
    result = await async_redis_example()
    print(result)
```

### 异步Web服务（使用FastAPI）

```python
from fastapi import FastAPI
import asyncio

app = FastAPI()

@app.get("/items/{item_id}")
async def read_item(item_id: int):
    # 模拟异步操作
    await asyncio.sleep(0.1)
    return {"item_id": item_id, "name": f"Item {item_id}"}

@app.post("/items/")
async def create_item(item: dict):
    await asyncio.sleep(0.1)
    return {"id": 1, **item}
```

## 常见模式

### 超时处理

```python
async def with_timeout():
    try:
        result = await asyncio.wait_for(async_operation(), timeout=5)
        return result
    except asyncio.TimeoutError:
        print("Operation timed out")
        return None
```

### 重试机制

```python
async def retry(coro_func, max_attempts=3, delay=1):
    for attempt in range(max_attempts):
        try:
            return await coro_func()
        except Exception as e:
            if attempt == max_attempts - 1:
                raise
            await asyncio.sleep(delay)
```

### 信号处理

```python
import signal

async def run_with_cancellation():
    loop = asyncio.get_event_loop()
    stop_event = asyncio.Event()
    
    def signal_handler():
        stop_event.set()
    
    for sig in (signal.SIGINT, signal.SIGTERM):
        loop.add_signal_handler(sig, signal_handler)
    
    await stop_event.wait()
```

### 限制并发数

```python
import asyncio

async def limited_gather(tasks, max_concurrent=5):
    semaphore = asyncio.Semaphore(max_concurrent)
    
    async def bounded_task(task):
        async with semaphore:
            return await task
    
    return await asyncio.gather(*[bounded_task(t) for t in tasks])
```

### 异步锁

```python
lock = asyncio.Lock()

async def protected_operation():
    async with lock:
        # 临界区代码
        await do_something()
```

### 异步条件变量

```python
condition = asyncio.Condition()

async def producer():
    async with condition:
        await produce()
        condition.notify_all()

async def consumer():
    async with condition:
        await condition.wait()
        await consume()
```

### 异步事件

```python
event = asyncio.Event()

async def setter():
    await asyncio.sleep(1)
    event.set()

async def waiter():
    await event.wait()
    print("Event received!")

async def main():
    await asyncio.gather(setter(), waiter())
```

## 性能优化

### 1. 避免阻塞事件循环

```python
# 错误：在协程中执行同步阻塞操作
async def bad_example():
    time.sleep(10)  # 阻塞整个事件循环

# 正确：使用asyncio.sleep或在线程池中运行
async def good_example():
    await asyncio.sleep(10)
    # 或者
    await asyncio.to_thread(blocking_io_operation)
```

### 2. 使用连接池

```python
async with aiohttp.ClientSession() as session:
    # 复用session，避免频繁创建连接
    async with session.get(url) as response:
        return await response.json()
```

### 3. 批量操作

```python
# 收集任务并一次性执行
tasks = [create_task(i) for i in range(1000)]
results = await asyncio.gather(*tasks)
```

### 4. 合理设置并发数

```python
# 根据服务器能力和网络条件调整
SEMAPHORE = asyncio.Semaphore(50)  # 限制最大并发为50
```

## 调试技巧

### 打印协程信息

```python
async def coro():
    return 42

print(coro)  # <coroutine object coro at 0x...>
print(asyncio.iscoroutinefunction(coro))  # True
```

### 查看运行中的任务

```python
async def main():
    task = asyncio.create_task(coro())
    print(asyncio.all_tasks())
    await task
```

### 异常处理

```python
async def main():
    try:
        await risky_coro()
    except Exception as e:
        print(f"Error: {e}")
        # 查看异常详情
        import traceback
        traceback.print_exc()
```

## 常见问题

### 1. 死锁

避免在持有锁时等待其他协程，而该协程又试图获取同一锁。

### 2. 协程未执行

忘记await协程，导致协程对象被创建但从未执行。

```python
# 错误
async_task()  # 协程未执行

# 正确
await async_task()
```

### 3. 事件循环在多线程中使用

```python
# 在主线程中创建事件循环
async def main():
    await asyncio.sleep(1)

# 不要在子线程中创建新的事件循环
# 使用 asyncio.run() 或正确传递事件循环
```

### 4. 资源泄漏

```python
# 确保所有连接和资源被正确关闭
async with aiohttp.ClientSession() as session:
    # 使用session
# session会自动关闭
```

## 与多线程的对比

| 特性 | 异步编程 | 多线程 |
|-----|---------|-------|
| 并发模型 | 单线程，协作式 | 多线程，抢占式 |
| 复杂度 | 较高（需避免阻塞） | 较低（自动切换） |
| 资源消耗 | 较低 | 较高 |
| 适用场景 | I/O密集型 | CPU密集型/多核利用 |
| 调试难度 | 较高 | 较高 |
| 全局解释器锁 | 不受影响 | 受影响 |

## 实践建议

1. **明确I/O边界**：识别所有I/O操作并将其异步化
2. **渐进式改造**：从独立模块开始，逐步扩展到整个应用
3. **使用异步兼容库**：aiohttp、asyncpg、aioredis等
4. **避免混合同步和异步代码**：不要在异步函数中调用同步阻塞操作
5. **设置合理的超时**：防止协程永久挂起
6. **监控和日志**：跟踪异步任务的执行状态和性能
7. **测试异步代码**：使用pytest-asyncio等工具

## 进阶主题

- asyncio Stream API：异步网络编程
- asyncio Subprocess：异步执行系统命令
- 自定义协程调度器
- 异步上下文变量
- 异步迭代器和生成器的高级用法
