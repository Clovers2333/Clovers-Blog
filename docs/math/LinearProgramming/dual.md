# 线性规划的对偶

## 原始问题与对偶问题的关系

### 定义

对于任何一个线性规划问题（称为原问题, primal, 记为 P），我们都可以将其约束条件改写作 $Ax \leq b$ 的形式，从而定义它的对偶问题 (dual, 记为 D)，

$$
(P) \quad \text{max } c^\top x \quad \text{s.t. } Ax \leq b \quad x \geq 0
$$

$$
(D) \quad \text{min } b^\top y \quad \text{s.t. } A^\top y \geq c \quad y \geq 0
$$

**原问题有几个变量，对偶问题就有几个约束；同理原问题有几个约束，对偶问题就有几个变量**。

### 符号的变化

- 若原问题约束为 $a_i^\top x \leq b_i$，那么对偶问题中有约束 $y_i \geq 0$；
- 若原问题约束为 $a_i^\top x \geq b_i$，那么对偶问题中有约束 $y_i \leq 0$。

因为两侧同乘符号有 $-a_i^\top x \leq -b_i$。令 $y'_i = -y_i$，令

$$
y' = (y_1, \dots, y_{i-1}, y'_i, y_{i+1}, \dots, y_m)^\top
$$

那么对偶问题可以写为

$$
\text{min} (b_1, \dots, b_{i-1}, -b_i, b_{i+1}, \dots, b_m) y'
$$

$$
\text{s.t. } (a_1, \dots, a_{i-1}, -a_i, a_{i+1}, \dots, a_m) y' \geq c
$$

$$
y' \geq 0
$$

将 $y_i = -y'_i$ 代入式中即可得。

所以就是说，改 P 问题 Ax 和 b 的符号，导致的是 D 问题的 A 和 b 取负了，为了使问题不变 y 也要取负数。

总的来说，约束条件的对应关系总结如下：
$$
(P) \quad \text{max } c^\top x \quad \Longleftrightarrow \quad (D) \quad \text{min } y^\top b
$$

$$
a_i^\top x \leq b_i \quad \Longleftrightarrow \quad y_i \geq 0
$$

$$
a_i^\top x \geq b_i \quad \Longleftrightarrow \quad y_i \leq 0
$$

$$
a_i^\top x = b_i \quad \Longleftrightarrow \quad y_i \text{ 无约束}
$$

$$
x_j \geq 0 \quad \Longleftrightarrow \quad A_j^\top y \geq c_j
$$

$$
x_j \leq 0 \quad \Longleftrightarrow \quad A_j^\top y \leq c_j
$$

$$
x_j \text{ 无约束} \quad \Longleftrightarrow \quad A_j^\top y = c_j
$$

## 弱对偶定理

设 $x,y$ 分别是 P 和 D 的可行解，则 $c^\top x \leq y^\top b$ 成立。

证明：
$$
y^\top b \geq y^\top (A x) = (y^\top A) x \geq c^\top x
$$

## 强对偶定理

若 P 有有限的最优解，那么 D 也有有限的最优解，而且二者最优解相同。

证明需要单纯性相关内容，先略。

## Minmax 定理

> 这实际上是线性映射的对偶在算法博弈论中的应用。

### 定义

在二人零和博弈中，记参与者为甲乙，博弈矩阵为 $A\in \mathbb{R}^{m\times n}$，甲（row player）的混合策略记作 $x\in \mathbb{R}^n$，乙（column player）的混合策略记作 $y\in \mathbb{R}^m$，那么这个博弈的效用是 $x^\top Ay$.

甲想要最小效用，乙想要最大效用，所以如果甲先进行行动，最终的效用是：
$$
\min_{x \in \Delta_n} \max_{y \in \Delta_m} x^\top A y
$$
如果是乙先行动，最终的效用是：
$$
\max_{y \in \Delta_m} \min_{x \in \Delta_n} x^\top A y
$$
**Minmax 定理**指出：这两个东西是相等的。

### 证明

考虑先手确定决策以后，后手其实没必要再混合决策了，取纯策略肯定可以取得极值，即：
$$
\min_{x \in \Delta_n} \max_{y \in \Delta_m} x^\top A y = \min_{x \in \Delta_n} \max_{j} x^\top A e_j = \min_{x \in \Delta_n} \max (A x)_j
$$
这样一来，我们整理一下两个式子，就变成了线性规划问题：

#### P 问题

$$
\min t
$$

$$
\text{s.t. } \sum_{j=1}^{m} a_{ij} x_j \leq t, \quad i = 1, \dots, n
$$

$$
\sum_{i=1}^{n} x_i = 1
$$

$$
x \geq 0
$$

#### D 问题

$$
\max s
$$

$$
\text{s.t. } \sum_{i=1}^{n} a_{ij} y_j \geq s, \quad j = 1, \dots, m
$$

$$
\sum_{j=1}^{m} y_i = 1
$$

$$
y \geq 0
$$

容易发现两个问题是对偶问题，根据强对偶定理，两个式子的最优值相等。

