# Lecture 4 - Floating Points

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
    - **可以理解成向 0 取整**
    - 若数为 `NaN` 或 `inf`，设成 Tmin.
- `int -> double`: exact conversion，因为 `int` 位数小于 `double`
- `int -> float`: 因为 `float` 有符号位和指数位，所以位数不够，用浮点数的方式取整。

一些练习：

![](https://github.com/Clovers2333/picx-images-hosting/raw/master/FPexercises.4uaosxtj89.webp)

- 第二、三、五个、倒数第二、三、四个正确，其余错误。