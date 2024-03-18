# fds 被拷打记录

fds 回忆卷：https://www.cc98.org/topic/5507473

## Introducution

- 一个算法不一定要有输入，但是一定要有输出.
- An algorithm must terminate after finite number of steps.（T）
- 程序不一定要有限.

## qsort 用法

```c
int cmp(const void* elem1, const void* elem2){
	return *(int*)elem1 - *(int*)elem2;
}//从小到大排
int sz=sizeof(arr)/sizeof(arr[0]);
qsort(arr, sz, sizeof(arr[0]), cmp_int);
```

## 时空复杂度分析

1.节点层数*每次规模

2.节点个数

分析取较大值

### 主定理

对于递推式 $T(n)=aT(\frac{n}{b})+O(n^d)$

则：
$$
T(n)=
\begin{cases}
O(n^d),d>log_ba \\
O(n^dlogn),d=log_ba\\
O(n^{log_ba}),d<log_ba
\end{cases}
$$
还是比较好推导的，按照层数推，当时只记定理现在想也想不起来。。

Fib 复杂度：$O(2^n)$

## 卡特兰数

$$
C(0)=1 \\
C(n+1)=\sum_{i=0}^{n}C(i)C(n-i)\\
C(n)=\binom{2n}{n}/(n+1)
$$

使用场景：$n$ 的括号序列；$n$ 的出入栈序列；$n+2$ 边形的三角剖分；$n+1$ 个叶子节点的国际满二叉树（只有 0 度点和 2 度点，补充：$n0=n2+1$）

递推式推导：第一个左括号个数等于右括号个数的位置。



## 二叉树、多叉树和堆

- 树节点的 $degree$ : 儿子节点个数，叶子高度是 0，根的深度也是 0.

- 线性构造二叉搜索树：先全部插成满二叉树，然后从下到上一个一个子树使有序。

    依据：$\sum height(i)=O(n)$

- 堆就是基于二叉搜索树，删除操作：把顶节点赋值成最后一个节点

- 表达式树：中序遍历是原表达式；栈：后序遍历进栈；注意 `^` 运算时从右往左结合。

- 对于二叉树，恒有：$n0=n2+1$

- 多叉树：$n0=n2+2n3+...+1$

- It is always possible to represent a tree by a one-dimensional integer array.（T）

    

### 线索二叉树（Threaded Binary Trees）

每一个节点都必须有左右儿子，如果没有，那么就按照题目给定的遍历顺序分别连向他们的前继后继。



### 堆

- 线性构造堆：先全部插成满二叉树，然后从下到上一个一个子树满足堆的性质，都进行一次 `pushdown`。

    依据：$\sum height(i)=O(n)$

- 插入元素：插在最后，然后 `pushup`

- 删除元素：把顶点和最后一个节点交换，删掉以后 `pushdown(1)`

- linear algorithm adjust：线性重新建堆



### 二叉搜索树

- 插入：找到对应的空位置直接插入

- 删除：找到右侧子树最小值然后继续递归；也可以把左侧最大值扔上去。

    ```c
    if ( T->Left && T->Right ) {  /* Two children */ 
        /* Replace with smallest in right subtree */ 
        TmpCell = FindMin( T->Right ); 
        T->Element = TmpCell->Element; 
        T->Right = Delete( T->Element, T->Right );  
    } 
    ```

    - 可以对删除打 `lazytag`，即我需要继续往下递归时才删除这个点（找到右侧子树的min，拿上来，然后把右侧子树的 min 打上删除的 lazytag）。



## 图

