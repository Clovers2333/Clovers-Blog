# Bits, Bytes and Integer

## Lecture 2

### 不同机器的数据类型所占位数

![](https://github.com/Clovers2333/picx-images-hosting/raw/master/Datatype_Size.7zq6kn3v8j.webp)

### 原码、反码和补码

原码：开头为符号位，后面均为数值位。

若为正数，原码=反码=补码；

若为负数，则反码为原码除了符号位每一位取反，补码为反码+1；

移码：针对

两个数相加，在计算机里面就是补码二进制相加。

- -128 补码（对于八位来说）为 `10000000`；原码和反码的可表示区间都是 `-127~127`，因为 0 被表示了两次。

- **负数补码转原数：$-2^n +$ 后面非符号位的数值**，即对于负数反过来表示。

    或：$[X_\textbf{T}]_补=(2^n+X_\textbf{T}) \% 2^n$，其中 $X_\textbf{T}$ 为原数。

#### 变形补码

- 不同于补码的单符号位，变形补码采用双符号位，2 个 bits 判断符号，剩下两个 bits 判断是否溢出。
- 假定变形补码的位数为 $n+1$，则变形补码可以表示成：$[X_\textbf{T}]_补=(2^{n+1}+X_\textbf{T}) \% 2^{n+1}$.
- 可以分析出当符号位为 01 或者 10 时，补码溢出。

### Expression Evaluation

- If there is a mix of unsigned and signed in a single expression, **signed values implicitly cast to unsigned.**

    e.g.  `2147483647U < -2147483647-1`；`(unsigned)-1 > -2`

- What is the return value of `f(Tmin)`?

    ```c
    int f(int x){
        if (x < 0) return -x;
        else return x;
    }
    ```

    - 仍然是 `Tmin`，用补码 `0` 去减 `10000...`，仍然是 `10000....`

- Another example:

    ```c
    int i;
    for (int i = n; i-sizeof(char) >= 0; i--);
    ```

    这样仍然不会停下，因为 `sizeof` 作为操作符它解析出来的是一个无符号的值。

### Sign Extension

- 对于低位数的数据类型映射到高位数：用最高位补齐

    即：如果是正数，那么就全补 0；如果是负数，那么就全补 1。在这种补位规则下，可以保证补码所表示的数仍相同。

- 对于高位数的数据映射到低位数：将高位截断。

    如果是在低位数表示范围内的，那么容易推出还是原数；如果大于表示范围，那么符号位会被截掉，溢出也没有办法了。

- **以上规则是针对有符号数，如果是无符号数扩展，直接加 0.**



## Lecture 3

### 二进制的运算

- 对于加减法，直接用补码按照二进制进行计算，然后截去溢出位，剩下的通过有/无符号来翻译成原数即可。

- 移位操作对于**负数非溢出**情况：（左移低位补 0，右移高位补 1）

$$
\begin{aligned}
x&=-2^n+w\\
x<<1&=-2^n+(w*2-2^n)=2x\\
x>>1&=-2^n+(w/2)
\end{aligned}
$$

   需要注意的点：$(-3)>>1=-2,(-1)>>1=-1$

- 对于一个数取反：所有位取反，然后 +1 （$-\textbf{Tmin} = \textbf{Tmin}$）

（有关乘法除法，以及加减法的溢出后面会讲）

### When to use unsigned

- Do use when performing modular arithmetic.
- Do use when using bits to represent sets.

### Byte-Oriented Memory Organization

- 对于 64 位计算机，所有的地址都用 64 个 bit 来保存，而你可以用到的最大的地址是 47 位。（可以把内存看成一个巨大无比的数组，下标为 0~2^47，每个元素的大小是 1 byte）
- 就算系统的内存没有 2^47 bytes，但是在程序运行时他会假定你有这么多内存（内存系统在虚拟内存中运行）。
- 从逻辑上来说，你有这么多内存，但对于有限的硬件，你只能访问内存中的一些地址，如果你访问了其他地址，就会报错 `Segmentation Fault`.
- 对于 64 位机，内存的最大容量是 47 位，这只是对地址空间的限制，不是对所有变量类型的限制。64 位机善于处理 64 位的数据。

### Byte Ordering

#### Convention（单一数据整合）

把内存看成数组，有时候数组中的一个位置不足以表示你这个变量，所以把数组中的连续位置整合来保存你这个变量。

- Little Endian: 对于一个数据，地址保存从低到高（大多数机子）
- Big Endian: 对于一个数据，地址保存从高到低（Internet）

#### Examining Data Representations

如何查看表示一个 `int` 四个字节的情况：

```c
typedef unsigned char *pointer;
void show_bytes(pointer start, size_t len){
    size_t i;
    for(i=0; i<len; i++){
        printf("%p\t0x%.2x\n", start+i, start[i]);
        puts("")
    }
}
// %p: print pointer, %x: print Hexadecimal

int a = 15213;
show_bytes((pointer) &a, sizeof(int));
/*
0x7fffb7f7dbc   6d
0x7fffb7f7dbd   3b
0x7fffb7f7dbe   00
0x7fffb7f7dbf   00
*/

/*
pointer 定义成 char* 因为 char 是一个字节的，而 pointer 的 +=1 就是按照指针的类型来加字节数；
start[i] = &(start+i)，然后用十六进制打出。
*/
```
