# Linux用户空间多线程与同步

Linux用户空间的多线程和同步是一个非常重要的概念，即便在工程中能够利用这部分内容，但是还需要更系统化的对这部分进行了解，因为我们花一些时间来整理这部分的笔记。Linux用户空间的多线程为POSIX线程，与进程之间的区别就不再赘述了。多线程的优点在于比进程有着**更小的开销，且共享信息更为便利**，而**带来的缺点难于同步、线程交互控制和调试**。

线程中引出一个**可重入**的概念，可**重入 <reentrant, [ˌriˈɛntrənt]>**，用于避免类似于fputs之类的函数，这些函数通常会有**全局性的缓冲区存储数据**。这也提示我们如果要编写一个reentrant的函数，对于修改全局性的缓冲存储数据，要格外的留心。另外，在编写reentrant的程序时，需要**`_REENTRANT`宏定义**来告诉编译器我们需要可重入的功能。reentrant还有另外一些知识点，在Linux中为了reentrant安全做了一些操作：

* 函数后加_r作为可重入安全标识：gethostbyname, 可重入版本 gethostbyname_r。
* 编译时候-D_REENTRANT会使用线程安全的函数。
* errno.h中顶一个的变量errno，加入了线程安全的处理。

## 1. pthread接口与编译

### 1.1 [pthread_create](https://linux.die.net/man/3/pthread_create)

**API Define:** 

```C
#include <pthread.h>

int pthread_create (pthread_t *thread,
                    pthread_attr_t * attr,
                    void *(*start_routine)(void*),
                    void *arg);
```

**Parameters:**

| Params                           | I/O   | Details                                                      |
| -------------------------------- | ----- | ------------------------------------------------------------ |
| pthread_t *thread                | Input | thread context                                               |
| pthread_attr_t* attr             | Input | thread attribute, if there is no attribute, the NULL shall be inputted. |
| void \*(\*start_routine)(void\*) | Input | New thread entry.                                            |
| void *arg                        | Input | arguments of new thread entry.                               |

**Return:**

* 0 : Success
* otherwise: 

### 1.2 [pthread_exit](https://linux.die.net/man/3/pthread_exit)

**API Define:** 

```C
#include <pthread.h>

void pthread_exit (void *retval);
```

**Parameters:**

| Params       | I/O    | Details                                                      |
| ------------ | ------ | ------------------------------------------------------------ |
| void *retval | Output | 返回指向某个对象的指针，**注意不可以返回一个指向局部变量的指针，局部变量在栈回溯后会消失。** |

**Return:**

* None

### 1.3 [pthread_join](https://linux.die.net/man/3/pthread_join)

**API Define:** 

```C
#include <pthread.h>

int pthread_join(pthread_t thread, 
                 void **retval);
```

**Parameters:**

| Params           | I/O    | Details                                                      |
| ---------------- | ------ | ------------------------------------------------------------ |
| pthread_t thread | Input  | 指定要等待的线程                                             |
| void \*\*retval  | Output | 返回指向某个对象的指针，**注意不可以返回一个指向局部变量的指针，局部变量在栈回溯后会消失。** |

**Return:**

On success, **pthread_join**() returns 0; on error, it returns an error number.

### 1.4  用户空间的同步机制

#### 1.4.1 基础的例子

请参考：test1_basic_thread on the [link]()

这里需要注意编译选项:

* -lpthread
* -D_REENTRANT

对于第一个编译选项，如果系统默认是NPTL线程库，无需`-lpthread`；`-D_REENTRANT`默认会使用线程安全的实现。

#### 1.4.2 10个线程竞争

这里塑造一个测试场景，10个线程竞争各自输出自己数值，0线程输出0,1线程输出1，以此类推到9号线程输出到9。这里线程竞争不加任何的同步机制，让线程彼此自由运行。

**注意，sprintf是线程不安全的。**

