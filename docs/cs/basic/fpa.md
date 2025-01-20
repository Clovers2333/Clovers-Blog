# 程序设计与算法基础

## 指针

#### 关于指针的一些声明

- `NULL`是⼀个预定定义的符号，表示0地址

- 指向不同类型的指针是不能直接互相赋值的；**不同类型的指针也不能比较大小！相同的指针可以比大小/相减，但是不可以相加！**

- `void *` 是不知道指向什么东西的指针，相当于它只是单纯地保存了地址，而没有保存这个地址上元素的类型（当地址被分配以后，它的类型就已经确定了）。所以说对于 `void *` ，可以进行的操作只有：**暂时保存某个指针的地址、把它的地址赋给另外一个元素类型正确的指针变量**。任何涉及类型的运算 `void *` 都不能执行。

    ```c
    int x=7,*p=&x;
    void *q=p;
    int *y=q;//以上都可以
    /*
    printf("%d\n",*q);
    *q=8;
    以上注释的操作都不行
    */
    ```

- 指针的作用：

    1）需要传入较大的数据时用作参数

    2）传入数组后对数组做操作

    3）函数返回时不止一个结果

    4）需要用函数来修改不止一个变量

    5）动态申请内存

- 开指针操作：`Node * ptr=(Node*)malloc(sizeof(Node))` ；`int *a=(int*)malloc(sizeof(int)*n)`

### 1.数组和指针的关系

- 数组名类型可以理解为一个封装了长度信息以及取址有所差别的 `const void *`

    对于取址的差别，即如果对数组 `a` 进行了一次取址操作，那么指针 `p` 的类型是 `int (*)[]`，而非 `int **`；但如果再取一次，那么系统就不管了，类型就是指向指针的指针。

    对于封装了长度信息，即数组和指针的 `sizeof` 结果是不一样的。

    当不涉及长度和取址的时候，数组退化成 `const void *` ；当把数组作为参数传入函数时，数组会退化成指针，即使参数变量取名时 `int []`。

- 在开一个 $n$ 的数组的时候，相当于预留了连续的 $n$ 个地址。

​		所以就有 $a[k]=*(a+k)$

### 2.指针的优先级

$*$ 本质上是解引用，也就是说把一个指向的值给解了，变成最原始的指针。

也就是说，p 在没有和 * 结合前都是值，一旦结合就是指针，**指向剩下还没结合的类型**。

括号 $()$ 的优先级最高，其次是数组 $[]$ ，然后是剩余的 $*$ ，然后是类型。

```c
//！！！！可以用“右结合”来记。
int p;				// p是一个普通的int型变量
int *p;				// p先与*结合，后与类型结合，所以p是一个指针，指针指向的内容是int型
int p[3];			// p先与[]结合，然后与类型结合，说明p是一个数组，数组里面的内容是int型
int *p[3];			// p先与[]结合，说明p是一个数组，然后再与*结合，说明数组p里面存放的是指针，然后与类型结合，说明指针指向的内容是int型，p是一个由返回整型数据的指针所组成的数组
int (*p)[3];		// ()的优先级最高，(*p)说明p是一个指针，然后与数组[3]结合，说明指针指向的是大小为3的一个数组，然后与类型结合，说明指针所指向的数组里面的内容为int型，p是一个指向由整型数据组成的数组的指针
int **p;			// p先和最近的*结合，说明p是一个指针，然后再与剩下的*结合，说明指针p指向的是一个指针，然后与int结合，说明指针p所指向的指针(*p)所指向的内容(**p)为int型
int p(int);			// p先与(int)结合，说明这是一个函数，函数的参数为int型，然后与外面的int结合，说明函数返回的内容是int型
int (*p)(int);		// ()的优先级最高，(*p)说明p是一个指针，然后与(int)结合，说明指针所指向的内容是函数，函数的参数为int型，然后与外面的int结合，说明指针p所指向的内容(*p)是一个函数，该函数的返回内容为int型
int *(*p(int))[3];	// 一层一层的剥开，p首先与(int)结合，说明p是一个函数（这里说了p是一个函数），函数的参数为int型，然后与*结合，说明函数返回的是一个指针，然后与数组[3]结合，说明是返回的指针指向的是一个数组，然后与*结合，说明数组里面的内容是指针，然后与外面的int结合，说明数组里面的指针指向的内容是int型。所以p是一个参数为int型并且返回的是一个指向内容为int*型的数组的函数，由于int*是指针，指针指向的内容为int型，所以p返回的就是一个指向内容为指向内容为int型的指针的数组的函数（太绕了，注意段句）。

//有关加减：就是从左到右，*++p是先进行加减操作，然后再和 * 结合；而 *p++ 是先解引用，返回 *p，然后 p++
//可以理解 ++ 和 * 是同级运算，比 [] 的优先级低。
```

