---
title: 文件遍历
---

=== "C++"

    === "filesystem"

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

    === "dirent.h"

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

=== "Python"

    === "os.walk"

        ~~~python
        import os

        def list_files(startpath):
            for root, dirs, files in os.walk(startpath):
                for name in files:
                    print(os.path.join(root, name))
                for name in dirs:
                    print(os.path.join(root, name))

        start_directory = '.'
        list_files(start_directory)
        ~~~