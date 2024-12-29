# 随机算法

随机算法分为以下两种：

- 拉斯维加斯算法(Las Vegas Algorithm): 这种算法在每次运行时都会使用随机性，但它保证最终会找到一个正确的解。也就是说，拉斯维加斯算法的输出总是正确的，但运行时间可能会有所不同。
- 蒙特卡罗算法(Monte Carlo Algorithm): 这种算法在每次运行时都会使用随机性，但它不一定能找到一个正确的解。也就是说，蒙特卡罗算法的输出可能不正确，但运行时间通常是固定的。

## 雇佣问题(The Hiring Problem)

- 每天都面试不同的申请者，持续 $N$ 天
- 面试成本 $C_i$，雇佣成本 $C_h$

**每次碰到更优秀的就雇佣：**

```c
int Hiring ( EventType C[ ], int N )
{   /* candidate 0 is a least-qualified dummy candidate */
    int Best = 0;
    int BestQ = the quality of candidate 0;
    for ( i=1; i<=N; i++ ) {
        Qi = interview( i ); /* Ci */
        if ( Qi > BestQ ) {
            BestQ = Qi;
            Best = i;
            hire( i );  /* Ch */
        }
    }
    return Best;
}
```

则在随机化的情况下，第 $i$ 个人被雇佣的概率为 $1/i$，期望总雇佣次数为调和级数 $O(ln N)$.

所以总成本为 $O(C_hlnN+NC_i)$.

**只 Hire 一个：**

先随机挑选`k`个面试，取出其中最好的，然后从第`k+1`个开始，如果比最好的还好的话，就雇佣。

**$S_i$: 第 $i$ 个申请者是最好的**

要使 $S_i$ 成立，需要满足：

- **A**: 最好的申请者在位置 $i$，概率为 $\dfrac{1}{N}$。
- **B**: 在位置 $i+1$ 到 $N$ 之间，没有人被选中。也就是说，$1$ 到 $i-1$ 中没有比 $i$ 个更好的（前 $i-1$ 个申请者的集合中，最优者不被选中），概率为 $\dfrac{k}{N(i-1)}$。

联合计算：

$$
\Pr[S_i] = \Pr[A \cap B] = \Pr[A] \cdot \Pr[B] = \frac{1}{N} \cdot \frac{k}{i-1} = \frac{k}{N(i-1)}.
$$

$$
\Pr[S] = \sum_{i=k+1}^{N} \Pr[S_i] = \frac{k}{N} \sum_{i=k+1}^{N} \frac{1}{i}.
$$

对上述和求积分作为上界和下界：

$$
\int_{k}^{N} \frac{1}{x} dx \leq \sum_{i=k+1}^{N} \frac{1}{i} \leq \int_{k}^{N-1} \frac{1}{x} dx
$$

$$
\ln N - \ln k - \frac{1}{N} \leq \Pr[S] \leq \ln N - \ln k - 1.
$$

对于给定的 $k$，最优概率依赖于解满足：

$$
\frac{k}{N} \ln \left(\frac{N}{k}\right) \leq \Pr[S] \leq \frac{k}{N} \ln \left(\frac{N-1}{k-1}\right).
$$

为了最大化这个概率，我们需要求该概率关于 $k$ 的值：

对上述求导取其零点求极值，我们得到：

$$
\frac{d}{dk} \left[\frac{k}{N} \ln \left(\frac{N}{k}\right)\right] = \frac{1}{N} (\ln N - \ln k - 1) = 0
$$

这意味着：

$$
\ln k = \ln N - 1 \quad \Rightarrow \quad k = \frac{N}{e}.
$$

带入：

$$
f(k) = \frac{k}{N} \ln \left(\frac{N}{k}\right) \quad \Rightarrow \quad \max(f(k)) = \frac{1}{e}.
$$

## 随机化快速排序

对于传统的快速排序，最坏情况是每次选择的pivot都是最小或最大值，导致每次划分都只能减少一个元素，导致时间复杂度为 $O(n^2)$.

现在我们希望能够选择 pivot，使得每一侧至少包含 $\dfrac{n}{4}$ 个元素。

即在 $[\dfrac{n}{4},\dfrac{3n}{4}]$ 之间选 pivot，期望下要选 2 次

所以每两层就期望把数据规模减成 3/4，一共有 log 层，每层 $O(N)$，总复杂度 $O(NlogN)$.

