---
title: 共享内存
date: 2024-07-14
---

进程间共享内存

<!-- more -->

## 使用name创建

~~~c++
#include <fcntl.h>
#include <sys/mman.h>
#include <unistd.h>
#include <sys/stat.h> 
#include <iostream>

int main() {
    const char *name = "/my_shared_memory";
    const size_t SIZE = 4096;  // 大小为4KB

    int shm_fd = shm_open(name, O_CREAT | O_RDWR, 0666);
    if (shm_fd == -1) {
        std::cerr << "shm_open failed\n";
        return 1;
    }

    if (ftruncate(shm_fd, SIZE) == -1) {
        std::cerr << "ftruncate failed\n";
        return 1;
    }

    void *ptr = mmap(0, SIZE, PROT_READ | PROT_WRITE, MAP_SHARED, shm_fd, 0);
    if (ptr == MAP_FAILED) {
        std::cerr << "mmap failed\n";
        return 1;
    }

    std::cout << "Shared memory created and mapped successfully\n";

    return 0;
}
~~~

## 使用key创建

~~~c++
int s_shmem_key = 888666;
int s_sem_id = -1;
int s_shmid = -1;
int s_shmsize = 123;
void *s_pshm = NULL;
int init_shm() {
    if ((s_shmid = shmget(key_t(s_shmem_key), s_shmsize, 0666 | IPC_CREAT)) < 0) {
        printf("shmget fail %s", strerror(errno));
        return -1;
    }
    s_pshm = shmat(s_shmid, 0, 0);
    if (s_pshm == (void*) -1) {
        printf("shmat fail %s", strerror(errno));
        return -1;
    }

    ((char *)s_pshm)[s_shmsize - 1] = '\0';

    return 0;
}
~~~

## 使用name同步

~~~c++
#include <fcntl.h>
#include <sys/mman.h>
#include <unistd.h>
#include <sys/stat.h>
#include <iostream>
#include <semaphore.h>
#include <cstring>

const char *name = "/my_shared_memory";
const size_t SIZE = 4096;
const char *sem_name = "/my_semaphore";

int main() {
    sem_t *sem = sem_open(sem_name, O_CREAT, 0666, 1);
    if (sem == SEM_FAILED) {
        std::cerr << "sem_open failed\n";
        return 1;
    }

    int shm_fd = shm_open(name, O_CREAT | O_RDWR, 0666);
    if (shm_fd == -1) {
        std::cerr << "shm_open failed\n";
        return 1;
    }

    if (ftruncate(shm_fd, SIZE) == -1) {
        std::cerr << "ftruncate failed\n";
        return 1;
    }

    void *ptr = mmap(0, SIZE, PROT_READ | PROT_WRITE, MAP_SHARED, shm_fd, 0);
    if (ptr == MAP_FAILED) {
        std::cerr << "mmap failed\n";
        return 1;
    }

    sem_wait(sem);

    strcpy((char *)ptr, "Hello, shared memory!");

    sem_post(sem);

    return 0;
}
~~~

## 使用key同步

~~~c++
int sync_shm(int shmid, int semid) {
  s_shmid = shmid;
  s_sem_id = semid;

  if (s_shmid < 0 || s_sem_id < 0) {
    return -1;
  }

  s_pshm = shmat(s_shmid, 0, 0);
  if (s_pshm == (void*) -1) {
    printf("shmat fail %s", strerror(errno));
    return -1;
  }

  ((char *)s_pshm)[s_shmsize - 1] = '\0';

  return 0;
}
~~~