# C1必答题

## Q：实现iringbuf

> [!IMPORTANT]
>
> 根据上述内容, 在NEMU中实现iringbuf. 你可以按照自己的喜好来设计输出的格式, 如果你想输出指令的反汇编, 可以参考itrace的相关代码; 如果你不知道应该在什么地方添加什么样的代码, 你就需要RTFSC了.

A：和实现监视点的思路很类似，使用缓冲池来实现。

![image-20260515173101543](/home/Yang/.config/Typora/typora-user-images/image-20260515173101543.png)







## Q：实现mtrace

> [!IMPORTANT]
>
> 这个功能非常简单, 你已经想好如何实现了: 只需要在`paddr_read()`和`paddr_write()`中进行记录即可. 你可以自行定义mtrace输出的格式.
>
> 不过和最后只输出一次的iringbuf不同, 程序一般会执行很多访存指令, 这意味着开启mtrace将会产生大量的输出, 因此最好可以在不需要的时候关闭mtrace. 
>
> 噢, 那就参考一下itrace的相关实现吧: 尝试在Kconfig和相关文件中添加相应的代码, 使得我们可以通过menuconfig来打开或者关闭mtrace. 另外也可以实现mtrace输出的条件, 例如你可能只会关心某一段内存区间的访问, 有了相关的条件控制功能, mtrace使用起来就更加灵活了.

![image-20260516152638885](/home/Yang/.config/Typora/typora-user-images/image-20260516152638885.png)



## （待定）Q：实现ftrace

> [!IMPORTANT]
>
> 根据上述内容, 在NEMU中实现ftrace. 你可以自行决定输出的格式. 你需要注意以下内容:
>
> - 你需要为NEMU传入一个ELF文件, 你可以通过在`parse_args()`中添加相关代码来实现这一功能
> - 你可能需要在初始化ftrace时从ELF文件中读出符号表和字符串表, 供你后续使用
> - 关于如何解析ELF文件, 可以参考`man 5 elf`
> - 如果你选择的是riscv32, 你还需要考虑如何从`jal`和`jalr`指令中正确识别出函数调用指令和函数返回指令
>
> 注意, 你不应该通过`readelf`等工具直接解析ELF文件. 在真实的项目中, 这个方案确实可以解决问题; 但作为一道学习性质的题目, 其目标是让你了解ELF文件的组织结构, 使得将来你在必要的时候(例如在裸机环境中)可以自己从中解析出所需的信息. 如果你通过`readelf`等工具直接解析ELF文件, 相当于自动放弃训练的机会, 与我们设置这道题目的目的背道而驰.















# C2 支持RV32E的单周期NPC

## 为NPC搭建基础设施

### Q：为NPC搭建sdb

> [!IMPORTANT]
>
> 你需要为NPC实现单步执行, 打印寄存器和扫描内存的功能, 而表达式求值和监视点都是基于打印寄存器和扫描内存实现的. 单步执行和扫描内存都很容易实现.
>
> 为了打印寄存器, 你需要访问RTL中的通用寄存器. 有如下两种方式进行访问, 你可以自行选择:
>
> 1. 通过DPI-C进行访问.
> 2. 通过Verilator编译出的C++文件来访问通用寄存器, 例如`top->rootp->NPC__DOT__isu__DOT__R_ext__DOT__Memory`, 具体的C++变量名与Verilog中的模块名和变量名有关, 可阅读编译出的C++头文件得知. 不过C++变量名可能会在修改RTL代码或更改Verilator版本时发生变化, 需要手动同步修改.

#### 1.实现单步执行

每一次执行1条指令，就停止执行，直到得到下一条命令

#### 2.如何实现命令`c`

并且，执行完`ebreak`后，如何实现不退出，而是返回继续接受命令的地方。

#### 3.如何实现打印寄存器

通过DPI-C进行访问，但是出了一堆bug。



### （待定）Q：为NPC添加trace支持

> [!IMPORTANT]
>
> 你已经在NEMU中实现itrace, mtrace和ftrace了, 尝试在NPC中实现它们. 其中, 实现itrace需要注意两点:
>
> - 需要通过DPI-C获取当前执行的指令
> - 需要链接capstone库, 具体可以参考`nemu/src/utils/filelist.mk`
>
> 在仿真环境获取到当前执行的指令和访存信息之后, mtrace和ftrace也就不难实现了.

itrace：获取 pc，机器码，反汇编的汇编指令

mtrace：写在里面mem_read和write即可

ftrace：暂时还不会



### Q：为NPC添加DiffTest支持

