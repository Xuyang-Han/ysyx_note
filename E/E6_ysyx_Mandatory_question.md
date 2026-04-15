# E6_PA1

## 开天辟地的篇章

### 最简单的计算机

#### Q：尝试理解计算机如何计算

> [!IMPORTANT]
>
> 在看到上述例子之前, 你可能会觉得指令是一个既神秘又难以理解的概念. 不过当你看到对应的C代码时, 你就会发现指令做的事情竟然这么简单! 而且看上去还有点蠢, 你随手写一个for循环都要比这段C代码看上去更高级.
>
> 不过你也不妨站在计算机的角度来理解一下, 计算机究竟是怎么通过这种既简单又笨拙的方式来计算`1+2+...+100`的. 这种理解会使你建立"程序如何在计算机上运行"的最初原的认识.

A：从指令里面读出要累加的数a，再读下一条指令进行a+1存入某个寄存器，再读下一条指令sum=a+(a+1)，再反复前三步，直到达到a+1=101，则停止累加，输出sum.



### 重新认识程序: 程序是个状态机

#### Q：从状态机视角理解程序运行

> [!IMPORTANT]
>
> 以上一小节中`1+2+...+100`的指令序列为例, 尝试画出这个程序的状态机.
>
> 这个程序比较简单, 需要更新的状态只包括`PC`和`r1`, `r2`这两个寄存器, 因此我们用一个三元组`(PC, r1, r2)`就可以表示程序的所有状态, 而无需画出内存的具体状态. 初始状态是`(0, x, x)`, 此处的`x`表示未初始化. 程序`PC=0`处的指令是`mov r1, 0`, 执行完之后`PC`会指向下一条指令, 因此下一个状态是`(1, 0, x)`. 如此类推, 我们可以画出执行前3条指令的状态转移过程:
>
> ```text
> (0, x, x) -> (1, 0, x) -> (2, 0, 0) -> (3, 0, 1)
> ```
>
> 请你尝试继续画出这个状态机, 其中程序中的循环只需要画出前两次循环和最后两次循环即可.

下列指令序列可以计算`1+2+...+100`, 

```
// PC: instruction    | // label: statement
0: mov  r1, 0         |  pc0: r1 = 0;
1: mov  r2, 0         |  pc1: r2 = 0;
2: addi r2, r2, 1     |  pc2: r2 = r2 + 1;
3: add  r1, r1, r2    |  pc3: r1 = r1 + r2;
4: blt  r2, 100, 2    |  pc4: if (r2 < 100) goto pc2;  //branch if less than
5: jmp 5              |  pc5: goto pc5;
```

对应的状态机：

```
(0, x, x) -> (1, 0, x) -> (2, 0, 0) -> (3, 0, 1) -> (4, 1, 1) 
-> (2, 0, 2) -> (3, 3, 2) -> ...
-> (2, 4851, 99) -> (3, 4950, 99) -> (2, 4950, 100) -> (3, 5050, 100)
-> (5, 5050, 100)
```



## RTFSC

### 框架代码初探