```c
int a=1;
int *q=&a;
int **p=&q;
*p++;//去出 p 指向的数据，然后将 p 向右移一格。
//这样定义可以，但是不能直接 **p=&&a, 因为地址是不会再有另外一个指针指向它的，指针的存在是因为新开了一个变量 q，所以必定会有指向 q 的指针。
```

#### 正确的添加限定

需要注意的是，只有出现在 `*` 与标识符之间的限定才会被赋予指针类型。举例来说，语句 `char * const p;` 声明了一个类型为 `char *` 的、值不可变的指针 `p`，它指向的对象类型为 `char`，值可通过指针 `p` 改变；而语句 `const char * p;` 声明了一个类型为 `const char *` 的、值可变的指针 `p`，它指向的对象类型为 `const char`，值不可通过指针 `p` 改变。

### 3.有关指针和数组的 sizeof

在 C 语言中，输出 `size_t` 元素的类型为 `%zd`；

```c
int a[]={0,1,2};
int *p=a;
printf("%zd %zd",sizeof(p),sizeof(a));//输出 8 12
```

### 4.二维数组的地址开法

比如你开一个 `int a[3][3]` 的二维数组，那么计算机会给你分配九个 `int` 的地址空间。

此时数组 $a$ 会被转换成 `int (*)[3]` 类型的指针（和 `int **` 是不同的数据类型），指针指向的类型是 $int[3]$。即 $a$ 是指向第一个 `int [3]` 数组，随后 `a+i` 是向右移动了 $i*sizeoof(int[3])$ 个字节。

`*(a+i)` 也可以看成 `a[i]`，那么下一层就可以改成 `a[i]+j`，即从 `a[i]` 指向的地址开始，向右移动 $j*sizeof(int)$ 个字节，指针指向的类型是 $int$。

由此我们可以看出，其实 $a[1][-1]$ 其实是指向 $a[0]$ 的最后一个元素。

### 5.函数指针

函数指针的值，实际上是函数第一条指令的地址，也即函数的入口地址。

#### 函数的内存布局

在可执行文件中，指令与数据被分别存储在代码段与数据段中。每个函数的指令往往被存储在代码段中的一段连续内存空间中，根据这一特点，我们可以将函数看作一种特殊的对象。

**C 语言允许通过调用函数指针来调用函数**

也就是说，函数和函数指针是一回事情，所以你从 `(&fun)()` 到 `(******fun)()` 都是可以的，但是不能 `(&fun_ptr)()`，因为你调用函数指针的指针，鬼知道他是不是函数的指针。

### 6.有关 typedef

https://zhuanlan.zhihu.com/p/637324207?utm_id=0

```c
typedef int (*PTR_TO_ARRAY)[3];
PTR_TO_ARRAY x;
```

可以这么定义。

### 7.有关野指针

就是在定义指针的时候你不知道把它指到哪去了，所以也自然不能把值往野指针的地址里面塞。



## 程算被拷打记录

### 一些易错题

- After executing the following code fragment, the value of variable `m` is:

    ```c
    int m;
    for( m=0; m<9; m++ ) m++;
    ```

    answer: 10

    

