## Lec 1

### Multi-Cycle

- 不同于单周期，单周期是一个指令占用一个周期
- 多周期会把一个指令分成多个部分/阶段，如计算、读写内存等，一个阶段占用一个周期
- 多周期可能劣于单周期，因为在阶段间实用的寄存器太重了

### ISA Classes

#### Stack Architecture

- stack 优点：操作简单、一个指针
- 例如 `c=a+b`——`push a, push b, add, pop c`

#### Accumulator Architecture

- 本质上是在 ALU 上面放了一个类似寄存器的东西，总得称为 accumulator.
- `c=a+b`——`load a, add b, store c`

#### GPR(General-Purpose Register Arch)

- 只有显示的操作
- 操作类型：direct memory access; load into temporary storage first

##### Register-Memory Arch

- any instruction can access memory
- 不同于 risc-v，ALU 的一个源来自寄存器，另一个源来自内存，因此任何操作都可以连接内存
- 连接内存有两条线，一条数据线，一条地址线

##### Load-Store Arch

- 就是 risc-v 的类型
- 所有操作都在寄存器中进行，只有 load 和 store 可以 access memory

目前大多数都是用 GPR，是在多方因素的考虑下的产物

## Lec 2

### Addressing Mode

- Register
- Immediate
- Displacement(of memory)

寻址模式：静态地址、动态地址（PC Relative）

### Little / Big Endian

- 小端序：低位存低位，把数据的最低位存在最低的地址，主流是用这种
- 大端序：低位存高位，把数据的最高位存在最低的地址，这样按照地址从小往大符合自然逻辑

### Data Alignment

- 见 CSAPP 笔记