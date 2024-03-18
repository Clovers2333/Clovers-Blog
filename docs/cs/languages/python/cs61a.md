## 前言

- **函数化思想和模块化思想**。

    对于 python 语言来讲，他的一大优势就是解释性且函数传参自由性强。无论是从 project 的设置，还是从平常课程上的演示代码，John 都鼓励学生用一个个函数来封装每一个小功能，这样既使代码各部分的独立性高，也使比较大的项目更易于长期维护。

    PS. Jupyter Notebook 也是一个不错的选择。

- ban assignments

## Classes

### Lecture 2

- 任何一个 `Expression` 都通过  `Environment` 决定，包括 `local environment` 和 `global environment`，`local` 优先。  

### Lecture 3

#### 普通函数的参数和输出

- `print` 一个没有返回值的函数会输出 `None`

    ```python
    >>> print(print(1),print(2))
    1
    2
    None None
    ```

- default value：`f(n, d=10)`：如果 `d` 没有被传参，那么默认为 10.

#### 一些 python 运行指令

- `python3 -m doctest *.py`: 为函数测试特定数据

    ```python
    def f(n, d):
        """
        >>> q, r = f(2013, 10)
        >>> q
        201
        >>> r
        3
        """
        return n//d, n%d
    ```

    这样就会测试在注释里面的数据（如果没有错，不会输出信息）

- `python -i *.py`: 把 `*.py` 作为环境，然后进行交互式编程。

### Lecture 4

#### Higher-order function

A function that takes a function as an argument value or returns a function as a return value.

- 在函数中定义函数：

```python
def f1():
    def f2():
        
    return f2
>>> f1()()
# 第一个括号是 f1 的参数，返回 f2；然后第二个括号是 f2 的参数
```

#### Lambda Expression

- used to evaluate functions

- `>>> f=lambda x: x*x`, A function with formal parameter `x` that returns the value of `x*x`.

- `lambda` 函数本身没有名字（或者可以认为这个函数的名字是 $\lambda$​ ），只是把这个函数的信息赋值到了一个变量上而已。

- 记录一下课件上对于函数 `f(x)` 的定义：

    ```
    - f(x) for user-defined function f: switch to a new environment; excute f's body.
    - return statement within f: switch back to the previous environment; f(x) now has a value.
    ```

- 反函数：

```python
def search(f):
    x=0
    while True:
        if f(x):
            return x
       	x+=1
def inverse(f):
    """return a function g that is the inversion of f"""
    return lambda y: search(labmda x: f(x)==y)

# 先把反函数赋值，不会进行解析。当传参 y 的时候，反函数就会遍历所有正整数 x，看有没有 f(x)=y，找到了直接返回。
# 所以用 sqr 定义 sqrt 无法对非完全平方数开根。 
```

- 以下这个函数实际不存在，但其实可以定义。