> [!IMPORTANT]
>
> DiffTest是处理器调试的一大杀手锏, 在为NPC实现更多指令之前, 为其搭建DiffTest是一个明智的选择. 在这里, DUT是NPC, 而REF则选择NEMU. 为此, 你需要
>
> - 在`nemu/src/cpu/difftest/ref.c`中实现DiffTest的API, 包括`difftest_memcpy()`, `difftest_regcpy()`和`difftest_exec()`. 此外`difftest_raise_intr()`是为中断准备的, 目前暂不使用
> - 在NEMU的menuconfig中选择共享库作为编译的目标
>
> ```text
> Build target (Executable on Linux Native)  --->
>   (X) Shared object (used as REF for differential testing)
> ```
>
> - 重新编译NEMU, 成功后将会生成动态库文件`nemu/build/riscv32-nemu-interpreter-so`
> - 在NPC的仿真环境中通过动态链接方式链接上述的动态库文件, 通过其中的API来实现DiffTest的功能, 具体可以参考NEMU的相关代码
>
> 尝试在打开DiffTest机制的情况下在NPC中正确运行dummy程序. 为了检查DiffTest机制是否生效, 你可以为NPC中`addi`指令的实现注入一个错误, 观察DiffTest是否能够按照预期地报告这一错误.
>
> 注意, 为了再次将NEMU编译成ELF, 你还需要修改NEMU中menuconfig的编译目标.

天哪！终于完成了！
操作流程：

```

```

Q：if (argc >= 3) { init_difftest(argv[2]);}
也就是说argc >= 3时，必然传入了so文件的路径吗，也就不需要difftest_en这个值控制了吗

A：你理解得很对：**`argc >= 3` 表示用户确实在命令行提供了第二个参数（通常是参考实现动态库的路径）**，因此 `init_difftest(argv[2])` 会被调用，这个函数内部通常会将一个全局标志（例如 `difftest_enabled`）设置为 `true`，并加载 `.so` 文件。之后其他地方只需检查 `difftest_enabled()` 即可知道是否需要进行差分测试。

所以，**确实不再需要额外的 `difftest_en` 变量来控制是否开启**——因为是否加载了 `.so` 文件本身就决定了差分测试是否可用。如果你仍然希望保留手动开关（例如虽然加载了 `.so` 但暂时关闭对比），可以再定义一个 `difftest_on` 变量，但多数简单实现中，`init_difftest` 成功后就直接开启，没有额外开关。

常见写法：

```c
static bool difftest_enabled = false;

void init_difftest(const char *so_file) {
    // 加载动态库，初始化参考模型...
    difftest_enabled = true;   // 开启差分测试
}

bool difftest_enabled() { return difftest_enabled; }

int main(int argc, char *argv[]) {
    // ...
    if (argc >= 3) init_difftest(argv[2]);
    // 之后使用 if (difftest_enabled()) 来判断
}
```

因此，**`argc >= 3` 就隐含了开启 difftest**，不需要额外的布尔变量来控制开启/关闭。如果你需要临时关闭（比如调试），可以再加一个命令行选项 `--no-difftest` 来覆盖。



## 实现RV32E指令集

### Q：搭建riscv32e-npc的AM

> [!IMPORTANT]
>
> 你需要完成如下内容:
>
> - 为`riscv32e-npc`提供`run`目标, 从而支持一键编译程序并仿真.
> - 实现`riscv32e-npc`中的`halt()`函数, 从而通知NPC的仿真环境结束仿真, 并让其得知程序的运行结果是否正确.

答：



Q：0x800003c8: 00055863    bge      a0, zero, 0x800003d8 bge省略的一定是rs2吗

是的，在 `bge` 这类 RISC-V 分支指令中，**省略的一定是第二个操作数（`rs2`）**。
`bge` 指令的完整语法是：`bge rs1, rs2, offset`。如果 `rs2` 是零寄存器（`x0`/`zero`），常见的汇编器（包括 Capstone 默认行为）会省略它，只显示 `bge rs1, offset`。这正是你之前看到的 `bge a0, 0x800003d8` 的原因——Capstone 默认省略了 `zero`。通过使用 `detail` 接口自行拼接操作数，你可以强制显示 `zero`，得到 `bge a0, zero, 0x800003d8`。


sh指令怎么确定wmask呢？
如何根据内存地址，来确定存入的是前半字还是后半字呢？

- **对于 `sb`**（store byte）：写入 1 个字节，字节位置由 `Mem_addr[1:0]` 决定（0~3），所以掩码为 `4'b0001 << Mem_addr[1:0]`。
- **对于 `sh`**（store half-word）：写入连续的 2 个字节。半字在 32 位 word 中的起始地址由 `Mem_addr[1]` 决定：
  - 若 `Mem_addr[1] == 0`：起始于字节 0（低半字），掩码为 `4'b0011`（字节0和1）
  - 若 `Mem_addr[1] == 1`：起始于字节 2（高半字），掩码为 `4'b1100`（字节2和3）
- **对于 `sw`**：写入全部 4 字节，掩码为 `4'b1111`。



Q：发生了wdata赋值一直错误的情况

A：原因是sv语句的多个命名，会导致最后只选择一个“支配类型”。

![image-20260601205532018](/home/Yang/.config/Typora/typora-user-images/image-20260601205532018.png)



#### 为什么嵌套条件表达式会导致错误？

在你的代码中，`srai` 分支使用了 `$signed(Reg[rs1]) >>> (imm & 32'h1f)`，这个表达式的类型是有符号数。但在同一长链的条件表达式中，其他分支可能引入了无符号操作，例如：

- `$unsigned(Reg[rs1]) / $unsigned(Reg[rs2])` （divu）
- `$unsigned(Reg[rs1]) << (imm & 32'h1f)` （slli）

