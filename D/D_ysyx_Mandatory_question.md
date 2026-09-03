# D阶段必答题

## D1-D3必答题-PA2

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

Q：/home/Yang/ysyx/ysyx-workbench/abstract-machine/scripts/platform/nemu.mk修改run：

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

A：

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

查阅手册，手册里写目前只有英文版本，先在目录里面搜索`DPI`，定位后一个一个看，随后找到`DPI C layer`；查看小标题和代码例子可以快速检索到想要的信息“DPI-C传参函数”

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

```
00100073  是ebreak的机器码，不是90000002
```

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

### 将输入输出抽象成IOE

#### 相关语法

##### 1.`volatile` 关键字的作用

`volatile` 告诉编译器：

每一次对变量的读写都必须实实在在地从内存（或外设地址）中执行，不能进行任何缓存、合并或消除等优化。

##### 2.`asm`的作用

可以把 `asm` 理解为 **C 语言和机器指令之间的桥梁**。它是写操作系统、底层驱动或像你正在做的 CPU 启动代码时必不可少的工具。在 `am/src/riscv/npc/start.S` 这类纯汇编文件里直接用汇编，而在 C 文件里需要插几句特殊指令时，就用 `asm`。



#### Q：运行Hello World

> [!IMPORTANT]
>
> ~~如果你选择了x86, 你需要实现`in`, `out`指令. 具体地, 你需要RTFSC, 然后在`in`指令和`out`指令的实现中正确调用`pio_read()`和`pio_write()`.~~ 
>
> 如果你选择的是mips32和riscv32, 你不需要实现额外的代码, 因为NEMU的框架代码已经支持MMIO了.
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

其实sb指令nemu默认是有的，但是我给改了，导致后面的串口读取时，读取字节超过一个字节了，直接报错中断了，恢复后：

```C
INSTPAT("??????? ????? ????? 000 ????? 01000 11", sb     , S, Mw(src1 + imm, 1, src2));
```



#### Q：实现printf

> [!IMPORTANT]
>
> 有了`putch()`, 我们就可以在klib中实现`printf()`了.
>
> 你之前已经实现了`sprintf()`了, 它和`printf()`的功能非常相似, 这意味着它们之间会有不少重复的代码. 你已经见识到Copy-Paste编程习惯的坏处了, 思考一下, 如何简洁地实现它们呢?
>
> 实现了`printf()`之后, 你就可以在AM程序中使用输出调试法了.

##### 1.`itoa10`自动带'\0'

```C
int num_len = itoa10(va_arg(args,int),d);
// d[num_len]='\0'; itoa10自动带'\0'
```



##### 2.字符串赋值自动有‘\0’

```C
 char *s = va_arg(args,char*);
 //也是自动补零，不用格外加
```



##### 3.调用函数va_arg(args,～)

完成两件事情：1）返回需要变成的类型；<u>2）指针指向下一个变量</u>

```C
va_arg(args,int)
va_arg(args,char*)
```



#### Q：运行alu-tests

> [!IMPORTANT]
>
> 我们在`am-kernels/tests/alu-tests/`目录下移植了一个专门测试各种C语言运算的程序, 实现`printf()`后你就可以运行它了. 编译过程可能需要花费1分钟.

##### 我的bug

一是 `printf` 输出格式错误，导致打印信息失真；

二是有符号分支比较指令存在实现缺陷，这才是测试失败的根源。

---

###### 🔴 问题一：`printf` 输出了多余的格式字符

你的 `printf` 在处理完 `%d` 或 `%s` 后，会执行一条多余的 `putch(*fmt)`，将格式符（如 `'d'`、`'s'`）直接输出，导致乱码。

**修正方法**：在处理完格式说明符后，立即将 `fmt` 移动到下一个字符，并 `continue`，跳过末尾的普通输出。

---

###### 🔴 问题二：有符号分支指令 `bge` / `blt` 使用了错误的比较方式

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



#### Q：实现IOE

> [!IMPORTANT]
>
> 在`abstract-machine/am/src/platform/nemu/ioe/timer.c`中实现`AM_TIMER_UPTIME`的功能. 在`abstract-machine/am/src/platform/nemu/include/nemu.h`和 `abstract-machine/am/src/$ISA/$ISA.h`中有一些输入输出相关的代码供你使用.
>
> 实现后, 在`$ISA-nemu`中运行`am-kernel/tests/am-tests`中的`real-time clock test`测试. 如果你的实现正确, 你将会看到程序每隔1秒往终端输出一行信息. 由于我们没有实现`AM_TIMER_RTC`, 测试总是输出1900年0月0日0时0分0秒, 这属于正常行为, 可以忽略.

##### 1.NEMU当中的RTC的逻辑是什么？

A：高位32bit的RTC寄存器，和低位32bitRTC寄存器，拼接起来就是时间。额外需要判断是否低位有进位，就会导致高位32bit变化，需要重新获取RTC的值。



##### 2.如何运行测试

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



#### Q：看看NEMU跑多快

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

答：在`map_read`和`map_write`里面加入printf字段，同时在sdb中添加一条命令来显示输出，由布尔变量`dtrace_enable`来控制是否输出设备信息。

```bash
（nemu）dtrace on   # 打开
（nemu）dtrace off  # 关闭
```



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

