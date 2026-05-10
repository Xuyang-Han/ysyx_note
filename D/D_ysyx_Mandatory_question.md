# D1-D3必答题

## PA2

### Q：理解YEMU如何执行程序

> [!IMPORTANT]
>
> YEMU可以看成是一个简化版的NEMU, 它们的原理是相通的, 因此你需要理解YEMU是如何执行程序的. 具体地, 你需要
>
> - 画出在YEMU上执行的加法程序的状态机
> - 通过RTFSC理解YEMU如何执行一条指令
>
> 思考一下, 以上两者有什么联系?

答：YEMU每次执行完一个操作，比如取指，状态机就会改变（PC的值），再继续译码，执行……，就会对应更改状态机相应的内容。



### Q：RTFSC理解指令执行的过程

> [!IMPORTANT]
>
> 这一小节的细节非常多, 你可能需要多次阅读讲义和代码才能理解每一处细节. 根据往届学长学姐的反馈, 一种有效的理解方法是通过做笔记的方式来整理这些细节. 事实上, 配合GDB食用效果更佳.
>
> 为了避免你长时间对代码的理解没有任何进展, 我们就增加一道必答题吧:
>
> > 请整理一条指令在NEMU中的执行过程.
>
> 除了`nemu/src/device`和`nemu/src/isa/$ISA/system`之外, NEMU的其它代码你都已经有能力理解了. 因此不要觉得讲义中没有提到的文件就不需要看, 尝试尽可能地理解每一处细节吧! 在你遇到bug的时候, 这些细节就会成为帮助你调试的线索.



### Q：运行第一个客户程序

> [!IMPORTANT]
>
> 在NEMU中实现上文提到的指令, 具体细节请务必参考手册. 实现成功后, 在NEMU中运行客户程序`dummy`, 你将会看到`HIT GOOD TRAP`的信息. 如果你没有看到这一信息, 说明你的指令实现不正确, 你可以使用PA1中实现的简易调试器帮助你调试.

#### 指令类型判断：

