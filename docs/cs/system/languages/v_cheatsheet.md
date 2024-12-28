# Verilog Cheatsheet

## 基本语法

### 标识符、常量和注释

#### 标识符

- 所描述对象的名字称为“标识符”；标识符可包含字母、数字、下划线和 $ 符号；以字母或下划线开头。

#### 常量

- 整数常量表示方式：`<size>'<base><value>`；可通过下划线来分割数字，提升可读性。
- `value` 中除了数字以外，还可以包含 `x`（非法值或不定态，0/1 之间电压值）、`z`（浮空值或高阻态，导线悬空）

#### 注释

同 C/C++.

### 数据类型

**推荐用 `logic` 全面替换 `wire` 和 `reg`**，`verilog` 语言会自己判定 `logic` 的类型。

- 网线类型（net，提供元件或模块之间的物理连接）：如 `wire`.
- 寄存器类型（register，抽象的存储元件）：`reg`；在给寄存器赋予新值之前，寄存器将保存上一次的赋值结果，这与触发器和锁存器的性质类似。
- 参数类型（parameter）：给常量赋予有意义的标识符（const）
- 向量和数组：`reg[MSB:LSB] <memory_name> [first_addr:last_addr]`，即定义了一个从 `MSB` 到 `LSB` 的向量，然后以这个向量为元素开了一个数组，索引为 `first_addr` 到 `last_addr`.

#### 模块端口函数

- 输入端口：
    - 在模块定义时，模块的输入端口被看作父模块到子模块输入端口的物理连接，所以只能定义为 `wire` 类型。
    - 在模块实例化时，可将子模块输入端口关联的父模块信号看成物理连接的驱动源，可以为 `wire` 类型或 `reg` 类型。
    - 输出端口既可以是 `wire` 也可以是 `reg`.
    - 如果定义输入输出没有说明数据类型，缺省为 `wire`.

### 运算符

- `&a`：将数组 `a` 中所有元素与起来
- 位拼接：`{a, {2{c, b}}} = {a, c, b, c, b}`

其余语法和 C 语言类似。



## Verilog 建模方式

### 结构化建模

1. 模块及结构化建模：即用户自己设计子模块。
2. 门级结构化建模：通过逻辑运算等语法，调用 Verilog 内建的基本门级元件
3. 开关级结构化建模

### 数据流建模（assign）

`assign 网线 = 表达式`

- 将右边的输出作为左边的输入

- 左边必须为 `wire`，右边都行

- 从物理结构来理解，**赋值语句的顺序 doesn't matter，这也是我们必须从硬件角度研究语言的原因**。

    ```verilog
    assign a = b + c;
    assign b = c & d;
    
    assign b = c & d;
    assign a = b + c;
    // 上面两组赋值没有区别
    ```

- 不能对相同的网线进行重复的驱动（即两个不同的 assign）

- 不能出现回路

### 行为建模

`always @ (事件信号列表) 过程语句`

- 过程语句中被赋值的只能是 `reg` 类型变量。

- 事件信号列表多个信号用 `,` 分隔，用关键字 `posedge/negedge` 来修饰，表示只检测上升沿或者下降沿。

    ```verilog
    always @ (posedge clk)
        q <= d
    // 当 clk 信号上升沿到来时，将 d 的值更新到 reg 变量 q
    ```

    事件信号列表也可以用 `*`，表示所有要用到的信号列表，就是说所有会影响这个模块运行的信号发生变化时，就会执行这个模块。

#### 行为建模中的过程语句

- `begin/end`：视为左花括号、右花括号。
- 阻塞赋值语句：`reg = expression`，在赋值语句结束以后立即更新（可用于描述组合逻辑电路）
- 非阻塞赋值语句：`reg <= expression`，在整个 always 结束以后并行更新（可用于描述时序逻辑电路）

**Verilog 语言允许再一个 always 语句中对同一个 reg 类型变量进行多次赋值，并以最后一次赋值为准，但不能在多个 always 语句里面对同一个 reg 类型变量进行赋值！**（这相当于多个电路逻辑同时连向同一个寄存器，相当于两个 assign，所以不行）

- case 语句：

```verilog
case (expr0)
    expr1: statement1
    expr2: statement2
    [default: statement] //可以不给出
endcase
```

- 循环语句：

```verilog
for(expr1; expr2; expr3) statement
```

语法和 C 语言相同，从硬件的角度来理解，不应该理解成电路在时间上重复，而应该理解成在空间上重复构建多个电路。

#### 组合逻辑和时序逻辑区别

**从硬件角度思考，一个是单纯的导线，另一个则是触发器、锁存器等。**

- 对于组合逻辑电路，使用 `always@(*)` 的时候，`if` 是需要搭配 `else` 使用的，因为 `always@(*)` 综合得到的电路是用 `wire` 搭建的，不然会导致环路错误（如果没有 `else` 就会保持）；

- 但是时序逻辑电路不需要，`always@(posedge clk)` 只是借用了 `reg` 的 `always` 块语法而已，它得到的电路使用真实的寄存器搭建的，是不会形成环路问题的（`else` 保持的情况已经写在触发器、锁存器里面了，不会有问题）