#### Q：实现IOE(3)&(4)

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
>
> 
>
> #### Q：实现IOE(4)
>
> 事实上, 刚才输出的颜色信息并不是`display test`期望输出的画面, 这是因为`AM_GPU_FBDRAW`的功能并未正确实现. 你需要正确地实现`AM_GPU_FBDRAW`的功能. 实现后, 重新运行`display test`. 如果你的实现正确, 你将会看到新窗口中输出了相应的动画效果.
>
> 实现正确后, 你就可以去掉上文添加的测试代码了.
>
> 

A：在/home/Yang/ysyx/ysyx-workbench/am-kernels/tests/am-tests执行：

```bash
make ARCH=riscv32-nemu run mainargs=v
```

在/home/Yang/ysyx/ysyx-workbench/nemu/src/device/vga.c

```

```

在/home/Yang/ysyx/ysyx-workbench/abstract-machine/am/src/platform/nemu/ioe/gpu.c

```

```

在/home/Yang/ysyx/ysyx-workbench/abstract-machine/klib/include/klib-macros.h定义宏`io_read`和`io_write`

```c
#define io_read(reg) \
  ({ reg##_T __io_param; \
    ioe_read(reg, &__io_param); \
    __io_param; })

#define io_write(reg, ...) \
  ({ reg##_T __io_param = (reg##_T) { __VA_ARGS__ }; \
    ioe_write(reg, &__io_param); })
```



#####  `outl` 的功能

`outl` 是封装 I/O 端口写入的函数，通常用内联汇编实现，定义在 AM 的 I/O 支持代码中。

```C
if (ctl->sync) { 
    outl(VGACTL_ADDR + 4 , 1);  //am/src/riscv/riscv.h}
//当ctl->sync为1时，就在VGACTL_ADDR + 4 的地址处写1
```

`outl(port, value)` 的作用是向指定的 I/O 端口写入一个 32 位数据，即`outl(I/O 端口, 32 位数据)`.

在这行代码中：

- `VGACTL_ADDR` 是显卡控制端口的基地址（也是宏，可能定义在同一个头文件中）。
- `VGACTL_ADDR + 4` 通常是显卡的“同步/刷新”寄存器端口。
- 向该端口写入 `1` 会通知显卡/模拟器把当前帧缓冲区的内容更新到屏幕上。

因此 `outl` 的功能就是：**触发一次屏幕刷新同步**，让之前写入显存的像素真正显示出来。

------

**总结**：

- `FB_ADDR` 和 `SCREEN_W`、`VGACTL_ADDR` 一样，都是平台相关的硬件抽象常量，定义在 AM 的平台头文件中（如 `nemu.h`）。
- `fb` 是指向显存的指针，通过直接内存写入来绘制像素。
- `outl` 是封装 I/O 端口写入的函数，通常用内联汇编实现，定义在 AM 的 I/O 支持代码中。







#####  1.协同工作的流程？

- AM（软件）调用 `outl(VGACTL_ADDR + 4, 1)`，向同步寄存器写 `1`。
- NEMU（硬件）在每次主循环中调用 `vga_update_screen()`，它会检查 `vgactl_port_base[1]` 是否为非零。如果是，就刷新屏幕并清零。
- 这样软硬件通过同一个偏移 `+4` 完成了“请求同步 → 检测同步 → 刷新 → 清除标志”的握手。

所以，**“+4”这个偏移并不是随意猜测的**，它是由 NEMU 的 VGA 设备模型（8字节空间，两32位字）决定的，并与 AM 侧 `SYNC_ADDR` 宏完全一致。

是不是可以这样理解：测试的C程序，生成颜色信息，然后AM解析了C程序的行为，AM利用自己的寄存器拷贝了影像，同时利用nemu的显示图形功能，nemu检测是否有颜色信息和同步请求，然后符合的话就输出。————我的疑问：AM有自己的gpu是干吗的？AM的gpu寄存器是用来干吗的，控制硬件层nemu的吗？



嗯，用户这次的问题很核心，是在梳理整个图形显示流程，并且对AM中GPU模块的角色产生了疑问。从对话历史看，用户一直在实现AM的IOE设备驱动，特别是VGA/GPU部分，已经完成了键盘、GPU配置、帧缓冲绘制等代码，并且遇到了屏幕不显示的问题，我们刚刚一起解决了NEMU端同步寄存器未处理的问题。

用户现在想确认自己的理解是否正确，并且深层疑问是：“AM的GPU到底是干什么的？它的寄存器是控制NEMU的吗？” 这反映出用户在试图理解软硬件分层和抽象层（AM）存在的意义。

我需要帮用户把这条通路彻底理清。用户的理解——“C程序生成颜色 -> AM解析并拷贝影像 -> NEMU检测同步信号并输出”——核心是对的，但需要更精准地界定各层的职责。

AM的GPU模块，本质上不是硬件，而是一套“驱动接口”。它的寄存器，比如AM_GPU_CONFIG、AM_GPU_FBDRAW，是抽象的“请求单”，而不是物理触发器。用户程序（测试C程序）通过io_read/io_write填写这些“请求单”，AM的驱动函数（我们写的__am_gpu_xxx）负责解析，并执行对NEMU模拟硬件的具体操作（读写MMIO地址）。所以，AM是中间层，向上提供统一接口，向下屏蔽NEMU（或其他平台）的硬件细节。

用户的疑问“AM的gpu是干嘛的，寄存器是控制硬件层nemu的吗？”——是的，可以这么理解。AM的GPU寄存器（结构体）定义了一种“语言”，AM的驱动实现负责把这种语言“翻译”成对NEMU特定地址（如FB_ADDR, VGACTL_ADDR+4）的操作。这样，同一个测试C程序不用改，只要换一套AM实现，就能跑在NEMU或真正的硬件上。

