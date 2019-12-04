## 概述

###知识储备

系统软件包括：操作系统，数据库管理系统，语言处理程序，服务性程序，标准库程序

并发和共享是操作系统最基本的特征：两者相互依存

BIOS是一组固化到计算机主板上的一个ROM芯片里的程序，是计算机的开启时运行的第一个程序，主要功能是为计算机提供最底层的、最直接的硬件设置和控制。

###OS提供的接口

1：联机和脱机两种命令接口，其中

+ 脱机：类似于批处理程序，一次性写好一次性执行
+ 联机：命令行，输入一句执行一句，涉及到【命令解释器】和【shell】

2：提供给程序的接口叫【系统调用】，又叫【广义指令】

### 发展，类型

单道批处理：最简单串行

多道批处理：中断技术，并发执行，吞吐量高，但不能交互，用户响应时间长

分时操作系统：多个用户同时使用一台计算机，能够人机交互（剥夺式调度进程）

实时操作系统：在严格时限内处理完任务（及时性），可靠性

---

【例】与单道程序系统相比，多道程序系统的优点是

系统开销小（错误：需要额外的开销来组织，调度作业

IO设备利用率高（√）

---

【例】采用（优先级+非抢占式调度算法）来缩短分时系统的系统响应时间

优先级：使得重要的作业能及时响应

非抢占：不重要的作业也不会因为被抢占而迟迟不能处理完毕

---

【例】多任务操作系统具有并发和并行的特点（√，并发执行进程；CPU与IO并行，CPU与通道并行）

### 核心/用户态

CPU执行不同权限指令时的状态

**核心态**：特权指令

+ I/O指令
+ 置中断指令
+ 存取用于内存保护的寄存器的指令
+ 送程序状态字到寄存器的指令
+ 与硬件关联紧密的模块
    + 时钟管理：处理和时间有关的信息，比如系统时间、进程时间片、延迟、CPU使用时间、各种定时器
    + 中断处理程序
    + 设备驱动
+ 运行频率较高的程序
    + 进程管理程序
    + 存储器管理程序
    + 设备管理程序
    + 原语：原子操作指令，不可被中断的（执行原语是会关闭中断功能）

**用户态**

+ 命令解释程序
+ 调用系统调用
+ 发生缺页
+ 发生外部中断



用户态进入核心态是通过**中断和异常**，这一过程由【硬件】完成；由核心态返回用户态是【操作系统】完成的

+ 中断：也叫【外中断】

    + 时间片中断
    + IO结束中断

+ 异常：也叫【内中断】，源自CPU执行的指令产生的错误，不能被屏蔽，一旦出现要立即处理，处理要依赖当前程序的运行现场

    + 地址越界
    + 虚存缺页
    + 专门的“陷入”指令，访管指令或 trap 指令
    
    

**系统调用的执行过程**：用户进程传递系统调用参数，通过 trap 指令主动让权给内核（进入内核态）这样产生的中断叫【访管中断】，将返回地址压入堆栈，执行被调用的内核服务程序，完后恢复程序现场返回用户态

用户态进入核心态后，使用的堆栈也由用户堆栈切换为系统堆栈，但系统堆栈属于该进程

----

【例】中断处理程序保存：<u>程序的断点（PC），程序状态字寄存器（PSW）</u>，子程序调用需保存：<u>程序的断点（PC）</u>，外部中断处理过程中，PC值由中断隐指令自动保存，操作系统保存：<u>通用寄存器</u>。

---

## 进程管理

### 进程相关概念

进程实体=程序段+相关数据段+PCB（进程控制块）

进程控制用的程序段是【原语】



**状态**

创建过程：

1. OS分配进程标识符PID，申请PCB，如果申请失败，则进程创建失败
2. 复制父进程的环境
3. 分配资源（堆栈，文件…），如果分配失败进程处于阻塞态/等待态
4. 复制父进程空间里的内容
5. 将进程设置为就绪状态，放入就绪队列



就绪态：进程已经获得了除了处理机之外的一切所需资源，一旦得到处理机后就可以运行

运行态：在单处理机环境下，创建了N个进程，每一时刻最多只有N-1个进程处于就绪状态，最多有 1 个处于运行态，最少有 0 个处于运行态（比如死锁导致全部阻塞）

阻塞态：进程正在等待某一事件（某种资源的获取、IO…）而暂停运行



终止过程：

1. 根据终止进程的进程标识符，检索PCB，查看进程状态
2. 若该进程有子进程则全部递归终止（即使正在运行也立即终止
3. 将该进程的所属资源还给父进程或者OS
4. 将该进程的PCB从PCB集合中移除

