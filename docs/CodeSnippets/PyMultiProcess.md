---
title: Python的异步多任务与事件循环
---

!!! warning "Python的异步多进程, 需要特别注意GIL"

!!! note "关于Python的事件循环"

    这里贴一下文档 [:octicons-link-16: Use EventLoop in thread or process pools](https://docs.python.org/3.13/library/asyncio-eventloop.html#executing-code-in-thread-or-process-pools)

    Python的异步相当于一个独特的事件循环，它有一个特点，如果在这个事件循环里运行了非异步的方法，那么整个异步都会等待这个“同步方法”运行完。假如你的同步方法又恰好是个CPU密集型工作的话，就可以很明显的观测到TPS降低，异步看起来“失效”了。

    解决的办法一种是给所有被调用的方法都套上`async`，不过大多数时候代码很庞大，所以按照文档套一层loop兴许是相对简单的办法。

    ??? example "用asyncio的loop，解决python异步函数被阻塞的问题"
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


=== "Python"

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