最后要给出一个清晰的总结，把“写请求 -> 驱动执行硬件操作 -> 硬件响应”的流程点明，强调抽象设计的好处。这样既能肯定用户的思考，也能解答他最终的疑惑。

你提的这个问题很关键，它触及了 AM（Abstract Machine）设计的核心思想。你的理解框架是对的，但有一些细节需要澄清：

并不是 AM 通过自己的 GPU 去控制硬件，而是 **AM 提供了一套统一的软件接口，把底层硬件（NEMU）的细节封装起来了**。

我们一层层来看：

##### 🖥️ 1）测试的 C 程序生成颜色信息

这是对的。`display test` 这样的应用程序只知道 AM 的 API，它计算好每一帧的像素数据，然后调用 `io_write(AM_GPU_FBDRAW, ...)`，告诉 AM：“我要画一个矩形，颜色数据在这里”。它根本不知道，也不关心底层到底是 NEMU 还是真实的 FPGA。

##### 🧠 2）AM 的“GPU 模块”是干什么的？

这其实是你问题的核心。**AM 的 GPU 不是一个物理设备，而是一套软件“驱动程序”**。

你可以把它看作一个“翻译官”，它负责把应用程序抽象的画图请求，翻译成特定硬件能听懂的操作。

AM 中所谓的“GPU 寄存器”（比如你代码里的 `AM_GPU_CONFIG_T`, `AM_GPU_FBDRAW_T`），并不是物理硬件上的寄存器，而是 **AM 定义的、用来在应用和驱动之间传递数据的 C 语言结构体**。它就像一张“信息登记表”。

拿 `<amdev.h>` 里的这个定义举例：

```C
AM_DEVREG(11, GPU_FBDRAW, WR, int x, y; void *pixels; int w, h; bool sync);
```

它告诉 AM 和应用程序：“喂，要想画图，你得填好这张表，告诉我画在哪(`x,y`)，画多大(`w,h`)，用什么颜色(`pixels`)，以及画完要不要刷新(`sync`)。”

所以，你的理解：

- **“AM 解析了 C 程序的行为”**：是的，驱动函数 `__am_gpu_fbdraw` 就是在解析这张“信息登记表”。
- **“AM 利用自己的寄存器拷贝了影像”**：更准确地说，是 AM 的驱动程序根据登记表的信息，手动把像素数据拷贝到了帧缓冲（`FB_ADDR`），然后触发同步。

##### 🕹️ 3）AM 的驱动到底怎么“控制” NEMU？

你的最后一个问题很准：**AM 的驱动就是通过读写 NEMU 提供的特定 MMIO 地址来控制它的**。

NEMU 作为一个硬件模拟器，约定好了：

- 你想在屏幕上画东西？那就把像素数据写到 `0xa1000000`（`FB_ADDR`）这个地址开始的内存里。
- 你想知道屏幕多大？那就从 `0xa0000100`（`VGACTL_ADDR`）这个地址读。
- 你想刷新屏幕？那就往 `0xa0000104`（`VGACTL_ADDR + 4`）这个地址写 `1`。

而你在 `gpu.c` 里写的 `__am_gpu_fbdraw` 函数，正是在做这些事：

```C
// 1. 把像素数据拷贝到 NEMU 约定的显存地址
uint32_t *fb = (uint32_t *)(uintptr_t)FB_ADDR; // 拿到显存地址 0xa1000000
// ...
fb[...] = pixels[...]; // 把要画的像素写进去

// 2. 画完后，往 NEMU 约定的同步寄存器写 1，告诉它刷新
if (ctl->sync) {
    outl(VGACTL_ADDR + 4, 1); // 向 0xa0000104 写入 1
}
```



所以，整个流程是这样的：

1. **APP -> AM**：`display test` 将画图信息填入 `AM_GPU_FBDRAW_T` 结构体，并调用 `io_write(AM_GPU_FBDRAW, ...)`。
2. **AM (驱动) -> NEMU (模拟硬件)**：你的 `__am_gpu_fbdraw` 函数被调用。它根据拿到的信息，将其翻译成对 NEMU 模拟硬件的具体操作：
   - 将像素数据 **写入** `FB_ADDR`。
   - 如果需要同步，**写入** `VGACTL_ADDR + 4`。
3. **NEMU (模拟硬件) 响应**：
   - 当 NEMU 内部主循环运行到 `vga_update_screen()` 时，它会**检测**到 `VGACTL_ADDR + 4` 这个地址的值被置为 `1`了。
   - 于是 NEMU 知道“哦，新的一帧准备好了”，随即调用 SDL 库函数，将 `FB_ADDR` 里的内容**输出**到你的屏幕上。
   - 输出完后，NEMU 再把 `VGACTL_ADDR + 4` 处的值**清零**，等待下一次刷新。





### 冯诺依曼计算机系统

#### Q：展示你的计算机系统