**异常终止**：存储区越界；保护错；非法指令；特权指令错；I/O故障等



**状态转换**

![在这里插入图片描述](assets/20190706002405844.jpg)

就绪态→运行态：被调度，进程切换

+ 保存CPU上下文，包括程序计数器和其它寄存器
+ 更新PCB信息，并将PCB移入就绪或等待队列
+ 通过调度算法选择另一个进程，更新PCB并执行
+ 更新内存管理的数据结构
+ 恢复处理及上下文



运行态→就绪态：运行时间片结束；可剥夺的操作系统中被更高优先权进程抢占

运行态→阻塞态：等待某一资源（如外设）或某一事件发生（如I/O操作完成）系统调用中断【主动行为，自己调用Block原语，即只有正在占有CPU的进程才能转换成阻塞态】

+ 检索PCB
+ 保护现场，更改状态，停止运行
+ 将该PCB插入相应事件的等待队列



阻塞态→就绪态：I/O操作结束，中断结束【被动的，需要其它进程协助，中断处理程序把进程变成就绪态，调用Wakeup原语】

+ 检索PCB
+ 把PCB从该事件的等待队列移出
+ 插入就绪队列，等待被调度

---

【例】就绪队列不空，就绪的进程数目越多，CPU的效率<u>不变</u>（只要就绪队列不空，CPU总是可以调度进程并运行，与就绪队列的大小无关）

【例】以下需要创建新进程的是

+ 设备分配【×】在系统中设置相应的数据结构即可，不需要启动进程
+ 用户登录【√】开机的时候登陆账户，要启动用户桌面、配置用户环境等

---



**进程控制块**：由系统维护，常驻内存，系统中的PCB数量是有限的，有两种组织形式

+ 链接：同一状态的PCB链接成一个队列，阻塞状态由于阻塞原因不同会有多条队列
+ 索引：同一状态的PCB在一个索引表里

---

【例】若一个进程实体由

B：共享正文段

C：数据堆段

D：数据栈段

四部分组成，则下列C语言程序中的数据位于哪一部分

未赋值的局部变量（D）

调用函数传递的实参（D）

malloc 动态分配的区域（C）

const 常量（B）

全局变量（B）

二进制代码（B）

---

###进程间通信

用户进程空间是独立的，进程间不能直接相互访问对方的进程空间

**共享存储**：通过特殊的系统调用，对共享的空间进行读写，这个过程中需要借助同步/互斥工具（P，V操作）

+ 低级方式：基于数据结构的共享
+ 高级方式：基于存储区的共享



**消息传递**：由系统提供发送，接收两个原语来传递格式化的信息

+ 直接通信：发送进程直接把消息挂在接收进程的消息缓冲队列上，等待接收进程取
+ 间接通信：发送进程把消息发给中间实体（信箱），接收进程从信箱取信息



**管道**：用于连接一个读进程和写进程的固定大小的共享文件（pipe文件/缓冲区），视为共享存储的优化

+ 是一种半双工通信，一次只能一个方向
+ 当缓冲区为空，才写入，写满时写操作阻塞
+ 当缓冲区满了，才读出，数据被读取就从缓冲区中删除，所以读完数据缓冲区变空，继续写操作

### 线程

同属一个进程的线程共享该进程所有资源，线程之间可以相互创建撤销，通信不需要操作系统干预

线程是：CPU执行的基本单元，程序执行流的最小单元，独立调度的基本单位，拥有自己的CPU现场

进程是：除了CPU外系统资源分配的基本单元

线程也有【线程控制块】一个进程的多个线程分布在多个CPU上同时执行可以加快进程执行速度

**分类**

用户级线程：应用程序自己维护，内核不管

内核级线程：由内核进行调度维护的

**模型**

+ 多对一：多个用户级线程映射到一个内核级线程，线程管理在用户空间进行，效率较高，但当一个线程使用内核服务时被阻塞会导致整个进程阻塞，同时这多个用户线程并不能并行在多核CPU上

+ 一对一：并发强，但开销大，会影响程序性能
+ 多对多：M 个用户级线程映射到 m 个内核级线程（M≥m），集前两者之所长，优点多

---

【例】系统动态dll库的系统线程，被不同进程调用时，它们是（相同）的线程

（同一个系统线程可以由系统调用被多个不同进程多次调用）



【例】降低进程优先级的合理时机是

A：时间片用完【√】降低优先级让其它进程被调度执行