根据 SystemVerilog 标准，在确定整个条件表达式的类型时，会综合考虑所有分支的返回类型（包括中间分支），并选择一个“支配类型”。如果整个表达式的最终类型被推断为**无符号**，那么即使你写了 `$signed(...)`，最终赋值给 `wdata` 之前，结果也会被强制转换为无符号数，导致算术右移变成逻辑右移（`80000000` 作为无符号数右移 24 位得到 `00000080`）。



#### 为什么 `case` 可以解决？

`case` 语句中每个分支的表达式独立求值，然后直接赋值给 `wdata`。`wdata` 被声明为 `wire [31:0]`（无符号），但赋值时保留右侧表达式的所有位，不会额外进行符号扩展或零扩展，只做位宽调整（这里位宽相同）。因此 `$signed(...) >>> ...` 的结果会原封不动地传给 `wdata`，保留了算术移位的正确位模式。

简而言之：

- 嵌套条件表达式：整个表达式类型可能被“污染”，导致期望的有符号操作变成了无符号操作。
- `case` 语句：各分支独立计算，类型明确，互不干扰。

##### 📌 最佳实践

对于多路选择，**推荐使用 `case` 或 `always_comb`**，特别是涉及有符号/无符号混合运算时，避免使用一长串的嵌套 `? :`。这样代码可读性更高，也更安全。



**`always_comb` 正是为了解决“随意使用 `always` 造成时序混乱”而设计的**，你应该使用 `always_comb` 来包裹 `case`。



#### Q：always_comb + case会翻译成什么电路呢

`always_comb` + `case` = **纯组合逻辑的并行多路选择器**，既避免了 `assign ? :` 的类型推断陷阱，又不会引入任何时序混乱，非常适合实现指令结果选择。



# C5-线程切换

## PA3-批处理系统

### 异常响应机制

#### riscv32-硬件层

（一）riscv32提供`ecall`指令作为自陷指令, 并提供一个mtvec寄存器来存放异常入口地址. 为了保存程序当前的状态, riscv32提供了一些特殊的系统寄存器, 叫控制状态寄存器(CSR寄存器). 在PA中, 我们只使用如下3个CSR寄存器（加一个mtvec寄存器）：

- mepc寄存器 - 存放触发异常的PC
- mstatus寄存器 - 存放处理器的状态
- mcause寄存器 - 存放触发异常的原因

riscv32触发异常后硬件的响应过程如下:

1. 将当前PC值保存到mepc寄存器
2. 在mcause寄存器中设置异常号
3. 从mtvec寄存器中取出异常入口地址
4. 跳转到异常入口地址

上述保存程序状态以及跳转到异常入口地址的工作, 都是硬件自动完成的, 不需要程序员编写指令来完成相应的内容. 

（二）由于异常入口地址是硬件和操作系统约定好的, 接下来的处理过程将会由操作系统来接管, 操作系统将视情况决定是否终止当前程序的运行(例如触发段错误的程序将会被杀死). 若决定无需杀死当前程序, 等到异常处理结束之后, 就根据之前保存的信息恢复程序的状态, 并从异常处理过程中返回到程序触发异常之前的状态. 

具体地：riscv32通过`mret`指令从异常处理过程中返回, 它将根据mepc寄存器恢复PC.



### 将上下文管理抽象成CTE（AM）

所以, 我们只要把这两点信息抽象成一种统一的表示方式, 就可以定义出CTE的API了. 对于切换原因, 我们只需要定义一种统一的描述方式即可. CTE定义了名为"事件"的如下数据结构(见`abstract-machine/am/include/am.h`):

```c
typedef struct Event {
  enum { ... } event;
  uintptr_t cause, ref;
  const char *msg;
} Event;
```

其中`event`表示事件编号, `cause`和`ref`是一些描述事件的补充信息, `msg`是事件信息字符串, 我们在PA中只会用到`event`. 然后, 我们只要定义一些统一的事件编号(上述枚举常量), 让每个架构在实现各自的CTE API时, 都统一通过上述结构体来描述执行流切换的原因, 就可以实现切换原因的抽象了.

对于上下文, 我们只能将描述上下文的结构体类型名统一成`Context`, 至于其中的具体内容, 就无法进一步进行抽象了. 这主要是因为不同架构之间上下文信息的差异过大, 比如mips32有32个通用寄存器, 就从这一点来看, mips32和x86的`Context`注定是无法抽象成完全统一的结构的. 所以在AM中, `Context`的具体成员也是由不同的架构自己定义的, 比如`x86-nemu`的`Context`结构体在`abstract-machine/am/include/arch/x86-nemu.h`中定义. 因此, 在操作系统中对`Context`成员的直接引用, 都属于架构相关的行为, 会损坏操作系统的可移植性. 不过大多数情况下, 操作系统并不需要单独访问`Context`结构中的成员. CTE也提供了一些的接口, 来让操作系统在必要的时候访问它们, 从而保证操作系统的相关代码与架构无关.

最后还有另外两个统一的API：