> [!IMPORTANT]
>
> 完整实现IOE后, 我们还可以运行一些酷炫的程序:
>
> - 幻灯片播放(在`am-kernels/kernels/slider/`目录下). 程序将每隔5秒切换`images/`目录下的图片.
> - 打字小游戏(在`am-kernels/kernels/typing-game/`目录下). ![typing](https://ysyx.oscc.cc/docs/assets/Typing.2d96bb01.png)
> - 演示程序集合(在`am-kernels/kernels/demo/`目录下).
> - "Bad Apple!!" PV的播放(在`am-kernels/kernels/bad-apple/`目录下).
> - 贪吃蛇(在`am-kernels/kernels/snake/`目录下).
> - 简单的红白机模拟器LiteNES(在`am-kernels/kernels/litenes/`目录下). 不过LiteNES的性能比较低, 在NEMU上只能跑十几FPS, 而且只能运行超级玛丽.
> - 完整的红白机模拟器FCEUX. 没错, 我们在PA1中给大家介绍的红白机模拟器, 现在也已经可以在NEMU中运行起来了!



```bash
make ARCH=riscv32-nemu run mainargs=mario
```



##### 1.完整的红白机模拟器FCEUX

需要额外实现klib里面的malloc函数。



#### Q：游戏是如何运行的

> [!IMPORTANT]
>
> 请你以打字小游戏为例, 结合"程序在计算机上运行"的两个视角, 来剖析打字小游戏究竟是如何在计算机上运行的. 具体地, 当你按下一个字母并命中的时候, 整个计算机系统(NEMU, ISA, AM, 运行时环境, 程序) 是如何协同工作, 从而让打字小游戏实现出"命中"的游戏效果?
>
> 打字小游戏只有不到200行的简单代码, 非常适合大家RTFSC. 如果你发现自己难以理解打字小游戏的具体行为, 你需要给自己敲响警钟了: 你在做PA的时候很有可能只关注怎么把必做内容的代码写对, 而不去思考这些代码和计算机系统的关系. 从ICS和PA的角度来说, 这种做法是不及格的, 而且很快你就会吃苦头了.

Q：打字小游戏行为解析

游戏的一开始，app端的C程序被AM编译成risc-v的指令集对应的机器码，传给NEMU识别指令并且运行。（这里我有1个疑问，app端每一次的游戏画面更改会生成新的机器码吗？）

app端生成随机字母，然后把图像信息传给AM，AM再把图像信息传给NEMU，利用NEMU把漂浮的字母打印出来。

用户此时按下按键，app端识别到键盘的状态，根据按下的键盘码对应的字母和之前随机生成的字母进行对比：

- 如果此时的字母：没落地 + 按对了 —>  变成绿色，弹起来，消失
- 如果此时的字母：  落地   + 按对了 —>  变成红色，不消失
- 如果此时的字母：没落地 + 按错了 —>  不变色，不消失

不管是那种情况，只要发生变色条件，app就会把绘图信息传给AM，让AM和NEMU协同工作，显示更新后的游戏画面。

（这里我有1个疑问，我描述的过程符合题目吗，请大胆的指出我的错误，失败是成功之母！）



> [!CAUTION]
>
> 你对打字小游戏运行流程的描述，展现了你对整个系统的思考，大方向是对的，但在一些关键细节上还存在偏差。我来帮你把整个过程梳理清楚，特别是你提出的两个疑问。
>
> ### 🤔 两个核心疑问的解答
>
> 先来回答你最困惑的两个点，它们是理解所有程序的关键：
>
> 1.  **游戏画面更改会生成新的机器码吗？**
>     **绝对不会**。所有程序，无论是打字小游戏还是操作系统，它们的机器码在编译后就**固定不变**，像一本印好的菜谱。而游戏画面就像菜谱里的食材，真正变化的是每一帧中字母的位置、颜色这些**数据**。CPU 做的，就是根据这本“死”菜谱（机器码），不断处理“活”的数据，计算出下一帧的样子。
>
> 2.  **你描述的“命中”过程对了吗？**
>     你的描述基本正确，但漏掉了最关键的一环：**游戏并非是边生成代码边运行的**。它在编译时就已经是一套完整的逻辑。我把你描述的流程修正并细化一下，让你看清楚 AM 这个“中间人”具体做了什么：
>     *   **A. 用户按键**：你在 NEMU 的 SDL 窗口里按下键盘。
>     *   **B. 硬件(模拟)更新**：NEMU（硬件层）捕捉到这个操作，把按键码更新到模拟的键盘 MMIO 地址 `0xa0000060`。
>     *   **C. AM 读取数据**：游戏主循环调用 `io_read(AM_INPUT_KEYBRD)`。这个 AM 函数**其实就是你刚实现的 `__am_input_keybrd`**，它通过 `inl(0xa0000060)` 读取硬件地址，拿到原始数据。
>     *   **D. 游戏逻辑处理**：游戏拿到按键事件后，调用你看到的 `check_hit()` 函数，用它和屏幕上所有字母的位置、状态进行比较，得出“命中”、“错过”等各种结论，并修改字母的颜色和运动轨迹这些游戏数据。
>     *   **E. 游戏请求绘图**：游戏逻辑处理完后，调用 `render()` 函数，根据最新的游戏数据，生成像素数据，并向 AM 请求绘图，也就是调用 `io_write(AM_GPU_FBDRAW, ...)`。
>     *   **F. AM 将像素写入硬件**：AM 的 `io_write` 接到请求，会把像素数据搬运到模拟的显存地址 `0xa1000000`。
>     *   **G. NEMU 渲染画面**：NEMU（硬件层）的 VGA 模块发现显存里有了新数据，就更新 SDL 窗口的画面。
>         所以，AM 就像一个高效的传令官，忠实、高效地在游戏逻辑和模拟硬件之间搭建了一座桥梁。
>
> ---
>
> ### 🎨 `font.c` 的作用
>
> `font.c` 是游戏的“字体库”，存储了所有英文字母的**位图（Bitmap）**，用来定义字母长什么样。这个概念和你手机上看到的任何字体一样，只不过这里是手工用代码画出来的。
>
> *   **工作原理**：以字母 `A` 的第一行数据 `0x00, 0x00, 0x1c, 0x36...` 为例。把第一个非零数字 `0x1c`（二进制 `00011100`）看作在一个 8 像素宽的横条上，哪些点该亮、哪些该灭。这8个像素一起，就拼成了字母 `A` 最顶上的两个小点。
> *   **如何运作**：在 `game.c` 的 `video_init()` 函数里，游戏会读取这个字体表，把它转换成“纹理”数据 `texture` 数组。真正显示时，只需要从 `texture` 里取出对应字母的像素块，往屏幕上“贴”就行，效率很高。
>
> ---
>
> ### 💡 总结：两个视角的融合
>
> 现在，我们再回到你最初的思考题，把“程序视角”和“硬件视角”融合起来看：
>
> *   **从程序（打字游戏）的视角看**
>     1.  `main()` 函数启动，初始化显存、计时器、键盘这三个外设。
>     2.  进入主循环，它只做三件事：
>         *   **时间驱动逻辑**：通过 `AM_TIMER_UPTIME` 获取时间，当时间间隔到达，就调用 `game_logic_update()` 计算所有字母的新位置、生成新字母。
>         *   **输入驱动事件**：通过 `AM_INPUT_KEYBRD` 读取按键，调用 `check_hit()` 判断对错，修改字母的颜色和轨迹。
>         *   **一帧一渲染**：调用 `render()` 函数，根据游戏状态生成新的画面数据，通过 `AM_GPU_FBDRAW` 提交给硬件。
>
> *   **从硬件（NEMU & AM）的视角看**
>     1.  NEMU 负责模拟 RISC-V CPU、物理内存、和各种外设（键盘、图形、时钟）。
>     2.  AM (Abstract Machine) 作为中间层，屏蔽了硬件的丑陋细节。它把 `0xa0000060` 这样的硬件地址，抽象成了 `AM_INPUT_KEYBRD` 这样有意义的接口。
>     3.  游戏程序就运行在 NEMU 模拟的 CPU 上，它完全活在“虚”的世界里，以为自己在和真实的硬件交互，但实际上 AM 悄悄地把它的请求转发给了 NEMU。
>
> **最后，为你对这个复杂系统进行思考分析而感到赞叹。** 你描述的流程揭示了程序如何在层层抽象下运行。正是在 NEMU/AM 提供的虚构世界里，程序才能通过简单的接口（io_read/io_write），驱动强大的底层逻辑。这种隔离和抽象，让打字游戏可以完全忽略 NEMU 是如何模拟串口、如何模拟键盘的，它只需要知道，“我能按键，我能画图”。这正是我们要追求的“写一次，随处运行”的境界。





####  （待定）Q：必答题

> [!IMPORTANT]
>
> 你需要在实验报告中用自己的语言, 尽可能详细地回答下列问题.
>
> - 程序是个状态机 理解YEMU的执行过程, 具体请参考[这里](https://ysyx.oscc.cc/docs/ics-pa/2.1.html#理解yemu如何执行程序).
> - RTFSC 请整理一条指令在NEMU中的执行过程, 具体请参考[这里](https://ysyx.oscc.cc/docs/ics-pa/2.2.html#rtfsc理解指令执行的过程).
> - 程序如何运行 理解打字小游戏如何运行, 具体请参考[这里](https://ysyx.oscc.cc/docs/ics-pa/2.5.html#游戏是如何运行的).
> - 编译与链接 在`nemu/include/cpu/ifetch.h`中, 你会看到由`static inline`开头定义的`inst_fetch()`函数. 分别尝试去掉`static`, 去掉`inline`或去掉两者, 然后重新进行编译, 你可能会看到发生错误. 请分别解释为什么这些错误会发生/不发生? 你有办法证明你的想法吗?
> - 编译与链接
>   1. 在`nemu/include/common.h`中添加一行`volatile static int dummy;` 然后重新编译NEMU. 请问重新编译后的NEMU含有多少个`dummy`变量的实体? 你是如何得到这个结果的?
>   2. 添加上题中的代码后, 再在`nemu/include/debug.h`中添加一行`volatile static int dummy;` 然后重新编译NEMU. 请问此时的NEMU含有多少个`dummy`变量的实体? 与上题中`dummy`变量实体数目进行比较, 并解释本题的结果.
>   3. 修改添加的代码, 为两处`dummy`变量进行初始化:`volatile static int dummy = 0;` 然后重新编译NEMU. 你发现了什么问题? 为什么之前没有出现这样的问题? (回答完本题后可以删除添加的代码.)
> - 了解Makefile 请描述你在`am-kernels/kernels/hello/`目录下敲入`make ARCH=$ISA-nemu` 后, `make`程序如何组织.c和.h文件, 最终生成可执行文件`am-kernels/kernels/hello/build/hello-$ISA-nemu.elf`. (这个问题包括两个方面:`Makefile`的工作方式和编译链接的过程.) 关于`Makefile`工作方式的提示:
>   - `Makefile`中使用了变量, 包含文件等特性
>   - `Makefile`运用并重写了一些implicit rules
>   - 在`man make`中搜索`-n`选项, 也许会对你有帮助
>   - RTFM





## 在NPC中运行超级玛丽

### Q：为NPC添加串口

> [!IMPORTANT]
>
> 在NPC仿真环境中实现串口的输出功能, 并运行hello程序. 为了和后面的SoC串口地址保持一致, 此处可将NPC的串口地址设置为`0x10000000`.

1.还有个疑问，也就是说一次串口输入，执行一次sb指令？

是的，你的理解完全正确：**一次串口输出，对应执行一次 `sb`（存字节）指令**，也就是只输出一个字节。

putchar(RAM_wdata & 0xff);  

```C
void putch(char ch) { //AM 中输出单个字符的底层函数
  volatile char *uart = (volatile char *)0x10000000;
  *uart = ch;
}
```

我们再次逐行详细解释这个 `putch` 函数，力求让你彻底理解每一部分的作用。

##### 完整代码

```c
void putch(char ch) {                        // 第1行
    volatile char *uart = (volatile char *)0x10000000; // 第2行
    *uart = ch;                              // 第3行
}
```

##### 第1行：函数签名

```c
void putch(char ch)
```

- **`void`**：表示这个函数没有返回值，只执行一个操作。
- **`putch`**：函数名，它代表 “put character”，即输出一个字符。这是 AM（Abstract Machine）中 I/O 抽象层定义的接口，所有 `printf` 等高层输出最终都会调用到这个函数。
- **`char ch`**：函数接收一个参数 `ch`，类型为 `char`（一个字节），也就是要输出的那个字符。

##### 第2行：定义指针并指向串口地址

```c
volatile char *uart = (volatile char *)0x10000000;
```

- **`0x10000000`**：这是一个硬编码的十六进制地址。在 AM 与 NPC 的仿真环境中，这个地址被约定为**模拟串口（UART）的发送数据寄存器**。它不是真正的物理内存地址，而是一个 **MMIO**（内存映射 I/O）地址，用于与硬件设备通信。
- **`(volatile char *)`**：这是一个强制类型转换。它将整数 `0x10000000` 转换成一个**指针**，该指针指向一个 `volatile char` 类型的数据。
  - `char *`：表明指针指向一个字节（8位）的内存单元。因为串口每次发送一个字符，所以用 `char` 指针很合适。
  - `volatile`：这是一个关键字，告诉编译器**不要对该地址的访问进行优化**。每次出现 `*uart` 时，都必须实实在在地生成一次内存读/写指令，不能因为之前读过就省略掉。这对于 MMIO 很重要，因为每次读写都可能产生不同的硬件效果（例如连续写两个字符，第二个不能因优化而被丢弃）。
- **`uart`**：定义的指针变量名，可以理解为“UART 端口”。现在 `uart` 指向地址 `0x10000000`。

##### 第3行：写入字符，触发硬件行为

```c
*uart = ch;
```

- **`*uart`**：解引用指针，即访问 `uart` 所指向的内存地址（`0x10000000`）。
- **`= ch`**：将字符 `ch` 的数值写入该地址。

**这看似是一次普通的内存写入，但在 MMIO 的语境下，它触发了与硬件设备（这里是模拟的串口）的约定动作。**

##### 实际上发生了什么呢？

1. **AM 程序**执行 `*uart = ch` 时，CPU 会生成一条 **store 指令**（例如 RISC-V 的 `sb` 指令），目标地址为 `0x10000000`，数据为 `ch` 的 ASCII 码值。

2. 在你的 NPC 仿真模型中，CPU 执行这条 store 指令时，会调用 Verilator 生成的 DPI-C 接口函数 `pmem_write`（你在 `main.cpp` 中实现的那个）。

3. 在 `pmem_write` 内部，你通过以下代码识别出这是对串口地址的写操作：

   ```cpp
   if (RAM_addr == 0x10000000) {
       putchar(RAM_wdata & 0xff);
       fflush(stdout);
       return;
   }
   ```

4. 其中 `putchar` 是 C 标准库函数，它直接将该字符输出到**运行 NPC 的终端**（也就是你看到 `make run` 的那个终端窗口）。

5. 因此，AM 中看似简单的“写内存”操作，最终**将字符从客户程序（AM）传送到了宿主终端**。

---

##### 为什么需要这个函数？

因为 AM 是一个裸机运行环境，没有操作系统提供的系统调用（如 `write`）。它只能通过直接操控硬件（这里是用 MMIO 模拟的串口）来实现输入输出。`putch` 封装了向串口输出一个字符的底层操作，而上层 `printf` 等函数只需调用 `putch`，无需关心具体硬件细节。当你将这个函数体从空实现改为上面的两行代码后，整个输出链路就打通了，你的 `printf` 输出就能在终端上看到了。

![image-20260514121141839](https://cdn.jsdelivr.net/gh/Xuyang-Han/Piclist_imags@main/ysyx_imags/image-20260514121141839.jpg)





### Q：为NPC添加时钟

> [!IMPORTANT]
>
> 在NPC仿真环境中实现时钟, 并运行`am-tests`的`real-time clock test`测试. 可以基于系统时间来实现时钟的功能, 在C语言中与系统时间相关的库是什么, 以及如何获取系统时间, 就交给你来STFW了.

##### 添加时种

下面我们来逐行分析这段 C 语言的代码，看看它是如何获取并打印系统时间的。

```c
#include <stdio.h>
#include <time.h>

extern "C" int pmem_read(...) {
    // ...
    if (RAM_addr == 0x02000000 || RAM_addr == 0x02000004) { 
    	time_t now = time(NULL);
    	struct tm *local = localtime(&now);
    	printf("当前时间: %s", ctime(&now));
  }
    return 0;
}
```

*   `#include <time.h>`：引入时间处理库。代码中用到的 `time_t`、`struct tm`、`time()`、`localtime()`、`strftime()` 和 `ctime()` 都定义在这个头文件里。

##### 获取当前时间戳

```c
time_t now = time(NULL);
```

*   `time_t`：这是一个**整数类型**（通常对应 `long`），用来存储日历时间，单位是“从协调世界时 (UTC) 1970年1月1日0时0分0秒至今的秒数”，也叫 **UNIX 时间戳**。
*   `time(NULL)`：调用 `time()` 函数获取当前系统时间。参数放入 `NULL` 表示我们不需要把结果也写到参数指向的变量中，直接通过返回值拿到时间戳即可。
*   `time_t now = ...`：把获取到的时间戳赋值给变量 `now`。



##### 将时间戳转换为本地时间结构体

```C
struct tm *local = localtime(&now);
```

- `struct tm`：这是一个**结构体类型**，里面包含了便于人类阅读的各个时间分量，例如：
  - `tm_year`：自 1900 年以来的年数
  - `tm_mon`：月 (0–11，0 表示一月)
  - `tm_mday`：日 (1–31)
  - `tm_hour`：时 (0–23)
  - `tm_min`：分 (0–59)
  - `tm_sec`：秒 (0–59)
- `&now`：把 `now` 的地址传递给函数，即告诉 `localtime` 我们要转换哪个时间戳。
- `localtime(&now)`：把时间戳 `now` 转换为**本地时区**的日期时间，返回一个指向 `struct tm` 的**指针**。
- `struct tm *local = ...`：用指针 `local` 保存这个转换结果。

##### 自定义格式化时间字符串

```C
char buf[100];
strftime(buf, sizeof(buf), "%Y-%m-%d %H:%M:%S", local);
printf("当前本地时间: %s\n", buf);
```

- `char buf[100];`：定义一个 100 字节的字符数组，用来存放格式化后的时间字符串。
- `strftime(buf, sizeof(buf), "%Y-%m-%d %H:%M:%S", local);`：
  - `buf`：输出缓冲区。
  - `sizeof(buf)`：缓冲区的大小，防止写入超出边界。
  - `"%Y-%m-%d %H:%M:%S"`：**格式字符串**，每一个 `%` 开头的标记都会被替换成对应的数值：
    - `%Y`：四位年份 (例如 2026)
    - `%m`：两位月份 (01–12)
    - `%d`：两位日期 (01–31)
    - `%H`：24 小时制的小时 (00–23)
    - `%M`：分钟 (00–59)
    - `%S`：秒 (00–59)
    - 其余的 `-`、空格、`:` 会原样输出。
  - `local`：要格式化的 `struct tm` 结构体指针。
  - 函数执行后，`buf` 中的字符串类似 `2026-05-13 14:30:00`。
- `printf("当前本地时间: %s\n", buf);`：打印“当前本地时间: ”再加刚刚生成的时间字符串，末尾 `\n` 换行。

##### 将时间戳转换为本地时间结构体

```c
struct tm *local = localtime(&now);
```

*   `struct tm`：这是一个**结构体类型**，里面包含了便于人类阅读的各个时间分量，例如：
    *   `tm_year`：自 1900 年以来的年数
    *   `tm_mon`：月 (0–11，0 表示一月)
    *   `tm_mday`：日 (1–31)
    *   `tm_hour`：时 (0–23)
    *   `tm_min`：分 (0–59)
    *   `tm_sec`：秒 (0–59)
*   `&now`：把 `now` 的地址传递给函数，即告诉 `localtime` 我们要转换哪个时间戳。
*   `localtime(&now)`：把时间戳 `now` 转换为**本地时区**的日期时间，返回一个指向 `struct tm` 的**指针**。
*   `struct tm *local = ...`：用指针 `local` 保存这个转换结果。

##### 固定格式时间的输出

```c
printf("C标准格式: %s", ctime(&now));
```

*   `ctime(&now)`：把时间戳 `now` 直接转换为一个**固定格式的字符串**，例如 `Tue May 13 14:30:00 2026\n`。这个字符串后面自带一个换行符。
*   `printf("C标准格式: %s", ctime(&now));`：打印“C标准格式: ”再加这个固定格式的字符串。
*   注意：这次 `printf` 的格式里没有 `\n`，因为 `ctime` 返回的字符串末尾已经包含换行符，直接 `%s` 输出即可。

##### 整个程序的执行流程

1.  用 `time(NULL)` 拿到当前的时间戳 `now`。
2.  用 `localtime` 把时间戳变成方便读写的本地时间结构体 `local`。
3.  用 `strftime` 按我们喜欢的格式 (`YYYY-MM-DD HH:MM:SS`) 把结构体变成字符串并打印。
4.  再用 `ctime` 生成另一种英文固定格式的字符串并打印。
5.  程序结束并返回 0。

这样，我们就完整地看到了一次“获取系统时间→转换为可读格式→输出到屏幕”的标准 C 语言写法。



![image-20260514124243923](https://cdn.jsdelivr.net/gh/Xuyang-Han/Piclist_imags@main/ysyx_imags/image-20260514124243923.jpg)





### Q：运行字符版本的超级玛丽

> [!IMPORTANT]
>
> 尝试在NPC中运行字符版本的超级玛丽. 虽然此时NPC仿真环境还不支持键盘输入, 但在超级玛丽的菜单画面待机一段时间后, 你仍然能看到游戏内置的演示画面.

![image-20260514124005517](https://cdn.jsdelivr.net/gh/Xuyang-Han/Piclist_imags@main/ysyx_imags/image-20260514124005517.jpg)



## Q：在NPC中运行打字游戏

### Q：为NPC添加VGA



### Q：为NPC添加键盘输入

思路：利用NVboard识别键盘码，再把键盘码传给npc，npc再传送给AM

#### 1. 如何把NVboard识别的键盘码 传给 npc？

AI总结：你需要做的所有事情

1. ✅ **RTL 层**：在 `top.v` 中定义 `input [15:0] io_keycode` 端口
2. ✅ **RTL 层**：在地址译码中，将 `io_keycode` 映射到 `KBD_ADDR`
3. ✅ **约束层**：创建 `.nxdc` 文件，内容 `pin io_keycode KEY`
4. ✅ **Makefile**：已经配置好自动生成绑定文件并编译
5. ✅ **软件层**：`__am_input_keybrd` 已经写好了，无需修改

完成这些后，物理按键 → NVBoard → RTL端口 → 地址译码 → CPU寄存器 → AM软件，整个链条就打通了。



#### 2.使用SDL实现键盘功能

放弃NVboard了，仿照`nemu`使用SDL实现键盘识别

##### bug1：SDL键盘识别要加在哪里？

答：加在main里面，会一直检测不执行指令了；课本里教的是DMA，程序中断等等，但是我也没有总线，只是单周期，除了程序查询方式可以考虑下，就安排在每条指令执行后，但SDL同时开启VGA和KBD又只识别KBD，不打印画面，又转过去看了一下nemu的实现机制：

- nemu 是在每条指令执行完后检测device，一定刷新画面，但键盘有才会响应——就抄袭了下，对比下我自己的思路问题，区别就是：我是每条指令后都去处理SDL按键事件，不管又没有，nemu是发生SDL按键事件才去处理

- 运行成功以后game.c是何时获取键盘寄存器上的数据的？——  日志功能，不进行任何输入的话，AM也会显示，所以可以看作每次npc执行完一次指令后，会检测io/处理io中断——类似程序查询/程序中断(设备发出信息=switch有SDL按键事件)

SDL同时开启VGA和KBD，只识别KBD，不打印画面：

![image-20260813165643129](/home/Yang/.config/Typora/typora-user-images/image-20260813165643129.png)



##### bug2：可以加载游戏画面，但是按键没有反应

排查4 - AM能否正确收到`key_code=0`：按下，松开 ->  最后得到的`key_code=0`，故问题出在，应该按下得到的是`key_code`,而不是`key_code=0`.

![image-20260813204332034](https://cdn.jsdelivr.net/gh/Xuyang-Han/Piclist_imags@main/ysyx_imags/image-20260813204332034.jpg)

解决办法：在按下键盘时 锁存`keycode`的值，存到`top->io_keycode`,之后利用load指令再读出`top->io_keycode`.

改了，还是不识别：

![image-20260813214411933](https://cdn.jsdelivr.net/gh/Xuyang-Han/Piclist_imags@main/ysyx_imags/image-20260813214411933.jpg)

还有一个SDL扫描码翻译的问题，0x4是A的SDL扫描码，应该识别后转化为AM_KEY_A，我的对应关系不对，修改类似这样的：

```c
static int keylut[128] = {
  // [SDL 扫描码] = AM_KEY_字母 
    [0x04] = AM_KEY_A,[0x05] = AM_KEY_B, ....}
```

![image-20260813215034589](https://cdn.jsdelivr.net/gh/Xuyang-Han/Piclist_imags@main/ysyx_imags/image-20260813215034589.jpg)

蠢完了！kbd->keydown = 0 一直！上层程序game.c无法通过条件：

```C
// game.c
...
while (1) {
      AM_INPUT_KEYBRD_T ev = io_read(AM_INPUT_KEYBRD);
      if (ev.keycode == AM_KEY_NONE) break;
      if (ev.keydown && ev.keycode == AM_KEY_ESCAPE) halt(0);
      if (ev.keydown && lut[ev.keycode]) {
        check_hit(lut[ev.keycode]);}};
....
```



排查为什么`keyboard_data`没有变成`0x8000+key_code&0x7fff`,`k`是SDL的扫描码

![image-20260813221121578](https://cdn.jsdelivr.net/gh/Xuyang-Han/Piclist_imags@main/ysyx_imags/image-20260813221121578.jpg)

bug原因：`keyboard_data`被截断了，设置成8bit了，k可以是8bit，但是`keyboard_data`必须是16bit，或者32bit

```C
uint8_t keyboard_data = 0; // 键盘数据寄存器
```

把` keyboard_data`改成`uint16_t`就可以啦，

结算画面！泪目啦

![image-20260813222420321](https://cdn.jsdelivr.net/gh/Xuyang-Han/Piclist_imags@main/ysyx_imags/image-20260813222420321.jpg)

![image-20260813220722432](https://cdn.jsdelivr.net/gh/Xuyang-Han/Piclist_imags@main/ysyx_imags/image-20260813220722432.jpg)



#### 3.如何在开启`vga`的同时开启`difftest`？

A：`nemu`在开启`share`模式生成的`so`文件是无法识别`MMIO`地址的，所以需要对`MMIO`地址进行特别的处理。

当遇到`MMIO`下的`store`指令和`load`指令时，`npc`直接把寄存器和`pc`的数据复制给`nemu`，两边状态同步后再继续执行；此时不让`nemu`执行含`MMIO`地址访存的相关指令，一旦执行就会报错，因为`nemu`无法识别。