B：进程刚完成I/O操作，进入就绪队列（为了让其尽快处理I/O结果应该提高优先级）

C：进程长期处于就绪队列（为了防止饥饿现象应该提高优先级）

D：进程从就绪态转为运行态（运行态的进程不应变动优先级）



【例】父进程创建子进程与主程序调用子程序有什么不同？

答：父子进程可以并发执行，而主程序要停在调用原点，子程序开始执行，返回结果后，主程序再继续执行

---

### CPU调度

**作业的三级调度**

+ 作业调度：又叫高级调度，从外存上处于后备状态的作业中挑选一个，为其分配内存及各种资源，建立相应的进程。本质上是内存与辅存之间的调度，每个作业只调入一次，调出一次。
+ 中级调度：又叫内存调度，将暂时不能运行的进程调至外存【挂起】，直到内存或资源满足运行需要后再调回内存，改状态为【就绪】
+ 进程调度：又叫低级调度，分配CPU给就绪进程，频率最高
    + 在以下条件下不能进行调度
        + 中断处理过程中
        + 进程处于操作系统内核程序的临界区中
        + 进行原子操作时
    + 在以下条件下能调度
        + 进程处于临界区中：有些临界区（比如打印机I/O）只要不破坏临界区访问规则，就可以进行调度
        + 当前程序无法进行，或时间片终止（非剥夺式）
        + 中断/trap处理结束后，置请求调度标志，则可不恢复到原进程，立即调度（剥夺式）



**性能指标**

等待时间：作业在就绪队列中等待的时间，是衡量调度算法优劣的性能指标

周转时间：= 作业完成时刻 - 作业提交时刻，是一个总时间，影响因素很多

响应时间：= 首次产生响应的时刻 - 用户提交请求的时刻，是交互式系统关心的指标

系统吞吐量：单位时间完成的作业数量，短作业更有优势

---

【例】某系统进程调度和进程切换总开销为 1μs，某时刻就绪队列有一个进程，相关信息如下，求该进程的周转时间

| 进程 | 等待时间/μs | 需要的CPU时间/μs |
| ---- | ----------- | ---------------- |
| P    | 30          | 12               |

答：1 + 30 + 12

这里要说的是，考研中，从这一时刻开始，意味着初始的调度时间也要算进去，所以最前面有一个 1 别忘了

---

**调度算法**

+ 先来先服务（FCFS）：（进程&作业调度）

    + 有利于CPU繁忙型的作业和长作业
    + 不利于IO繁忙型的作业和短作业

+ 短作业优先（SJF）：（进程&作业调度）

    + 平均等待时间，平均周转时间最少
    + 长作业迟迟得不到调度会饥饿

+ 优先级调度：（进程&作业调度）

    + 动态/静态优先级
    + 剥夺/非剥夺
    + 优先级高低排序：系统进程＞用户进程；交互型＞非交互型；I/O型＞计算型

+ 高响应比优先：对SJF和FCFS综合平衡

    $$
    响应比R_p=\frac{等待时间+要求服务的时间}{要求服务的时间}
    $$

    + 当等待时间相同时，短进程优先
    + 当进程长短一样时，先来的优先，克服饥饿
    
+ 时间片轮转：每次结束一个时间片就切换进程，剥夺式的

    + 若时间片很大，每个进程在一个时间片内都能完成，就退化成FCFS

+ 多级反馈队列：（究极融合版）动态调整进程优先级和时间片大小

    + 第一级：FCFS，优先级最高，时间片最短
    + 第二级：FCFS
    + ··· ···
    + 第 n 级：时间片轮转
    + 新进程进入内存，首先放在第一级队列末尾，如果一个时间片没完成，就降到第二级，以此类推逐级下降
    + 当第 i 级队列全部执行完毕，才执行第 i+1 级，若 i > 1且此时有新进程进到第一级，则新进程抢占CPU

###临界资源

**临界资源**：一次只能被一个进程使用的资源

**临界区**：进程中访问临界资源的代码



基本原则

+ 空闲让进：临界区空闲时，进程想进就进
+ 忙则等待：临界区被占有时，其它想进来的进程必须等待
+ 让权等待：在等待临界区的进程，就不要再占有CPU了，防止忙等待
+ 有限等待：等临界区的进程也不能等太久，必须要在有限时间内进入临界区



**软件实现法**

+ 单标志位：用一个 turn 标记允许进入临界区的进程号

    + turn = 1，P1进入，随后 turn = 0，但此时 P0 并不想进入，而 P1 也被挡在外面。违背了空闲让进

