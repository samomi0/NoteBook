---
title: 文件遍历
date: 2024-07-14
---

文件遍历

<!-- more -->

## os.walk

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