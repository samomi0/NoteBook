---
title: 自定义logging
date: 2023-11-10
---

利用python的logging模块，实现一个同时支持文件导出的日志器。

<!-- more -->

## mylogger.py

~~~python
import logging
from logging import handlers
import threading


class Log:
    _instance_lock = threading.Lock()

    def init(self, *args, **kwargs):
        # 日志等级 默认: INFO
        self.level = (kwargs["level"] if kwargs.get("level") else "INFO")
        # self.level = (kwargs["level"] if kwargs.get("level") else "DEBUG")
        # 时间戳格式 默认: %Y-%m-%d-%H:%M:%S
        self.datefmt = (kwargs["datefmt"] if kwargs.get("datefmt") else "%Y-%m-%d_%H:%M:%S")
        # 日志输出格式 默认: [时间] [模块名] [函数名] [日志等级] 进程号 线程号 打印内容
        self.format = logging.Formatter((kwargs["format"] if kwargs.get("format") else 
        "%(asctime)s.%(msecs)03d [%(module)s] [%(funcName)s] [%(levelname)s] %(process)d %(thread)d %(message)s"), self.datefmt)
        # 是否控制台输出 默认: 是
        self.console = (kwargs["console"] if kwargs.get("console") else True)
        # 日志文件名前缀 默认: log
        self.filename = "./logs/" + (kwargs["filename"] + "_" if kwargs.get("filename") else "") + "log" 
        # 日志切分时间 默认: 0点
        self.when = (kwargs["when"] if kwargs.get("when") else "MIDNIGHT")
        # 日志滚动保留个数 默认: 1
        self.backupCount = (kwargs["backupCount"] if kwargs.get("backupCount") else 1)
 
        self.level_map = {
            "DEBUG" : logging.DEBUG,
            "INFO"  : logging.INFO,
            "WARNING" : logging.WARNING,
            "ERROR" : logging.ERROR,
            "CRITICAL" : logging.CRITICAL
        }

        self.logger = logging.getLogger("__name__")
        self.logger.setLevel(self.level_map.get(self.level))
        
        # 控制台日志handler
        if self.console:
            stream_handler = logging.StreamHandler()
            stream_handler.setFormatter(self.format)
            self.logger.addHandler(stream_handler)
        
        # 文件日志handler
        file_handler = handlers.TimedRotatingFileHandler(
            filename=self.filename,
            when=self.when,
            backupCount=self.backupCount,
            delay = True,
            encoding="utf-8"
        )
        file_handler.setFormatter(self.format)
        self.logger.addHandler(file_handler)

    def __new__(cls, *args, **kwargs):
        if not hasattr(Log, "_instance"):
            with Log._instance_lock:
                if not hasattr(Log, "_instance"):
                    Log._instance = object.__new__(cls)
                    Log._instance.init(*args, **kwargs)
                    Log._instance.logger.debug("logger init success")
        return Log._instance

mylogger = Log().logger
~~~

## 使用

~~~python
from mylogger import mylogger

def test():
    mylogger.info("hello world")

if __name__ == "__main__":
    test()
~~~