+ 双标志位先检查：两个进程都先检查对方的占有标志位，如果对方不占有，再把自己的占有位设为 true

    + 两个进程在占有前同时检查，都发现对方不在，然后一起进入。违背了忙则等待

+ 双标志位后检查：两个进程都先把自己的占有位设为 true，再检查对方的占有位，如果对方不占有就进入

    + 两个进程在检查前同时设置自己的占有位，结果检查的时候都发现对方正在占有，于是无限等待。导致饥饿，违背了有限等待

+ Peterson：使用 turn 来解决饥饿现象，使用 flag 来实现互斥访问

    ```c
    /*Pi进程*/
    flag[i] = true;
    turn = j;
    while(flag[j] && turn = j);
    critical section;
    flag[i] = false;
    
    /*Pj进程*/
    flag[j] = true;
    turn = i;
    while(flag[i] && turn = i);
    critical section;
    flag[j] = false;
    ```

    + 甲：我来啦 flag[甲] = true；乙：我来啦 flag[乙] = true
    + 乙：你先 turn = 甲
    + 甲：你先 turn = 乙
    + 乙：好的我先【精髓是只让一次，先让者先用】
    + 甲等待中 while(flag[乙] && turn = 乙);
    + 乙：我好了 flag[乙] = false
    + 甲：我来了 while 循环跳出



**硬件实现方法**

+ 关中断：简单粗暴，没有中断进程就不切换，用户程序具有控制中断的能力是一件很恐怖的事情，整个系统都会无法运行

+ 硬件指令法：原语指令，由硬件逻辑实现，不被中断

    + TestAndSet(&lock)：读出 lock 的当前值，并把 lock 设置为 true；即正在占用，又叫 TSL(&lock)

        ```c
        while TestAndSet(&lock)； // 检查临界段的标志位 lock，初值为 false
        ```

    + Swap(&lock, &key)：交换两个字节的内容

        ```c
        key = true;
        while(key != false)
            Swap(&lock, &key);	// 用于捕捉 lock = flase 的一瞬间
        ```

    + 缺点一：注意到进程在等待时，while循环里，依然在耗费CPU，处于忙等待状态，违背了让权等待

    + 缺点二：进程只有在被调度进CPU后，才能执行它的while，如果进程一直不被调度，就饥饿了



### 信号量

注意到前述互斥访问临界区时，处于等待状态的进程主动 check while 循环条件，换句话说，处于等待状态的进程还会被CPU执行，占有CPU资源。有没有什么办法能让它们不被调度，然后当它们的临界区空闲后，有一个程序再去唤醒等待这个临界资源的进程。

```c
typedef struct{
    int value;		   // 该临界资源的数量
    struct process *L; // 正在等待该临界资源的进程链表
} semaphore;

// P 操作
void wait(semaphore S){
    S.value --;
    if(S.value < 0){
     	add this process to S.L;// 尾部
    	block(S.L); 	// 主动让权
    }
}

// V 操作
void signal(semaphore S){
    S.value ++;
    if(S.value <= 0){
     	remove head process P from S.L;// 首部，FIFO
    	wakeup(P); 	// 唤醒
    }
}
```

信号量的值，即 S.value 在 > 0 时，就是可用资源的数量，在 ≤ 0 时，是正在等待该资源的进程数量

一个进程对某一信号量进行 P 操作后进入临界区，退出临界区后，依然是该进程进行 V 操作



**PV操作的伪码**

关键：设置信号量，并赋值

+ 找到互斥点，设置相应的互斥信号量 mutex，初值必为 1；或数组 semaphore chopstick[5] = {1, 1, 1, 1, 1}
+ 设置资源数量信号量，semaphore empty = n；semaphore full = 0



**读者-写者问题**

> 描述：一组读进程和一组写进程，写进程与写进程互斥，写进程与读进程互斥，读进程之间可以共享文件
>
> 设计一个读写公平的程序

思考

+ 首先对于写进程，互斥就完事儿了，需要一个 rw
+ 对于读进程，要考虑本身进程与写进程的互斥，还要考虑本身进程与其它读进程的共享，不能我自己读完了就可以把控制权交给写进程，还要看看此时还有没有其它读进程在读，当读进程的数量为 0 时，才能移交控制权。所以需要设置一个计数变量 count，用于统计读进程数
+ 对于这个计数变量的操作，为了保证计数的正确性，也要设置一个互斥信号 mutex

伪码如下

