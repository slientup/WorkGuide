-[python](#python)
-[java](#java)
-[linux](#linux)
### python
#### nat_remote_collector采集器`10天`左右 cpu就`100%`
**可能性分析**：这段程序通过`多进程`+`多线程`的方式采集，怀疑线程未释放掉
**排障记录**：
- `top`命令查看python进程cpu非常高
![top命令](https://github.com/slientup/WorkGuide/blob/master/top.png)

- `perf top -g -p 4920` 查看该进程具体哪一个方法cpu高  `system_call_fastpath`系统调用非常高
 ![top命令](https://github.com/slientup/WorkGuide/blob/master/perf.png)

- `top -H -p 4920` 查看当前进程下有多少线程  `2531`
![top命令](https://github.com/slientup/WorkGuide/blob/master/much_thread.png)

- `htop -p 4920` 更友好的命令
![top命令](https://github.com/slientup/WorkGuide/blob/master/htop.png)

- 暂时尝试解决方法：更换多线程的实现方式 去掉pool池的方式
```
logging.info("nat port collect start")
thread_list = []
for host in self.task_list:
    t = threading.Thread(target=self.get_nat_result, args=(host,))
    t.start()
    thread_list.append(t)
for t in thread_list:
    t.join()
logging.info("nat port collect end")

```


### java
### linux