![img](https://cdn.jsdelivr.net/gh/Xuyang-Han/Piclist_imags@main/ysyx_imags/45a35fbdbd0b284d1eb35ea1f7b44f80.jpg)

#### 我的bug：

##### 1.得到合适的汇编程序

Li指令是伪指令，需要根据elf文件转变为rv32指令，（elf文件是介于源代码和硬件语言之间）：

```shell
riscv64-unknown-elf-objdump -d -M no-aliases string-riscv32-nemu.elf要转化的elf文件
#/home/Yang/ysyx/ysyx-workbench/am-kernels/tests/cpu-tests/build/dummy-minirv-npc.elf
```

从而得到正确的汇编程序，再根据手册来进行寻找并实现指令功能。

###### Li指令（此处是C扩展，并非dummy程序里面的Li指令）

![image-20260413153013702](https://cdn.jsdelivr.net/gh/Xuyang-Han/Piclist_imags@main/ysyx_imags/image-20260413153013702.jpg)

##### 2.jal指令的立即数imm

间接跳转指令，J型指令,处理imm时我发现自己没看懂手册，正确的解读应该是：指令IR[31]是imm[20]，指令IR[30:21] 是 imm[10:1]，注意imm[0]未在指令中定义，是硬连线为0.

![image-20260415151204699](https://cdn.jsdelivr.net/gh/Xuyang-Han/Piclist_imags@main/ysyx_imags/image-20260415151204699.jpg)

![img](https://cdn.jsdelivr.net/gh/Xuyang-Han/Piclist_imags@main/ysyx_imags/941c7e32bc83e27e987b15355797b932.jpg)

指令操作：

reg数据——pc+imm

reg地址——rd

pc_next=jal的pc+符号扩展imm



 ==jalr指令——sel=1==

目标地址由寄存器 rs1与<u>符号扩展</u>的 12 位 I 型立即数相加得到，并将计算结果的最低有效位清零。跳转指令后续指令的地址（pc+4）被写入寄存器 rd。如果不需要保存返回地址，可以使用寄存器 x0 作为目标寄存器。

```
jalr	rd,rs1,imm
jalr	rd,imm(rs1)
```

![image-20260118123333574](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260118123333574.png)

###### 译码操作：

读rs1的值+imm 并且将计算结果的最低有效位清零    作为   跳转PC，

再把jalr指令的下一条指令地址存入寄存器rd中（具体方式搜索目录“PC不匹配的延伸问题：jalr指令如何实现保存下一条PC？”）。



##### 3.auipc指令

AUIPC（Add Upper Immediate to pc）用于构造 pc 相对地址，采用 U 型格式。AUIPC 通过 U-立即数 构造一个 32 位偏移量，最低 12 位填充 0，然后将该偏移量加到 AUIPC 指令所在的地址，并将结果存入寄存器 rd.

```
auipc rd, imm  #imm是20位
```

![1](https://cdn.jsdelivr.net/gh/Xuyang-Han/Piclist_imags@main/ysyx_imags/1.jpg)

###### 指令操作

reg数据：20bit的imm + 低12bit补充零 = 扩展为32bit，

reg地址：rd



##### 4.nemu实现取imm（以下取自deepseek）

我们详细解释一下 `SEXT(x, len)` 这个宏如何利用 GCC 的位域（bit-field）特性来实现符号扩展，以及其中 `__x` 和 `.n = x` 的作用。

首先，写出这个宏的完整定义：

```C
#define SEXT(x, len) ({ struct { int64_t n : len; } __x = { .n = x }; (uint64_t)__x.n; })
```



###### 1.整体结构：GCC 语句表达式

`({ ... })` 是 GCC 的 C 语言扩展，称为**语句表达式（statement expression）**。它允许在大括号内写任意语句块，并将最后一条语句的值作为整个表达式的值返回。这类似于一个匿名函数，但没有函数调用开销。

所以，整个 `SEXT` 宏在预处理后，会被替换为一个表达式，其值就是最终符号扩展后的 `uint64_t` 结果。

###### 2. 位域结构体定义与变量 `__x`

```C
struct { int64_t n : len; } __x;
```

- 这里定义了一个**匿名结构体类型**，它只有一个成员 `n`，类型为 `int64_t`（有符号64位整数），但只占用 `len` 个比特位（位域宽度为 `len`）。
- 然后立即声明了一个该类型的变量 `__x`。变量名前的 `__` 是一种常见命名约定，表示这是宏内部使用的临时变量，避免与调用者作用域内的其他变量名冲突（类似 `__typeof__` 的用法）。

###### 3. 初始化：`.n = x`

```C
__x = { .n = x };  //因为是匿名结构体，所以是  .n
```

这是 C99 的指定初始化器（designated initializer），它显式地给结构体成员 `n` 赋初值 `x`。

此时，编译器会执行关键动作：

1. 将 `x` 的值截断到 `len` 位宽度（位域的物理限制）。
2. 因为 `n` 是**有符号**类型，截断后的 `len` 位数据的最高位被当作符号位，然后按照 **int64_t 的补码规则**扩展到 64 位，并存入变量 `__x.n` 中。

###### 4. 返回扩展后的值并转换类型

```C
(uint64_t)__x.n;
```

- 从 `__x.n` 取出已经完成符号扩展的 64 位有符号值。
- 强制转换为 `uint64_t` 无符号类型。这个转换并不改变位模式，而是将补码形式的二进制位原封不动地解释为无符号整数。这对于后续的纯位运算（如移位、按位或）非常重要，避免了有符号数右移可能带来的符号扩展干扰。

###### 5. 符号扩展的具体例子

假设 `len = 7`，`x = 0b0101010`（十进制42，符号位为0）。

- 初始化 `__x = { .n = 0b0101010 }`。
- 由于最高位（第6位）为0，编译器将 `n` 视为正数，高位全部填0。
- `__x.n` 的64位二进制为 `...0000000000101010`。
- 转换为 `uint64_t` 后，位模式不变。

假设 `len = 7`，`x = 0b1101010`（十进制106，但作为7位有符号数，符号位为1，值为 -22）。

- 初始化 `__x = { .n = 0b1101010 }`。
- 编译器识别最高位为1，将 `n` 扩展为 `int64_t` 的负数补码：高位全部填1。
- `__x.n` 的64位二进制为 `...111111111101010`（64位中的低7位是 `1101010`，高57位全是1）。
- 转换为 `uint64_t` 后，位模式不变，就是一个巨大的64位无符号数（`0xfffffffffffffff6`），但其低7位仍然是 `0x6a` 的截断。

###### 6. 为什么这种方法“零开销”？

整个操作**完全发生在编译期**。编译器知道位域的结构体布局，并在语义分析阶段直接将初始化、截断和扩展计算为常量（如果 `x` 是常量），或者生成极少的位操作指令（如带符号扩展的加载指令）。在优化级别 `-O1` 以上，变量 `__x` 本身通常会被优化掉，直接得到计算结果。

###### 总结表

| 组件                          | 作用                                               |
| :---------------------------- | :------------------------------------------------- |
| `({ ... })`                   | 将多条语句包装成一个表达式，并返回最后一条语句的值 |
| `struct { int64_t n : len; }` | 定义一个长度为 `len` 的有符号位域类型              |
| `__x`                         | 临时结构体变量，名字中的 `__` 防止宏展开后重名     |
| `.n = x`                      | 将 `x` 存入位域，触发编译器自动符号扩展            |
| `(uint64_t)__x.n`             | 取出扩展后的值并转为无符号数，保持位模式           |

这样，`SEXT` 宏就优雅地复用了 C 语言的位域符号扩展规则，无需手写位运算逻辑，且生成的代码效率极高。



运行命令

```shell
make ARCH=$ISA-nemu ALL=dummy run
make ARCH=riscv32-nemu ALL=dummy run
```



###  Q：实现更多的指令

> [!IMPORTANT]
>
> 你需要实现更多的指令, 以通过上述测试用例.
>
> 你可以自由选择按照什么顺序来实现指令. 经过PA1的训练之后, 你应该不会实现所有指令之后才进行测试了. 要养成尽早做测试的好习惯, 一般原则都是"实现尽可能少的指令来进行下一次的测试". 你不需要实现所有指令的所有形式, 只需要通过这些测试即可. 如果将来仍然遇到了未实现的指令, 就到时候再实现它们.
>
> 框架代码已经实现了部分指令, 但可能未编写相应的模式匹配规则. 此外, 部分函数的功能也并没有完全实现好(框架代码中已经插入了`TODO()`作为提示), 你还需要编写相应的功能.
>
> 由于`string`和`hello-str`还需要实现额外的内容才能运行(具体在后续小节介绍), 目前可以先使用其它测试用例进行测试.

#### mulhu指令

```C
INSTPAT("0000001 ????? ????? 011 ????? 01100 11", mulhu  , R, 
R(rd) = ((uint64_t)src1 * (uint64_t)src2) >> 32;
```

把乘积的高32bit存入寄存器rd，必须先把乘数扩展成64bit才可以再进行相乘。



####  lh指令

I型指令

```C
 INSTPAT("??????? ????? ????? 001 ????? 00000 11", lh     , I,
 R(rd) = (int32_t)(Mr(src1 + imm, 2) & 0xffff));
```

**错误**：`(int32_t)` 转换这个无符号值到有符号整数时，**并不会进行符号扩展**。例如，读取到 `0xffff` 会保持为 `65535`（`0x0000ffff`），而不是 RISC‑V 要求的 `-1`（`0xffffffff`）。

修改：需要先转为16bit的有符号数：

```C
 INSTPAT("??????? ????? ????? 101 ????? 00000 11", lh     , I,
 R(rd) = (int32_t)(int16_t)(Mr(src1 + imm, 2) & 0xffff));
```



#### lw指令是I型指令

将一个 32 位的值从内存加载到寄存器 rd 中。

![image-20260118181932966](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260118181932966.png)

#### bne指令——sel=9

对于判断指令，我选择了bne，即不相等就跳转：

![img](https://cdn.jsdelivr.net/gh/Xuyang-Han/Piclist_imags@main/ysyx_imags/155c04ed547b1ddc323bacb34c8b61fa.jpg)

![image-20260416094206398](https://cdn.jsdelivr.net/gh/Xuyang-Han/Piclist_imags@main/ysyx_imags/image-20260416094206398.jpg)

译码操作：

读rs1的值 和 读rs2的值，进行比较，

若相等，则pc = pc+“1” ;

若不相等，则pc = pc+imm . ——跳转



#### bge指令

B型指令

![image-20260428171540312](/home/Yang/.config/Typora/typora-user-images/image-20260428171540312.png)

译码操作：

读rs1的值 和 读rs2的值，当做有符号数进行比较，

若rs1 < rs2，  则pc = pc+“1” ;

若rs1 >= rs2，则pc = pc+imm . ——跳转



#### blt指令

![image-20260428174412173](https://cdn.jsdelivr.net/gh/Xuyang-Han/Piclist_imags@main/ysyx_imags/image-20260428174412173.jpg)



#### sltiu指令

I型指令

![image-20260427094137155](https://cdn.jsdelivr.net/gh/Xuyang-Han/Piclist_imags@main/ysyx_imags/image-20260427094137155.jpg)

![image-20260427095153244](https://cdn.jsdelivr.net/gh/Xuyang-Han/Piclist_imags@main/ysyx_imags/image-20260427095153244.jpg)

指令操作：

无符号扩展imm，比较imm和rs1：

- rs1 < imm , rd = 1
- rs1 > imm , rd = 0



#### srli指令

I型指令的特化模式

![image-20260427104218551](https://cdn.jsdelivr.net/gh/Xuyang-Han/Piclist_imags@main/ysyx_imags/image-20260427104218551.jpg)

![image-20260427104741940](https://cdn.jsdelivr.net/gh/Xuyang-Han/Piclist_imags@main/ysyx_imags/image-20260427104741940.jpg)

指令操作

逻辑右移，高位补0

`srli rd, rs1, imm` 汇编格式，功能：将寄存器 `rs1` 中的值逻辑右移 `imm` 位（高位补 0），结果写入寄存器 `rd`。



#### sub指令

R型指令

![image-20260427135506296](https://cdn.jsdelivr.net/gh/Xuyang-Han/Piclist_imags@main/ysyx_imags/image-20260427135506296.jpg)

![image-20260427135613776](https://cdn.jsdelivr.net/gh/Xuyang-Han/Piclist_imags@main/ysyx_imags/image-20260427135613776.jpg)

指令操作
`sub rd, rs1, rs2` 汇编格式，功能：将寄存器 `rs1`减去`rs2` 中得到的差，存入寄存器`rd`.



#### div指令

R型指令

```
div	rd,rs1,rs2
```

![image-20260428172509741](https://cdn.jsdelivr.net/gh/Xuyang-Han/Piclist_imags@main/ysyx_imags/image-20260428172509741.jpg)

![image-20260428173235268](https://cdn.jsdelivr.net/gh/Xuyang-Han/Piclist_imags@main/ysyx_imags/image-20260428173235268.jpg)

指令操作

有符号数  rs1  整除  有符号数 rs2。



#### rem指令

R型指令

```
rem	rd,rs1,rs2
```

指令操作

本质是求余数，

有符号数  rs1  整除  有符号数 rs2 ，余数的符号和rs1相同。



#### xor指令

R型

```
xor	rd,rs1,rs2
```

将rs1和rs2按位异或，再把结果存入rd中。



### Q：通过批处理模式运行NEMU

> [!IMPORTANT]
>
> 我们知道, 大部分同学很可能会这么想: 反正我不阅读Makefile, 老师助教也不知道, 总觉得不看也无所谓.
>
> 所以在这里我们加一道必做题: 我们之前启动NEMU的时候, 每次都需要手动键入`c`才能运行客户程序. 但如果不是为了使用NEMU中的sdb, 我们其实可以节省`c`的键入. NEMU中实现了一个批处理模式, 可以在启动NEMU之后直接运行客户程序. 请你阅读NEMU的代码并合适地修改Makefile, 使得通过AM的Makefile可以默认启动批处理模式的NEMU.
>
> 你现在仍然可以跳过这道必做题, 但很快你就会感到不那么方便了.

/home/Yang/ysyx/ysyx-workbench/abstract-machine/scripts/platform/nemu.mk修改run：

```makefile
run: ...
	... ARGS="-b $(NEMUFLAGS)" ...
```

`-b`意思是批处理,自动执行`c`



### Q： RISC-V指令测试（待测试）

> [!IMPORTANT]
>
> 我们把开源社区的一些RISC-V指令测试集移植到AM:
>
> - https://github.com/NJU-ProjectN/riscv-tests-am
> - https://github.com/NJU-ProjectN/riscv-arch-test-am
>
> 如果你选择了RISC-V指令集, 你可以通过这些测试集来测试你的指令实现是否正确. 克隆相应仓库后, 可通过类似运行`cpu-tests`的方式来运行这些测试集, 详细命令可参考相应的`README.md`.



### （待定）未定义行为的编译优化

 [这篇文章](https://homes.cs.washington.edu/~akcheung/papers/apsys12.pdf)列举了一些让你大开眼界的花式编译优化例子, 看完之后你就会刷新对程序行为的理解了.





###  Q：实现字符串处理函数

> [!IMPORTANT]
>
> 根据需要实现`abstract-machine/klib/src/string.c`中列出的字符串处理函数, 让`cpu-tests`中的测试用例`string`可以成功运行. 关于这些库函数的具体行为, 请务必RTFM.

#### 1.`strcmp`的实现

`strcmp`的一种情况：若 `s1` 是 `s2` 的前缀（如 `"ab"` vs `"abc"`），循环会在 `s1[i]=='\0'` 处停止。我的代码中 `if(s1[i]=='\0') return 0;` 会**错误地返回 0**，但实际上 `"ab"` 应该小于 `"abc"`。

```C
int strcmp(const char *s1, const char *s2) {
  size_t i=0;
  while(s1[i] == s2[i] && s1[i] !='\0'){i++;}
  if(s1[i] =='\0') return 0;
  return s1[i] - s2[i];
}
```

应该修改为：

```C
int strcmp(const char *s1, const char *s2) {
  size_t i=0;
  while(s1[i] == s2[i] && s1[i] !='\0'){i++;}
  return (unsigned char)s1[i] - (unsigned char)s2[i];
}
```

#### 2.`strncpy`的实现

使用2个`for`循环来实现，一定要区分`n`，`src[]`，`dst[]`，三者的长度大小关系，分别对应的2种情况。

#### 3.memcpy和memmove之间的区别？

##### memcpy：

不管源地址和目的地址的大小，均正向复制，即低地址到高地址。

##### memmove：

- 源地址 > 目的地址，正向复制；

- 源地址 < 目的地址，反向复制；

##### memcmp：

返回的是两个字符串的长度之差



###  Q：实现常用的库函数sprintf

> [!IMPORTANT]
>
> 实现`abstract-machine/klib/src/stdio.c`中的`sprintf()`, 具体行为可以参考`man 3 printf`. 目前你只需要实现`%s`和`%d`就能通过`hello-str`的测试了, 其它功能(包括位宽, 精度等)可以在将来需要的时候再自行实现.

#### 1.`man 3 printf()`的行为

利用`#include <stdarg.h>`,来实现：

```c
  va_list args;

 // 让 args 指向 fmt 后面的第一个可变参数
  va_start(args, fmt); 

  //解析 fmt，取出参数
  type type = va_arg(args, type);
	
 //释放args的地址空间
  va_end(args);
```

一个基础的指针语法错误：

```C
char *fmt=NULL;
fmt++;   //指针向后移动一个位置
*fmt++;  //指针指向的data，执行data+1
```





#### 2.库函数调用逻辑

| 自程序到硬件             | 每层功能                                       |
| ------------------------ | ---------------------------------------------- |
| 测试程序 (dummy.c)       | 调用 sprintf, strcmp 等                        |
| klib (sprintf, string.c) | 库函数，互相调用。使用 C 指针、局部变量        |
| AM 运行时环境 (TRM)      | 提供 halt(), putch() 等，使用内嵌汇编/特殊指令 |
| RISC-V CPU (NEMU 模拟)   | 解释执行机器码，内部调用 vaddr_read 等         |

##### A：在AM下，可以实现库函数调用库函数吗？

Q：可以，AM 的做法：AM 将基础字符串函数（string.c）和格式化输出（stdio.c/sprintf）分开编译，但最终都链接进同一个客户程序。因此，只要按顺序实现了 strlen、strcpy、strcat 等，sprintf 就可以放心地调用它们。

##### A：什么情况下不能调用Mr()这样的硬件实现函数？

Q：只有在NEMU实现指令的时候才能调用Mr()，因为这是硬件层面，不能跨层调用。



#### 3.`'0' + (n % 10)` 的作用

就是**把 0~9 的数字转换成对应的字符 '0'~'9'**。这是 C 语言中将一位整数快速转换为字符的标准写法。



# D4必答题

## 实现完整的minirv处理器

从指令类型来看, minirv的指令涵盖的功能包括加法, 位拼接, 访存和跳转. 我们可以根据这些功能, 结合处理器的工作流程给NPC划分模块:

- IFU(Instruction Fetch Unit): 

- IDU(Instruction Decode Unit): 负责对当前指令进行译码, 准备执行阶段需要使用的数据和控制信号

- EXU(EXecution Unit): 负责根据控制信号控制ALU, 对数据进行计算

- LSU(Load-Store Unit): 负责根据控制信号控制存储器, 从存储器中读出数据, 或将数据写入存储器




### DPI-C如何调用？

1.在.v声明，或者使用package的方式

```verilog
import "DPI-C" function void ebreak();
```

2.在.cpp中定义

```Cpp
//DPI-C实现ebreak
extern "C" void ebreak(){
  Verilated->gotFinish(true);   // 设置全局完成标志
  std::cout << "ebreak() called, simulation will stop." << std::endl;
  return;}  
```

3.可以在.v和.cpp实现调用

.cpp中当作普通函数调用即可；.v中引用包后进行调用。



Q：DPI-C机制必须在顶层top.v里import吗，可以定义在顶层，但是在下层的.v文件调用吗？package的操作，会不会想C语言那样多次引用包导致重复引用？

###### A：

- DPI-C的原理是一座桥，与C语言的.h文件不同，.sv是一个通道，原文件只有一个，但是可以有无数个桥通向它，所以不会出现重复引用；

- C语言的.h文件是文本复制，所以会出现重复加载的情况。

- package操作2步走：定义与引用

  - .sv可以使用package的方法来进行集中定义，然后在需要的.v文件中进行引用调用：

    ```verilog
    //在~/vsrc/dpi_pkg.sv中定义
    import "DPI-C" function void ebreak();
    import "DPI-C" function void pmem_write(input int waddr, input int wdata, input byte wmask);
    ```

  - 在.v文件中进行引用：

    ```verilog
    //ebreak   
    import dpi_pkg::ebreak;//精准引用，*是全部引用
    always @(posedge clk) begin
      if(sel == 4'h8)  ebreak();
    end
    ```

#### DPI-C如何寻找形式参数对应？

查阅手册《》，手册里写目前只有英文版本，先在目录里面搜索`DPI`，定位后一个一个看，随后找到`DPI C layer`；查看小标题和代码例子可以快速检索到想要的信息“DPI-C传参函数”

- 找到小标题`H.5 svdpi.h include file`——需要带头文件；
- 找到小标题`H.6.1 Types of formal arguments`——形式参数有关



### `char*` 和 `char**` 的区别

- **`char\*`**：指向字符的指针，通常用来表示一个字符串（如 `"hello"`）。例如 `char* str = "filename.bin";`。
- **`char\**`**：指向 `char*` 的指针，相当于字符串数组。在 `main(int argc, char** argv)` 中，`argv` 是一个数组，每个元素 `argv[i]` 是一个 `char*` 字符串，分别代表各个命令行参数。



### always块内reg的真实含义

可以当作是综合寄存器，我在描述F6所写的电路的时候，在实现lw和lbu指令时，无法在一个时钟周期内实现取数据和读数据

对于实现sw和sb指令：我采取DPI-C技术，在cpp内部实现，都当成sw指令，而后在cpp内部实现划分。

另一种解决办法：两个always块加同步

```verilog
// 访存阶段（第一个 always 块）
always @(posedge clk) begin
    mem_data_reg <= dmem_rdata;        // 访存数据寄存
    mem_wb_reg   <= mem_write_reg;     // 写回使能寄存
    mem_rd_reg   <= rd;                // 目标寄存器号寄存
end

// 写回阶段（第二个 always 块）
always @(posedge clk) begin
    if (mem_wb_reg)
        rf[mem_rd_reg] <= mem_data_reg;
end
```



## 搭建面向minirv的AM运行时环境

###  Q：一键编译并在NPC上运行AM程序

> [!IMPORTANT]
>
> 在AM项目中, Makefile并没有为`minirv-npc`提供`run`目标. 尝试为`minirv-npc`提供`run`目标, 使得键入`make ARCH=minirv-npc ALL=dummy run`即可把AM程序编译并在NPC上运行. 不过目前`minirv-npc`的`halt()`函数是一个死循环, 你可以通过查看波形来检查NPC是否成功进入了`halt()`函数.

#### 1.如何是实现npc跨目录自由地执行AM程序?

答：重点是如何进行将bin文件从AM的build文件内部  修改AM的Makefile，

```makefile
run: insert-arg
	$(MAKE) -C $(NPC_HOME) ISA=$(ISA) run IMG=$(IMAGE).bin
```

1. `$(MAKE)` 是 Make 的内置变量，会调用 `make` 命令。
   `-C $(NPC_HOME)` 会让 make 进入 NPC 目录去执行。
2. 在命令行中写 `IMG=$(IMAGE).bin`，是将 `IMG` 变量及其值**作为命令行参数传递给被调用的 make 进程**。
   这种方式传递的变量优先级最高，会覆盖 NPC 的 Makefile 中 `IMG ?= ...` 的默认值。
3. 在 NPC 的 Makefile 中，通过 `IMG ?=` 或直接使用 `$(IMG)` 即可接收这个变量，并传递给仿真器可执行文件。



#### 2.遇到pc地址越界报错问题

问题本质：我是利用静态数组来定义 内存数组M 的，但是数组大小是64bit，所以导致连续的内存不够用，不适合在编译时静态分配，从而产生超大的`.bss`段。

修改方案：

```cpp
// 原来：unsigned char M[SIZE];
std::vector<unsigned char> M(SIZE);
```

就是把静态分配，改为动态分配数组。



### Q：实现minirv-npc中的halt()函数

> [!IMPORTANT]
>
> 为了可以自动地结束程序, 你需要在`minirv-npc`中实现TRM的halt()函数, 在其中添加一条`ebreak`指令. 这样以后, 在NPC上运行的AM程序在结束的时候就会执行`ebreak`指令, 从而通知NPC的仿真环境结束仿真.
>
> 实现之后, 你就可以通过一条命令自动在NPC上运行AM程序并自动结束仿真了.

#### 1.修改在trm.c中`halt(code)`函数

AM里面`~/ysyx-workbench/abstract-machine/am/src/riscv/npc`的`trm.c`里面AM运行时的启动序列：

```
_start (启动代码，设置栈等)
  └── _trm_init (初始化堆、命令行参数，并调用 main)
        └── main(mainargs)  // 你的测试程序
              └── return ret;
        └── halt(ret);      // 退出
```

对应源码：

```C
void _trm_init() {
  int ret = main(mainargs);
  halt(ret);
}
```

##### halt函数的实现原理：

 `_trm_init()`  —>  `main(mainargs)`  —>` halt(ret) `  —>  `halt把ret的值写入a0，再调用ebreak` —>  `ebreak判断a0的值`

若`a0 == 0`，则 `HIT GOOD`；

若`a0 != 0`， 则 `HIT BAD`.

一般上述流程，只要可以正确调用`main`，就会执行`halt(0)`，即 `HIT GOOD` ；

如果中途调用了`halt(code)`，就说明触发了AM的check函数，会调用`halt(1)`,最后会显示 `HIT BAD`.

check的宏定义在`/home/Yang/ysyx/ysyx-workbench/abstract-machine/klib/include/klib-macros.h`

```c
//check：用来检查错误，有错则返回halt(1)
#define panic_on(cond, s) \
  ({ if (cond) { \
      putstr("AM Panic: "); putstr(s); \
      putstr(" @ " __FILE__ ":" TOSTRING(__LINE__) "  \n"); \
      halt(1); \
 }})
```



#### 我的疑问：

###### 1.报错“多次定义`main`函数”

我在`trm.c`里面更改了`main`函数，加了一句`return 0`，最后会报错说我多次定义了`main`函数，但是我找不到第一次定义的`main`函数在哪里，报错如下：

```
Yang@yyl-Ubuntu:~/ysyx/ysyx-workbench/am-kernels/tests/cpu-tests$ make ARCH=minirv-npc ALL=dummy run V=1
# Building dummy-run [minirv-npc]
# Building am-archive [minirv-npc]
# Building klib-archive [minirv-npc]
# Creating image [minirv-npc]
+ LD -> build/dummy-minirv-npc.elf
riscv64-linux-gnu-ld: /home/Yang/ysyx/ysyx-workbench/abstract-machine/am/build/am-minirv-npc.a(trm.o): in function `main':
trm.c:(.text.startup.main+0x0): multiple definition of `main'; /home/Yang/ysyx/ysyx-workbench/am-kernels/tests/cpu-tests/build/minirv-npc/tests/dummy.o:dummy.c:(.text.startup.main+0x0): first defined here
make[1]: *** [/home/Yang/ysyx/ysyx-workbench/abstract-machine/Makefile:142: /home/Yang/ysyx/ysyx-workbench/am-kernels/tests/cpu-tests/build/dummy-minirv-npc.elf] Error 1
test list [1 item(s)]: dummy
[         dummy] ***FAIL***
Yang@yyl-Ubuntu:~/ysyx/ysyx-workbench/am-kernels/tests/cpu-tests$ make clean
rm -rf Makefile.* build/
Yang@yyl-Ubuntu:~/ysyx/ysyx-workbench/am-kernels/tests/cpu-tests$ make ARCH=minirv-npc ALL=dummy run V=1
# Building dummy-run [minirv-npc]
+ CC tests/dummy.c
# Building am-archive [minirv-npc]
# Building klib-archive [minirv-npc]
# Creating image [minirv-npc]
+ LD -> build/dummy-minirv-npc.elf
riscv64-linux-gnu-ld: /home/Yang/ysyx/ysyx-workbench/abstract-machine/am/build/am-minirv-npc.a(trm.o): in function `main':
trm.c:(.text.startup.main+0x0): multiple definition of `main'; /home/Yang/ysyx/ysyx-workbench/am-kernels/tests/cpu-tests/build/minirv-npc/tests/dummy.o:dummy.c:(.text.startup.main+0x0): first defined here
make[1]: *** [/home/Yang/ysyx/ysyx-workbench/abstract-machine/Makefile:142: /home/Yang/ysyx/ysyx-workbench/am-kernels/tests/cpu-tests/build/dummy-minirv-npc.elf] Error 1
test list [1 item(s)]: dummy
[         dummy] ***FAIL***
```



###### 2.无法正确编译`putch`函数

同样是在`trm.c`里面的`putch`函数如下：

```C
void putch(char ch) { //AM 中输出单个字符的底层函数
  asm volatile(
    "mv a0, %0\n"
    "mv a1, 1\n"
    "ebreak\n"
    :
    : "r"(ch)
    : "a0", "a7"
  );
  while(1);
}
```

报错无法编译：

```C
# Creating image [minirv-npc]
+ LD -> build/string-minirv-npc.elf
riscv64-linux-gnu-ld: /home/Yang/ysyx/ysyx-workbench/abstract-machine/am/build/am-minirv-npc.a(start.o): in function `_start':
(entry+0xc): undefined reference to `_trm_init'
riscv64-linux-gnu-ld: (entry+0x10): undefined reference to `_trm_init'
make[1]: *** [/home/Yang/ysyx/ysyx-workbench/abstract-machine/Makefile:140: /home/Yang/ysyx/ysyx-workbench/am-kernels/tests/cpu-tests/build/string-minirv-npc.elf] Error 1
test list [1 item(s)]: string
[        string] ***FAIL***
```



### Q：为NPC实现HIT GOOD/BAD TRAP

> [!IMPORTANT]
>
> NEMU可以输出"程序是否成功结束执行"的信息, 尝试在NPC中实现相似的功能, 这样以后, 你就可以快速了解程序在NPC上是否成功结束了.
>
> 得益于minirv指令集的完备性, 之前你在`riscv32-nemu`上能运行的程序, 都能通过重新编译到`minirv-npc`, 从而运行在NPC上. 你无需为了运行它们而在NPC上实现更多的指令.

##### 我的bug：`ebreak`指令无法正确译码

我的``ebreak``指令设置错了，手册没找对指令，op应该是system，导致我在trm.c里面的ebreak无法被npc识别，一直陷入死循环。

![image-20260503132003406](https://cdn.jsdelivr.net/gh/Xuyang-Han/Piclist_imags@main/ysyx_imags/image-20260503132003406.jpg)

system应该是固定的`1110011`:

![image-20260503132034814](https://cdn.jsdelivr.net/gh/Xuyang-Han/Piclist_imags@main/ysyx_imags/image-20260503132034814.jpg)

###  Q：在NPC上运行更多程序

> [!IMPORTANT]
>
> 在`minirv-npc`上运行`cpu-tests`和`riscv-tests`, 来测试NPC的实现是否正确.
>
> 由于minirv将其他指令转换成上述8条指令时, 需要满足程序不会使用`sp`, `gp`和`tp`寄存器的假设. 但`riscv-arch-test`中的测试过程确实使用了这些寄存器, 不符合这个假设, 因此`riscv-arch-test`无法在`minirv-npc`中正确运行.

答：没毛病，测试都`HIT GOOD`.



# D5 设备和输入输出



## PA2-输入输出

所以, 在程序看来, 访问设备 = 读出数据 + 写入数据 + 控制状态.



### `volatile` 关键字的作用

`volatile` 告诉编译器：

每一次对变量的读写都必须实实在在地从内存（或外设地址）中执行，不能进行任何缓存、合并或消除等优化。



### `asm`的作用

可以把 `asm` 理解为 **C 语言和机器指令之间的桥梁**。它是写操作系统、底层驱动或像你正在做的 CPU 启动代码时必不可少的工具。在 `am/src/riscv/npc/start.S` 这类纯汇编文件里直接用汇编，而在 C 文件里需要插几句特殊指令时，就用 `asm`。



### Q：运行Hello World

> [!IMPORTANT]
>
> 如果你选择了x86, 你需要实现`in`, `out`指令. 具体地, 你需要RTFSC, 然后在`in`指令和`out`指令的实现中正确调用`pio_read()`和`pio_write()`. 如果你选择的是mips32和riscv32, 你不需要实现额外的代码, 因为NEMU的框架代码已经支持MMIO了.
>
> 实现后, 在`am-kernels/kernels/hello/`目录下键入
>
> ```bash
> make ARCH=$ISA-nemu run
> ```
>
> 如果你的实现正确, 你将会看到程序往终端输出一些信息(请注意不要让输出淹没在调试信息中).
>
> 需要注意的是, 这个hello程序和我们在程序设计课上写的第一个hello程序所处的抽象层次是不一样的: 这个hello程序可以说是直接运行在裸机上, 可以在AM的抽象之上直接输出到设备(串口); 而我们在程序设计课上写的hello程序位于操作系统之上, 不能直接操作设备, 只能通过操作系统提供的服务进行输出, 输出的数据要经过很多层抽象才能到达设备层. 我们会在PA3中进一步体会操作系统的作用.

我的bug：

一开始在前面实现的sb指令是这样的：

```C
INSTPAT("??????? ????? ????? 000 ????? 01000 11", sb     , S,{
     int sel = (src1+imm) & 0x3 ;
     int32_t mem_data_fu = -(BITS(Mr(src1 + imm,4), sel*8+8, sel*8) << sel*8);
     word_t data = mem_data_fu + Mr(src1 + imm,4); //得到挖空版mem_data
     word_t w_data = data + (src2 << sel*8) ;
     Mw(src1 + imm, 1, w_data);
   });
```

其实sb指令nemu默认是有的，但是我给改了，导致后面的串口读取时，读取字节超过一个字节了，直接报错中断了。



### Q：实现printf

> [!IMPORTANT]
>
> 有了`putch()`, 我们就可以在klib中实现`printf()`了.
>
> 你之前已经实现了`sprintf()`了, 它和`printf()`的功能非常相似, 这意味着它们之间会有不少重复的代码. 你已经见识到Copy-Paste编程习惯的坏处了, 思考一下, 如何简洁地实现它们呢?
>
> 实现了`printf()`之后, 你就可以在AM程序中使用输出调试法了.

#### 1.`itoa10`自动带'\0'

```C
int num_len = itoa10(va_arg(args,int),d);
// d[num_len]='\0'; itoa10自动带'\0'
```



#### 2.字符串赋值自动有‘\0’

```C
 char *s = va_arg(args,char*);
 //也是自动补零，不用格外加
```



#### 3.调用函数va_arg(args,～)

完成两件事情：1）返回需要变成的类型；<u>2）指针指向下一个变量</u>

```C
va_arg(args,int)
va_arg(args,char*)
```



### Q：运行alu-tests

> [!IMPORTANT]
>
> 我们在`am-kernels/tests/alu-tests/`目录下移植了一个专门测试各种C语言运算的程序, 实现`printf()`后你就可以运行它了. 编译过程可能需要花费1分钟.

#### 我的bug

一是 `printf` 输出格式错误，导致打印信息失真；

二是有符号分支比较指令存在实现缺陷，这才是测试失败的根源。

---

##### 🔴 问题一：`printf` 输出了多余的格式字符

你的 `printf` 在处理完 `%d` 或 `%s` 后，会执行一条多余的 `putch(*fmt)`，将格式符（如 `'d'`、`'s'`）直接输出，导致乱码。

**修正方法**：在处理完格式说明符后，立即将 `fmt` 移动到下一个字符，并 `continue`，跳过末尾的普通输出。

---

##### 🔴 问题二：有符号分支指令 `bge` / `blt` 使用了错误的比较方式

原先的实现：

```c
src1 = (int32_t)src1;
src2 = (int32_t)src2;
```

由于 `src1` 的类型是 `word_t`（即 `uint32_t`），将 `int32_t` 赋值给 `uint32_t` 时，会发生**隐式类型转换**，数值会被重新解释为无符号数。例如 `0x80000000`（-2147483648）赋给 `uint32_t` 后，仍然是 `0x80000000`（2147483648），导致比较又变成无符号比较，有符号分支完全失效。

**修正方法**：不要修改 `src1` 和 `src2` 本身，直接在条件表达式中进行强制转换。

```c
// bge : rs1 >= rs2 时跳转
INSTPAT("??????? ????? ????? 101 ????? 11000 11", bge, B, {
    if ((int32_t)src1 >= (int32_t)src2) { s->dnpc = s->pc + imm; }
});

// blt : rs1 < rs2 时跳转
INSTPAT("??????? ????? ????? 100 ????? 11000 11", blt, B, {
    if ((int32_t)src1 < (int32_t)src2) { s->dnpc = s->pc + imm; }
});
```

`bgeu` 和 `bltu` 无需修改，它们原本就是无符号比较，已经正确。



### Q：实现IOE

> [!IMPORTANT]
>
> 在`abstract-machine/am/src/platform/nemu/ioe/timer.c`中实现`AM_TIMER_UPTIME`的功能. 在`abstract-machine/am/src/platform/nemu/include/nemu.h`和 `abstract-machine/am/src/$ISA/$ISA.h`中有一些输入输出相关的代码供你使用.
>
> 实现后, 在`$ISA-nemu`中运行`am-kernel/tests/am-tests`中的`real-time clock test`测试. 如果你的实现正确, 你将会看到程序每隔1秒往终端输出一行信息. 由于我们没有实现`AM_TIMER_RTC`, 测试总是输出1900年0月0日0时0分0秒, 这属于正常行为, 可以忽略.

#### 1.NEMU当中的RTC的逻辑是什么？

A：高位32bit的RTC寄存器，和低位32bitRTC寄存器，拼接起来就是时间。额外需要判断是否低位有进位，就会导致高位32bit变化，需要重新获取RTC的值。



#### 2.如何运行测试

```shell
make ARCH=riscv32-nemu run mainargs=t
```

命令会把`mainargs`传入`main`函数中，作为`args[0]`.

运行结果部分：

```
[src/monitor/monitor.c:34 welcome] Build time: 16:34:54, May  6 2026
Welcome to riscv32-NEMU!
For help, type "help"
1900-0-0 %02d:%02d:%02d GMT (1 second).
1900-0-0 %02d:%02d:%02d GMT (2 seconds).
1900-0-0 %02d:%02d:%02d GMT (3 seconds).
1900-0-0 %02d:%02d:%02d GMT (4 seconds).
1900-0-0 %02d:%02d:%02d GMT (5 seconds).
1900-0-0 %02d:%02d:%02d GMT (6 seconds).
1900-0-0 %02d:%02d:%02d GMT (7 seconds).
1900-0-0 %02d:%02d:%02d GMT (8 seconds).
1900-0-0 %02d:%02d:%02d GMT (9 seconds).
1900-0-0 %02d:%02d:%02d GMT (10 seconds).
1900-0-0 %02d:%02d:%02d GMT (11 seconds).
1900-0-0 %02d:%02d:%02d GMT (12 seconds).
1900-0-0 %02d:%02d:%02d GMT (13 seconds).
1900-0-0 %02d:%02d:%02d GMT (14 seconds).
1900-0-0 %02d:%02d:%02d GMT (15 seconds).
1900-0-0 %02d:%02d:%02d GMT (16 seconds).
1900-0-0 %02d:%02d:%02d GMT (17 seconds).
```



### Q：看看NEMU跑多快

> [!IMPORTANT]
>
> 有了时钟之后, 我们就可以测试一个程序跑多快, 从而测试计算机的性能. 尝试在NEMU中依次运行以下benchmark(已经按照程序的复杂度排序, 均在`am-kernel/benchmarks/`目录下; 另外跑分时请关闭NEMU的监视点, trace以及DiffTest, 同时取消menuconfig中的 `Enable debug information`并重新编译NEMU, 以获得较为真实的跑分):
>
> - dhrystone
> - coremark
> - microbench
>
> 成功运行后会输出跑分. 其中microbench跑分以`i9-9900K @ 3.60GHz`的处理器为参照, `100000`分表示与参照机器性能相当, `100`分表示性能为参照机器的千分之一. 除了和参照机器比较之外, 也可以和小伙伴进行比较. 如果把上述benchmark编译到`native`, 还可以比较`native`的性能.
>
> 另外, microbench提供了四个不同规模的测试集, 包括`test`, `train`, `ref`和`huge`. 你可以先运行`test`规模, 它可以较快地运行结束, 来检查NEMU实现的正确性, 然后再运行`ref`规模来测量性能. 具体的运行方法请阅读README.
>
> 此外, `huge`规模一般用于真机的测试, 在NEMU中需要运行很长时间, 我们不要求你运行它.

1.运行dhrystone的`dry.c`，结果显示 `100000`分表示与参照机器性能相当。

![image-20260510154903324](https://cdn.jsdelivr.net/gh/Xuyang-Han/Piclist_imags@main/ysyx_imags/image-20260510154903324.jpg)



2.运行测试集coremark，结果显示 `100000`分表示与参照机器性能相当。

![image-20260507142419365](https://cdn.jsdelivr.net/gh/Xuyang-Han/Piclist_imags@main/ysyx_imags/image-20260507142419365.jpg)



3.运行测试集microbench

运行命令

```shell
 make ARCH=riscv32-nemu run  mainargs=test
```

test

![image-20260507150358444](https://cdn.jsdelivr.net/gh/Xuyang-Han/Piclist_imags@main/ysyx_imags/image-20260507150358444.jpg)



train

![image-20260507150612713](https://cdn.jsdelivr.net/gh/Xuyang-Han/Piclist_imags@main/ysyx_imags/image-20260507150612713.jpg)



ref

![image-20260507151152483](https://cdn.jsdelivr.net/gh/Xuyang-Han/Piclist_imags@main/ysyx_imags/image-20260507151152483.jpg)



### 设备访问的踪迹 - dtrace

#### Q：实现dtrace

> [!IMPORTANT]
>
> 这个功能非常简单, 你可以自行定义dtrace输出的格式. 注意你可以通过`map->name`来获取一段设备地址空间的名字, 这样可以帮助你输出可读性较好的信息. 同样地, 你也可以为dtrace实现条件控制功能, 提升dtrace使用的灵活性.

答：在`map_read`和`map_write`里面加入printf字段，同时在sdb中添加一条命令来控制是否输出，由布尔变量`dtrace_enable`来控制是否输出设备信息。



#### Q：实现IOE(2)

> [!IMPORTANT]
>
> 在`abstract-machine/am/src/platform/nemu/ioe/input.c`中实现`AM_INPUT_KEYBRD`的功能. 实现后, 在`$ISA-nemu`中运行`am-tests`中的`readkey test`测试. 如果你的实现正确, 在程序运行时弹出的新窗口中按下按键, 你将会看到程序输出相应的按键信息, 包括按键名, 键盘码, 以及按键状态.

##### 1.NEMU键盘输入原理

`__am_input_keybrd` 实现键盘输入的原理和之前的 RTC 时钟如出一辙：NEMU 会将键盘状态映射到一个特定的 MMIO 地址，AM 只需要去读取这个地址，然后解析数据并填充到 `AM_INPUT_KEYBRD_T` 结构体中。

键盘码和按键状态这两个信息被打包进了同一个32位整数里。根据AM的硬件约定：

- **低15位** (bit 14:0) 存放的是**按键码** (`keycode`)。
- **最高位** (bit 15) 存放的是**按键状态** (`keydown`)：`1` 代表按下，`0` 代表释放。

它们俩可以利用所给的掩码 `KEYDOWN_MASK=0x8000`来进行取数。

**NEMU 无按键时**

NEMU 的实现中，无按键时直接读到 `0`，且 `keydown` 为 `0`，`keycode` 为 `0` 就等价于 `AM_KEY_NONE`，所以不需要额外处理，但需要明白 `0` 代表无按键。



##### 2.在黑框里输入！

不然没办法输出相应的按键信息, 包括按键名, 键盘码, 以及按键状态。我一直在终端输入，找了一个多小时，真是笑死自己了！



### VGA

#### Q：实现IOE(3)

> [!IMPORTANT]
>
> 事实上, VGA设备还有两个寄存器: 屏幕大小寄存器和同步寄存器. 我们在讲义中并未介绍它们, 我们把它们作为相应的练习留给大家. 具体地, 屏幕大小寄存器的硬件(NEMU)功能已经实现, 但软件(AM)还没有去使用它; 而对于同步寄存器则相反, 软件(AM)已经实现了同步屏幕的功能, 但硬件(NEMU)尚未添加相应的支持.
>
> 好了, 提示已经足够啦, 至于要在什么地方添加什么样的代码, 就由你来RTFSC吧. 这也是明白软硬件如何协同工作的很好的练习. 实现后, 向`__am_gpu_init()`中添加如下测试代码:
>
> ```diff
> --- abstract-machine/am/src/platform/nemu/ioe/gpu.c
> +++ abstract-machine/am/src/platform/nemu/ioe/gpu.c
> @@ -6,2 +6,8 @@
> void __am_gpu_init() {
> +  int i;
> +  int w = 0;  // TODO: get the correct width
> +  int h = 0;  // TODO: get the correct height
> +  uint32_t *fb = (uint32_t *)(uintptr_t)FB_ADDR;
> +  for (i = 0; i < w * h; i ++) fb[i] = i;
> +  outl(SYNC_ADDR, 1);
>  }
> ```
>
> 其中上述代码中的`w`和`h`并未设置正确的值, 你需要阅读`am-tests`中的`display test`测试, 理解它如何获取正确的屏幕大小, 然后修改上述代码的`w`和`h`. 你可能还需要对`gpu.c`中的代码进行一些修改. 修改后, 在`$ISA-nemu`中运行`am-tests`中的`display test`测试, 如果你的实现正确, 你会看到新窗口中输出了全屏的颜色信息.

 



#### Q：实现IOE(4)

> [!IMPORTANT]
>
> 事实上, 刚才输出的颜色信息并不是`display test`期望输出的画面, 这是因为`AM_GPU_FBDRAW`的功能并未正确实现. 你需要正确地实现`AM_GPU_FBDRAW`的功能. 实现后, 重新运行`display test`. 如果你的实现正确, 你将会看到新窗口中输出了相应的动画效果.
>
> 实现正确后, 你就可以去掉上文添加的测试代码了.





## 在NPC中运行超级玛丽

### Q：为NPC添加串口和时钟

> [!IMPORTANT]
>
> 在NPC仿真环境中实现串口的输出功能, 并运行hello程序. 为了和后面的SoC串口地址保持一致, 此处可将NPC的串口地址设置为`0x10000000`.