- `bool cte_init(Context* (*handler)(Event ev, Context *ctx))`用于进行CTE相关的初始化操作. 其中它还接受一个来自操作系统的事件处理回调函数的指针, 当发生事件时, CTE将会把事件和相关的上下文作为参数, 来调用这个回调函数, 交由操作系统进行后续处理.
- `void yield()`用于进行自陷操作, 会触发一个编号为`EVENT_YIELD`事件. 不同的ISA会使用不同的自陷指令来触发自陷操作, 具体实现请RTFSC.

CTE中还有其它的API, 目前不使用, 故暂不介绍它们.

需要注意的是, 上文介绍的异常和事件是两个层次的概念: 异常是处理器硬件层次的机制, 事件是AM对异常的一种封装. 因此, 自陷异常和自陷事件也是不同层次的概念, 异常号和事件编号也并不相同.

特别地, 为了简化ISA的设计, 处理器通常只会提供一条自陷指令, 软件层次上的多个事件可能都通过相同的自陷指令来实现, 因此CTE需要额外的方式区分它们. 



### 触发第一个异常

#### 设置异常入口地址（AM）

在触发自陷操作前, 首先需要按照ISA的约定来设置异常入口地址, 将来切换执行流时才能跳转到正确的异常入口. 这显然是架构相关的行为, 因此我们把这一行为放入CTE中, 而不是让`am-tests`直接来设置异常入口地址. 当我们选择`yield test`时, `am-tests`会通过`cte_init()`函数对CTE进行初始化, 其中包含一些简单的宏展开代码. 

这最终会调用位于`abstract-machine/am/src/$ISA/nemu/cte.c`中的`cte_init()`函数，她会做2件事情：

- 对于riscv32来说, 直接将异常入口地址设置到mtvec寄存器中即可.
- 注册一个事件处理回调函数,这个回调函数由 程序`yield test`提供

#### 触发自陷操作（nemu）

为了支撑自陷操作, 同时测试异常入口地址是否已经设置正确, 你需要在NEMU中实现`isa_raise_intr()`函数 (在`nemu/src/isa/$ISA/system/intr.c`中定义)来模拟上文提到的异常响应机制, 并在执行自陷指令的时候调用它.



#### Q：实现异常响应机制

> [!IMPORTANT]
>
> 你需要实现上文提到的新指令, 并实现`isa_raise_intr()`函数. 然后阅读`cte_init()`的代码, 找出相应的异常入口地址.
>
> 如果你选择mips32和riscv32, 你会发现status/mstatus寄存器中有非常多状态位, 不过目前完全不实现这些状态位的功能也不影响程序的执行, 因此目前只需要将status/mstatus寄存器看成一个只用于存放32位数据的寄存器即可.
>
> 实现后, 重新运行`yield test`, 如果你发现NEMU确实跳转到你找到的异常入口地址, 说明你的实现正确(NEMU也可能因为触发了未实现指令而终止运行).





### 保存上下文

#### Q：重新组织Context结构体

> [!IMPORTANT]
>
> 你的任务如下:
>
> - 实现这一过程中的新指令, 详情请RTFM.
> - 理解上下文形成的过程并RTFSC, 然后重新组织`abstract-machine/am/include/arch/$ISA-nemu.h` (如果你选择RISC-V, 则文件名为`riscv.h`) 中定义的`Context`结构体的成员, 使得这些成员的定义顺序和 `abstract-machine/am/src/$ISA/nemu/trap.S`中构造的上下文保持一致.
>
> 需要注意的是, 虽然我们目前暂时不使用上文提到的地址空间信息, 但你在重新组织`Context`结构体时仍然需要正确地处理地址空间信息的位置, 否则你可能会在PA4中遇到难以理解的错误.
>
> 实现之后, 你可以在`__am_irq_handle()`中通过`printf`输出上下文`c`的内容, 然后通过简易调试器观察触发自陷时的寄存器状态, 从而检查你的`Context`实现是否正确.



####  Q：必答题(需要在实验报告中回答) - 理解上下文结构体的前世今生

> [!IMPORTANT]
>
> 你会在`__am_irq_handle()`中看到有一个上下文结构指针`c`, `c`指向的上下文结构究竟在哪里? 这个上下文结构又是怎么来的? 具体地, 这个上下文结构有很多成员, 每一个成员究竟在哪里赋值的? `$ISA-nemu.h`, `trap.S`, 上述讲义文字, 以及你刚刚在NEMU中实现的新指令, 这四部分内容又有什么联系?
>
> 如果你不是脑袋足够灵光, 还是不要眼睁睁地盯着代码看了, 理解程序的细节行为还是要从状态机视角入手.





`ysyx-workbench/nemu/src/isa/riscv32/include`的`isa-def.h`中添加CSR寄存器

```C
typedef struct {
  word_t gpr[MUXDEF(CONFIG_RVE, 16, 32)];
  vaddr_t pc;
  // 新增 CSR 寄存器
  vaddr_t mtvec;
  vaddr_t mepc;
  word_t mstatus;
  word_t mcause;
} MUXDEF(CONFIG_RV64, riscv64_CPU_state, riscv32_CPU_state);
```





```
80001484 <yield>:
80001484:	fff00893           li  a7,-1
80001488:	00000073           ecall
8000148c:	00008067           ret
```