```c
int count = 0;
S mutex = 1;
S rw = 1;

writer(){
    while(1){
        P(rw);			// 读写互斥
        write;
        V(rw);
    }
}

reader(){
    while(1){
        P(mutex);
        if(count == 0)	// 第一个读进程锁上读写互斥信号
            P(rw);
        count++;		// 读进程进场的标志
        V(mutex);
        
        read;
        
        P(mutex);
        count--;		// 读进程退场的标志
        if(count == 0)	// 最后一个读进程释放读写互斥信号
            V(rw);
        V(mutex);
    }
}
```

到目前为止，确实能正常工作，但存在一个问题，即当有一个进程在读的时候，所有的写进程都会等待，哪怕是先来的写进程，而其它后到的读进程可以自由读取。如果有源源不断的读进程，那么写进程会”饿死“，不符合”读-写公平“的原则。

所以要做到当文件正在被读/写时，下一个访问文件的进程应该按照先来后到的顺序，读进程先来那么就读，写进程先来就写，还需要一个互斥信号量表示”下一个访问权在我手里“ next = 1，先来的进程，P 掉 next

```c
int count = 0;
S mutex = 1;
S rw = 1;
S next = 1;

writer(){
    while(1){
        P(next)		// 手握下一个访问权
        P(rw);
        write;
        V(rw);
        V(next)		// 放出下一个访问权
    }
}

reader(){
    while(1){
        P(next)		// 手握下一个访问权，如果没握住就等，握住了才能有count++表示成功进场
        P(mutex);
        if(count == 0)
            P(rw);
        count++;
        V(mutex);
        V(next)		// 放出下一个访问权
        
        read;
        
        P(mutex);
        count--;
        if(count == 0)
            V(rw);
        V(mutex);
    }
}
```





**管程**

是：一组数据以及对这组数据的操作组成的软件模块，封装了同步/互斥操作，对进程隐蔽了同步细节，简化了同步功能的调用接口。使编写并发程序和写串行一样简便



提出的背景：

信号量机制的缺点：进程自备同步操作，P/V操作大量分散在各个进程中，不易管理，易发生死锁



特点：

- 管程内的数据只能由管程内的方法访问与操作，外部进程无法直接访问
- 每次仅允许一个进程在管程内执行某个方法
- 管程的互斥访问完全由编译程序在编译时自动添加，保证正确，不是进程无法被创建和撤销



目的：

+ 把分散在各进程中的临界区集中起来进行管理
+ 防止进程有意或无意的违法同步操作
+ 便于用高级语言来书写程序，也便于程序正确性验证



### 死锁



**产生的原因**

+ 互斥：进程对所需资源进行排它性控制
+ 不剥夺：进程已经获得的资源不会被剥夺，只能自己主动释放
+ 请求并保持：进程保持住了一个资源后，又对其它资源提出了请求
+ 循环等待：进程之间循环等待资源释放



**预防**

从破坏产生死锁的原因入手

- 破坏互斥：不可行，有的资源必须互斥
- 强行剥夺：需要通过额外开销来保存现场，否则会导致进程前功尽弃。常用于易于保存和恢复现场的资源，如CPU的寄存器以及内存资源
- 破坏请求并保持：静态分配资源，进程运行前一次性占有它所需的所有资源。资源严重浪费
- 破坏循环等待：顺序资源分配法，系统没所有资源编号，每个进程只能按编号递增的顺序请求资源，阻止环的形成。编程麻烦

**避免**

在资源动态分配的过程中，防止系统进入不安全状态

不安全状态不一定都会变成死锁，但安全状态一定没有死锁

什么是安全状态：有一堆进程，它们已占有了一些资源，系统还剩下一些资源，我如果能找到一个合适的顺序，按照这个顺序把资源依次分配给这些进程（每个进程结束会释放它占有的资源，所以资源数量随着进程的结束会越来越多），那么这个顺序就叫【安全序列】，当前处于安全状态

银行家算法

核心：多个进程，多个资源的 Need 矩阵，与当前可用资源向量 Available 的比较

思路：试探性的分配，如果分配后能找到安全序列，则此次分配安全，否则拒绝分配

**检测及解除**

资源分配图：

![1575361736973](assets/1575361736973.png)

+ 大圆圈：代表一个进程
+ 方块：一类资源
+ 方块里的小圆圈：一个资源
+ 进程到资源的箭头：请求边，进程正在请求一个该类资源
+ 资源到进程的箭头：分配边，进程已拥有一个该类资源



**死锁定理**（检测）：资源分配图不可完全简化时，当前状态为死锁

资源图简化：找到能正常运行的进程，把该进程的出边和入边删除，直到所有的边被消去，则完全简化



死锁解除

