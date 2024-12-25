## Synchronization

竞争条件：多个进程并发访问和操作同一数据并且执行结果与特定访问顺序有关

为了防止竞争条件，需要确保一次只有一个进程可以操作变量 `count`，为了实现这个目的，我们需要进程按照一定方式进行同步。

解决临界区问题的方法必须满足下面三条要求：

1. Mutual Exclusion/互斥访问：在同一时刻，只有一个进程可以执行临界区（critical section 只能有一个进程在跑）；
2. Progress/空闲让进：当没有线程在执行临界区代码时，必须在申请进入临界区的线程中选择一个线程，允许其执行临界区代码，保证程序执行的进展；
3. Bounded Waiting/有限等待：当一个进程申请进入临界区后，必须在有限的时间内获得许可并进入临界区，不能无限等待。

### Peterson's Method

Peterson 解决方案适用于两个进程交替执行临界区与剩余区。

两个进程共享两个数据项：`int turn; bool flag[2];`

```c
while(true){
	flag[i]=true;
	turn=j;
	while(flag[j]&&turn==j);
	/*critical section */
	flag[i]=false;
	/*remainder section （一般是用于释放进程）*/
}
```

但是对于现代的计算机，指令可能会被重新排序，例如：

线程一执行：`while(!flag); print x;`

线程二执行：`x = 100; flag = true;`

对于线程二，可能会先执行 `flag = true`，导致出现错误。

### 硬件同步支持

#### 内存屏障

- 内存模型：计算机体系结构如何确定它向应用程序提供哪些内存保证。
    - 强排序：一个处理器的内存修改对其他处理器立即可见
    - 弱排序：内存修改可能不会立即背其他处理器看到
- 内存屏障：强制将内存中任何修改传播到其他处理器的指令，当执行内存屏障指令时，系统确保在执行任何后续加载/存储前，先完成现在的加载和存储（相当于在这里打了个标记，我之前和我之后不能交换）

#### 硬件指令

很多现代操作系统提供特殊硬件指令，用于检测和修改字的内容，或者作为**不可中断的指令（原子/Atomically/Uninterruptably） 地交换两个字**，我们可以通过使用这些特殊指令，相对简单地解决临界区问题。

**原子操作的定义**：

- 原子操作是指在执行过程中不会被中断的操作。
- 在多线程环境中，原子操作可以确保一个线程在执行该操作时，其他线程无法同时访问或修改同一资源。

这里使用指令 `test_and_set()` 和 `compare_and_swap()` 抽象了这些指令背后的主要概念。

##### `test_and_set()`

```c
bool test_and_set(bool *target) {
    bool rv = *target;
    *target = true;
    return rv;
}
```

本质上就是返回 `target` 的值，并将其修改成 `true`.

利用 `test_and_set()` 来实现互斥：

```c
do {
    // Entry section
    while (test_and_set(&lock))
        ;   // Do nothing
    // Critical section
    // Exit section
    lock = false;
    // Remainder section
} while (true);
```

倘若某个时刻 `lock` 为 `false`，很可能有不止一个线程调用了 `test_and_set()`，但是由于 `test_and_set()` 是原子的，所以只可能有一个线程返回 `false`。

但是 `test_and_set()` 并不能确保有限等待，下面就是一个例子：

![drawing](https://note.v1ceversaa.cc/System/images/img-OSC/Synchronization-3.png)

谁也不知道 T2 会等到哪里去（这取决于调度），我们可以这样修改代码，这里共用的数据结构是 `bool waiting[n]` 和 `int lock`：

```c
do {
    waiting[i] = true;
    while (waiting[i] && test_and_set(&lock)) ;
    waiting[i] = false;

    // Critical section
    // Exit section
    j = (i + 1) % n;
    while ((j != i) && !waiting[j])
        j = (j + 1) % n;
    if (j == i)
        lock = false;
    else
        waiting[j] = false;

    // Remainder section
} while (true);
```

我们首先将 `waiting[i]` 置为 `true`，然后检查 `lock`，如果 `waiting[i]` 被释放**或者** `lock` 为 `false`，那么就进入临界区，同时上锁（第 3 行）。在退出临界区的时候，我们向下检查后面的线程，寻找下一个正在等待的线程（第 8 到 10 行），若是没有找到（`j == i`），那么就释放锁，这是因为没有线程在等待（第 11 行和第 12 行）；否则，就将 `waiting[j]` 置为 `false`，释放该线程，继续循环，让其进入临界区（第 14 行）。

##### `copare_and_swap()`

`compare_and_swap()` 也是以原子的方式对两个字进行操作，但是是使用基于交换两个字的内容的机制：

```c
int compare_and_swap(int *value, int expected, int new_value) {
    int temp = *value;
    if (*value == expected)
        *value = new_value;
    return temp;
}
```

只有当 `*value` 的值等于 `expected` 的时候，才会将 `*value` 设置为 `new_value`，并且返回 `*value` 的原始值。

互斥的实现也是类似的，但是这里 `compare_and_swap(&lock, 0, 1)` 和 `test_and_set(&lock)` 的逻辑是反的，需要注意一下，但是两条指令**并没有本质区别**。

加上有限等待的实现，这里共用的数据结构是 `bool waiting[n]` 和 `int lock`：

```c
while (true) {
    waiting[i] = true;
    key = 1;
    while (waiting[i] && key == 1) 
        key = compare_and_swap(&lock, 0, 1);
    waiting[i] = false;

    // Critical section
    // Exit section
    j = (i + 1) % n;
    while ((j != i) && !waiting[j])
        j = (j + 1) % n;
    if (j == i)
        lock = 0;
    else
        waiting[j] = false;

    // Remainder section
}
```

x86 实现了 `cmpxchg` 这个指令，可以实现 `compare_and_swap()` 的功能，为了强制执行原子执行，可以使用 `lock` 前缀，使得在目标操作数更新的时候**锁定总线**。汇编的一般形式是 `lock cmpxchg <destination operand>, <source operand>`。ARM 也有类似的指令。

### 原子变量

通常，指令 `compare_and_swap()` 并不直接用来实现互斥，而是作为实现别的工具的基本构建块。例如我们用这个指令，搭建一个**原子变量**，对于这个变量的操作都是原子的。

有界缓冲区问题可以通过实现对整型变量的原子递增递减来解决：