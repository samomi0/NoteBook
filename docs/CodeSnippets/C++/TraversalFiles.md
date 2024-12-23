---
title: 文件遍历
date: 2024-07-14
---

文件遍历

<!-- more -->

## filesystem

~~~c++
#include <iostream>
#include <filesystem>

namespace fs = std::filesystem;

int main() {
    std::string path = ".";

    try {
        for (const auto& entry : fs::directory_iterator(path)) {
            std::cout << entry.path() << std::endl;
        }
    } catch (const fs::filesystem_error& e) {
        std::cerr << "Filesystem error: " << e.what() << std::endl;
    } catch (const std::exception& e) {
        std::cerr << "General exception: " << e.what() << std::endl;
    }

    return 0;
}
~~~

## dirent.h

~~~c++
#include <iostream>
#include <dirent.h>
#include <cstring>

int main() {
    const char *path = ".";
    DIR *dir = opendir(path);

    if (dir == nullptr) {
        std::cerr << "Error opening directory: " << std::strerror(errno) << std::endl;
        return 1;
    }

    struct dirent *entry;
    while ((entry = readdir(dir)) != nullptr) {
        std::cout << entry->d_name << std::endl;
    }

    closedir(dir);
    return 0;
}
~~~