- 对于时序电路，只推荐以下四种行为建模：

    ```verilog
    always@(posedge clk) // 上升沿触发
    
    always@(negedge clk) // 下降沿触发
    
    always@(posedge clk or negedge rstn) // 上升沿触发和下降沿复位
    
    always@(posedge clk or posedge rstn) // 上升沿触发和上升沿复位
    ```

    - 多边沿、多时钟触发都会无法构成时序电路。
    - 异步触发因为毛刺而产生数据竞争：

    ```verilog
    reg [1:0] d;
    wire problem;
    reg cond1;
    reg cond2;
    assign problem = cond1 & cond2;
    
    always@(posedge problem)begin
        d <= d + 2'b1;
    end
    // what if the speed of signal cond1/cond2 is different?
    // solution: use a uniform signal---clk.
    ```

#### 其他 `always` 语法

**always_comb:**

你还在为 `always@(*)` 语法的看起来在做 `reg` 编程实际上在做 `wire` 编程苦恼吗？

`always_comb` 关键字就解决了这个问题，它直接把 `always` 块变成组合逻辑电路的，内部被赋值的 `logic` 都会被综合为 `wire`，然后会对 `if-else` 缺失、`case-default` 缺失等问题进行检查。大家可以用`logic+always_comb`语法全面替代`reg+always@(*)`语法，这样在享受 `always` 便利的行为描述语法的同时，也可以享受编译器的电路问题检查。

```verilog
// 修改前
reg d;
always@(*)begin
    if(a) d <= c;
    else d <= b;
end

// 修改后
logic d;
always_ff begin
    if(a) d <= c;
    else d <= b;
end
```

**always_ff:**

内部赋值的 `logic` 全部变成 `reg`.

```verilog
always_ff@(posedge clk)begin
    if(problem) d <= d + 2'b1;
end
```



## 高级语法

### `generate` 语句

`generate ... endgenerate` 语句块用于硬件代码生成。

```verilog
    wire [3:0] a [3:0];
    wire [3:0] b [3:0];

    assign b[0] = a[0];
    assign b[1] = a[1];
    assign b[2] = a[2];
    assign b[3] = a[3];
//-----------------------------

    wire [3:0] a [3:0];
    wire [3:0] b [3:0];

    genvar i;
    generate
        for(i=0;i<4;i=i+1)begin
            assign b[i] = a[i];
        end
    endgenerate
```

`genvar i` 定义了一个仅用于 `generate` 语句内部的变量 `i`，这个变量仅仅是一个用于代码生成的变量，它不会生成有效的电路，因而也不会提高生成硬件的复杂度。

一个 `genvar` 可以在多个 `generate` 块中使用。

可以在 `generate` 块内部使用 `for` 循环对 `genvar` 进行赋值遍历，这里的语法和 C 语言的 for 循环语法在语法和语义上是一致的，最大的区别可能是 `genvar` 的递增仅支持 `i=i+1`，而不支持 `i+=1` 和 `i++`.

### `integer` 变量

`integer` 变量是 32 位的变量，可以认为它是一个 32 位 bit 宽的 `reg`

除此之外还有 `short`、`longint` 分别是 16 位变量和 64 位变量。

```verilog
    integer i;
    initial begin
        ...
        for(i=0;i<8;i=i+1)begin
            data=i[3:0];
            #5;
        end
        ...
    end
```

 `initial` 块的 `for` 循环语法不同于 `generate` 块，它使用 `integer` 得到变量而不是 `genvar` 变量。

`initial` 块的 `for` 循环和 `generate` 的 `for` 循环在语法上是保持一致的，但是语义上有细微区别：generate 的 for 循环生成的语句用于电路描述，相互之间是**空间并列关系**；`initial` 的 `for` 循环生成的语句用于仿真激励，依次之间是由激励的**时间先后关系**的。

### 参数化编程

```verilog
    `define LEN 4
    wire [3:0] a [`LEN-1:0];
    wire [3:0] b [`LEN-1:0];

    genvar i;
    generate
        for(i=0;i<`LEN;i=i+1)begin
            assign b[i] = a[i];
        end
    endgenerate
```

```verilog
    localparam LEN=4; //相当于 const
    wire [3:0] a [LEN-1:0];
    wire [3:0] b [LEN-1:0];

    genvar i;
    generate
        for(i=0;i<LEN;i=i+1)begin
            assign b[i] = a[i];
        end
    endgenerate
```

假设我们将上面的二维数组赋值封装为一个模块 `dummy`。如果我们希望二维数组宽度为 4，则可以将 `LEN` 修改为 4；如果我们希望二维数组宽度为 5，则可以将 `LEN` 修改为 5，那么这个时候我们就可以在封装模块的时候定义一个 `parameter`.

```verilog
module dummy #(parameter LEN = 4) (
    input ...
    output ...
);
    
dummy #(.LEN(LEN))
    A1(.a(),
       .b());
```

## 其它语法

### 调试与对拍

- `$random()`：提供宽度为 32 位的随机数。

- `$display`：和 C 里面的 `printf` 非常类似：

    ```verilog
    $display("Start simulation");        //显示字符串
    $display("data_play = %h hex", 100);     //显示data_play的16进制数（或者其他进制）
    $display("Simulation time is %t", $time);     //显示仿真的时间
    ```

- 

