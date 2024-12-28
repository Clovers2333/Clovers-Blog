# Optimizing Program Performance

- CPE：每个元素需要消耗的时钟周期数
- 编译器优化必须基于正确性，所以编译器的优化会很保守，会考虑最坏情况再进行优化。
- 优化很重要的步骤：看汇编

### Reducing Function Calls

- 代码移动：例如将固定的函数值移出循环，避免重复调用；将函数调用转化成直接访问地址（效果其实都是要看是否是瓶颈）

### Reducing Memory References

- 将读写内存操作改为开临时变量——因为一个是在对内存操作，而另一个只是对寄存器进行操作

### Using Modern Processors

- 利用指令并行