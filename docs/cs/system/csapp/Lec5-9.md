# Machine-Level Programming

## Lecture 5 - Overview & Moving Operations

### Definitions

- Architecture (also ISA: instruction set architecture)：指令集，编译器所编译的目标。

![](https://github.com/Clovers2333/picx-images-hosting/raw/master/Machine_Programming_Definitions.7p3d8s7et6.webp)

- 上图介绍了在汇编语言的视角下，计算机一些最基本的组件。
    - Register file 即寄存器，暂存数据和地址（有限容量的高速存储部件，cache 是扩充，寄存器更快）（并不是按照数字下标索引，而是有特定的名字）。
    - Condition codes 用来保存最近几次运算的结果，用于条件语句。
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

- 用一个寄存器 `rbx` 保存 dest 的地址。

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

    ```assembly
    movabsq $0xffffffffffffffff, %rax
    movb $0, %al  # rax = 0xffffffffffffff00
    movw $0, %ax  # rax = 0xffffffffffff0000
    movl $0, %rax # rax = 0xffffffff00000000
    movl $0, %eax # rax = 0x0000000000000000
    ```

#### Operand Types

- Immediate：Constant integer data.（`$0x4, $-147`）
- Register: One of 16 integer registers. (`%rsp` is reserved)
- Memory: 8 consecutive bytes of memory at address given by register.( `(%rax)` )

#### `mov` Operand Combinations

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
    - **leaq 非常重要的作用：create pointers.**

一些其它的运算操作：

![](https://github.com/Clovers2333/picx-images-hosting/raw/master/Assembly_Arithmetic_Operations.7p3dfqezgz.webp)

![](https://github.com/Clovers2333/picx-images-hosting/raw/master/Assembly_Arithmetic_Operations2.5q76pechg8.webp)

- 和 `mov` 类似，被赋值的寄存器是在后面的！
- Example:

![](https://github.com/Clovers2333/picx-images-hosting/raw/master/Assembly_Arithmetic_Example.6bgubp9csc.webp)



## Lecture 6 - Condition Codes & Loops

### Condition Codes

#### Implicit Setting

- CF：进位标志（对于 unsigned）
- SF：正负标志（若为负数则设置为 1）
- ZF：是否为 0
- OF：是否溢出

例如在普通进行加法的时候，这四个 flag 就会隐式设置：

![](https://github.com/Clovers2333/picx-images-hosting/raw/master/Condition_Codes_AddSetting.5mnkt0fv9v.webp)

注意！`leaq` 操作时不会设置。

对于 Compare/Test 操作，这四个符号相当于是在对于 `a-b` 作判断。

- `cmpq b, a` like computing `a-b` without setting destination.
- `testq b, a` like computing `a&b` without setting destination（可以 `testq a, a` 来判断 a 的正负性）

**以上两个操作在机器代码中编写参数的方式与我们正常所期望的相反！**

#### SetX Instructions

![](https://github.com/Clovers2333/picx-images-hosting/raw/master/SetX_Instructions.3nre2p1vgn.webp)

以上这些 Instruction 都是基于这 4 个 Flag 的组合。

**Example:**

![](https://github.com/Clovers2333/picx-images-hosting/raw/master/Condition_Code_Example1.7smzettn7u.webp)

- `setg %al` 表示将 Flag 组合运算的值赋值到 `%al` 上。
- `movzbl` 表示低位扩展，高位补 0（`b`: Byte (8 bits); `w`: word (16 bits); `l`: long (32 bits) ）
- 然后第三行的 `Dest` 设置为 32 位的 `%eax`，是因为对于一个 64 位寄存器只赋值其前 32 位，高位会自动补上 0.

#### Jumping - Branches

![](https://github.com/Clovers2333/picx-images-hosting/raw/master/JX_Instructions.9rj656beq1.webp)

![](https://github.com/Clovers2333/picx-images-hosting/raw/master/Conditional_Branch_Example.70a3x3yxdv.webp)

（类似于 goto？）

事实上，gcc 在将上述 source code 转成汇编时，并不会右边的汇编代码，如果你不添加特定的编译指令的话，他会进行优化——conditional move.

![](https://github.com/Clovers2333/picx-images-hosting/raw/master/Conditional_Moves.7ljrjfzhbx.webp)

#### Switch Statement

- 不同于我们之前所想的，丢入一个 x，程序会对每一个 case 进行检索，时间复杂度 $O(Case)$；事实上，在编译过程中会对每一种 case 进行地址索引，通过空间换时间的方式来实现 $O(1)$ 锁定 case.

- 对于 default 的情况，编译器会比较聪明，在编译的时候也会进行很多优化，例如 case 有 `x = 1,2,4,5` ，那么就会在开头用无符号大小判断一下 `x` 是不是 `> 5`，如果是直接进入 `default`（也同时干掉了负数），然后对 `0, 1, 2, 3, 4, 5` 进行索引（0, 3 索引到 `default` ）

- 更聪明的是，如果你的 case 是负数，它会通过 add bias 的等方式来使你的最小值为 0.

- 更更聪明的是，如果 case 值太过分散，那么它会采用 BST 或者直接改成 if-else，来实现时空综合效率的最大化。

    （CMU 的学生把我想到的 Corner Cases 全问到了hhhh）

![](https://github.com/Clovers2333/picx-images-hosting/raw/master/switch_statement_example.1e8dksed0y.webp)

![](https://github.com/Clovers2333/picx-images-hosting/raw/master/Jump_Table.9nzk90w228.webp)

- Direct Jumping: `jmp .L8`（内存都是在的，也可以直接编号访问，但能拿来运算处理的只有寄存器（？））

- Indirect Jumping: `jmp *.L4(, %rdi, 8)`

    - Start of the table: `.L4`
    - Addresses are 8 bytes
    - Fetch target by effective Address `.L4 + x*8 (0 <= x <= 6)`

    （Jump Table 中保存各个 case 的 label（其实就是该 case 地址的别名），表头+一定字节的地址索引到具体的 case label，然后再 Jump 到那个 label 运行 case） 

- Fall Through 并不是编译器不够聪明，而是设计者就想给你这么一种选项（其实个人感觉这是 C 语言设计比较反人类的一点）

- 如果先前赋值的 `w` 仅在 case 块中用到，为了节省时间，会在要用到这个赋值的块中才加这句话。

![](https://github.com/Clovers2333/picx-images-hosting/raw/master/switch_code_block1.67x8h02b2q.webp)

所以虽然在个人在写 Source Code 的时候很抵触 case 语法，但其实作者在编写过程中还是花了一定心思的，它的底层也是值得去探究的。

**Bad Cases for conditional move:**

```c
val = Test(x) ? f1(x) : f2(x);
val = p ? *p : 0;
val = x > 0 ? x*=7 : x+=3;
```

- 慎用三目运算符，因为两个结果都会被计算。
- 可能会导致运算时间过长、边界报错、有 side-effect 等问题。

### Loops

#### Do-While Loop

![](https://github.com/Clovers2333/picx-images-hosting/raw/master/do_while_loop.5mnkujykn3.webp)

#### While Loop

##### Example 1 - Jump to Middle

![](https://github.com/Clovers2333/picx-images-hosting/raw/master/Jump_to_Middle.45hfst765n.webp)

##### Example 2 - O1 Optimization

- 改成先判断一次，如果不成立直接结束，否则转变成 do-while loop.

    ```c
    if (!Test)
        goto done;
    loop:
    	Body
        if (Test) goto loop;
    done;
    ```

#### For Loop

```c
Init;
while (Test){
    Body
    Update;
}
```

- 在采用 O1 优化的时候，Complier 可以帮你去掉一些显而易见的判断语句，比如 Init 之后 while 转 do-while 的初始判断，若显然成立，直接删掉。

## Lecture 7 - Procedure

- ABI: Application Binary Interface，即应用程序二进制接口。它定义了应用程序与操作系统之间进行交互的方式和规范，确保不同的软件组件能够正确地协同工作。

### x86-64 Stack

- 从一个很高的地址开始存数据，**从高往低**。
- 寄存器 `%rsp` 储存栈的最低点，也即栈顶。
- 汇编语言中的 `push/pop` 都是指对栈的操作。 

### Calling Conventions

#### Passing Control 

![](https://github.com/Clovers2333/picx-images-hosting/raw/master/Control_Flow_Example.70a4sxdm2c.webp)

- 对于`call` 语句：当调用函数 `mult2` 时，我们暂时将下一个语句的地址存到 `stack` 里面，然后程序计数器 `%rip` 跳转到函数的地址开始运行函数，当函数运行结束以后，再取栈顶的地址开始运行。
- 对于 `ret` 语句：reverse the effect of `call`，把 `%rip` 跳回栈顶，然后栈 `pop`.

#### Passing Data

- 通过 `%rdi, %rsi, %rdx, %rcx, %r8, %r9` 六个寄存器来传输值，通过 `%rax` 来返回值。
- 如果传输值超过六个？——把多余的参数放在栈里面。

#### Managing Local Data

主要是对于栈的管理，对于一个递归树形结构的管理。（[reference](https://blog.csdn.net/xungjhj/article/details/70946057)）

![](https://github.com/Clovers2333/picx-images-hosting/raw/master/Stack_Frame.2yy5lddnnj.webp)

- 参数 `1~n` 是传给 caller 的参数，上面的地址保存一些需要暂存的变量。
- 栈帧（stack frame）表示一个函数所占的栈空间，`%rsp` 保存栈顶的地址，`%rbp` 保存当前运行函数的最高地址。
- 当函数 return 时，通过 `%rbp` 返回函数的最高地址，然后在这个地址中把 `%rbp` 更新成 father 函数的 `%rbp`。
- 然后再读取返回地址，也就是调用 `son` 以后 `father` 要继续运行的语句的地址，把栈上的参数弹出，要返回的变量进行恢复，继续运行返回语句即可。

以下是一个递归的汇编代码：

![](https://github.com/Clovers2333/picx-images-hosting/raw/master/recursion_example.6pnb6nfaob.webp)

其中 `pushq %rbx` 是暂存 `%rbx` 中的值；`popq %rbx` 是返回 `%rbx` 的值。

为什么不用 `%rbp` 作栈帧了？——因为每一层递归只需要在栈中 push 一个元素，所以简单地 pop 就行了。

