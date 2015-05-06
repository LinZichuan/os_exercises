# 同步互斥(lec 18) spoc 思考题


- 有"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的ucore_code和os_exercises的git repo上。

## 个人思考题

### 基本理解
 - 什么是信号量？它与软件同步方法的区别在什么地方？
 - 什么是自旋锁？它为什么无法按先来先服务方式使用资源？
 - 下面是一种P操作的实现伪码。它能按FIFO顺序进行信号量申请吗？
```
 while (s.count == 0) {  //没有可用资源时，进入挂起状态；
        调用进程进入等待队列s.queue;
        阻塞调用进程;
}
s.count--;              //有可用资源，占用该资源； 
```

> 参考回答： 它的问题是，不能按FIFO进行信号量申请。
> 它的一种出错的情况
```
一个线程A调用P原语时，由于线程B正在使用该信号量而进入阻塞状态；注意，这时value的值为0。
线程B放弃信号量的使用，线程A被唤醒而进入就绪状态，但没有立即进入运行状态；注意，这里value为1。
在线程A处于就绪状态时，处理机正在执行线程C的代码；线程C这时也正好调用P原语访问同一个信号量，并得到使用权。注意，这时value又变回0。
线程A进入运行状态后，重新检查value的值，条件不成立，又一次进入阻塞状态。
至此，线程C比线程A后调用P原语，但线程C比线程A先得到信号量。
```

### 信号量使用

 - 什么是条件同步？如何使用信号量来实现条件同步？
 - 什么是生产者-消费者问题？
 - 为什么在生产者-消费者问题中先申请互斥信息量会导致死锁？

### 管程

 - 管程的组成包括哪几部分？入口队列和条件变量等待队列的作用是什么？
 - 为什么用管程实现的生产者-消费者问题中，可以在进入管程后才判断缓冲区的状态？
 - 请描述管程条件变量的两种释放处理方式的区别是什么？条件判断中while和if是如何影响释放处理中的顺序的？

### 哲学家就餐问题

 - 哲学家就餐问题的方案2和方案3的性能有什么区别？可以进一步提高效率吗？

### 读者-写者问题

 - 在读者-写者问题的读者优先和写者优先在行为上有什么不同？
 - 在读者-写者问题的读者优先实现中优先于读者到达的写者在什么地方等待？
 
## 小组思考题

1. （spoc） 每人用python threading机制用信号量和条件变量两种手段分别实现[47个同步问题](07-2-spoc-pv-problems.md)中的一题。向勇老师的班级从前往后，陈渝老师的班级从后往前。请先理解[]python threading 机制的介绍和实例](https://github.com/chyyuu/ucore_lab/tree/master/related_info/lab7/semaphore_condition)

```

题目
进程A1、A2、...、An1通过m个缓冲区向进程B1、B2、...、Bn2不断发送消息。发送和接收 工作遵循下列规则： 每个发送进程一次发送一个消息，写入一个缓冲区，缓冲区大小等于消息长度； 对每个消息，B1，B2，Bn2都须各接收一次，读入各自的数据区内； m个缓冲区都满时，发送进程等待，没有可读消息时，接收进程等待。 试用P、V操作组织正确的发送和接收工作。

#coding=utf-8
#!/usr/bin/env python

import threading
import time

m = 5
condition = threading.Condition()
buffer_num = 0
buf = ['0','0','0','0','0']

class Send(threading.Thread):
    def __init__(self):
        threading.Thread.__init__(self)
    def run(self):
        global condition, buffer_num
        while True:
            if condition.acquire():
                if buffer_num < m:
                    buffer_num += 1
                    print "put into buffer %s" %(buffer_num)
                    buf[buffer_num-1] = self.name         ## write in buffer 
                    condition.notify()
                else:
                    print "no buffer space! waiting..."
                    condition.wait()
                condition.release()
                time.sleep(1)


class Recv(threading.Thread):
    def __init__(self):
        threading.Thread.__init__(self)
    def run(self):
        global condition, buffer_num
        while True:
            if condition.acquire(): 
                if buffer_num > 0:
                    buffer_num -= 1
                    print "get from buffer %s, get content is %s" %(buffer_num, buf[buffer_num])  ##get the content of the buffer 
                    condition.notify()
                else :
                    print "no content! waiting..."
                    condition.wait()
                condition.release()
                time.sleep(2)

if __name__ == "__main__":
    for s in range(0, 6):
        s = Send()
        s.start()

    for r in range(0, 6):
        r = Recv()
        r.start()

```
