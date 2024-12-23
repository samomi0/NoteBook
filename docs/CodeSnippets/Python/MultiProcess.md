---
title: 异步多任务
date: 2024-07-17
---

python的异步多进程, 需要特别注意GIL

<!-- more -->

[关于事件循环](EventLoop.md#loop)

## code

~~~python
import asyncio
import concurrent.futures
import time

async def worker_async(id):
    print(f"Async task {id} started")
    await asyncio.sleep(2)  # 模拟I/O操作
    print(f"Async task {id} finished")

def worker_process(id):
    print(f"Process {id} started")
    time.sleep(2)  # 模拟CPU密集型操作
    print(f"Process {id} finished")

async def main():
    loop = asyncio.get_running_loop()
    num_async_tasks = 3
    num_processes = 2

    async_tasks = [worker_async(i) for i in range(num_async_tasks)]

    with concurrent.futures.ProcessPoolExecutor(max_workers=num_processes) as executor:
        process_tasks = [loop.run_in_executor(executor, worker_process, i) for i in range(num_processes)]

        await asyncio.gather(*async_tasks, *process_tasks)

if __name__ == "__main__":
    asyncio.run(main())
~~~