+ 资源剥夺：抢占资源
+ 撤销进程：按优先级撤销进程
+ 进程回退：进程主动释放资源，要求保存进程历史信息，设置还原点

---

【例】系统产生死锁的原因可概括为

独占资源分配不当（√）

系统资源不足（×）资源不足导致的是饥饿，根本就跑不了

【例】”死锁“与”饥饿“的区别

答：死锁是相互的，是多个进程互相锁住，而饥饿是指单个进程因各种原因一直不能执行

---

## 内存管理

###程序运行的步骤

+ 编译：将源代码编译成目标文件，比如 .obj
+ 链接：将目标文件与所需的库函数链接在一起，形成装入模块，**产生整个程序完整的逻辑地址空间**，产生 exe 文件
    + 静态链接：在程序运行之前，先将各目标模块及它们所需的库函数链接成一个完整的可执行程序，以后不再拆开。
    + 装入时动态链接：在装入内存时，釆用边装入边链接的链接方式。
    + 运行时动态链接：对某些目标模块的链接，是在程序执行中需要该目标模块时，才对它进行的链接。其优点是便于修改和更新，便于实现对目标模块的共享。
    + 动态链接与程序的逻辑结构有关，所以段式存储有利于动态链接
+ 装入：双击exe，将装入模块调入内存运行
    + 绝对装入。在编译时，就知道程序将驻留在内存的哪个位置，产生绝对地址的目标文件。按照地址直接装入。
        + 这里程序中的逻辑地址与实际内存地址完全相同，故不需对程序和数据的地址进行修改。
    + 可重定位装入：又称为静态重定位，每个模块内部从 0 号单元开始编逻辑地址，将模块装入到内存的适当位置时，对目标程序中指令和数据重定位，地址变换通常是在装入时一次完成的。
        + 作业装入内存时，必须分配其要求的全部内存，如果内存不够，就不能装入
        + 作业一旦进入内存，在整个运行期间不能在内存中移动，也不能再申请内存空间。
    + 动态运行时装入，也称为<u>动态重定位</u>，装入程序在把装入模块装入内存后，模块中还是相对地址，到程序真正要**执行时**才地址转换（动态重定位），这种方式需要**一个**重定位寄存器的支持。这种程序在装入后还可能被换出，所以同一模块的物理地址可能会变
        + 动态重定位的特点是可以将程序分配到不连续的存储区中；
        + 在程序运行之前可以只装入它的部分代码即可投入运行，然后在程序运行期间，根据需要动态申请分配内存；
        + 便于程序段的共享，可以向用户提供一个比存储空间大得多的逻辑地址空间。

+ 运行：需要界地址寄存器（包含最大逻辑地址）和重定位寄存器，首先界地址寄存器判断逻辑地址是否越界，若没越界则通过重定位寄存器计算得物理地址
    + CPU调度程序选择进程执行时，派遣程序会初始化 界地址寄存器 和 重定位寄存器
    + 动态重定位，地址转换

### 覆盖和交换

覆盖：内存中有一个覆盖区，运行一个大型进程时，执行的部分调入内存的覆盖段，没执行的部分在外存。所以当正在运行的部分大于内存时，程序还是跑不了

交换：把处于等待状态或时间片刚过的程序从内存【换出】到外存，把另一个准备竞争CPU的进程【换入】

+ 磁盘上有一个交换区用于交换，这一区域独立于文件系统，使用起来可以很快
+ 交换需要备份储存内存映像
+ 换出的进程是完全处于空闲状态的

### 连续分配

+ 单一连续分配：用于单用户单任务的系统中，内存分为系统区和用户区，一次就只有一个进程在用户区，啥算法都不要，简单，无外碎片，有内碎片，内存不够可以使用覆盖技术
+ 固定分区分配：将用户区分成大小相等的或大小不等的许多小分区，每个进程占一块分区，这就衍生出了问题
    + 进程太大一个小区域放不下
    + 进程太小没有放满一个分区【内碎片】，内存空间利用率低
    + 进程装入后就不会改变位置了，可以采用静态重定位
+ 动态分区分配：并不事先划分好小分区，而是根据装入进程的大小动态建立分区。缺点很明显
    + 随着进程的换入换出，内存中会出现越来越多的小碎片【外碎片】，解决方式是”紧凑“，相对费时
    + 使用 ”拼接技术“ 对空闲区合并，另外在紧凑会使得程序在运行过程中的地址发生变化
    + 分配策略：当有多个合适的空闲块，选择哪一个块分配给将要进来的进程呢
        + 首次适应：地址递增的顺序查找，找到第一个即可【最好，最快的】【外碎片集中在低地址】
        + 最佳适应：按容量递增，找到第一个合适的【外碎片最多】
        + 最坏适应/最大适应：按容量递减的顺序，即找最大的
        + 邻近适应：循环首次适应算法，从上次查找结束的位置开始继续查找 ”首次适应“【外碎片集中在内存的末尾】

