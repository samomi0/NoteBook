---
title: Gunicorn —— 绿色独角兽
date: 2024-07-17
authors: ["Misaki"]
categories: ["Gunicorn", "Python"]
---

> *说真的，什么教程都不如直接看官方手册*
>
> WSGI: [Gunicorn Docs](https://docs.gunicorn.org/en/stable) + [Flask Docs](https://flask.palletsprojects.com/en/3.0.x/)
> 
> ASGI: [Uvicorn Docs](https://www.uvicorn.org/) + [Fastapi Docs](https://fastapi.tiangolo.com/zh)

## 为什么选择它

常见的python web应用框架有Fastapi或者Flask，与之对应的http服务器则是Uvicorn和Gunicorn。

<!-- more -->

由于Flask的默认启动方式并不合适用于生产，我更习惯选择Fastapi
> Do not use the development server when deploying to production. It is intended for use only during local development. It is not designed to be particularly secure, stable, or efficient.
> 
> [相关资料](https://flask.palletsprojects.com/en/3.0.x/deploying/)

然而Fastapi是一个异步的web框架，与之搭配的uvicorn又并没有像Gunicorn这样非常方便的配置文件和钩子机制。

好在uvicorn提供了一个兼容的worker可以使用，于是就有了如下的组合:

**Fastapi + uvicorn.workers.UvicornWorker + Gunicorn**

## 如何开始

0. 记得安装好它们

    ~~~sh
    pip install fastapi uvicorn gunicorn
    ~~~

1. 你的fastapi代码

    ~~~py
    from fastapi import FastAPI

    app = FastAPI()

    @app.get("/")
    def read_root():
        return {"Hello": "World"}

    @app.get("/items/{item_id}")
    def read_item(item_id: int, q: str = None):
        return {"item_id": item_id, "q": q}
    ~~~

2. 启动

    ~~~sh
    gunicorn -k uvicorn.workers.UvicornWorker main:app
    ~~~

## 更多配置

像这样指定一个配置文件来启动app，然后在配置文件里自定义更多你想要的内容

~~~sh
gunicorn -c gunicorn_conf.py main:app
~~~

### gunicorn_conf.py

~~~py
import os
import multiprocessing

# 获取环境变量
max_workers_str = os.getenv("MAX_WORKERS")
host = os.getenv("HOST", "0.0.0.0")
port = os.getenv("PORT", "8080")
bind_env = os.getenv("BIND")
use_loglevel = os.getenv("LOG_LEVEL", "info")
accesslog_var = os.getenv("ACCESS_LOG", "-")
errorlog_var = os.getenv("ERROR_LOG", "-")
graceful_timeout_str = os.getenv("GRACEFUL_TIMEOUT", "120")
timeout_str = os.getenv("TIMEOUT", "120")
keepalive_str = os.getenv("KEEP_ALIVE", "5")
backlog_str = os.getenv("BACKLOG", "2048")

# 设置 Gunicorn 配置
use_max_workers = int(max_workers_str) if max_workers_str else (multiprocessing.cpu_count() * 2 + 1)
use_bind = bind_env if bind_env else f"{host}:{port}"
use_accesslog = accesslog_var or None
use_errorlog = errorlog_var or None

# Gunicorn config variables

# 工作进程数量
workers = use_max_workers
worker_class = "uvicorn.workers.UvicornWorker"

# 绑定地址和端口
bind = use_bind

# 日志配置
errorlog = use_errorlog
accesslog = use_accesslog
loglevel = use_loglevel

# 超时时间
timeout = int(timeout_str)
graceful_timeout = int(graceful_timeout_str)

# 保持活动时间
keepalive = int(keepalive_str)

# 请求队列大小
backlog = int(backlog_str)


# For debugging and testing
log_data = {
    "loglevel": loglevel,
    "workers": workers,
    "bind": bind,
    "backlog": backlog,
    "graceful_timeout": graceful_timeout,
    "timeout": timeout,
    "keepalive": keepalive,
    "errorlog": errorlog,
    "accesslog": accesslog,
    # Additional, non-gunicorn variables
    "use_max_workers": use_max_workers,
    "host": host,
    "port": port,
}
print(log_data)
~~~

### hooks

这允许用户在 Gunicorn 的不同生命周期阶段执行自定义代码。这些 hooks 可以用来执行启动前初始化、日志记录、信号处理等操作。具体详细的说明可以在文档里找到，这里做简单示例。

~~~py
# add in gunicorn_conf.py

WORKER_ENV = {}

def get_worker_id(worker):
    return f'WORKER_{worker.age}'

def when_ready(server):
    server.log.info(f"on when_ready")

def pre_fork(server, worker):
    server.log.info(f"on pre_fork")

def post_fork(server, worker):
    server.log.info(f"on post_fork")

def post_worker_init(worker):
    worker.log.info(f"worker {get_worker_id(worker)} init success")

def child_exit(server, worker):
    server.log.info(f"worker:{worker_id} exited")
~~~