gpr[0]   = 0xb9b9b9b9 

mcause   = 0xb 

mstatus  = 0x0 

mepc     = 0x80001488



### 事件分发

`__am_irq_handle()`的代码会把执行流切换的原因打包成事件, 然后调用在`cte_init()`中注册的事件处理回调函数, 将事件交给`yield test`来处理. 在`yield test`中, 这一回调函数是`am-kernels/tests/am-tests/src/tests/intr.c`中的`simple_trap()`函数. `simple_trap()`函数会根据事件类型再次进行分发. 不过我们在这里会触发一个未处理的事件:

```text
AM Panic: Unhandled event @ am-kernels/tests/am-tests/src/tests/intr.c:12
```

这是因为CTE的`__am_irq_handle()`函数并未正确识别出自陷事件. 根据`yield()`的定义, `__am_irq_handle()`函数需要将自陷操作打包成编号为`EVENT_YIELD`的事件.

#### Q：识别自陷事件

> [!IMPORTANT]
>
> 你需要根据`yield()`触发自陷异常时的异常号和其他状态(若有), 在`__am_irq_handle()`中将这次自陷操作打包成编号为`EVENT_YIELD`的自陷事件. 重新运行`yield test`, 如果你的实现正确, 你会看到识别到自陷事件之后输出一个字符`y`.





### 恢复上下文

代码将会一路返回到`trap.S`的`__am_asm_trap()`中, 接下来的事情就是恢复程序的上下文. `__am_asm_trap()`将根据之前保存的上下文内容, 恢复程序的状态, 最后执行"异常返回指令"返回到程序触发异常之前的状态.

不过这里需要注意之前自陷指令保存的PC, 对于x86的`int`指令, 保存的是指向其下一条指令的PC, 这有点像函数调用; 而对于mips32的`syscall`和riscv32的`ecall`, 保存的是自陷指令的PC, 因此软件需要在适当的地方对保存的PC加上4, 使得将来返回到自陷指令的下一条指令.

 从加4操作看CISC和RISC

事实上, 自陷异常只是其中一种异常类型. 有一种故障类异常, 它们返回的PC和触发异常的PC是同一个, 例如缺页异常, 在系统将故障排除后, 将会重新执行相同的指令进行重试, 因此异常返回的PC无需加4. 所以根据异常类型的不同, 有时候需要加4, 有时候则不需要加.

这时候, 我们就可以考虑这样的一个问题了: 决定要不要加4的, 是硬件还是软件呢? CISC和RISC的做法正好相反, CISC都交给硬件来做, 而RISC则交给软件来做. 思考一下, 这两种方案各有什么取舍? 你认为哪种更合理呢? 为什么?

代码最后会返回到`yield test`触发自陷操作的代码位置, 然后继续执行. 在它看来, 这次时空之旅就好像没有发生过一样.

#### Q：恢复上下文

> [!IMPORTANT]
>
> 你需要实现这一过程中的新指令. 重新运行`yield test`. 如果你的实现正确, `yield test`将不断输出`y`.

```shell
cd cd /home/Yang/ysyx/ysyx-workbench/am-kernels/tests/am-tests
make ARCH=riscv32-nemu run mainargs=i
```



#### Q：必答题(需要在实验报告中回答) - 理解穿越时空的旅程

> [!IMPORTANT]
>
> 从`yield test`调用`yield()`开始, 到从`yield()`返回的期间, 这一趟旅程具体经历了什么? 软(AM, `yield test`)硬(NEMU)件是如何相互协助来完成这趟旅程的? 你需要解释这一过程中的每一处细节, 包括涉及的每一行汇编代码/C代码的行为, 尤其是一些比较关键的指令/变量. 事实上, 上文的必答题"理解上下文结构体的前世今生"已经涵盖了这趟旅程中的一部分, 你可以把它的回答包含进来.
>
> 别被"每一行代码"吓到了, 这个过程也就大约50行代码, 要完全理解透彻并不是不可能的. 我们之所以设置这道必答题, 是为了强迫你理解清楚这个过程中的每一处细节. 这一理解是如此重要, 以至于如果你缺少它, 接下来你面对bug几乎是束手无策.



### 异常处理的踪迹 - etrace

处理器抛出异常也可以反映程序执行的行为, 因此我们也可以记录异常处理的踪迹(exception trace). 你也许认为在CTE中通过`printf()`输出信息也可以达到类似的效果, 但这一方案和在NEMU中实现的etrace还是有如下区别:

- 打开etrace不改变程序的行为(对程序来说是非侵入式的): 你将来可能会遇到一些bug, 当你尝试插入一些`printf()`之后, bug的行为就会发生变化. 对于这样的bug, etrace还是可以帮助你进行诊断, 因为它是在NEMU中输出的, 不会改变程序的行为.
- etrace也不受程序行为的影响: 如果程序包含一些致命的bug导致无法进入异常处理函数, 那就无法在CTE中调用`printf()`来输出; 在这种情况下, etrace仍然可以正常工作

事实上, QEMU和Spike也实现了类似etrace的功能, 如果在上面运行的系统软件发生错误, 开发者也可以通过这些功能快速地进行bug的定位和诊断.

