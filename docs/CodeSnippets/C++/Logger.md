---
title: 简易Logger
date: 2023-11-09
---

利用可变参数列表实现一个简易无依赖的日志系统。

<!-- more -->

## 定义

~~~c++
#include <iostream>
#include <ctime>
#include <cstdarg>
#include <string>

enum LogLevel {
    DEBUG,
    INFO,
    WARN,
    ERROR
};

class Logger {
public:
    static void log(LogLevel level, const char* format, ...);

private:
    static std::string getCurrentTime();
};
~~~

## 实现

~~~c++
void Logger::log(LogLevel level, const char* format, ...) {
#ifndef DEBUG_LOG
    if (level == DEBUG) {
        return;
    }
#endif
    std::string levelStr;
    switch (level) {
        case DEBUG:
            levelStr = "DEBUG";
            break;
        case INFO:
            levelStr = "INFO";
            break;
        case WARN:
            levelStr = "WARN";
            break;
        case ERROR:
            levelStr = "ERROR";
            break;
    }

    std::string timeStr = getCurrentTime();
    std::string prefix = timeStr + " [" + levelStr + "] ";
    
    std::cout << prefix;
    va_list args;
    va_start(args, format);
    std::vprintf(format, args);
    va_end(args);
    std::cout << std::endl;
}

std::string Logger::getCurrentTime() {
    std::time_t now = std::time(nullptr);
    char buffer[100];
    std::strftime(buffer, sizeof(buffer), "%Y-%m-%d %H:%M:%S", std::localtime(&now));
    return buffer;
}
~~~

## 如何使用

~~~c++
int a;
int b;
Logger::log(DEBUG, "a=%d, b=%d", a, b);
~~~
