---
title: 事件循环
date: 2024-07-15
---

用asyncio的loop，解决python异步函数被阻塞的问题

<!-- more -->

[资料](https://docs.python.org/zh-cn/3/library/asyncio-eventloop.html#executing-code-in-thread-or-process-pools)

## loop

~~~python
import os
import asyncio
import concurrent.futures

max_cpu_cores = int(os.getenv("MAX_CPU_CORES", 4))
_pool = concurrent.futures.ThreadPoolExecutor(max_workers=min(max_cpu_cores, (os.cpu_count() or 1)))

def func_impl(param):
    return param

async def func():
    param = "hello"
    loop = asyncio.get_running_loop()
    result = await loop.run_in_executor(_pool, func_impl, param)
    return result
~~~