[这个页面](https://ysyx.oscc.cc/docs/ics-pa/nemu-isa-api.html)对ysyx-workbench/nemu的内容进行了说明，可以以后查阅。



### 运行第一个客户程序

#### Q：理解框架代码

> [!IMPORTANT]
>
> 你需要结合上述文字理解NEMU的框架代码.
>
> 如果你不知道"怎么才算是看懂了框架代码", 你可以先尝试进行后面的任务. 如果发现不知道如何下手, 再回来仔细阅读这一页面. 理解框架代码是一个螺旋上升的过程, 不同的阶段有不同的重点. 你不必因为看不懂某些细节而感到沮丧, 更不要试图一次把所有代码全部看明白.

#### Q：优美地退出

> [!IMPORTANT]
>
> 为了测试大家是否已经理解框架代码, 我们给大家设置一个练习: 如果在运行NEMU之后直接键入`q`退出, 你会发现终端输出了一些错误信息. 请分析这个错误信息是什么原因造成的, 然后尝试在NEMU中修复它.

```C
static int cmd_q(char *args) {
  nemu_state.state=NEMU_QUIT;
  return -1;//返回后直接退出程序
}
```

##### 1）为什么退出会报错？

A：因为没有正确设置state，仅仅返回-1是无法正常退出的，必须满足：

```c
int good =
    (nemu_state.state == NEMU_END && nemu_state.halt_ret == 0) ||
    (nemu_state.state == NEMU_QUIT);
```

也就是，退出状态，或是正常中止状态。



##### 2）退出的流程是什么？

A：识别用户命令过程：

`nemu-main.c`  调用  ` init.c`, 并且 返回 `is_exit_status_bad();`对状态进行检测：返回0正常退出，返回1报错退出  —-> 

`init.c` 调用 `sdb_mainloop()`  —->   

  `sdb_mainloop()` 识别用户命令，再执行相应的命令函数  `cmd_q()` ，只要返回>0，就会循环识别用户输入的命令 ，返回负数就会跳出循环 —->  

`cmd_q()` 设置`nemu_state.state=NEMU_QUIT`,并且`return -1`，一层层向上返回到`nemu-main.c`  ，再返回 `is_exit_status_bad();`，检测到状态为QUIT，正常退出。



延伸知识点：

`init.c` 可以看作是 NEMU 模拟器启动后选择执行路径的入口点，它决定了是直接运行程序还是进入调试器等待命令。



## 基础设施: 简易调试器

### Q: 实现单步执行, 打印寄存器, 扫描内存

> [!IMPORTANT]
>
> 熟悉了NEMU的框架之后, 这些功能实现起来都很简单, 同时我们对输出的格式不作硬性规定, 就当做是熟悉GNU/Linux编程的一次练习吧.
>
> NEMU默认会把单步执行的指令打印出来(这里面埋了一些坑, 你需要RTFSC看看指令是在哪里被打印的), 这样你就可以验证单步执行的效果了.
>
> 不知道如何下手? 嗯, 看来你需要再阅读一遍[RTFSC小节](https://ysyx.oscc.cc/docs/ics-pa/1.3.html)的内容了. 如果你已经忘记了某些注意事项, 重新去阅读一遍也是应该的.

A：这三条指令识别的内核是一样的，即把输入命令拆解成不同的部分，然后识别进行相应的操作。

#### 1)单步执行：

##### 执行原理：

```bash
si num #执行num步
si     #默认执行1步
```

把输入的字符串，除去命令外后余下字符串赋值给变量`args`，若`args==NULL`，则默认执行1步；若`args!=NULL`，则转化为无符号数`num`，传入`cpu_exec(num)`.

##### 命令 c 执行过程：

`cmd_si()`  —->     `cpu_exec(-1)`      —->   `execute(-1)`       

`execute(n)`   会进行for循环，由于`n=-1`所以会一直执行到直到程序状态改为`RUNNING`，然后自行跳出循环。

调用  `exec_once(&s, cpu.pc)` ，传入 s 的地址和当前 PC 值。这个函数会模拟执行一条指令，并更新 s 中的信息（如新的 PC 等）：

 `exec_once(&s, cpu.pc)`  —->   ` isa_exec_once(s);`   —–>   `decode_exec(Decode *s)`  

```C
static void execute(uint64_t n) {//传入无符号数-1
  Decode s;
  for (;n > 0; n --) {
    exec_once(&s, cpu.pc);//传入 s 的地址和当前 PC 值
    g_nr_guest_inst ++; //计算全局已执行的客户指令条数
    trace_and_difftest(&s, cpu.pc); //跟踪
    if (nemu_state.state != NEMU_RUNNING) break;
    IFDEF(CONFIG_DEVICE, device_update());//疑问：这个设备更新是state吗
  }
}
```

遇到指令ebreak，会把state改为stop,

```C
#define INSTPAT_MATCH(s, name, type, ... /* execute body */ ) { \
  ...
  INSTPAT("0000000 00001 00000 000 00000 11100 11", ebreak , N, NEMUTRAP(s->pc, R(10))); // R(10) is $a0
  ...
  return 0;
                                                               
#define NEMUTRAP(thispc, code) set_nemu_state(NEMU_END, thispc, code)       
//将模拟器的全局状态 nemu_state.state 设置为 NEMU_END，并记录结束时的 PC 地址（thispc）和返回值（code，通常来自 a0 寄存器）                                                               
```

==疑问：和谁对比？也没有写啊？==我的疑问：写入的指令存到哪里去了，然后就能直接对比？

之后再返回`execute(n)` 后，判断状态不为`RUNNING`，就会跳出循环，然后 `execute(-1)` 返回 `cpu_exec(-1)` 返回`cmd_si()`…..，最后返回0,由于不是负数，所以会循环执行用户交互界面。



#### 2）打印寄存器：

```bash
info r
```

由于寄存器只有32个，所以并没有指定寄存器名称，调用` isa_reg_display() `，循环打印寄存器`cpu.gpr[i]`即可.

#### 3）打印内存：

```bash
x 10 0x80000000  #
x 10 0x100000	 # 地址会越界  范围为[0x80000000, 0x87ffffff]
```

==疑问：既然会越界，为什么讲义会提到0x100000这个范围？应该是场景不对==

和打印寄存器原理一样，识别2个变量，所读内存块数量，还有起始内存地址。

可以利用`strtok`函数提取再赋值给不同类型的变量：

```C
char *n_str = strtok(args, " ");//args提取第二个命令词
char *addr_str = strtok(NULL, " ");//args提取第三个命令词
```

判断不为空，再进行进一步赋值。



再利用`strtok`函数提取再赋值给不同类型的变量：

```C
uint32_t n=strtoull(n_str,NULL,10);   //取出指定 十进制  字节数n
paddr_t addr=strtoull(addr_str,NULL,16);//取出指定 十六进制 地址addr
```

`paddr_t`是NEMU内部定义内存地址的变量。



## 表达式求值

### Q：实现算术表达式的词法分析

> [!IMPORTANT]
>
> 你需要完成以下的内容:
>
> - 为算术表达式中的各种token类型添加规则, 你需要注意C语言字符串中转义字符的存在和正则表达式中元字符的功能.
> - 在成功识别出token后, 将token的信息依次记录到`tokens`数组中.

A：根据KISS原则，我先进行了十进制的加减乘除 的表达式解析，然后又加上了对寄存器名称的解析。



### Q：实现算术表达式的递归求值

> [!IMPORTANT]
>
> 由于ICS不是算法课, 我们已经把递归求值的思路和框架都列出来了. 你需要做的是理解这一思路, 然后在框架中填充相应的内容. 实现表达式求值的功能之后, `p`命令也就不难实现了.

A：这块逻辑有点复杂，主要要是解析带多层括号的表达式，需要使用递归，具体看代码吧。

需要能处理带负数的表达式



### Q：实现表达式生成器

> [!IMPORTANT]
>
> 根据上文内容, 实现表达式生成器. 实现后, 就可以用来生成表达式求值的测试用例了.
>
> ```text
> ./gen-expr 10000 > input
> ```
>
> 将会生成10000个测试用例到`input`文件中, 其中每行为一个测试用例, 其格式为
>
> ```text
> 结果 表达式
> ```
>
> 再稍微改造一下NEMU的`main()`函数, 让其读入`input`文件中的测试表达式后, 直接调用`expr()`, 并与结果进行比较. 为了容纳长表达式的求值, 你还需要对`tokens`数组的大小进行修改.
>
> 随着你的程序通过越来越多的测试, 你会对你的代码越来越有信心.

```bash
make            # 编译
make n=10 run   # 连续生成10条比表达式
```



## 监视点

### Q：实现监视点池的管理

> [!IMPORTANT]
>
> 为了使用监视点池, 你需要编写以下两个函数(你可以根据你的需要修改函数的参数和返回值):
>
> ```c
> WP* new_wp();
> void free_wp(WP *wp);
> ```
>
> 其中`new_wp()`从`free_`链表中返回一个空闲的监视点结构, `free_wp()`将`wp`归还到`free_`链表中, 这两个函数会作为监视点池的接口被其它函数调用. 需要注意的是, 调用`new_wp()`时可能会出现没有空闲监视点结构的情况, 为了简单起见, 此时可以通过`assert(0)`马上终止程序. 框架代码中定义了32个监视点结构, 一般情况下应该足够使用, 如果你需要更多的监视点结构, 你可以修改`NR_WP`宏的值.
>
> 这两个函数里面都需要执行一些链表插入, 删除的操作, 对链表操作不熟悉的同学来说, 这可以作为一次链表的练习.

A：只设置了一个监视点，所以只在监视点池的第一处进行操作，`IN=0`恒成立，也就在free时没有判断监视点编号的情况。



### Q：实现监视点

> [!IMPORTANT]
>
> 你需要实现上文描述的监视点相关功能, 实现了表达式求值之后, 监视点实现的重点就落在了链表操作上.
>
> 由于监视点的功能需要在`cpu_exec()`的每次循环中都进行检查, 这会对NEMU的性能带来较为明显的开销. 我们可以把监视点的检查放在`trace_and_difftest()`中, 并用一个新的宏 `CONFIG_WATCHPOINT`把检查监视点的代码包起来; 然后在`nemu/Kconfig`中为监视点添加一个开关选项, 最后通过menuconfig打开这个选项, 从而激活监视点的功能. 当你不需要使用监视点时, 可以在menuconfig中关闭这个开关选项来提高NEMU的性能.
>
> 在同一时刻触发两个以上的监视点也是有可能的, 你可以自由决定如何处理这些特殊情况, 我们对此不作硬性规定.

A：说实话，一开始感觉这个监视点过程有些复杂，但是理清楚就好多了。

1. `make menuconfig`  打开  监视点设置检测  –>  
2. `watch   expr`(表达式，不要纯数字，不会触发监视点，因为数字式不变化)     –>
3. `set  sp(寄存器名称)    num(要赋值的数值) `     –>
4. `si  num   /   c`    ，此时就会触发监视点.

上面过程的原理：

打开监视点功能，只要所监视的寄存器/表达式有变化，也就是旧值和新值不同，就会触发监视点。（个人理解：带有监视寄存器的表达式，其实就是监视寄存器，因为常数部分不会变化）



#### 延伸问题：宏是如何管理监视点的开启和关闭的？

设置了宏`CONFIG_WATCHPOINT`，并且包裹了监视点的代码，那NEMU是如何识别这个宏是否被启动的，只有在Kconfig里有`config  WATCHPOINT` ？

A：Kconfig 提供了 用户 设置nemu功能的交互平台，如果修改了Kconfig里的`config  WATCHPOINT` ，那么就会生成一个.config，里面会记录`CONFIG_WATCHPOINT=y`, NEMU 会读取 `.config` 文件，并自动生成一个 C 语言头文件，再将所有 `=y` 的选项定义为宏。例如：

```C
#define CONFIG_WATCHPOINT 1
```

之后，如果宏`CONFIG_WATCHPOINT` 被定义，那么就会执行该宏所包含的代码，否则忽略该宏。



## 如何阅读手册

### Q：必答题

> [!IMPORTANT]
>
> - riscv32
>   - riscv32有哪几种指令格式?
>   - LUI指令的行为是什么?
>   - mstatus寄存器的结构是怎么样的?

R型，I型，S型，U型，B型（基于S型），J型（基于U型）

LUI指令——构造32bit常数：

```
lui rd,imm 
#把imm放到高20bit，低位补零，并存入rd寄存器
```



> [!IMPORTANT]
>
> shell命令 
>
> - 完成PA1的内容之后, `nemu/`目录下的所有.c和.h和文件总共有多少行代码? 
> - 你是使用什么命令得到这个结果的? 和框架代码相比, 你在PA1中编写了多少行代码? 
> - 除去空行之外, `nemu/`目录下的所有`.c`和`.h`文件总共有多少行代码?
>
> (Hint: 目前`pa0`分支中记录的正好是做PA1之前的状态, 思考一下应该如何回到"过去"?) 
>
> 你可以把这条命令写入`Makefile`中, 随着实验进度的推进, 你可以很方便地统计工程的代码行数, 例如敲入`make count`就会自动运行统计代码行数的命令. 

A：

```bash
find . | grep '\.c$\|\.h$' | xargs wc -l
find . -name '*.[ch]' | xargs grep  '^' | wc -l
find . -name '*.[ch]' | xargs grep -v '^$' | wc -l
```

 命令`xargs wc -l`的含义？

- **`xargs`**：从标准输入读取数据，并将其作为参数传递给指定的命令。
- **`wc -l`**：统计每个文件的行数（`-l` 选项表示只显示行数）。当多个文件作为参数时，`wc -l` 会输出每个文件的行数，并在最后给出总和。

- **`[ch]`**：匹配单个字符 `c` 或 `h`。

`wc -l` 是一个 Linux/Unix 命令行命令，用于统计输入中的**行数**。

- `wc` 是 **word count**（单词计数）的缩写，是一个用于统计文件或标准输入中各种信息的工具。
- `-l` 选项（小写字母 L）告诉 `wc` 只统计**行数**（lines），而不是默认的单词数或字节数。



> [!IMPORTANT]
>
> RTFM 打开`nemu/scripters/build.mk`文件, 你会在`CFLAGS`变量中看到gcc的一些编译选项. 请解释gcc中的`-Wall`和`-Werror`有什么作用? 为什么要使用`-Wall`和`-Werror`?

A：

- `-Wall`：启用大多数常见的警告信息，帮助开发者发现潜在问题。
- `-Werror`：将所有警告当作错误处理，导致编译失败，强制开发者修复警告。
- 为什么要使用：提高代码质量，避免潜在错误，遵循严谨的编程风格，特别是在教学或项目开发中确保代码规范。







```bash
Yang@yyl-Ubuntu:~/ysyx/ysyx-workbench/nemu$ gdb /home/Yang/ysyx/ysyx-workbench/nemu/build/riscv32-nemu-interpreter
run
....
(gdb) bt
No stack.
(gdb) run
Starting program: /home/Yang/ysyx/ysyx-workbench/nemu/build/riscv32-nemu-interpreter 
Downloading separate debug info for system-supplied DSO at 0x7ffff7fc3000
Downloading separate debug info for /lib/x86_64-linux-gnu/libreadline.so.8                                                                                                      
[Thread debugging using libthread_db enabled]                                                                                                                                   
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
Downloading separate debug info for /lib/x86_64-linux-gnu/libtinfo.so.6
warning: could not find '.gnu_debugaltlink' file for /lib/x86_64-linux-gnu/libtinfo.so.6                                                                                        
Downloading separate debug info for /lib/x86_64-linux-gnu/libtinfo.so.6
[src/utils/log.c:30 init_log] Log is written to stdout                                                                                                                          
[src/utils/log.c:30 init_log] Log is written to stdout
[src/memory/paddr.c:50 init_mem] physical memory area [0x80000000, 0x87ffffff]
[src/memory/paddr.c:50 init_mem] physical memory area [0x80000000, 0x87ffffff]
[src/monitor/monitor.c:53 load_img] No image is given. Use the default build-in image.
[src/monitor/monitor.c:53 load_img] No image is given. Use the default build-in image.

Program received signal SIGSEGV, Segmentation fault.
0x0000555555559d8c in init_wp_pool ()

```

