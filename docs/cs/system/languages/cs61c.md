# RISC-V

> 为了这个语言专门开了 cs61c 这个坑。

## Basic Introduction

### What is ISA

- Instruction Set Architecture, an abstract model that generally defines how software controls the CPU in a computer or a family of computers.
- In general, an ISA defines the supported instructions, data types, registers; the hardware support for managing main memory, fundamental features (such as the memory consistency, addressing modes, virtual memory), and the input/output model of implementations of the ISA.
- The level between software & hardware.

### CISC & RISC

- CISC: Complex Instruction Set Computing
    - difficult to learn
    - less work for the compiler
    - complicated hardware runs more slowly
- RISC(Reduced): Simpler/smaller instruction set

### What is RISC-V?

- Open-source Instruction Set specification
- Appropriate for all levels of computing
    - 32 / 64 / 128-bits
    - microcontrollers / supercomputers

- [RISC-V / x86 / ARM 之间的对比](https://www.sohu.com/a/736796607_121097259)

![](https://github.com/Clovers2333/picx-images-hosting/raw/master/risc_arm_x86.pf5rgb89y.webp)

- Little Endian: 小端序，一个多位数，低位存小地址，高位存大地址（和正常阅读顺序相反）

### How to Use Green Card?

- 本节课使用 32 位机，greencard 中的以 `w` 为后缀的是为了 >32 位机准备，使得操作统一在 32 位。
- `FMT` 是指操作系统把汇编语言转化为机器语言的方式，具体可见 greencard 中的右下角。

## Registers

- The more registers you have, the slower it is to access any of them.
- number: 32, x0~x31.

![](https://github.com/Clovers2333/picx-images-hosting/raw/master/riscv_registers.3d4m1tz1eg.webp)

![](https://github.com/Clovers2333/picx-images-hosting/raw/master/riscv_registers2.8s34tyaj59.webp)

- Registers have no type.
- **Register zero(x0) always has the value 0 and cannot be changed.**

## Basic Instructions

### Arithmetic Instructions

- `op dst, src1, src2`：`dst = src1 <op> src2`.
- `opi dst, src, imm`：
    - `addi t1, s1, 5`：`a = b + 5`.
    - 没有 `subi`：因为完全可以通过加负数来实现（但是 `sub` 是有的！！！）
- `mul dst, src1, src2;  mulh dst, src1, src2`：`src1*src2`，第 32 位用 `mul` 计算，高 32 位用 `mulh` 计算.
- `div dst, src1, src2;  rem dst, src1, src2`.

### Compare Instructions

- `slt dst, reg1, reg2`：`dst = (reg1 < reg2)`
- `slti dst, reg1, imm`. 

### Data Transfer

`memop reg, off(bAddr)`

- access memory at `bAddr+off`.
- **Load Word(lw)**: Takes data at address `bAddr+off` **from** the memory and place it into reg.
- **Store Word(sw)**: store the reg to the address.
- word: 32-bits, 4 bytes.（通用于 RISC-V）

![](https://github.com/Clovers2333/picx-images-hosting/raw/master/data_transfer_eg.4913hc0j2a.webp)

- 有关小端序和 `sign extention` 的例子

![](https://github.com/Clovers2333/picx-images-hosting/raw/master/byte_instruction_eg.92pydgxrvj.webp)

- **Half-Word Instructions：**

![](https://github.com/Clovers2333/picx-images-hosting/raw/master/half_word_instructions.7eglns93rj.webp)

### Control Flow Instructions

- `beq reg1, reg2, label`: If `reg1 == reg2` goto label, otherwise go to the next instruction.
- `bne reg1, reg2, label`: not equal.
- `j label`: jump.
- `blt reg1, reg2, label`: `reg1 < reg2`.
- `bge reg1, reg2, label`: `reg1 >= reg2`.

#### Program Counter

A special register that keeps track of the memory address of the next instruction to be executed in a program.

### Loops

### Environmental Call

- `ecall` is a way for an application to interact with the operating system.
- The value in register `a0` is given to the OS which performs special functions.（不同操作系统所用的寄存器可能不相同）
    - Print values
    - Exit the program
    - Allocating more memory for the program
    - [For more information](https://github.com/ThaumicMekanism/venus/wiki/Environmental-Calls)

### Function in RISC-V

- `a0-a7`: eight argument registers to pass parameters.(more arguments go to the stack)
- `a0-a1`: two argument registers also used to return values.
- `sp`: "stack pointer", holds the current memory address of the "bottom" of the stack.

#### Jumping and Returning

- `jal ra sum`(jump and link): 设置 `ra` 为该指令的下一个地址(link)，然后去执行 `sum` 函数，执行结束以后再执行 `jr ra`，返回 `ra` 继续执行。

- 在实际的 risc-v 中，其实没有 `j label` 这个语句，它是通过 `jal x0 label` 来实现的，因为对 `x0` 的写入是无效的。

    （`jal x0 ra` 是错误的，因为第二个参数必须是一个 `label`）

- `jr ra` 可用 `jalr x0, ra, 0` 来实现，其中 0 是程序跳到 `ra` 的后续偏移量。

#### Local Storage for Variables

- 主要靠移动栈指针 `sp` 来实现。

- 核心问题出在函数嵌套上面——`ra` is overwritten?

- `s0-s11` 作为 saved registers，当 callee 被调用时，要先把 caller 用的 `s` 全部存到栈里面。

    - `s0`: frame pointer（参考 x86 笔记—— stack frame）

- 对于 `ra` 寄存器，在 caller 调用 callee 时，会提前把它进行存储。

    其他要存储的寄存器：`t0-t6(tempporary registers), a0-a7(return address and arguments)`

![](https://github.com/Clovers2333/picx-images-hosting/raw/master/calling_stack_frame.32hsircdui.webp)

**（注意！栈的保存是从高地址到低地址）**

- It's callee's duty to store the registers it wants to use to the stack before overrites it.

### Instruction Formats

原则：尽量 32 位中相同的位置在不同的 Formats 中能有相同的作用。

#### R-Format

有 dest 三寄存器运算(`rd`: register destination)。

#### I-Format

有 dest 二寄存器一立即数运算；如读取数据操作。

#### S-Format

无 dest 二寄存器一立即数运算；如保存数据操作。

#### SB-Format

用于 branch 操作。

- Extensions to RISC-V base ISA support 16-bit compressed instructions.
- 16-bit = half-word.
- To enable this, RISC-V scales the branch offset to be half-words even there are no 16-bit instructions.
- PC-Relative Addressing: 以 PC 为中心定位代码地址（所以一个完整的代码无论存在哪里相对偏移量都是不变的）

![](https://github.com/Clovers2333/picx-images-hosting/raw/master/sbFormats.26lbaei5p1.webp)

#### U-Format

Dealing With Large Immediates.

- `lui` writes the upper 20 bits of the destination with the immediate value.
- Together with `addi` to set low 12 bits.
- The key to PC-Relative Addressing: `auipc x10 0`.

#### UJ-Format

- For branches, we assumed that we won't branch too far, but for general jumps(`jal`), we may jump anywhere in code memory.
- notice: `jalr` is I-Format.(`rd = PC+4; PC = rs1 + offset` )

