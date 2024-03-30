# Machine-Level Programming

## Lecture 5

### Definitions

- Architecture (also ISA: instruction set architecture)：指令集，编译器所编译的目标。

![](https://github.com/Clovers2333/picx-images-hosting/raw/master/Machine_Programming_Definitions.7p3d8s7et6.webp)

- 上图介绍了在汇编语言的视角下，计算机一些最基本的组件。
    - Register file 即寄存器，暂存数据和地址（有限容量的高速存储部件，cache 是扩充，寄存器更快）（并不是按照数字下标索引，而是有特定的名字）。
    - Condition codes 用来保存最近几次运算的结果，用于条件语句
    - 不可见的 Memory 被视为一大块数组，为虚拟内存。

### Turning C into Assembly Code

![](https://github.com/Clovers2333/picx-images-hosting/raw/master/Turning_C_to_ObjectCode.2krojil1vi.webp)

- 依次进行的操作为：预处理（展开头文件、替换操作符、展开宏定义）、编译、汇编、链接。
- 以下是一个具体的例子：

```c
long plus(long x, long y);
void sumstore(long x, long y, long *dest){
    long t = plus(x, y);
    *dest = t;
}
```

```assembly
sumstore:
	pushq	%rbx
	movq	%rdx, %rbx
	call	plus
	movq	%rax, (%rbx)
	popq	%rbx
	ret
```

- `%<name>`：寄存器的名称。
- `pushq`：push somthing to the stack.
- `movq`：move sth. from one place to another.
- `call`：call a procedure.
- `ret`：return from a specific function.
- **Each line written in text will turn into an instruction in the object code.**

**以下是对于这个例子中赋值语句的解释：**

![](https://github.com/Clovers2333/picx-images-hosting/raw/master/Assembly_Assigning_Exanple.ibvvim1by.webp)

- 用一个寄存器 `rbx` 保存 dest 的地址

- `mov` 语句的意思：将 `rax` 寄存器中的数值，移到 `rbx` 寄存器上的地址所指向的位置。

    （如果没有括号，那就是简单地把 `rax` 上的数字移到 `rbx` 上保存）

- 汇编上的每个 text，实则都有一个编码，所以在汇编过程中，会把一条汇编语句转成一个二进制的 instruction，一般在 1~14 bytes（？）

    （汇编语言：低级语言；二进制：机器语言）
    
- Disassembler: 把 Object Code 转化成 Assembly Code；但是不能完全还原 Source Code，因为很多信息例如变量名称会在转化成 Assembly Code 时丢失。

    ```bash
    objdump -d sum > sum.d
    ```

    "objdump"：系统自带的反汇编程序。

    同时在 `gdb` 逐步调试 Object Code 时，可运行 `dissassemable <function>` 来反编译其中一个函数。

### How to Get an Assembly Code

- 汇编语言的文件后缀：`.s`；反汇编语言的后缀：`.d`

- 把 C 语言编译成汇编语言的命令：`gcc -Og -S *.c`

    当你调用 gcc 时，实际上你调用的不仅仅是一个程序，而是一整个程序序列，已完成整个编译过程。

    这里的两个编译选项，`-S` 是停止，只进行编译成汇编的那步；`-Og` 是一种优化的规范，使得汇编代码更好读懂（g：调试级别；如果采用 O1, O2 这种优化效率，那么更加难读懂）。

### Assembly Characteristics: Data Types

- Integer data of 1, 2, 4 or 8 bytes.
    - 地址或指针都以整型存储
- Floating point data of 4, 8 or 10 bytes.（表示方法和 C 语言大相径庭） 
- No aggregate types（较大型的数据类型）such as arrays or structures.
    - Just contiguously allocated bytes in memory.
    - Built-up in more superior languages.

### x86-64 Integer Registers

- `%r`：表示 64 位；`%e`：表示 32 位。（IA32：8 个寄存器；x86-64：16 个寄存器；`%rsp, %esp`：栈指针）

### Moving Data

- `movq <source>, <dest>`.

#### Operand Types

- Immediate：Constant integer data.（`$0x4, $-147`）
- Register: One of 16 integer registers. (`%rsp` is reserved)
- Memory: 8 consecutive bytes of memory at address given by register.( `(%rax)` )

#### `movq` Operand Combinations

- 看上去三种数据类型总共有九种组合方式，但其实只有五种：`Imm` 作为常数不可能成为 `dest`；`Mem-Mem` 的操作也会被分解成 `Mem-Reg-Mem`.
- `movq (%rax) %rbx`：从内存中取一个值，从内存中读取它并将其复制到寄存器中。（`%rax` 本身是一个数字，用数字表示地址；但加括号以后就变成了地址所指的值，相当于 C 语言中的解引用 `*`）

![](https://github.com/Clovers2333/picx-images-hosting/raw/master/movq_operand_combination.51dx5bddta.webp)

但是相对于巨大的 Memory 空间，16 个寄存器该如何实现访问内存的功能呢？

#### Displacement

- D(R)：`Mem[Reg[R]]-->Mem[Reg[R]+D]`.

    `movq 8(%rbp), %rdx`：取 `%rbp` 地址右移 8 个字节后的所指向的元素。

- D(Rb, Ri, S)：`Mem[Reg[Rb]+S*Reg[Ri]+D]`.

    `S`：根据表示的数据类型来决定位移字节的大小，这样更加方便，如 `int：4, double：8`.

    （也被称为 Complete Memory Addressing Modes）

### Address Computation

对于上面的 Complete Addressing，因为在汇编语言中，地址本质上就是整型，所以我们不仅可以用它来算地址，也可以用于变量的计算。

对于正常的 Complete Addressing，因为语法是 `(Rb, Ri, S)` ，我们在计算完地址的同时会同时及进行解引用操作，返回地址所指的数值；而 `leaq` 函数则提供了计算地址之后只返回地址，而不进行解引用操作的一种方式。

- `leaq Src, Dst`：
    - `Src` is address mode expression.
    - Set `Dst` to address denoted by expression.
    - 可以视为一个比较特殊的运算操作。

一些其它的运算操作：

![](https://github.com/Clovers2333/picx-images-hosting/raw/master/Assembly_Arithmetic_Operations.7p3dfqezgz.webp)

![](https://github.com/Clovers2333/picx-images-hosting/raw/master/Assembly_Arithmetic_Operations2.5q76pechg8.webp)

- 和 `mov` 类似，被赋值的寄存器是在后面的！
- Example:

![](https://github.com/Clovers2333/picx-images-hosting/raw/master/Assembly_Arithmetic_Example.6bgubp9csc.webp)