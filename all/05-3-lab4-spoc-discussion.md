# lab4 spoc 思考题

- 有"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的ucore_code和os_exercises的git repo上。

## 个人思考题

### 总体介绍

(1) ucore的线程控制块数据结构是什么？
>proc_struct

### 关键数据结构

(2) 如何知道ucore的两个线程同在一个进程？
>cr3寄存器是一样的

(3) context和trapframe分别在什么时候用到？
>context在切换进程的时候用到。trapframe在发生中断的时候用到，一部分是由硬件填写的，另外一部分是由中断服务例程填写的用于保存寄存器原来的值。

(4) 用户态或内核态下的中断处理有什么区别？在trapframe中有什么体现？

### 执行流程

(5) do_fork中的内核线程执行的第一条指令是什么？它是如何过渡到内核线程对应的函数的？
```
tf.tf_eip = (uint32_t) kernel_thread_entry;
/kern-ucore/arch/i386/init/entry.S
/kern/process/entry.S
```
>第一条是pushl %edx。 用call %ebx过渡到内核线程对应的函数。

(6)内核线程的堆栈初始化在哪？
```
tf和context中的esp
```

(7)fork()父子进程的返回值是不同的。这在源代码中的体现中哪？

(8)内核线程initproc的第一次执行流程是什么样的？能跟踪出来吗？

## 小组练习与思考题

(1)(spoc) 理解内核线程的生命周期。

> 需写练习报告和简单编码，完成后放到git server 对应的git repo中

### 掌握知识点
1. 内核线程的启动、运行、就绪、等待、退出
2. 内核线程的管理与简单调度
3. 内核线程的切换过程

### 练习用的[lab4 spoc exercise project source code](https://github.com/chyyuu/ucore_lab/tree/master/related_info/lab4/lab4-spoc-discuss)


请完成如下练习，完成代码填写，并形成spoc练习报告

### 1. 分析并描述创建分配进程的过程

> 注意 state、pid、cr3，context，trapframe的含义

```
先分配进程所需要的物理空间，当前进程初始化为UNINIT状态，cr3初始化为boot_cr3，pid初始化为-1。然后分配内核堆栈所需的空间，将分配的堆栈首地址保存到proc的kstack中。然后将父进程的tf复制给proc的tf，但是proc的tf的reg_eax要置为0，作为子进程的do_fork返回值，初始化proc的context。然后将proc加入进程列表中，并且wakeup子进程使之RUNNABLE。
```
### 练习2：分析并描述新创建的内核线程是如何分配资源的

> 注意 理解对kstack, trapframe, context等的初始化

```
kstack为在内存中分配的一页，trapframe的起始地址在kstack的栈顶，proc的context的eip和esp都置为当前进程的位置
```

当前进程中唯一，操作系统的整个生命周期不唯一，在get_pid中会循环使用pid，耗尽会等待

### 练习3：阅读代码，在现有基础上再增加一个内核线程，并通过增加cprintf函数到ucore代码中
能够把进程的生命周期和调度动态执行过程完整地展现出来

### 练习4 （非必须，有空就做）：增加可以睡眠的内核线程，睡眠的条件和唤醒的条件可自行设计，并给出测试用例，并在spoc练习报告中给出设计实现说明

### 扩展练习1: 进一步裁剪本练习中的代码，比如去掉页表的管理，只保留段机制，中断，内核线程切换，print功能。看看代码规模会小到什么程度。