请参考：test2_10_threads_no_sync on the [link](https://github.com/carloscn/clab/blob/master/macos/test_thread/test_thread.c)

```log
test_thread.c:test2_10_threads_no_sync:168--create 10 threads
0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000111111111111111111111111111111111111111111111111111111111111111111111343333333333333333333444431522222222555554333787872151test_thread.c:test2_10_threads_no_sync:179--created 10 treads.
4444test_thread.c:test2_10_threads_no_sync:182--5thread 0 exit (null)
852386133636333333333666666666666688886666666166666666666669799999997777676767777777777777777777777777777777777777777777779999999999343373777777598222222222226441111777332121111111111111111146311613888888888888888888888888888888888888888888888888888888888888888888888888888888888666666631222222222222222222222222222222222222222222222222222222222222222222222222222228888888888555555555555555577373763test_thread.c:test2_10_threads_no_sync:184--66666666666666666669433333359549555549494474544444444444755557773699496669696699799999999999999999999997thread 1 exit (null)
333937555test_thread.c:test2_10_threads_no_sync:186--thread 2 exit (null)
77797999999999999999999999999999999999996666699999666666666666666666674444445555555339999953337777744477773444433333333335444444447777777774444444444444444444444444444444444444555555555555533333333333333333333333444444444555555555555555555555555555555555555553test_thread.c:test2_10_threads_no_sync:188--thread 3 exit (null)
test_thread.c:test2_10_threads_no_sync:190--thread 4 exit (null)
test_thread.c:test2_10_threads_no_sync:192--thread 5 exit (null)
test_thread.c:test2_10_threads_no_sync:194--thread 6 exit (null)
test_thread.c:test2_10_threads_no_sync:196--thread 7 exit (null)
test_thread.c:test2_10_threads_no_sync:198--thread 8 exit (null)
test_thread.c:test2_10_threads_no_sync:200--thread 9 exit (null)
test_thread.c:test2_10_threads_no_sync:201--test 2 finish 
main : test done !!
```

线程执行的过程完全是随机的。

#### 1.4.3 [信号量semaphore](https://man.archlinux.org/search?q=semaphore&go=Go)

semaphore <*ˈseməfɔːr*>

```c
#include <fcntl.h>           /* For O_* constants */
#include <sys/stat.h>        /* For mode constants */
#include <semaphore.h>

sem_t *sem_open(const char *name, int oflag);
sem_t *sem_open(const char *name, int oflag,
                mode_t mode, unsigned int value);
int sem_post(sem_t *sem);
int sem_wait(sem_t *sem);
int sem_trywait(sem_t *sem);
int sem_timedwait(sem_t *restrict sem,
                  const struct timespec *restrict abs_timeout);
int sem_close(sem_t *sem);
```

信号量S是线程同步的重要手段，sem_post和sem_wait彼此之间配合，被称为PV操作。P使之增加，V使之减少。wait函数如果S为0的时候就要等待；如果S>0的时候，wait就跳过，并且进行S--。

**注意，MACOS已经舍弃了sem_init和sem_destroy两个函数，需要使用[sem_open](https://man.archlinux.org/man/sem_open.3p.en)和[sem_unlink](https://man.archlinux.org/man/sem_unlink.3)**

例子：

* 设定一个共享的全局空间work_area，设定一个全局空间备份区域back_area
* mainloop：等待用户输入字符串fgets，存储进入work_area
* thread01: 打印用户的字符的个数，并在用户输入字符串尾部加`!`
* thread02: 把thread01修改的字符串存储备份空间，并且work_area清零。

分析：

mainloop处理fgets字符串之后存入work_area，如果不加线程同步机制，thread01和thread02不断的竞争，最终达到稳态的时候，thread01写入1个！，thread02就会把他清掉。最后备份区域要么是0，要么是1个！。因此需要加同步机制，需要顺序是：

* mainloop写入字符串，t01和t02全部等待
* mainloop释放信号允许t01开始处理，需要让t02等待
* t01处理完毕之后，释放信号允许t02处理
* t02开始处理完，结束一个走起

因此，mainloop任务之后需要post一个信号量sem_01, 在t01的handler之后需要post一个信号量sem_02。t01 wait，mainloop的信号量；t02 wait t01的信号量。

请参考：test2_thread_sem.c on the [link](https://github.com/carloscn/clab/blob/master/macos/test_thread/test_thread_sem.c)

#### 1.4.4 spinlock和mutex

spinlock和mutex在功能上是一样的，关于不同往上已经很多的资料在阐述spinlock和mutex的不一样的。在Linux的user space，需要使用pthread中给定的spinlock和mutex，pthread_mutex_t和pthread_spinlock_t，在MACOS里面已经废弃掉了spinlock，可能考虑spinlock的忙等待十分的耗费CPU。我也做了相关的实验，塑造一种"死锁"场景，子线程不断的获取锁，分别看CPU的占用。实验结果和理论一致，spinlock会占用整个CPU的资源，使CPU根本无调度的窗口期。如果塑造两个spinlock的死锁，可以观察到两个CPU的调度资源被吃空，CPU使用率一度达到200%。这里可以总结一些spinlock和mutex的区别：

* spinlock忙等待，吃掉所有cpu资源；mutex使线程进入sleep状态。mutex由此引发的一个问题就是，sleep和wakeup占用了系统的调度，并没有spinlock高效。我理解，可能在某些很重要的时序场合spinlock应该被考虑使用。spinlock处于忙等待，自然的CPU无法调度，不能被其他task打断。
* 在Linux Userspace的pthread接口中，并没有给出spin_lock_irq等接口，在Linux内核中是有的。spinlock由于是忙等待，没有涉及调度，也没有涉及sleep，因此只要不屏蔽中断，是可以用于中断上下文的；而mutex涉及sleep，是不可以用于中断上下文的（内核进入中断之前会关闭系统的调度，一旦睡眠，内核调度无法响应别的进程，系统会hang死，**中断服务程序一定不能sleep**）
* semaphore和mutex一致，同样使用调度。

使用spinlock的死锁场景：

<img width="1489" alt="image-20220322101014839" src="_media/159394291-728e7b0d-1418-4dab-8125-38b226a3b074-20220331200549679.png">

使用mutex的死锁场景：

<img width="1429" alt="image-20220322101028140" src="_media/159394313-b41d16a6-60f4-42b2-b2a5-60efc8eaa192-20220331200549945.png">

在ARMv7架构（单核imx6ULL芯片测试）
![WeChat Image_20220329114610](_media/160529284-5d2c28d5-c8bc-47be-a469-e17784fd993a-20220331200549651.png)
似乎CPU并没有跑满，原因暂时未知。

我觉得这个文献里面对比pthread_mutex和pthread_spinlock的调度性能对比可以参考一下[^4]

这里面实验为：

* https://github.com/carloscn/clab/blob/master/linux/test_thread/test_thread_mutex.c
* https://github.com/carloscn/clab/blob/master/linux/test_thread/test_thread_spinlock.c

#### 1.4.5 线程的属性

在pthread建立的时候，pthread_create的第二个参数为pthread_attr_t，这里面可以设定线程的运行属性[^2]。

```c
int pthread_attr_init(pthread_attr_t *attr);
int pthread_attr_destroy(pthread_attr_t *attr);
```

接着linux提供很多接口set和get属性：

* **detachstate:** 脱离线程，该线程完成任务直接退出，不需要汇报主线程。可以配置下列宏定义

  * PTHREAD_CREATE_JOINABLE：默认配置
  * PTHREAD_CREATE_DETACHED：设定之后不需要pthread_join了

* **schdpolicy:** 线程的调度方式

  * SCHED_OTHER: 默认配置
  * SCHED_RP：时间片流转， 当采用SHCED_RP策略的进程的时间片用完，系统将重新分配时间片，并置于就绪队列尾。放在队列尾保证了所有具有相同优先级的RR任务的调度公平。实时进程将得到优先调用，实时进程根据实时优先级决定调度权值，分时进程则通过nice和counter值决定权值，nice越小，counter越大，被调度的概率越大，也就是曾经使用了cpu最少的进程将会得到优先调度[^1]。 
  * SCHED_FIFO：先到先服务。一旦占用cpu则一直运行。一直运行直到有 更高优先级任务到达或自己放弃。如果有相同优先级的实时进程（根据优先级计算的调度权值是一样的）已经准备好，FIFO时必须等待该进程主动放弃后才可以运行这个优先级相同的任务。而RR可以让每个任务都执行一段时间[^1]。**需要设置实时优先级rt_priority(1-99**) 

  * 注意，设定调度策略之后，同样不需要设定pthread_join了

    ```c
    int max_priority;
    int min_priority;
    struct sched_param sv;
    
    ret = pthread_attr_setschedpolicy(&thread_attr, SCHED_OTHER);
    
    max_priority = sched_get_priority_max(SCHED_OTHER)；
    min_priority = sched_get_priority_min(SCHED_OTHER);
    sv.sched_priority = min_priority;
    ret = pthread_attr_setschedparam(&thread_attr, &sv);
    ```

* **schedparam**: 和schedpolicy结合使用，上边的例子
* **inheritsched:** 调度由属性明确地设置
  * PTHREAD_EXPLICIT_SCHED：默认，明确的设置
  * PTHREAD_INHERIT_SCHED：沿用创建者使用的参数
* **scope**:线程调度的计算方式
  * PTHREAD_SCOPE_SYSTEM: 目前只有这一种取值
* **stacksize**: 设置线程的栈大小，字节，定义_POSIX_THREAD_ATTR_STACKSIZE的时候才会支持该属性的设定。Linux目前支持的线程栈很大，所以这个功能对于linux来说有点多余。

#### 1.4.6 取消线程

接口如下[^3]：

```c
int pthread_cancel(pthread_t thread);
int pthread_setcancelstate(int state, int *oldstate);
int pthread_setcanceltype(int type, int *oldtype);
```

对于setcanceltype:

* PTHREAD_CANCEL_ASYNCHRONOUS：立即取消线程
* PTHREAD_CANCEL_DEFERRED:在接受取消消息请求后，一直等待知道线程执行下属函数之一才会取消
  * pthread_cond_timewait
  * pthread_testcancel
  * sem_wait
  * sigwait

#### 1.4.7 条件变量

除了mutex和spinlock，在pthread层级接口还提供了条件变量，和mutex打配合完成更高效的线程与线程之间的同步。比如这样的场景，小明爸爸每隔三天会向桌子上放一个苹果，小明可以一天吃一个苹果，小明不知道爸爸什么时候来放苹果，所以小明的策略是写作业写一个小时就去桌子上查看一下是不是有苹果，这样效率就十分低。条件变量就可以提供一个通知机制，小明爸爸放苹果之后，通知小明去取，小明再去桌子拿苹果吃，这样显著提高效率。

* 小明爸爸可以使用pthread_cond_signal或者pthread_cond_boardcast来通知小明
* 小明可以持有pthread_cond_wait来等待爸爸的通知。

```c
int pthread_cond_init(pthread_cond_t *cond,pthread_condattr_t *cond_attr);
int pthread_cond_wait(pthread_cond_t *cond,pthread_mutex_t *mutex);
int pthread_cond_timewait(pthread_cond_t *cond,pthread_mutex *mutex,const timespec *abstime);
int pthread_cond_destroy(pthread_cond_t *cond);
int pthread_cond_signal(pthread_cond_t *cond);
int pthread_cond_broadcast(pthread_cond_t *cond);
```

这里说下signal和boardcast的区别，大刚有两个孩子，小明和小红，大刚三天放2个苹果，小红和小明一次拿一个，并且小明和小红使用cond_wait等待着大刚的cond_signal。如果大刚使用cond_signal，那么一次只能通知一个人，而且是轮着来，这样就非常没有效率。但如果大刚使用cond_boardcast，那么小明和小红可以同时收到消息。

关于条件变量的测试：https://github.com/carloscn/clab/blob/master/linux/test_thread/test_thread_cond.c

这里创建了一个生产者和两个消费者，竞争cond状态。

## 2 Topics

我相信，关于多线程会有很多TOPIC值的探讨，这里遇到什么TOPIC都会整理下来。

### 2.1 pthread使用多核

之前我以为多核调度是进程层面的事情，也很好奇，后来查了多方材料，原来pthread在内核层面是支持多核心执行的，具体调度我给Linux内核的学习留白，但在Linux userspace，pthread是提供配置多核运行和获取在哪个核心的接口[^5]。

```C
#define _GNU_SOURCE             /* See feature_test_macros(7) */
#include <pthread.h>
int pthread_setaffinity_np(pthread_t thread, size_t cpusetsize,
                           const cpu_set_t *cpuset);
int pthread_getaffinity_np(pthread_t thread, size_t cpusetsize,
                           cpu_set_t *cpuset);
```

这里也有相应的example来测试线程在哪个核心上面，配置8个核心，会根据实际配置核心输出。

* 注意：_GNU_SOURCE务必define，否则CPU_SET_SIZE这些宏定义找不到

```C
#define _GNU_SOURCE
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>
#define handle_error_en(en, msg) \
        do { errno = en; perror(msg); exit(EXIT_FAILURE); } while (0)
int
main(int argc, char *argv[])
{
    int s;
    cpu_set_t cpuset;
    pthread_t thread;
    thread = pthread_self();
    /* Set affinity mask to include CPUs 0 to 7. */
    CPU_ZERO(&cpuset);
    for (int j = 0; j < 8; j++)
        CPU_SET(j, &cpuset);
    s = pthread_setaffinity_np(thread, sizeof(cpuset), &cpuset);
    if (s != 0)
        handle_error_en(s, "pthread_setaffinity_np");
    /* Check the actual affinity mask assigned to the thread. */
    s = pthread_getaffinity_np(thread, sizeof(cpuset), &cpuset);
    if (s != 0)
        handle_error_en(s, "pthread_getaffinity_np");
    printf("Set returned by pthread_getaffinity_np() contained:\n");
    for (int j = 0; j < CPU_SETSIZE; j++)
        if (CPU_ISSET(j, &cpuset))
            printf("    CPU %d\n", j);
    exit(EXIT_SUCCESS);
}
```

我分别在ubuntu虚拟机和armv7(cortex-A7单核)上面测试得到不一样的结论：

![image-20220331220717666](_media/image-20220331220717666.png)

* 在ubuntu虚拟机上面这里面调度了两个core
* 在armv7a(cortex-A7单核)上面测试只有一个核心

## Ref

[^1]: [inux进程/线程调度策略(SCHED_OTHER,SCHED_FIFO,SCHED_RR)](https://blog.csdn.net/u012007928/article/details/40144089)
[^2]: [archlinux-man-page-pthread-attr-init](https://man.archlinux.org/man/core/man-pages/pthread_attr_init.3.en)
[^3]: [archlinux-man-page-pthread-cancel](https://man.archlinux.org/man/pthread_cancel.3) 
[^4]: [Pthread_Mutex_t Vs Pthread_Spinlock_t （转载](https://www.cnblogs.com/diyunpeng/archive/2011/06/07/2074059.html)
[^5]: [archlinux-man-pthread_setaffinity_np](https://man.archlinux.org/man/pthread_setaffinity_np.3)