- 图的表示方法：邻接矩阵，邻接表，[十字链表，邻接多重表][https://blog.csdn.net/daocaoren_/article/details/98726562]
- 有逆天题目节点是 1 ，但是从 0 下标开始存
- 并查集：按秩合并（union-by-size），路径压缩（path compression）
- Strongly connected directed graph G：任意 `i` 到 `j` 都有路径。

### 最短路

- `dijkstra`：复杂度 $O(VlogV+ElogV)=O(ElogV)$

- `SPFA`：每次松弛成功就进队，如果有一个点进队 `n` 次那么就有负环。

    ```c
    void  WeightedNegative( Table T )
    {   /* T is initialized by Figure 9.30 on p.303 */
        Queue  Q;
        Vertex  V, W;
        Q = CreateQueue (NumVertex );  MakeEmpty( Q );
        Enqueue( S, Q ); /* Enqueue the source vertex */
        while ( !IsEmpty( Q ) ) {
            V = Dequeue( Q );
            for ( each W adjacent to V )
    	if ( T[ V ].Dist + Cvw < T[ W ].Dist ) {
    	    T[ W ].Dist = T[ V ].Dist + Cvw;
    	    T[ W ].Path = V;
    	    if ( W is not already in Q )
    	        Enqueue( W, Q );
    	} /* end-if update */
        } /* end-while */
        DisposeQueue( Q ); /* free memory */
    }
    ```

- `AOE——activity on edge`，有向无环图，活动在边上； `AOV——activity on verticles`，有向无环图，活动在点上。

    ![](https://github.com/Clovers2333/picx-images-hosting/raw/master/AOE.8z6a4lc2el.webp)

    LC：最长路；EC：最短路。



### 网络流

- 分层

- 可反悔边

- 当前弧优化

- 最小费用最大流

- dijkstra 增广：每次选择增广值最大的路径（$O(Elog(cap_{max}))*O(ElogV)$），$cap_{max}$ 是最大的流量

- bfs 优化：每次选择边数最小的增广，且一起增广：$O(E)*O(EV)$

    我们证明了 Dinic 算法中下一分层图的最短路径长度与前一分层图相比**严格递增**

    https://www.zhihu.com/question/34374412/answer/2306847964

    

### 生成树

Kruskal ：排序以后并查集

Prim：类似 dijkstra。



### DFS

- articulation point：割点，删掉这个点以后图会变成至少两个连通图

- biconnected graph：没有割点的图

- biconnected component：没有割点的分量 

- 求割点：如果是子树的话和 low 取 min，如果是回溯的话和 dfn 取 min；如果当(u,v)为树边且`low[v] >= dfn[u]`时，节点u才为割点；如果根节点有两个儿子就是割点。

- 欧拉路：经过每条边；欧拉回路：经过每条边，且回到原点；哈密顿路：经过每个点；哈密顿回路：……

    （欧拉回路充要：联通且都是偶数点）



## 排序

**十种排序方法的具体分析及图示：** https://www.cnblogs.com/onepixel/articles/7674659.html（这个 b 总结错误一堆，谨慎观看）

额外空间：归并——$O(n)$，堆：$O(k)$，其它 $O(1)$

稳定排序：插入、冒泡、归并、基数、桶

不稳定排序：希尔、选择、快排、堆



### 快排

有关快排的 `Median3:` 把开头、中间、结尾排序，然后把中间和结尾排序，把结尾作为 `pivot`。

```c
void  Qsort( ElementType A[ ], int Left, int Right ) {   
    int  i,  j; ElementType  Pivot; 
    if ( Left + Cutoff <= Right ) {  /* if the sequence is not too short */
        Pivot = Median3( A, Left, Right );  /* select pivot */
        i = Left;     
        j = Right – 1;  /* why not set Left+1 and Right-2? */
        for( ; ; ) { 
            while ( A[ + +i ] < Pivot ) { }  /* scan from left */
            while ( A[ – –j ] > Pivot ) { }  /* scan from right */
            if ( i < j ) Swap( &A[ i ], &A[ j ] );  /* adjust partition */
            else     break;  /* partition done */
        } 
        Swap( &A[ i ], &A[ Right - 1 ] ); /* restore pivot */
        Qsort( A, Left, i - 1 );      /* recursively sort left part */
        Qsort( A, i + 1, Right );   /* recursively sort right part */
    }  /* end if - the sequence is long */
    else/* do an insertion sort on the short subarray */
        InsertionSort( A + Left, Right - Left + 1 );
}
```

快排 3 个 run，最后会有三个元素在应有的位置。



### 希尔排序

步长：可以取 $2^k$，也可以取 $2^k-1$。

希尔排序每一次是做插入排序。



- Bucket Sort：桶排序

- Radix Sort：基数排序（LSD：从最低位开始排序；MSD：从高位开始排）



## 哈希

- 哈希表：

    ![](https://github.com/Clovers2333/picx-images-hosting/raw/master/Hash-Table.5c0qh2g9ww.webp)

1.常见的对 $key$ 操作方法：取余法、平方取中、折叠相加法、数值分析法（变化最大的位数）

- identifier：就是 key；key : hash 函数的参数；hash value：hash的结果

2.对于字符串的向数字映射方法：乘 32，只需要左移

3.冲突解决方法：（区分探测次数和探测失败次数）

	1）seperate chaining:每个 key 值都对应了一个链表
	
	2）线性冲突探测法（也叫开放地址）
	
	3）平方冲突探测法（也叫开放地址，$1^2,-1^2,2^2,-2^2$，可以正负也可以不正负；二次探测不能保证有空位一定能找到；如果哈希模数是质数，且表中目前元素一半都不到，那么一定能找到）
	
	4）双哈希法（不仅是判断是否相同，还要有“find、delete”函数）
	
		映射到的还是单一的数组，先用 M1 取模，如果冲突了用 M2 取模函数作为探测的步长，双哈希可以使得探测的步长每次不一样，具有随机的效果。

**再哈希：把长度*2，然后扩张到素数，然后把所有元素都重新插入一遍。**