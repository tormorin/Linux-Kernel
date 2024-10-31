# 进程原理及系统调用
## 1.进程两种形式
**没有用户虚拟地址空间的进程叫内核线程**<br>
**共享用户虚拟地址空间的进程叫用户线程**<br>
**共享同一个用户虚拟地址空间的所有用户线程叫线程组**<br>
## 2.进程的生命周期
**创建状态**：创建新进程时，进程处于创建状态<br>
**就绪状态**：进程在创建后，被调度器选中，进入就绪状态<br>
**运行状态**：进程获得CPU时间片后，进程开始运行<br>
**阻塞状态**：进程运行过程中，遇到IO操作或等待某种条件，则进程进入阻塞状态<br>
**退出状态**：进程运行结束，进程进入退出状态<br>
## 3.Linux内核提供API函数来设置进程状态
**TASK_RUNNING**：可运行状态或者可就绪状态<br>
**TASK_INTERRUPTIBLE**：可中断睡眠状态(浅睡眠状态)，进程处于该状态时，可以被信号唤醒，但不能被抢占<br>
**TASK_UNINTERRUPTIBLE**：不可中断睡眠状态(深睡眠状态)，进程处于该状态时，不能被信号唤醒，也不能被抢占，我们可以通过ps命令查看被标记为D的进程<br>
**TASK_STOPPED**：停止状态，进程处于该状态时，不能被调度，只能由父进程通过kill命令杀死<br>
**EXIT_ZOMBIE**：僵尸状态，进程已经终止，但父进程还没有回收它的资源，只能由父进程通过wait命令回收<br>
**EXIT_DEAD**：死亡状态，进程已经终止，父进程已经回收了它的资源，可以被回收利用<br>
## 4.进程优先级
**限期进程优先级是-1**
**实时进程优先级是1-99**：优先级数值越大，优先级越高，实时进程的调度频率越高。
**普通进程的静态优先级是100-139**：优先级数值越小，优先级越高，可以通过修改nice值来调整优先级，优先级等于120+nice值。


# Linux内核中的 `task_struct` 源码分析

`task_struct`是Linux内核中用于描述进程（或任务）的关键数据结构，它包含了与进程相关的各种信息，如进程状态、优先级、调度信息、内存管理信息等。以下是对`task_struct`的一些重要成员和功能的分析。

## 1. 定义和位置

`task_struct`通常在`include/linux/sched.h`文件中定义。这个结构体非常庞大，包含了处理进程所需的各种信息。

## 2. 主要成员

- **pid_t pid;**
  - 存储进程的唯一标识符（PID）。

- **char comm[TASK_COMM_LEN];**
  - 存储进程的名称（通常是命令名），长度为`TASK_COMM_LEN`。

- **struct mm_struct *mm;**
  - 指向进程的内存描述符，包含了进程的虚拟地址空间的信息。

- **struct mm_struct *active_mm;**
  - 指向当前活动的内存描述符，通常在进程调度时使用。

- **volatile long state;**
  - 表示进程的状态，比如可运行、睡眠、停止等。

- **int priority;**
  - 存储进程的优先级，用于调度。

- **struct list_head children;**
  - 指向子进程的链表，方便管理父子进程关系。

- **struct task_struct *parent;**
  - 指向父进程的指针。

## 3. 状态管理

`task_struct`中的状态字段（如`state`）用来描述进程的当前运行状态：

- `TASK_RUNNING`：可运行状态，准备执行。
- `TASK_INTERRUPTIBLE`：可中断的睡眠状态，进程在等待某个条件满足。
- `TASK_UNINTERRUPTIBLE`：不可中断的睡眠状态，通常是等待I/O操作。

## 4. 调度信息

调度器使用`task_struct`来管理进程调度，此处可能包含：

- `struct sched_entity se;`：用于CFS（完全公平调度）算法的调度实体。
- `struct sched_rt_entity rt;`：用于实时调度的调度实体。

## 5. 内存管理

内存管理的信息由指向内存描述符的指针（`mm`）提供，内存描述符包含了页表、VMA（虚拟内存区域）等信息，这些都是内核在进行内存管理时所需的重要信息。

## 6. 其他信息

此外，`task_struct`还包含了与信号、定时器、文件描述符、权限等相关的信息，这些都与进程的日常操作及其生命周期密切相关。
