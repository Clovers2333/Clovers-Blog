> CMU 镇系神课，系统一暂时没有太多涉及，但还是能看完尽量看完吧。

## Lecture 1

Course Overview.



## Lecture 2 - Bits, Bytes and Integer

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



## Lecture 3 - Bits, Bytes and Integer

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

## Lecture 4 - Floating Point

主要介绍 IEEE Encoding.

### Presision options

$$
v=(-1)^sM2^E
$$

- 单精度：1 位符号位(s)，8 位指数位(exp)，23 位数值位(frac)
- 双精度：1 位符号位，11 位指数位，52 位数值位

### Normalized Values

- 过大过小情况占用：$Exp \neq 000...0$ and $Exp \neq 111...1$.

- 指数位不用补码，而是 $E=Exp-Bias$.

    - Exp: unsigned value of exp filed.
    - Bias: $2^{k-1}-1$.（如对于 single precision: `Bias = 127, Exp: 1...254; E: -126...127`）
    - 不采用补码的原因：对于 `int` ，最常见的运算是加减运算，所以采用补码；而对于 `double` 对指数的加减法并不常见，“比较大小”成为了更常见的运算，所以用 `unsigned` 更快。

- Signficand coded with implied leading 1.

    对于数值位全部标准化，即化成 `1.xxxxxxxx`，然后因为第一位肯定是 1，所以我们还可以再节省一位，`frac` 全部用来表示小数点后的数值。

但是你会发现，如果只有 normalized values，你无法表示 0，这就是为什么留了两个 exp 值的原因。

### Denormalized Values

- Condition: `Exp = 000...0`.

- exp: $E=1-Bias$（不是 normalized 里面的 `0-Bias` ！）

- Signficand coded with implied 0.

    即化成 `0.xxxxxxxx`，然后因为第一位肯定是 1，所以我们还可以再节省一位，`frac` 全部用来表示小数点后的数值。

    这就可以解释为什么 $E=1-Bias$ 了：如果还和原来一样，会导致`1.xxxxxx * 2^{-bias}` 这个次数表示不出来。

- 对于浮点数而言，存在 `+0` 和 `-0`.

### Special Values

- Condition: `Exp = 111...1`.
- Case 1: `frac = 000...0`——Represent value `infinity`
    - `1.0/0.0, 1.0/(-0.0)`.
- Case 2: `frac != 000...0`——Represent `NaN(Not-a-Number)`
    - `sqrt(-1), inf-inf, inf*0`.

一个简单的例子：

![](https://github.com/Clovers2333/picx-images-hosting/raw/master/Floatnum.839spiozdx.png)

- 浮点数所表示数值的疏密程度：在零点附近密集，在无穷处稀疏。

### Rounding

对于浮点数的加法/乘法，需要截去溢出位，并且进行 rounding.

接下来重点介绍 IEEE Encoding 的 Default Rounding——Nearest Even Rounding.

- 若 >0.5，向上取整
- 若 <0.5，向下取整
- 若 =0.5，向最近偶数取整

统计学的解释：如果你有一组均匀分布的数据，那么他们向上或向下取整的概率都是 50%.

以上是对于十进制的情况，那么对于二进制的情况？

- "Even" when least significant bit is 0（最低位是 0）
- "Half way" when bits to right of rounding position = `100...`

### Addition and Multiplication in FP

- 浮点数加法不具有结合性（overflow；inexactness of rounding）
    - `3 + inf - inf = 0`
    - `3 + (inf - inf) = 3`
- 浮点数乘法不具有结合性（overflow；inexactness of rounding）
    - `1e20 * 1e20 * 1e-20 = inf` (`1e20 = 10^20`)
    - `1e20 * (1e20 * 1e-20) = 1e20`

### Floating Point in C

- `double/float -> int`
    - 把指数乘进去以后扔掉小数位
    - 可以理解成向 0 取整
    - 若数为 `NaN` 或 `inf`，设成 Tmin.
- `int -> double`: exact conversion，因为 `int` 位数小于 `double`
- `int -> float`: 因为 `float` 有符号位和指数位，所以位数不够，用浮点数的方式取整。

一些练习：

![](https://github.com/Clovers2333/picx-images-hosting/raw/master/FPexercises.4uaosxtj89.webp)

- 第二、三、五个、倒数第二、三、四个正确，其余错误。