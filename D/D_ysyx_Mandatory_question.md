# D1必答题

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

#### 我的bug：

##### 1.得到合适的汇编程序

Li指令是伪指令，需要根据elf文件转变为rv32指令，（elf文件是介于源代码和硬件语言之间）：

```shell
riscv64-unknown-elf-objdump -d -M no-aliases 要转化的elf文件 
#/home/Yang/ysyx/ysyx-workbench/am-kernels/tests/cpu-tests/build/dummy-minirv-npc.elf
```

从而得到正确的汇编程序，再根据手册来进行寻找并实现指令功能。

###### Li指令（此处是C扩展，并非dummy程序里面的Li指令）

![image-20260413153013702](https://cdn.jsdelivr.net/gh/Xuyang-Han/Piclist_imags@main/ysyx_imags/image-20260413153013702.jpg)

##### 2.jal指令的立即数imm

间接跳转指令,J型指令,处理imm时我发现自己没看懂手册，正确的解读应该是：指令IR[31]是imm[20]，指令IR[30：21]是imm[10：1].

![image-20260415151204699](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260415151204699.png)

![img](https://cdn.jsdelivr.net/gh/Xuyang-Han/Piclist_imags@main/ysyx_imags/941c7e32bc83e27e987b15355797b932.jpg)

指令操作：

reg数据——pc+“4”

reg地址——rd

pc_next=jal的pc+符号扩展imm



##### 3.auipc指令

AUIPC（Add Upper Immediate to pc）用于构造 pc 相对地址，采用 U 型格式。AUIPC 通过 U-立即数构造一个 32 位偏移量，最低 12 位填充 0，然后将该偏移量加到 AUIPC 指令所在的地址，并将结果存入寄存器 rd。

```
auipc rd, imm  #imm是20位
```

![image-20260413143733203](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260413143733203.png)

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









# D4必答题

从指令类型来看, minirv的指令涵盖的功能包括加法, 位拼接, 访存和跳转. 我们可以根据这些功能, 结合处理器的工作流程给NPC划分模块:

- IFU(Instruction Fetch Unit): 

- IDU(Instruction Decode Unit): 负责对当前指令进行译码, 准备执行阶段需要使用的数据和控制信号

- EXU(EXecution Unit): 负责根据控制信号控制ALU, 对数据进行计算

- LSU(Load-Store Unit): 负责根据控制信号控制存储器, 从存储器中读出数据, 或将数据写入存储器

  



DPI-C如何调用？

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

查阅手册《》，手册里写目前只有英文版本，先在目录里面搜索`DPI`，定位后一个一个看，随后找到`DPI C layer`；查看小标题和代码例子可以快速检索到想要的信息“DPI-C传参函数”

- 找到小标题`H.5 svdpi.h include file`——需要带头文件；
- 找到小标题`H.6.1 Types of formal arguments`——形式参数有关







### `char*` 和 `char**` 的区别

- **`char\*`**：指向字符的指针，通常用来表示一个字符串（如 `"hello"`）。例如 `char* str = "filename.bin";`。
- **`char\**`**：指向 `char*` 的指针，相当于字符串数组。在 `main(int argc, char** argv)` 中，`argv` 是一个数组，每个元素 `argv[i]` 是一个 `char*` 字符串，分别代表各个命令行参数。





always块内reg的真实含义

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