#### Q：实现etrace

> [!IMPORTANT]
>
> 你已经在NEMU中实现了很多trace工具了, 要实现etrace自然也难不倒你啦.

```
y遇到自陷trap！pc at 0x80001448
y遇到自陷trap！pc at 0x80001448
y遇到自陷trap！pc at 0x80001448
y遇到自陷trap！pc at 0x80001448
y遇到自陷trap！pc at 0x80001448
```



## PA4 - 多道程序

### 内核线程

#### 创建内核线程上下文

Q：为谁创建上下文？如何创建？

A：`yield-os`提供了一个测试函数`f()`, 为测试函数`f()`创建一个上下文, 然后切换到它来执行。创建内核线程的上下文是通过CTE提供的`kcontext()`函数。



#### 线程/进程调度





#### 疑问

上下文切换中，如何把PCB的指针cp返回给AM的cte.c的`__am_irq_handle()`函数?

实现CTE的`__am_irq_handle()`函数是在`trap.S`，但是怎么判断谁是指针cp呢？  （难道是sp？）



####  `yield-os`程序逻辑

1. **初始化两个执行流**：`main` 通过 `kcontext` 为 `pcb[0]` 和 `pcb[1]` 分配栈并准备好第一个待执行的上下文（`Context`）。
2. **安装调度器**：`cte_init(schedule)` 将 `schedule` 函数注册为异常/调度回调，它的作用是在每次“切换事件”发生时决定下一个要运行的执行流。
3. **主动切换**：两个执行流都在 `f` 函数里通过 `yield()` 主动请求放弃 CPU。`yield()` 内部会触发自陷指令（例如 RISC-V 的 `ebreak`），从而进入内核态并调用 `schedule`。
4. **上下文切换**：`schedule` 保存当前 `Context` 到自己的 `PCB`，然后切换到另一个 `PCB`，最后返回新的 `Context` 指针。从内核返回时，CPU 会自动恢复到新的执行流。
5. **交替输出**：因为两个执行流会不厌其烦地：
   1. 输出一个字母 (A 或 B)
   2. 立即 `yield` 切换给对方
   3. 对方同样输出字母并切换回来
      这就形成了无限交替输出 `A B A B ...` 的效果。







#### Q：实现上下文切换（1）&（2）

> [!IMPORTANT]
>
> （1）实现上下文切换1
>
> 根据讲义的上述内容, 实现以下功能:
>
> - CTE的`kcontext()`函数
> - 修改CTE中`__am_asm_trap()`的实现, 使得从`__am_irq_handle()`返回后, 先将栈顶指针切换到新进程的上下文结构, 然后才恢复上下文, 从而完成上下文切换的本质操作
>
> 正确实现后, 你将看到`yield-os`不断输出`?`, 这是因为我们还没有为`kcontext()`实现参数功能, 不过这些输出的`?`至少说明了CTE目前可以正确地从`yield-os`的`main()`函数切换到其中一个内核线程.
>
> 
>
> （2）实现上下文切换2
>
> 根据讲义的上述内容, 修改CTE的`kcontext()`函数, 使其支持参数`arg`的传递.
>
> 因为`f()`中每次输出完信息都会调用`yield()`, 因此只要我们正确实现内核线程的参数传递, 就可以观察到`yield-os`在两个内核线程之间来回切换的现象.



出现的bug：



原因：

是因为ecall指令返回的问题，返回的是ecall本身，而不是ecall的下一条指令，所以导致A和B来回切换，但不往下执行。



删掉LOAD

```
(nemu) c
new thread: entry=8000004c, cp=80008980, sp=80010980, arg=1
new thread: entry=8000004c, cp=80010980, sp=80018980, arg=2
[yield-os]A cp=80008980
[yield-os]B cp=80010980
AB address (0x00000000) is out of bound at pc = 0x80000024
[src/isa/riscv32/reg.c:31 isa_reg_display] 开始打印寄存器内容:
Reg[$0]=0
Reg[ra]=0x80000230
Reg[sp]=0x80000938
Reg[gp]=0
Reg[tp]=0
Reg[t0]=0xb
Reg[t1]=0x20300
Reg[t2]=0x80000314

Reg[s0]=0x1869f
Reg[s1]=0x41
Reg[a0]=0x80000938
Reg[a1]=0x80000968
Reg[a2]=0xffffffff
Reg[a3]=0
Reg[a4]=0x8000097c
Reg[a5]=0x80008980

Reg[a6]=0
Reg[a7]=0xffffffff
Reg[s2]=0
Reg[s3]=0
Reg[s4]=0
Reg[s5]=0
Reg[s6]=0
Reg[s7]=0

Reg[s8]=0
Reg[s9]=0
Reg[s10]=0
Reg[s11]=0
Reg[t3]=0
Reg[t4]=0
Reg[t5]=0
Reg[t6]=0
```



保留LOAD