---

【例】分区分配内存管理方式的主要保护措施是（A）

A：界地址保护

B：程序代码保护

C：数据保护

D：栈保护

---

###非连续分配

####页式

把进程和内存都分成相同大小的块，进程中的块称为页 page，内存中的块称为页框 page frame，页与页框对应。会产生页内碎片，但无外碎片

每个进程会有一张页表，页表项记录了该进程逻辑页号所对应的物理块号。要覆盖全部逻辑地址的页

逻辑地址由页号和页内位移组成

+ 页号是页表中的页表项的号，页表项里面是物理地址的页框号
+ 页内位移是页框内的位移，具体字节

![1575372756493](assets/1575372756493.png)

20位的页号表示逻辑地址页数为$2^{20}$，页表项的大小至少是20位（3B，通常取4B），12位的页内偏移表示页的大小为$2^{12}$Bytes。

主存储器的访问以字节为单位

根据逻辑地址取一个数或一条指令要两次访存

+ 第一次访问页表
+ 第二次根据物理地址访问块

优化：增设一个高速缓冲器——快表（TLB）快表不在内存中。如果命中快表，就直接拼接物理地址访问块，一次访存即可

![1575370452737](assets/1575370452737.png)

二级页表

就上述32位逻辑地址，12位页内偏移，一页4K，其一级页表本身有 $2^{20}$ = 1M 页表项，即页表有4M = 1K页，挺大的，能不能为这1K 个页再建一个页表，于是乎二级分页出来了

如何划分二级分页逻辑地址结构呢？

+ 首先有一个规则是多级分页中的顶级页表最多占一个页框

+ 在这里一个页框 4KB，一个页表项是 4B，所以二级页表最多有 1K 个页表项，即通过二级页表可以索引 1K 个一级页表。所以一级页号要有 10 位，用来定位一级页表所在的框
+ 找到了一级页表所在的页框后，就要进一步找到这个一级页表里面的页表项，因为这个页表项里面有正式的物理块号。那么一个页框 4KB，即 1K 个页表项，所以还需要 10 位的二级页号来定位页表项
+ 10 + 10 + 12 = 32 完美~

![1575374982183](assets/1575374982183.png)

![1575375028443](assets/1575375028443.png)

---

【例】页表的起始地址放在（B）中

A：内存

B：寄存器

解释：页表的功能由一组专门的存储器实现，其始地址放在页表基址寄存器（PTBR）中，快速完成地址映射



【例】操作系统采用分页式管理时，要求（A）

A：每个进程一张页表，且进程的页表留在内存中

B：每个进程一张页表，且只有执行进程的页表留在内存中

多进程并发执行时，系统中只设置一个页表寄存器（PTR），存放页表在内存中的始地址和长度。进程未执行时，页表的始地址和长度放在PCB中，当进程被调度执行时，才把这两个数据调入PTR

---

####段式

分页通过硬件机制实现，对用户完全透明

分段方式在用户编程时就确定了

物理上：段间离散，段内连续【没有内碎片，有外碎片】；给出的逻辑地址是二维的【段号，段内位移】

段表项两个内容

+ 段长：这个段的长度
+ 物理基址：这个段在内存的地址

两个越界检查

+ 段号要小于段表长
+ 端内位移要小于段长



**段的共享**：不能修改的代码称为【纯代码】或【可重入代码】不属于临界资源



#### 段页式

页式：有效利用内存

段式：反映程序逻辑结构，有利于段的共享

段页式

+ 程序分段，段内分页；一个程序一个段表，每个段一个页表
+ 内存分页

逻辑地址结构：【段号S + 页号P + 页内偏移W】

地址映射过程

+ 根据段号在程序的段表中查找到 S 号段表项，这个段表项里有 S 段的页表首地址
+ 根据找到的页表首地址 + 页号P 得到 P号页表项，页表项里有物理块号
+ 合并找到的物理块号 和 页内偏移W 得到最终物理地址
+ 3 次访存

优化：快表【段号，页号，页帧号，保护码】

### 虚拟内存

操作系统将暂时不用的内容换出到外存上，把要用的页调进内存的**驻留集**中，这样系统好像为用户提供了一个比实际内存大的存储器。存储器的大小由系统的地址结构决定。

