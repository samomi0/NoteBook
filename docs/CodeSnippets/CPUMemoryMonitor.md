---
title: CPU内存监控
---

!!! info "Python Only"

    利用psutil和matplotlib实现对指定进程的资源监控，并画图保存

~~~shell title="depends"
pip3 install psutil matplotlib -i https://pypi.tuna.tsinghua.edu.cn/simple
~~~

~~~python
import psutil
import time
import argparse
import logging

def parse_arguments():
    parser = argparse.ArgumentParser()
    parser.add_argument("-p", "--pid", help="pid of target", type=int)
    parser.add_argument("-d", "--duration", default=10, help="Duration Time", type=int)
    parser.add_argument("-i", "--interval", default=1, help="interval", type=int)
    parser.add_argument("--save_image", default=False, help="save image?", type=bool)
    args = parser.parse_args()

    if not args.pid:
        print("Please set pid")
        exit(1)

    return args

class Monitor:
    _process_id = 1
    _process = None
    _timestamps = []
    _memory_usage = []
    _cpu_usage = []

    def __init__(self, process_id):
        self._process_id = process_id
        try:
            self._process = psutil.Process(self._process_id)
        except psutil.NoSuchProcess:
            print(f"进程ID {self._process_id} 不存在或已终止。")
            exit(-1)

    def save_usage_image(self):
        import matplotlib.pyplot as plt

        # 绘制内存使用率图表
        plt.figure(1)
        plt.plot(self._timestamps, self._memory_usage)
        plt.xlabel('time(s)')
        plt.ylabel('Memory(MB)')
        plt.title('Usage')
        plt.grid(True)

        # 保存图表
        plt.savefig("memory_usage_plot_" + str(self._process_id) + ".png")

        # 绘制CPU使用率图表
        plt.figure(2)
        plt.plot(self._timestamps, self._cpu_usage)
        plt.xlabel('time(s)')
        plt.ylabel('CPU(%)')
        plt.title('CPU Usage')
        plt.grid(True)

        # 保存图表
        plt.savefig("cpu_usage_plot_" + str(self._process_id) + ".png")

    def process(self, duration, interval):
        start_time = time.time()

        while time.time() - start_time <= duration:
            cur_time = time.time() - start_time
            try:
                mem_info = self._process.memory_info()
                cur_mem_usage = mem_info.rss / (1024 * 1024) # 单位：MB
                self._memory_usage.append(cur_mem_usage)  

                cpu_percent = self._process.cpu_percent(interval=interval)
                self._cpu_usage.append(cpu_percent)

                logging.info(f"Memory usage: {cur_mem_usage:.6f}MB, CPU usage: {cpu_percent}%")

                self._timestamps.append(cur_time)
            except:
                logging.warn(f"进程ID {self._process_id} 不存在或已终止。")
                break

if __name__ == "__main__":
    args = parse_arguments()

    process_id = args.pid
    duration = args.duration
    interval = args.interval
    b_save_img = args.save_image if args.save_image else False

    print("b_save_img:", b_save_img)

    log_file_name = str(process_id) + ".log"
    logging.basicConfig(level=logging.DEBUG,
                        format='%(asctime)s - %(levelname)s - %(message)s',
                        filename=log_file_name)

    monitor = Monitor(process_id)
    monitor.process(duration, interval)

    if b_save_img:
        monitor.save_usage_image()        
~~~
