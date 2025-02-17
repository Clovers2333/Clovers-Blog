## Data Lab

- 具体注释已经写在代码里了

## Boom Lab

- 反汇编：`objdump -d bomb > bomb.s`
- `main` 函数调用：

![](https://github.com/Clovers2333/picx-images-hosting/raw/master/boom_lab1.wiiboe4pn.webp)

### Phase_1

```assembly
0000000000400ee0 <phase_1>:
  400ee0:       48 83 ec 08             sub    $0x8,%rsp
  400ee4:       be 00 24 40 00          mov    $0x402400,%esi
  400ee9:       e8 4a 04 00 00          call   401338 <strings_not_equal>
  400eee:       85 c0                   test   %eax,%eax
  400ef0:       74 05                   je     400ef7 <phase_1+0x17>
  400ef2:       e8 43 05 00 00          call   40143a <explode_bomb>
  400ef7:       48 83 c4 08             add    $0x8,%rsp
  400efb:       c3                      ret
```

- `test`：如果 `%eax` 为 0，那么设置 ZF 位为 1.
- `je`：如果 ZF 为 1，那么就 jump

### Phase_2

```assembly
0000000000400efc <phase_2>:
  400efc:       55                      push   %rbp
  400efd:       53                      push   %rbx
  400efe:       48 83 ec 28             sub    $0x28,%rsp
  400f02:       48 89 e6                mov    %rsp,%rsi
  400f05:       e8 52 05 00 00          call   40145c <read_six_numbers>
  400f0a:       83 3c 24 01             cmpl   $0x1,(%rsp)
  400f0e:       74 20                   je     400f30 <phase_2+0x34>
  400f10:       e8 25 05 00 00          call   40143a <explode_bomb>
  400f15:       eb 19                   jmp    400f30 <phase_2+0x34>
  400f17:       8b 43 fc                mov    -0x4(%rbx),%eax
  400f1a:       01 c0                   add    %eax,%eax
  400f1c:       39 03                   cmp    %eax,(%rbx)
  400f1e:       74 05                   je     400f25 <phase_2+0x29>
  400f20:       e8 15 05 00 00          call   40143a <explode_bomb>
  400f25:       48 83 c3 04             add    $0x4,%rbx
  400f29:       48 39 eb                cmp    %rbp,%rbx
  400f2c:       75 e9                   jne    400f17 <phase_2+0x1b>
  400f2e:       eb 0c                   jmp    400f3c <phase_2+0x40>
  400f30:       48 8d 5c 24 04          lea    0x4(%rsp),%rbx
  400f35:       48 8d 6c 24 18          lea    0x18(%rsp),%rbp
  400f3a:       eb db                   jmp    400f17 <phase_2+0x1b>
  400f3c:       48 83 c4 28             add    $0x28,%rsp
  400f40:       5b                      pop    %rbx
  400f41:       5d                      pop    %rbp
  400f42:       c3                      ret
```

#### read_six_numbers

```assembly
000000000040145c <read_six_numbers>:
  40145c:       48 83 ec 18             sub    $0x18,%rsp
  401460:       48 89 f2                mov    %rsi,%rdx
  401463:       48 8d 4e 04             lea    0x4(%rsi),%rcx
  401467:       48 8d 46 14             lea    0x14(%rsi),%rax
  40146b:       48 89 44 24 08          mov    %rax,0x8(%rsp)
  401470:       48 8d 46 10             lea    0x10(%rsi),%rax
  401474:       48 89 04 24             mov    %rax,(%rsp)
  401478:       4c 8d 4e 0c             lea    0xc(%rsi),%r9
  40147c:       4c 8d 46 08             lea    0x8(%rsi),%r8
  401480:       be c3 25 40 00          mov    $0x4025c3,%esi
  401485:       b8 00 00 00 00          mov    $0x0,%eax
  40148a:       e8 61 f7 ff ff          call   400bf0 <__isoc99_sscanf@plt>
  40148f:       83 f8 05                cmp    $0x5,%eax
  401492:       7f 05                   jg     401499 <read_six_numbers+0x3d>
  401494:       e8 a1 ff ff ff          call   40143a <explode_bomb>
  401499:       48 83 c4 18             add    $0x18,%rsp
  40149d:       c3                      ret
```

- 通过 `%rdi, %rsi, %rdx, %rcx, %r8, %r9` 六个寄存器来传输值，通过 `%rax` 来返回值。
- 所以这个 `read_six_numbers` 函数本质上是，先把参数寄存器设置好，然后调用 `sscanf` 函数。

```
int sscanf(const char *buffer, const char *format, [argument]...)
```

其中，`buffer`是输入的字符串，`format`是字符串的格式，`argument`是根据 `format` 提取出来的内容保存的位置，而该函数的返回值为格式化参数的数目，然后看看运行这个函数前干了什么。

观察 `main` 函数，发现 `%rdi` 保存字符串的地址，再观察该函数和 `phase_2`，`%rsi`保存的是函数`phase_2`的栈顶地址。

- 第三个参数`%rdx`保存栈顶地址
- 第四个参数`%rcx`保存栈顶地址向上偏移`0x4`
- 第五个参数`%r8`保存栈顶指针向上偏移`0x8`
- 第六个参数`%r9`保存栈顶指针向上偏移`0xc`

并且该函数输入超过6个参数的话，就会将其他的参数保存到内存地址中，其中有两个参数保存在内存中，对应的汇编代码为：

```assembly
401467:       48 8d 46 14             lea    0x14(%rsi),%rax
40146b:       48 89 44 24 08          mov    %rax,0x8(%rsp)
401470:       48 8d 46 10             lea    0x10(%rsi),%rax
401474:       48 89 04 24             mov    %rax,(%rsp)
```

并且越靠前的参数，保存的地址越小，所以

- 第七个参数保存在地址`%rsp`处，内容是栈顶指针向上偏移`0x10`
- 第八个参数保存在地址`%rsp+0x8`处，内容是栈顶指针向上偏移`0x14`

最后修改了第二个参数`%rsi`保存`0x4025c3`，输出发现是输入的格式："%d %d %d %d %d %d"

前两个 phase 都不太熟悉操作，希望后面能自己独立做一下。

### Phase_3

```assembly
0000000000400f43 <phase_3>:
  400f43:       48 83 ec 18             sub    $0x18,%rsp
  400f47:       48 8d 4c 24 0c          lea    0xc(%rsp),%rcx
  400f4c:       48 8d 54 24 08          lea    0x8(%rsp),%rdx
  400f51:       be cf 25 40 00          mov    $0x4025cf,%esi
  400f56:       b8 00 00 00 00          mov    $0x0,%eax
  400f5b:       e8 90 fc ff ff          call   400bf0 <__isoc99_sscanf@plt>
  400f60:       83 f8 01                cmp    $0x1,%eax
  400f63:       7f 05                   jg     400f6a <phase_3+0x27>
  400f65:       e8 d0 04 00 00          call   40143a <explode_bomb>
  400f6a:       83 7c 24 08 07          cmpl   $0x7,0x8(%rsp)
  400f6f:       77 3c                   ja     400fad <phase_3+0x6a>
  400f71:       8b 44 24 08             mov    0x8(%rsp),%eax
  400f75:       ff 24 c5 70 24 40 00    jmp    *0x402470(,%rax,8)
  400f7c:       b8 cf 00 00 00          mov    $0xcf,%eax
  400f81:       eb 3b                   jmp    400fbe <phase_3+0x7b>
  400f83:       b8 c3 02 00 00          mov    $0x2c3,%eax
  400f88:       eb 34                   jmp    400fbe <phase_3+0x7b>
  400f8a:       b8 00 01 00 00          mov    $0x100,%eax
  400f8f:       eb 2d                   jmp    400fbe <phase_3+0x7b>
  400f91:       b8 85 01 00 00          mov    $0x185,%eax
  400f96:       eb 26                   jmp    400fbe <phase_3+0x7b>
  400f98:       b8 ce 00 00 00          mov    $0xce,%eax
  400f9d:       eb 1f                   jmp    400fbe <phase_3+0x7b>
  400f9f:       b8 aa 02 00 00          mov    $0x2aa,%eax
  400fa4:       eb 18                   jmp    400fbe <phase_3+0x7b>
  400fa6:       b8 47 01 00 00          mov    $0x147,%eax
  400fab:       eb 11                   jmp    400fbe <phase_3+0x7b>
  400fad:       e8 88 04 00 00          call   40143a <explode_bomb>
  400fb2:       b8 00 00 00 00          mov    $0x0,%eax
  400fb7:       eb 05                   jmp    400fbe <phase_3+0x7b>
  400fb9:       b8 37 01 00 00          mov    $0x137,%eax
  400fbe:       3b 44 24 0c             cmp    0xc(%rsp),%eax
  400fc2:       74 05                   je     400fc9 <phase_3+0x86>
  400fc4:       e8 71 04 00 00          call   40143a <explode_bomb>
  400fc9:       48 83 c4 18             add    $0x18,%rsp
  400fcd:       c3                      ret
```

- 先观察 `sscanf` 的参数：
    - `mov $0x4025cf,%esi`，print 后发现输入格式是 `%d %d`；
    - `%rdx, rcx`作为第三、四个参数，存储输入的两个整数；
    - 补充一下，下面的 cmp 和 jg 是比较 dest 和 src，前 src 后 dest，若 dest > src，则进行 jg 后面跳转的 label
- `jmp *0x402470(,%rax,8)`：`*` 指示这是一个间接跳转，内存位置 = 基址 + (变址寄存器的值 * 缩放因子)，跳转到的是计算得到的地址内存所指向的地址（地址运算如果不是 lea 都会自动解引用）

ok 啊，到这里卡住了，一是没想到这本质上是一个 switch，再者是不知道怎么用 gdb 看这个表。

本质上就是根据第一个输入，来跳到 switch 表的位置，然后 第二个输入符合表中数值即可。

首先用 `x/w 0x402478`，检查如果输入第一个数为 1，那么跳转的地址是什么，发现内存中的值为 `400fb9`.

然后看一下这个位置的命令是什么，`mov $0x137,%eax`，直接 `print /d 0x137` 即可

### Phase_4

```assembly
000000000040100c <phase_4>:
  40100c:       48 83 ec 18             sub    $0x18,%rsp
  401010:       48 8d 4c 24 0c          lea    0xc(%rsp),%rcx
  401015:       48 8d 54 24 08          lea    0x8(%rsp),%rdx
  40101a:       be cf 25 40 00          mov    $0x4025cf,%esi
  40101f:       b8 00 00 00 00          mov    $0x0,%eax
  401024:       e8 c7 fb ff ff          call   400bf0 <__isoc99_sscanf@plt>
  401029:       83 f8 02                cmp    $0x2,%eax
  40102c:       75 07                   jne    401035 <phase_4+0x29>
  40102e:       83 7c 24 08 0e          cmpl   $0xe,0x8(%rsp)
  401033:       76 05                   jbe    40103a <phase_4+0x2e>
  401035:       e8 00 04 00 00          call   40143a <explode_bomb>
  40103a:       ba 0e 00 00 00          mov    $0xe,%edx
  40103f:       be 00 00 00 00          mov    $0x0,%esi
  401044:       8b 7c 24 08             mov    0x8(%rsp),%edi
  401048:       e8 81 ff ff ff          call   400fce <func4>
  40104d:       85 c0                   test   %eax,%eax
  40104f:       75 07                   jne    401058 <phase_4+0x4c> # eax 为 0 则跳转
  401051:       83 7c 24 0c 00          cmpl   $0x0,0xc(%rsp)
  401056:       74 05                   je     40105d <phase_4+0x51>
  401058:       e8 dd 03 00 00          call   40143a <explode_bomb>
  40105d:       48 83 c4 18             add    $0x18,%rsp
  401061:       c3                      ret
```

#### func4

```assembly
0000000000400fce <func4>:
  400fce:       48 83 ec 08             sub    $0x8,%rsp
  400fd2:       89 d0                   mov    %edx,%eax
  400fd4:       29 f0                   sub    %esi,%eax
  400fd6:       89 c1                   mov    %eax,%ecx
  400fd8:       c1 e9 1f                shr    $0x1f,%ecx # 提取符号位
  400fdb:       01 c8                   add    %ecx,%eax
  400fdd:       d1 f8                   sar    %eax # 右移没有参数默认为 1
  400fdf:       8d 0c 30                lea    (%rax,%rsi,1),%ecx
  400fe2:       39 f9                   cmp    %edi,%ecx
  400fe4:       7e 0c                   jle    400ff2 <func4+0x24>
  400fe6:       8d 51 ff                lea    -0x1(%rcx),%edx
  400fe9:       e8 e0 ff ff ff          call   400fce <func4>
  400fee:       01 c0                   add    %eax,%eax
  400ff0:       eb 15                   jmp    401007 <func4+0x39>
  400ff2:       b8 00 00 00 00          mov    $0x0,%eax
  400ff7:       39 f9                   cmp    %edi,%ecx
  400ff9:       7d 0c                   jge    401007 <func4+0x39>
  400ffb:       8d 71 01                lea    0x1(%rcx),%esi
  400ffe:       e8 cb ff ff ff          call   400fce <func4>
  401003:       8d 44 00 01             lea    0x1(%rax,%rax,1),%eax
  401007:       48 83 c4 08             add    $0x8,%rsp
  40100b:       c3                      ret
```

- 通过 `%rdi, %rsi, %rdx, %rcx, %r8, %r9` 六个寄存器来传输值，通过 `%rax` 来返回值。
- 调用 `func4` 的时候，第一个参数为第一个数字，第二个参数为 0，第三个参数为 14.
- ok 啊，继续不会，看到递归就头大了，其次对位运算也不是很有把握。
- gpt，启动！

```c
int func4(int a=arg1, int b=0, int c=14){
  if(c-b>=0){
    int ans=(c-b)/2;
  }else{
    int ans=(c-b+1)/2;
  }
  int temp1=ans+b;
  if(temp1-a<=0){
    int ans=0;
    if(temp1-a>=0){
      return ans;
    }else{
      b=temp1+1;
      int ans = func4(a,b,c);
      ans=ans*2+1;
      return ans;
    }
  }else{
    c = temp1-1;
    int ans = func4(a,b,c);
    return ans*2;
  }
} 
```

- 笑死了 突然发现第二个值是 0，并且第一个数 < 14，那么不是只要一个个试就行了。（盲猜不会是负数）
- 实际上看代码也行，发现要求返回值 `%eax=0`, 那么直接令 arg1=7 即可。

### Phase_5

```assembly
0000000000401062 <phase_5>:
  401062:       53                      push   %rbx
  401063:       48 83 ec 20             sub    $0x20,%rsp
  401067:       48 89 fb                mov    %rdi,%rbx
  40106a:       64 48 8b 04 25 28 00    mov    %fs:0x28,%rax
  401071:       00 00
  401073:       48 89 44 24 18          mov    %rax,0x18(%rsp)
  401078:       31 c0                   xor    %eax,%eax
  40107a:       e8 9c 02 00 00          call   40131b <string_length>
  40107f:       83 f8 06                cmp    $0x6,%eax
  401082:       74 4e                   je     4010d2 <phase_5+0x70>
  401084:       e8 b1 03 00 00          call   40143a <explode_bomb>
  401089:       eb 47                   jmp    4010d2 <phase_5+0x70>
  40108b:       0f b6 0c 03             movzbl (%rbx,%rax,1),%ecx
  40108f:       88 0c 24                mov    %cl,(%rsp)
  401092:       48 8b 14 24             mov    (%rsp),%rdx
  401096:       83 e2 0f                and    $0xf,%edx
  401099:       0f b6 92 b0 24 40 00    movzbl 0x4024b0(%rdx),%edx
  4010a0:       88 54 04 10             mov    %dl,0x10(%rsp,%rax,1)
  4010a4:       48 83 c0 01             add    $0x1,%rax
  4010a8:       48 83 f8 06             cmp    $0x6,%rax
  4010ac:       75 dd                   jne    40108b <phase_5+0x29>
  4010ae:       c6 44 24 16 00          movb   $0x0,0x16(%rsp)
  4010b3:       be 5e 24 40 00          mov    $0x40245e,%esi
  4010b8:       48 8d 7c 24 10          lea    0x10(%rsp),%rdi
  4010bd:       e8 76 02 00 00          call   401338 <strings_not_equal>
  4010c2:       85 c0                   test   %eax,%eax
  4010c4:       74 13                   je     4010d9 <phase_5+0x77>
  4010c6:       e8 6f 03 00 00          call   40143a <explode_bomb>
  4010cb:       0f 1f 44 00 00          nopl   0x0(%rax,%rax,1)
  4010d0:       eb 07                   jmp    4010d9 <phase_5+0x77>
  4010d2:       b8 00 00 00 00          mov    $0x0,%eax
  4010d7:       eb b2                   jmp    40108b <phase_5+0x29>
  4010d9:       48 8b 44 24 18          mov    0x18(%rsp),%rax
  4010de:       64 48 33 04 25 28 00    xor    %fs:0x28,%rax
  4010e5:       00 00
  4010e7:       74 05                   je     4010ee <phase_5+0x8c>
  4010e9:       e8 42 fa ff ff          call   400b30 <__stack_chk_fail@plt>
  4010ee:       48 83 c4 20             add    $0x20,%rsp
  4010f2:       5b                      pop    %rbx
  4010f3:       c3                      ret
```

显然中间是一个循环：

![](https://github.com/Clovers2333/picx-images-hosting/raw/master/boom_lab2.2krvlu99rv.webp)

循环结束以后，发现会和 “flyers”对比，相等则通过，所以现在只需要知道循环在干什么。

一些寄存器实在是忘了干啥的了，看看题解解释：

```assembly
0x000000000040108b <+41>:    movzbl (%rbx,%rax,1),%ecx   # %rax是地址偏移量，M[%rax + %rbx]保存到%rcx中。
0x000000000040108f <+45>:    mov    %cl,(%rsp)           # %rcx低8位保存到M[%rsp]中
0x0000000000401092 <+48>:    mov    (%rsp),%rdx          # M[%rsp]赋值到%rdx
0x0000000000401096 <+52>:    and    $0xf,%edx            # %rdx只留低4位
0x0000000000401099 <+55>:    movzbl 0x4024b0(%rdx),%edx  # M[%rdx + 0x4024b0]赋值给%edx
0x00000000004010a0 <+62>:    mov    %dl,0x10(%rsp,%rax,1) #将%rdx低4位保存到M[0x10 + %rsp + %rax]中
0x00000000004010a4 <+66>:    add    $0x1,%rax            # %rax++
```

- `x/s 0x4024b0` 后发现字符串是 `maduiersnfotvbylSo you think you can stop the bomb with ctrl-c, do you?`

相当于是输入的字符取后四位作为下标，然后在上述字符串中取出 `flyers`.

直接超了,  `ionefg`.

### Phase_6

- 留坑。
