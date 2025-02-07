# 线性规划的几何基础

## 一般的线性规划问题

\[
\begin{array}{rl}
\text{max} & c^\top x \\
\text{s.t.} & Ax \leq b \\
            & Cx = d \\
            & x \geq 0
\end{array}
\]

其中 \( c, x \in \mathbb{R}^n \)，\( b, d \in \mathbb{R}^m \)，\( A, C \in \mathbb{R}^{m \times n} \)。\( x \geq 0 \) 代表向量 \( x \) 中的每个分量都非负。

## 仿射集、凸集

### 仿射组合

对于空间 $\mathbb{R}^n$ 中的两个点 $x_1, x_2$，那么$\left\{ \theta x_1 + (1-\theta) x_2 : \theta \in \mathbb{R} \right\}$定义了过这两点的直线。

我们称 $\theta x_1 + (1-\theta) x_2 = x_1 + \theta (x_1 - x_2), \theta \in \mathbb{R}$ 为 $x_1, x_2$ 的**仿射组合** (Affine Combination)。

### 仿射集

如果一个非空集合 $Q$ 满足：对任意 $x_1,x_2 \in Q$，其仿射组合也属于 $Q$，则称 $Q$ 是一个**仿射集**。

### 凸组合

类似于仿射组合的定义，只是 $\theta \in [0,1]$.

### 凸集

凸集 $Q$ 任意两点均属于 $Q$，则 $Q$ 为凸集。

**性质：**任意两个凸集的交也是凸集。

## 多面体

**定义**：称集合$\{x : a^\top x = b\}$（其中$a \in \mathbb{R}^n \neq 0, x \in \mathbb{R}^n, b \in \mathbb{R}$）为**超平面**(Hyperplanes)，一个超平面将 $\mathbb{R}$ 划分成两个**半空间**。

**定义**：一个多面体 (polyhedron) 是为有限个线性等式和不等式的解集：

$$\{x : a_i^\top x \leq b_i, i = 1, \dots, p, c_j^\top x = d_j, j = 1, \dots, q\}$$

因此一个多面体是有限个半空间和超平面的交集。也可以将其写作：$\{x : Ax \leq b, Cx = d\}$

由于我们可以用正负半空间之交集表示一个超平面，因此等式约束可以用两个不等式表示，于是每一个多面体也都可以写成：$\{x : Ax \leq b\}$

**性质：在多面体中，顶点、极点、基本可行解是等价的。**