![](https://github.com/Clovers2333/picx-images-hosting/raw/master/if_else_suite.101wo3xg22.png)

```python
def if_(c, t, f):
	if c: t
	else: f
```

- 对于 python 来说，对于一个极小的数，也会被近似到 0.

#### Self-refrence

```python
def f(x):
    print(x)
    return f
>>> f(1)(2)(3)...
```

### Lecture 7

#### Function Decorators

```python
@trace1
def triple(x):
    return 3*x

# the same as follows:
def triple(x):
    return 3*x
triple = trace1(triple)
```



### Lecture 10

#### 有关二维数组定义

```python
>>> a = [[i+j*10 for i in range(0, 10)] for j in range(0, 10)]
>>> a
[[0, 1, 2, 3, 4, 5, 6, 7, 8, 9], [10, 11, 12, 13, 14, 15, 16, 17, 18, 19], [20, 21, 22, 23, 24, 25, 26, 27, 28, 29], [30, 31, 32, 33, 34, 35, 36, 37, 38, 39], [40, 41, 42, 43, 44, 45, 46, 47, 48, 49], [50, 51, 52, 53, 54, 55, 56, 57, 58, 59], [60, 61, 62, 63, 64, 65, 66, 67, 68, 69], [70, 71, 72, 73, 74, 75, 76, 77, 78, 79], [80, 81, 82, 83, 84, 85, 86, 87, 88, 89], [90, 91, 92, 93, 94, 95, 96, 97, 98, 99]]
```

- 不需要解释吧，python 真的是一个非常自然的语言。
- **python 里面一个比较常见的表达方法：** `<value> <restriction>`，即前面先写值，后面写对这个值的修饰或者是限制，诸如 `if, for` 等等。



## Homework

### HW01

#### 有关函数嵌套问题

- 如果把函数套参数作为另一个函数的参数，那么会先运行那个函数把结果跑出来再传进外围的函数。
- `with_if_statement` 和 `with_if_function` 输出不同的原因是两个函数其实都没有返回值，而后者因为是传参，所以会把两个函数都执行一遍，所以就会输出两个数；而前者只会执行一个函数。

### HW02

#### Lambda 函数实例——创建单行阶乘函数

```python
def make_anonymous_factorial():
    """Return the value of an expression that computes factorial.
    >>> make_anonymous_factorial()(5)
    120
    """
    return (lambda f: lambda n: f(f, n))(lambda f, n: 1 if n == 1 else mul(n, f(f, sub(n, 1))))
```

- 作为写代码者，当然不用写这么臭的 lambda 表达式，但是总归还是要看懂的：对于 lambda 表达式，我们就机械性地将其展开： `:` 前的作为参数，`:` 后的就是它的返回值。
- 所以这个 lambda 表达式可以展开成：

```python
def make_anonymous_factorial():
  def func1(f):
    def func2(k):
      return f(f, k)
    return func2
  def func3(f, k):
    if k == 1:
      return k
    else:
      return mul(k, f(f, sub(k, 1)))
  return func1(func3)
```

### Lecture 11

#### 运用函数封装结构体

- 结构体本身的所有变量用一个 `list` 保存

- 通过 `selection` 函数取出 `list` 中的值，通过对不同 `selection` 函数的命名来实现抽象化

- 使用 higher-order function 使结构体更具抽象化：

    ```python
    def rational(n, d):
        def select(name):
            if name == 'n':
                return n
            elif name =='d':
                return d
        return select
    
    def numer(x):
        return x('n')
    def denom(x):
        return x('d')
    ```

#### Dictionary

```python
>>> d = {'I': 1, 'V': 5, 'X': 10}
>>> d['I']
>>> d.keys() # 索引列表
>>> d.values() # 值列表
>>> d.items() # 返回 pair 列表
>>> 'X' in d # output: True
>>> d.get('X', 0) # 如果 X 有值，那么就返回值；否则返回后面的 default 值
>>> {x: x*x for x in range(1, 10)} # 一下定义了十个值
```





## Lab

## Project

### Hog

#### python 引用变量的顺序

1.当前作用域局部变量

2.外层作用域变量  

3.当前模块中的全局变量  

4.python 内置变量

#### global 和 nonlocal 的区分

- `global` 表示将变量声明为全局变量

- `nonlocal` 表示将变量声明为外层变量

- 二者用途相同，适用场合为补集

- 非常神奇的一个现象，先引用后修改会报错：

    ```python
    a = 1
    def fun():
        print(a) # 先引用
        a = 2 # 再修改
    fun()
    ```

    如果一定要用外面的 `a`，那么就要加一句 `global a`. 注意这里不能定义成 `nonlocal`；但如果是非全局变量，那么就必须定义成 `nonlocal`.

#### 函数未知参数个数

```python
>>> def printed(f):
...     def print_and_return(*args):
...         result = f(*args)
...         print('Result:', result)
...         return result
...     return print_and_return
>>> printed_pow = printed(pow)
>>> printed_pow(2, 8)
Result: 256
256
>>> printed_abs = printed(abs)
>>> printed_abs(-10)
Result: 10
10
```

​	使函数参数为 `*args` 即可。

#### 函数 `announce_highest` 中的 higher-order function

```python
def announce_highest(who, last_score=0, running_high=0):
    """
    >>> f0 = announce_highest(1) # Only announce Player 1 score gains
    >>> f1 = f0(12, 0)
    >>> f2 = f1(12, 9)
    9 point(s)! The most yet for Player 1
    >>> f3 = f2(20, 9)
    >>> f4 = f3(20, 30)
    21 point(s)! The most yet for Player 1
    >>> f5 = f4(20, 47) # Player 1 gets 17 points; not enough for a new high
    >>> f6 = f5(21, 47)
    >>> f7 = f6(21, 77)
    30 point(s)! The most yet for Player 1
    """
    assert who == 0 or who == 1, 'The who argument should indicate a player.'
    # BEGIN PROBLEM 7
    # Be cautious about the side effect
    # you cannot assign a new value to the name in parent frames
    def f(score0, score1):
        last=last_score; high=running_high
        delta = 0
        if who == 0:
            delta = score0-last
            last = score0
        if who == 1:
            delta = score1-last
            last = score1
        if delta > high:
            print(delta, "point(s)! The most yet for Player", who)
            high = delta
        return announce_highest(who, last, high)
        # don't return f, return announce instead!
    return f
```

其实这里涉及的就是 higher-order functions，在函数中定义函数。怎么理解呢？

就是 `f0` 是 `announce` 定义的 `f` 函数，`f1` 是给 `f0` 丢了俩参数后返回 `announce`、然后 `announce` 又返回 `f`…… （带参数继续调用，不带参数直接返回）

比如在调用 `f6` 的时候，他就会递归调用 `f5,f4...f0`.

笔者在写的时候犯了一个错误，就是在保证 `f1-fn` 变量能够传递的同时，同时改变了全局变量（采用 `nonlocal` 并且 `f` 函数返回自身的方法），这导致在进行一系列变化以后，如果再想要在 `f1` 上拉出一个状态 `f2'`，这个时候 `f1` 的参数已经被改了。

而如果返回 `announce` 函数，因为 `announce` 函数可以传参，而 `f` 不行，所以可以保证在传递参数的同时，不改变先前的全局变量。

PS. 在乱写代码的时候发现一个语法：就是定义函数参数的时候有默认值的要放在后面。



### Cat

没啥好说的，打字模拟机，有关字符串的一些处理方法都写在 `cheatsheet` 里面了。

写到 problem 9 后面不想写了，放了大 Q 老师的代码上去。