```
(nemu) c
new thread: entry=8000004c, cp=80008988, sp=80010988, arg=1
new thread: entry=8000004c, cp=80010988, sp=80018988, arg=2
[yield-os]A cp=80008988
[yield-os]B cp=80010988
AB address (0xffffff7c) is out of bound at pc = 0x80000324
[src/isa/riscv32/reg.c:31 isa_reg_display] 开始打印寄存器内容:
Reg[$0]=0
Reg[ra]=0x800000ac
Reg[sp]=0xffffff74
Reg[gp]=0
Reg[tp]=0
Reg[t0]=0
Reg[t1]=0
Reg[t2]=0

Reg[s0]=0x1869f
Reg[s1]=0x41
Reg[a0]=0x41
Reg[a1]=0
Reg[a2]=0
Reg[a3]=0
Reg[a4]=0
Reg[a5]=0x186a0

Reg[a6]=0
Reg[a7]=0xffffffff
Reg[s2]=0
Reg[s3]=0
Reg[s4]=0
Reg[s5]=0
Reg[s6]=0
Reg[s7]=0

Reg[s8]=0
Reg[s9]=0
Reg[s10]=0
Reg[s11]=0
Reg[t3]=0
Reg[t4]=0
Reg[t5]=0
Reg[t6]=0
```





##### 每次调用 `yield()`会发生什么？

执行trap.s， 保存上下文，切换上下文，把PCB的*cp返回给CTE的am...hanle()， 再恢复上下文调用yield-os的f()

###### 1. 用户线程调用 `yield()`

```C
void yield() {
  asm volatile("li a7, -1; ecall");}
```

- 设置 `a7 = -1`（全1），执行 `ecall` 指令。
- CPU 触发异常（mcause = 11，自陷），硬件做：
  - `mepc ← pc`（保存返回地址，即 `ecall` 下一条指令）
  - `mstatus ← ...`（保存状态）
  - `pc ← mtvec`（跳转到异常入口 `__am_asm_trap`）





[yield-os] schedule: return cp = 80008a48
count=1
ev.event=1
[yield-os] schedule: return cp = 80010a48
count=1
ev.event=1
[yield-os] schedule: return cp = 8001099c
ev.event=1
[yield-os] schedule: return cp = 8001899c
ev.event=1



 

```
addi sp, sp, -CONTEXT_SIZE    
# 分配空间，sp = 栈指针，指向栈的当前可用最低地址（在 RISC‑V 中，栈向低地址方向增长）。
```



### OS中的上下文切换

#### RT-Thread(选做)



##### 创建上下文

对于上下文的创建, 你需要实现rt-thread-am/bsp/abstract-machine/src/context.c中的rt_hw_stack_init()函数:

```c
rt_uint8_t *rt_hw_stack_init(void *tentry, void *parameter, rt_uint8_t *stack_addr, void *texit);
```

它的功能是以stack_addr为栈底创建一个入口为tentry, 参数为parameter的上下文, 并返回这个上下文结构的指针. 此外, ==若上下文对应的内核线程从tentry返回, 则调用texit==, RT-Thread会保证代码不会从texit中返回. 需要注意:

1. 传入的stack_addr可能没有任何对齐限制, 最好将它对齐到sizeof(uintptr_t)再使用.

2. CTE的kcontext()要求不能从入口返回, 因此需要一种新的方式来支持==texit的功能. 

   一种方式是构造一个包裹函数, 让包裹函数来调用tentry, 并在tentry返回后调用texit, 然后将这个包裹函数作为kcontext()的真正入口, 不过这还要求我们将tentry, parameter和texit这三个参数传给包裹函数, 应该如何解决这个传参问题呢?

//上下文创建
rt_uint8_t *rt_hw_stack_init(void *tentry, void *parameter, 
                          rt_uint8_t *stack_addr, void *texit) {
  (uintptr_t)stack_addr                          

  assert(0);
  return NULL;
}

1.怎么实现rt_hw_stack_init函数
2.包裹函数是什么？再写一个函数吗











```
高地址  +--------------------+  ← 原 stack_addr (栈顶)
        |                    |
        |  wrapper_arg       |  ← wa
        |  (tentry, etc.)    |
        +--------------------+  ← 新 stack_top (指向参数块起始)
        |                    |
        |  Context           |  ← ctx_addr，线程的初始栈指针
        | (regs, mepc, etc.) |
低地址  +--------------------+  ← 栈底
```

##### 问题1：为什么要再放置Context？

A：`Context` 结构体用于模拟一次“异常/中断”发生时自动压栈的CPU状态。在上下文切换时，调度器会执行 `mret` 从 `Context` 中恢复 `mepc`、`mstatus`、通用寄存器等。如果不放置 `Context` 并正确初始化，就无法实现第一次切换到新线程时跳转到包装函数。

简而言之：`wrapper_arg` 是给包装函数（C语言）用的参数；`Context` 是给 `mret`（汇编）用的硬件状态。两者缺一不可。











## 在NPC中添加CSR

### Q：添加mcycle

> [!IMPORTANT]
>
> 具体地, 你需要在NPC中实现`csrrs`指令, 并添加`mcycle`. 为此, 你需要阅读RISC-V特权架构手册, 从中找到`mcycle`和`mcycleh`的编号等信息.
>
> 实现后, 尝试通过内联汇编多次读出`mcycle`寄存器, 检查其值是否自动递增. 在内联汇编中可以使用伪指令`csrr`, 其对应的真实指令即为`csrrs`.