- 以下代码，语法正确的是：

    ```c
    A. while();
    B. for();
    C. for(;;);
    D. do{}while();
    ```

    answer: C

    explanation：循环体在语法层面也是一个语句；所以结尾的分号是必备的（要么就是大括号，所以 `do ; while (1);` 也是合法语句），表示这个语句执行结束了。而循环体另外一个重要的元素就是循环条件，所以 `while` 里面必须填东西、`for` 里面必须有 `(;;)`。

    

- 在C语言程序中，若对函数类型未加显式说明，则函数的隐含类型为

    answer: `int`

    

- 正确的输出为：

    ```c
    int x = 5, y = 7;
    void swap ( )
    {
          int z ;
          z = x ;  x = y ;  y = z ;
    }
    int main(void)
    {
          int x = 3, y = 8;
          swap ( ) ;
          printf ("%d,%d \n", x , y ) ;
          return 0 ;
    }
    ```

    answer: 3 8

    

- 若函数调用时的实参为变量,下列关于函数形参和实参的叙述中正确的是()。

    A.函数的实参和其对应的形参共占同一存储单元

    B.形参只是形式上的存在, 不占用具体存储单元

    C.同名的实参和形参占同一存储单元

    D.函数的形参和实参分别占用不同的存储单元

    answer: D

    explanation：形参在执行函数的时候，会有属于它的存储单元；实参在外面也有它的存储单元（哪怕是一个 `expression`，也会有一个存储单元对应这个 `expression` 的结果）

    