允许作业在运行过程中，进行换进和换出

实现方法：在之前的分页、分段、段页式的基础上增加了【请求调页】功能和【页面置换】功能

新页表项结构如下

| 物理块号 | 状态位P          | 访问字段A              | 修改位M  | 外存地址         |
| -------- | ---------------- | ---------------------- | -------- | ---------------- |
|          | 该页是否调入内存 | 访问次数或未访问的时间 | 是否修改 | 调入该页时，参考 |

请求访问某一页的基本过程

+ CPU检索快表，若命中快表，修改访问位（写指令：和修改位），合并物理地址，万事大吉
+ 该页不在快表里，就访问内存，若在内存里，更新快表，修改访问位（写指令：和修改位），合并物理地址
+ 内存里也没有这个页，产生缺页中断，保存CPU现场，转入缺页中断处理程序：
    + 从外存中找到该页，若内存未满，启动IO，将该页调入内存，修改页表，回到最初的起点，检索快表
    + 若内存满了，就要【选择内存中的一页换出 ①】，如果要换出的页被修改过了，则将该页写回外存
    + 没被修改过，则直接启动IO，将该页调入内存将它覆盖掉，修改页表，回到最初的起点，检索快表



**页面置换算法**：选择内存中的哪一页换出

+ Optimal（OPT）：选择最长时间内不再被访问的页面，理想最佳情况，用来评价其它置换算法
+ FIFO：选驻留时间最久的页。【有Belady异常：当分配的物理块变多，缺页次数不减反增】
+ LRU：过去一段时间最久没有被访问过的页面，性能较好，要寄存器和栈的硬件支持，是堆栈类算法【堆栈类算法不会有Belady异常】
+ 时钟（CLOCK）置换：又称为最近未用（NRU）
    + 将所有帧围成一个圈，每个帧有一个使用位 u，刚进入的新帧 u = 1
    + 请求一个页，命中的话，把圈圈里的这个页的 u = 1，其它屁事没有
    + 否则的话，缺页中断处理，开始转动罪恶的指针，指针经过的帧如果 u == 1的话，u就变成0，如果 u是0的话，那就它了，换入的页就占这个位置，然后**指针往后移动一位**
+ 改进后的（CLOCK）：在考虑了使用与未使用的基础上，还考虑了修改与未修改，由于每次替换已修改的页都需要写回，所以在未使用的页里，优先选择调出未修改的页。再添加一个修改位 m = 0，发生修改时 m = 1
    1. 缺页处理时，先转一圈，挑出遇到的第一个 u=0, m=0 的崽，就它了
    2. 如果挑不出这样的，再转罪恶之第二圈，抓到一个 u=1，m=0 的崽，转这圈时，指针经过的帧 u == 1都变成0
    3. 如果还是找不到，没事，这时圈里只会有(u=0, m=0)和( u=0，m=1)的崽，再回到第 1 步即可



**比较**：LRU要维持一个线型链表，要改动结构，而NRU维持一个稳定的环状，改动的只是一个使用位，代价更低



**驻留集分配策略**

+ 固定分配局部置换：每个进程固定数量的物理块
+ 可变分配全局置换：每个进程初始分配物理块，运行时每次缺页，从系统中分配一个块加进去
+ 可变分配局部置换：每个进程初始分配物理块，运行时当缺页率过高，就从系统中加一个块进去，当缺页率过低，则可以减少该进程的物理块



**两种调页时机**

+ 请求调页：运行时一次只调入一页

+ 预调页：运行前一次调入若干相邻页

通常两种方式一起使用



**从哪里调页** —— 外存 = 交换区 + 文件区

交换区是连续分配的方式，文件区是离散分配，所以交换区的磁盘I/O更快

+ 系统有足够的交换区，那么在程序运行前，就需要将与程序有关的文件全复制到交换区
+ 系统交换区不够：对于没有被修改过的页，直接从文件系统调入，调入后被修改了，那调出时要调到交换区内。



**抖动**：<u>某个程序频繁访问的页面</u>数，多于可用的物理块数，则不得不频繁的进行页面的换入换出，占用大量CPU时间，这就叫抖动

某个程序频繁访问的页面叫【工作集】，所以如果不想发生抖动，分配给进程的驻留集要大于工作集

【工作集】的确定：由 时间 t 和 工作集窗口大小 S 确定，将 S 内的所有的页去重就得到 工作集

比如：t 时刻，S 中的内容是[ 2，3，5，3，2]，则工作集为{2，3，5}