添加一个新的测试程序csr.c

```C
#include <stdint.h>
#include <stdio.h>

int main() {
    int n = 10;
    while (n--) {
        uint32_t lo, hi;
        asm volatile ("csrr %0, 0xB00" : "=r"(lo));
        asm volatile ("csrr %0, 0xB01" : "=r"(hi));
        printf("mcycle = 0x%d %d\n", hi, lo);
        // 延时或执行一些指令，让计数器变化
        for (volatile int i = 0; i < 1000; i++);
    }
    return 0;
}
```

执行结果如下，其中`mcycle`寄存器初始值不为0，是正常结果：

![image-20260619105026853](https://cdn.jsdelivr.net/gh/Xuyang-Han/Piclist_imags@main/ysyx_imags/image-20260619105026853.jpg)



#### 为什么在 `main` 中读取初始值不是 0？

`mcycle` 是一个**物理计数器**，它从仿真开始（时间 0）就一直在计数，**不受软件控制**。从按下 `c` 后 CPU 开始运行，到执行到 `csr.c` 中的第一条 `csrr` 指令，已经过去了大量的时钟周期。这些周期消耗在：

1. **复位释放和启动**：从复位生效到释放，以及取第一条指令。
2. **AM 启动代码**：`_start` 中的栈指针设置、`.bss` 段清零等。
3. **`_trm_init` 执行**： `csr.c` 的 `main` 中读取寄存器CSR，但 `_trm_init` 已经此时执行完毕。
4. **函数调用开销**：从 `_trm_init` 跳转到 `main` 也需要周期。

所以在 `main` 中第一次读到 `0x9202`（约 3.7 万周期），完全合理。比在 `_trm_init` 开头读到的值会稍大一些，但数量级一致。



#### Q：添加学号CSR并输出学号

> [!IMPORTANT]
>
> 具体地, 你需要在NPC中实现`csrrs`指令, 并添加如下两个CSR:
>
> - `mvendorid` - 从中读出`ysyx`的ASCII码, 即`0x79737978`
> - `marchid` - 从中读出学号数字部分的十进制表示, 假设你的学号为`ysyx_22068888`, 若将读出的信息解释为整数, 则应为`22068888`, 即`0x150be98`
>
> 实现后, 可在TRM进入`main()`函数前, 通过内联汇编读出上述两个CSR的值, 然后通过`printf()`输出它们.

A：在执行所有指令之前，就进行寄存器的读取，此时电路寄存器已经完成初始化。

![image-20260619100133806](https://cdn.jsdelivr.net/gh/Xuyang-Han/Piclist_imags@main/ysyx_imags/image-20260619100133806.jpg)





## 在NPC中运行RT-Thread

### Q：在NPC中运行RT-Thread

> [!IMPORTANT]
>
> 在NPC中实现简单的异常处理机制, 并运行RT-Thread.

A：我感觉是只需要实现硬件的4条指令，因为AM和程序部分都已经实现了。看提示应该也是这样，先试试。

#### 我的bug

##### 1.如何正确找到.svh

```makefile
# 注意 -ldl 必须加上（用于 dlopen 等）
gcc: $(CAPSTONE_LIB) $(CSRC) $(VSRC)
	verilator --trace --cc --exe --build -j 0 -Wall --top top \
	-I$(NPC_HOME)/vsrc \  # 这一行找到inlcide文件
	$(NPC_HOME)/vsrc/dpi_pkg.sv \
	$(CSRC) $(VSRC) \
	-CFLAGS "-I$(CAPSTONE_INC)" \
	-LDFLAGS "-L$(CAPSTONE_DIR) -lcapstone -Wl,-rpath,$(CAPSTONE_DIR) -ldl" \

```



##### 2.函数问题

1. **解决 `VARHIDDEN`**：
   函数参数名从 `CSR_addr` 和 `wdata` 改为 `addr` 和 `data`，不再与模块端口（`CSR_addr`）和内部信号（`wdata`）冲突。

2. **解决 `NORETURN` 和 `UNDRIVEN`**：
   `csr_write` 函数不需要返回值，因此将其声明为 `function void`，不再有返回值赋值的问题。

3. **解决 `IGNOREDRETURN`**：
   由于 `csr_write` 现在是 `void`，调用它时不会返回值，所以 `EXU.v` 中的 `csr_write(CSR_addr, wdata);` 语句不再产生“忽略返回值”的警告。

   ```verilog
   function [31:0] csr_read(input [11:0] addr);
   function void csr_write(input [11:0] addr,input [31:0] data);
   ```

   





# ELF

Li指令是伪指令，需要根据elf文件转变为rv32指令，（elf文件是介于源代码和硬件语言之间）：

```shell
riscv64-unknown-elf-objdump -d -M no-aliases string-riscv32-nemu.elf要转化的elf文件
#/home/Yang/ysyx/ysyx-workbench/am-kernels/tests/cpu-tests/build/dummy-minirv-npc.elf
```

从而得到正确的汇编程序，再根据手册来进行寻找并实现指令功能。