- 以下说法正确的是：

    A.一个C语言源文件（.c文件）必须包含main函数

    B.一个C语言源文件（.c文件）可以包含两个以上main函数

    C.C语言头文件（.h文件）和源文件(.c文件）都可以进行编译

    D.在一个可以正确执行的C语言程序中，一个C语言函数的声明（原型）可以出现任意多次

    answer: D

    explanation：.c 文件不一定要包含 main 函数，多文件编译的时候肯定有程序不带 main 函数；函数声明即给出了函数名、返回值类型、参数列表（重点是参数类型）等与该函数有关的信息，称为函数原型（Function Prototype）。函数原型可以多次出现。

    

- 数组名作为参数时，以下叙述正确的是（ ）。

    A.函数调用时是将实参数组的所有元素值传给形参。

    B.函数调用时是将实参数组的第一个元素值传给形参。

    C.函数调用时是将实参数组的首地址传给形参。

    D.函数调用时是将实参数组的所有元素的地址传给形参。

    answer: C

    

- 关于递归函数以下叙述错误的是（ ）。

    A.递归函数是自己调用自己。

    B.递归函数的运行速度很快。

    C.递归函数占用较多的存储空间。

    D.递归函数的运行速度一般比较慢。

    answer: B

    explanation：占用较多空间——栈空间

    

- **这道题很重要（隐式转换）**

    For the function declaration `void f(char ** p)`，the definition __ of `var` makes the function call `f(var)` incorrect。

    A.`char var[10][10];`

    B.`char *var[10];`

    C.`void *var = NULL;`

    D.`char *v=NULL, **var=&v;`

    answer: A

    explanation:

    ![](https://github.com/Clovers2333/picx-images-hosting/raw/master/Hidden-conversion.3nrdjvgmo5.webp)

    

- For definitions: `char s[2][3]={"ab", "cd"}, *p=(char *)s;` the expression __ is correct and its value is equivalent to the element `s[1][1]`.

    A.`*(s+3)`

    B.`*s+2`

    C.`p[1][1]`

    D.`*++p+2`

    answer: D

    

- 以下正确的是：

    A.f 函数头改为`int f(int *a, int n)`，无需其它改变，效果完全一样。

    B.`main`函数中增加` int *p=a`;
    将` f(a,n);`改为`f(p,n);`，效果完全一样。

    C.用数组名作`f()`的参数，则`main()`中的数组与`f()`中的数组是同一数组。

    D.用数组名作`f()`的参数，实现了实参与形参双向的值传递。

    answer: D

    explanation：所谓的双向传递，就是在函数内修改形参的值，也会改变外面实参的值。但事实上参数类型是一个指针，形参只是新开了一个内存空间取保存传入的那个地址，所以原先的实参指针所代表的地址并不会变化。

    

- 结构体成员的类型必须是基本数据类型（）

    answer：F

    explanation：可能也是结构体

### C 语言标识符(identifier)（W1）

自定义的名称，用于表示变量、函数、数组等程序实体，必须由字母、数字、下划线组成，且首位不能是数字。`a$b_c` 也是标识符。



### while 括号内 --k 还是 k--（W1）

对于程序：

```c
int k=3,x=0;
while(k--){printf("%d\n",k); x++;}
printf("%d %d\n",x,k);
```

输出为：

```
2
1
0
3 -1
```

对于程序：

```c
int k=3,x=0;
while(--k){printf("%d\n",k); x++;}
printf("%d %d\n",x,k);
```

输出为：

```
2
1
2 0
```

`k--` 还是 `--k` 决定了是先拿 `while` 判断 `k` 再运算，还是先运算 `k` 再判断。

对于程序：

```c
int i, sum;
for (i = 1; i <= 10; i++){
    sum = sum + i;
}
```

这里 `i++` 和 `++i` 是一样的，但是要注意 `sum` 的初始值赋值了没。



### 运算优先级（W1）

![](https://github.com/Clovers2333/picx-images-hosting/raw/master/operation_priority.6pn9l3hyvf.webp)

- 其中单目运算符、赋值运算是**右结合**；赋值运算的运算优先级很低很低！

- 四则运算大于移位运算大于关系运算大于位运算（与大于异或大于或）大于条件运算大于赋值运算。

- 对于关系类运算符，运算顺序是左结合，就是从左到右一个个算。

- 逗号表达式：从左到右依次计算结果，并返回最右边的那个式子的结果；逗号的优先级是所有表达式最低的，比赋值运算还低。

- 对于 `&&` ：左边是 `False` 就不算了；对于`||`：左边是 `True` 就不算了

    

### 有关浮点数（W2）

为什么浮点数叫浮点数？——在一个定点数上加一个阶码，阶码用于移动小数点位置。

`float` 和 `double `的区别：

![](https://github.com/Clovers2333/picx-images-hosting/raw/master/The-difference-between-float&double.60u012ufv4.webp)

`float` 和 `double` 的默认输出都是保留 6 位小数

### 有关循环体（W3）

- 循环体如包括有一个以上的语句，则必须用一对大括号{}括起来，组成复合语句，复合语句在语法上被认为是一条语句。

    

### 有关 switch（W3）

```c
switch (expression){
    case value1:
        statement1;
        break;
    case value2:
    	statement2;
        break;
    default:
        default_statement;
}
```

先找到和条件相同的数值，随后依次往下进行（即如果该 case 没有 break ，则会继续进行下一个 case）

如果没有数值，就找 default，如果没有 default，那么这个 switch 无效

### 关键字(keyword)、操作符(operator)和函数的区别（W4）

- 操作符（`static operator`，如 `sizeof` ）会在编译阶段直接处理计算。

​		而函数则会在代码段中以 call 的形式去调用（代码运行时）

- 操作符不需要任何头文件都能运行，因为其是由编译器直接实现（或者说他的实现是由编译器完成的）

​		而函数的实现是在C语言库中实现的

- 程序运行时，会先将函数的参数入栈，然后再跳到函数的代码段运行，而操作符直接在原地计算

关键字：如 int，double 等已被语言使用的词



### 有关函数（W5）

- Function is a piece of code, which takes zero or many parameters, does one thing and returns zero or one value.

- `()` is the evidence of a function call

- 形参（`former parameter`）和实参（`real parameter`）的区别：

    ![](https://github.com/Clovers2333/picx-images-hosting/raw/master/The-difference-between-former&real-parameter.7i052tyklv.webp)

- `Local are to the execution rather than the function.`: 函数内的变量不是以函数为单位定义的，而是每执行一次就新分配内存空间，执行完释放。

- 函数可以嵌套调用但不能嵌套定义。

- **函数声明：只是说明有这个函数（函数里面的参数类型），不占用空间；函数定义：有函数体。一个程序同一个函数只能有一次定义，但是可以有多次声明。**

- **C 语言 main 函数的参数：**

    `main(int argc, char **argv)`

    其中 `argc` 表示编译命令的变量参数； `argv` 则表示了这些变量的名称

    ```c
    int main(int argc,char **argv)
    {
        for(int i=0;i<argc;i++)
            printf("%s\n",argv[i]);
    }
    /*
    编译命令：./me are you ok?
    输出：
    C:\Users\Clovers\Desktop\code\me.exe
    are
    you
    ok?
    */
    ```



### 有关递归（W7）

- 递归的4个基础 ：

    Base Case：⾄少存在⼀种情况，不再进⼀步递归就可以直接得到结果 

    Make Progress：每⼀次递归都是向着base case前进 

    Always Believe：始终相信递归是可⾏的 

    Compound Interest Rule：不要在不同的递归调⽤中做相同的计算

    

### 有关排序（W8）

**十种排序方法的具体分析及图示：** https://www.cnblogs.com/onepixel/articles/7674659.html

- 快速排序

- 注意冒泡排序的尾部优化

    ```c
    function bubbleSort(arr) {
        var len = arr.length;
        for (var i = len - 1; i >= 0; i--) {
            var loc = -1;
            for (var j = 0; j < len - 1 - i; j++) {
                if (arr[j] > arr[j+1]) {        // 相邻元素两两对比
                    var temp = arr[j+1];        // 元素交换
                    arr[j+1] = arr[j];
                    arr[j] = temp;
                    loc = j;
                }
            }
            i = loc + 1;
        }
        return arr;
    }
    ```

    时间复杂度：和逆序对个数有关。（$\sqrt{逆序对个数} * n$）？

- 时间复杂度测试程序：

    ```perl
    #include <time.h>
    time_t then = time(0);
    printf(“%llu\n”, time(0)-then);
    ```



### 有关字符串（W11）

- C 语言中的字符串其实是一个以终止字符结尾的字符数组 `\0`

- 字符串的大小计入中止字符，而其长度不计入。

​	如 `""` 为大小为 1 的字符数组，唯一元素是 `\0`. 一个 `char` 占用 1 个字节

- 数组的大小可以比字符串字面量的大小少 1，此时**中止字符将被忽略**。举例来说，语句 `char str[3] = "abc";` 等价于语句 `char str[3] = {'a', 'b', 'c'};` ，它不是一个字符串。

    合格的字符串必须结尾是`\0`，如 `char str[4]="abcd"`，那么它就等价于语句 `char str[4]={'a','b','c','\0'}`；对于字符指针 `char *` ，只有当它所指的字符数组有结尾的 `\0`，才能说它所指的是字符串）。

    `\0` 不算在字符长度判断里，但是算在字符串大小里。

    如果中止字符被忽略，那么就会一直输出字符串直到读到中止字符。

- `\b` 意味着光标左移一格；`\r` 意味着把光标移到行首。

- **以下内容非常很重要！**

    对于 `char *s="Hello"` ，`s` 是一个指针，初始化指向字符串的第一个字母。但注意，在这种情况下， `"Hello"` 它是一个不可变字符串，所以 `s` 的类型是 `const char *`，也就意味着不能改变字符串中的任意一个元素。

    但对于数组 `char s[]="Hello"`，那就没有问题（在这个定义语句中，决定了 `s` 的长度）。

    在编译程序的时候，字符串 `"Hello"` 不会改变（可以只看成一个文本，在可执行文件中保存了，在用 `.txt` 打开可执行文件的时候，这个字符串也会原封不动地出现）。后者是把他赋值到了一个可变的数组上，但前者那种情况只是把 `s` 指了过去，指向这个文本所在的位置，而这个文本是存储在静态存储区上的。或者说它不是程序运行时所开的空间——只有这些才可以被改变。它是编译之后就存在的文本——可读不可写，不能改变值。

    在 `printf("%s",s)` 时，秉持着输出字符串的原则，传入参数是一个 `char *`，从这个地址一直往后输出输出到 `\0` 为止（所以说字符数组必须要有 `\0` 才能算是字符串）。

- `scanf(“%7s” , string);` 最多读入七位；`printf("%7d\n",x);` 输出长度为 7.

- `char *string; scanf(“%s” , string);` 不行，因为你不知道 `string` 开了多大，读入可能溢出。

- 字符数组的不同定义：

    ![](https://github.com/Clovers2333/picx-images-hosting/raw/master/Differences-of-the-char-array-definition.8kzudpuegz.webp)

- 一些字符串函数的用法：（`string.h` 库）

    - `int strcmp(const char* str1, const char* str2);`比较两个字符串是否相等，相等返回 0 ，$str1>str2$ 返回 $> 0$ ，小于则返回 $<0$ 。
    - `char * strcpy ( char * destination, const char * source );` 把后者复制到前者，一直到 `\0` 。
    - `char * strncpy ( char * destination, const char * source, size_t n);` 把 `source` 前 `n` 个复制。
    - `void * memcpy( void * destination, const void * source, size_t n);` `n`：要拷贝多少字节。
    - `char *strcat(char *dest, const char *src);` 把后者移到前者末端，要有足够空间。
    - `char *strchr(const char *str, int c);` 查找 `c` 出现的第一个位置，返回指针（`int` ASCII）。
    - `char *strstr(const char* s2,const char* s1); ` 查找 `str` 出现的第一个位置，返回指针。
    - `int putchar(char c);` 返回 `c` 的 ASCII 码；
    - `int getchar();` 返回 `int` 是为了读入失败时返回 `EOF(-1)`



### 可变数组（W13）

```c
typedef struct { 
	int *array; 
	int size; 
} Array;
Array array_create(int init_size) { 
    Array a; 
    a.array = (int*)malloc(sizeof(int)*init_size); 
    a.size = init_size; 
    return a; 
}
void array_free(Array *a) { 
    free(a->array); 
    a->array = NULL; 
    a->size = 0; 
}
int array_size(const Array *a) { 
    return a->size; 
}
int* array_at(Array *a, int index) { 
    if ( index >= a->size ) { 
    array_inflate(a, index-a->size); 
    } 
    return &(a->array[index]); 
}
void array_inflate(Array *a, int more_size) { 
    int* p = (int*)malloc(sizeof(int)*(a->size+more_size)); 
    memcpy((void*) p, (void*) a->array, a->size*sizeof(int)); 
    free(a->array); 
    a->array = p; a->size = a->size+more_size; 
}//太暴力辣
```



### 有关数据类型（W14）

`sizeof:` `char 1; short 2(-32768~32767); int=long 4; float: 4 (8位有效); double 8(16位有效)`。

`double` 存储分配：

![](https://github.com/Clovers2333/picx-images-hosting/raw/master/Internal-Presentation-of-Floating.1ov6tjb4cn.webp)

- `bool` 所在库：`<stdbool.h>`

- `int` 取决于 CPU；但是 `long` 是固定的四个字节。



### 原码、反码和补码（W14）

原码：开头为符号位，后面均为数值位。

若为正数，原码=反码=补码；

若为负数，则反码为原码除了符号位每一位取反，补码为反码+1；

两个数相加，在计算机里面就是补码二进制相加。

- -128 补码（对于八位来说）为 `10000000`；原码和反码的可表示区间都是 `-127~127`，因为 0 被表示了两次。
- **补码：$-2^n +$ 后面非符号位的数值**，即对于负数反过来表示。



### 有关进制和`\`转义（W14）

- A literal with a leading 0 is an octal（8进制）
- A literal with a leading 0x is a hexadecimal（16进制）
- `%o` for octal,`%x` for hexadecimal
- `\x` 后面接 0-9 A-F 字符通过十六进制来表示一个字符。不过并不会有长度限制，比如 `'\x00000041'` 也是一个字符，同样遇到范围外的字符就结束。
- 后面可以接最多**三个数字**来使用八进制表示一个字节，且遇到 0-7 范围之外的数字会结束当前字节，比如 `'\101'` 也表示 'A'，而 `'\08'` 因为 8 超过了八进制范围，所以这是两个字符放到了一个单引号里，不是正确的用法，如果写成字符串，则 `"\08"` 表示两个字符。



### 编译预处理指令（宏）(W14)

![](https://github.com/Clovers2333/picx-images-hosting/raw/master/Compiling-process.7ljr0jrnb5.webp)

- `#` 开头的是编译预处理指令 

​		它们不是C语⾔的成分，但是C语⾔程序离不开它们

- `#define` ⽤来定义⼀个宏（`#define <名字> <值>`）；

    在C语⾔的编译器开始编译之前，编译预处理程序(cpp)会把程序中的名字换成值

- 宏定义不存在类型问题，宏名无类型，它的参数也无类型。

- **如果⼀个宏的值中有其他的宏的名字，也是会被替换的**

    如果⼀个宏的值超过一行，最后一行之前的行末需要加 `\`

- 带参数的宏的规则——一切都要括号（整个值要括号，参数出现的每个地方都要括号，没有**可能**会出问题）

    `#define RADTODEG(x) ((x) * 57.29578)`

    `#define MIN(a,b) ((a)>(b)?(b):(a))`

- 可以只 `#define flag` 这样，因为还有一种语句叫做 `#ifdef`

    ```c
    #define flag
    #ifdef flag
    #define format %3lf
    #else
    #define format %2lf
    
    printf("format", 1.2345);
    //比如有好多 printf ，这样我们可以通过只改变宏是否 define 来整体修改输出格式
    ```

- `#define` 语句最好还是别带分号：

    ```c
    #define PRETTY_PRINT(msg) printf(msg);
    if (n < 10)
    	PRETTY_PRINT("n is less than 10");
    else
    	PRETTY_PRINT("n is at least 10");
    //这个代码会 CE，因为相当于 if();;else
    ```

- 部分宏会被 `inline` 函数替代：

    `inline` 相当于在编译的时候，在函数调用的地方将直接原地展开，就是类似于 `#define`。

    内联（`inline`）是以代码膨胀（复制）为代价，仅仅省去了函数调用的开销，从而提高函数的执行效率。如果执行函数体内代码的时间，相比于函数调用的开销较大，那么效率的收获会很少。另一方面，每一处内联函数的调用都要复制代码，将使程序的总代码量增大，消耗更多的内存空间。

    使用方法：不是在声明函数的时候加 `inline` ，而是在定义的时候加。



### 多文件编程（W14）

- 编译指令：`gcc <a.c> <b.c> ... -o me`

- 编译过程：其实就是把所有函数入口都先准备好，所有全局变量都先开好，然后找到 `main` 开始运行。

- `#include` 是 `""` 还是 `<>`：`""` 要求编译器首先在当前目录(`.c`文件所在的目录)寻找这个文件，如果没有，到编译器指定的目录去找； `<>` 让编译器只在指定的目录去找

- 编译器自己知道自己的标准库头文件在哪，环境变量和编译器命令行参数也可以指定寻找头文件的目录

- 一些反直觉的东西：（好像只是在 Windows 中的垃圾 dev 里面有）

    ![](https://github.com/Clovers2333/picx-images-hosting/raw/master/Misunderstanding-of-include.5j3ycht2a1.webp)

- 不对外公开函数：在函数前面加上 `static ` 就使得它成为只能在所在的编译单元中被使用的函数（编译单元：一个文件）

- 一般来说，头文件（`.h`） 放声明不放定义（是规则不是法律）；最好把所有要声明的都给声明了；多文件编译过程中，如果要用外部的变量，必须提前声明。（提前 `incldue` 或者用 `extern`）

    

### 变量的声明（W14）

`int i;` 是变量的定义； `extern int i;` 是变量的声明

声明是不产生代码的东西；定义是产生代码的东西。



### 文件操作（W15）

- `printf/scanf` 的返回值：输出/读入字符数

- `while()` 一直读入如何判断结束：getchar读到了EOF、scanf 返回小于要求读的数量。

- `fopen(const char *path, const char *mode);` `path` 是文件名，`mode` 是打开文件的方式

- r 以只读方式打开文件，该文件必须存在。
    r+ 以可读写方式打开文件，该文件必须存在。
    w 打开只写文件，若文件存在则文件长度清为0，即该文件内容会消失。若文件不存在则建立该文件。
    w+ 打开可读写文件，若文件存在则文件长度清为零，即该文件内容会消失。若文件不存在则建立该文件。

- 需要一个`FILE*`型的指针接收`fopen`的返回值，拿着这个返回值就可以调用相应的接口对文件进项操作。

- 读入失败：`FILE* p=NULL`；关闭： `fclose(p)`；

    ```c
    a='string';
    fid=fopen('d:\char1.txt','w');
    fprintf(fid,'%s',a);
    fclose(fid);
    fid1=fopen('d:\char1.txt','rt');
    fid1=fopen('d:\char1.txt','rt');
    int x;
    fscanf(fid1,'%d',&x);
    ```

- 格式化输入输出（printf，scanf）还有两种变体：sprintf，sscanf 第一个参数为字符串（char\*），会输出到字符串中 / 从字符串中读取输入；fprintf，fscanf 第一个参数为文件指针（FILE\*），会输出到文件中 / 从文件中读取输入。



### 有关“链接错误”（W15）

链接错误有两种类型：

1）缺定义。当程序中出现对某个外部对象的使用，而连接程序找不到对应的定义时，将产生这个错误。（有声明但是没有定义）
2）重复定义。当被连接的各个部分中出现某个名字的多个定义时，将产生这个错误。

e.g. ![](https://github.com/Clovers2333/picx-images-hosting/raw/master/An-example-of-linking-error.4jnuzbqb3y.webp)

答案应该是链接错误（题目错了应该 `include` 一下 `stdio.h`）。

因为 `include` 相当于文本替换，多文件编译的时候你会发现 `include` 了两遍 `header.h` 导致 `school` 被定义了两次。

​	——所以这就是强调 `.h` 里面不要放定义，最好只放声明。

​	`#include<stdio.h>` 两次不会出现问题也是因为它里面只有声明。

### 有关存储类别（W15）

#### 1.自动变量（auto变量）

函数中的局部变量，如果不专门声明为（static）静态存储类别，都是动态地分配存储空间的，数据存储在动态存储区中。

形参与函数中定义的局部变量，都是自动变量，在调用结束就会自动释放这些存储空间

#### 2.静态变量（static变量）

（1）静态**局部**变量属于静态存储类别，在静态存储区分配存储单元，在程序整个运行期间都不释放。

（2）如果在定义局部变量时不赋初值，则对静态局部变量来说，编译时自动赋初值0（对数值型变量）或空字符'\0'（对字符变量）。

（3）虽然静态局部变量在函数调用结束后仍然存在，但其他函数是不能引用它的。因为它是局部变量，只能被本函数引用。

并且，加了 `static` 后只能在本编译单元内使用。

#### 3.寄存器变量（register变量）

一些变量使用频繁，存取变量要花费不少时间，提高效率，允许把局部变量放在CPU中的寄存器中，直接从寄存器中存取，可以提高执行效率，这种变量叫做寄存器变量

以上 3 种局部变量的存储位置是不同的：自动变量存储在动态存储区，静态变量存储在静态存储区、寄存器存储在CPU中的寄存器中。

#### 4.外部变量(extern变量)

从其它编译单元来的变量；所以这个 `extern` 也只是个声明而非定义。