# B1必答题

## Simple Bus协议

一般称主动发起通信的模块为master(主设备), 称响应通信的模块为slave(从设备)

异步总线

```
+-----+inst信号---> +-----+
| IFU | valid ---> | IDU |
+-----+ <--- ready +-----+
```

### 我的疑问

#### 1）异步总线是何时取值？

异步总线是每一个时钟周期都取指并译码一条指令吗？

答：异步总线没有时钟周期，是依靠握手信号来执行指令的；同步总线是每一个时钟周期取出一条指令并译码。

#### 2）异步总线的实现机制？

> ysyx讲义中的状态转移图：
>
> 1. 一开始处于空闲状态`idle`, 将`valid`置为无效
>    1. 如果不需要发送消息, 则一直处于`idle`状态
>    2. 如果需要发送消息, 则将`valid`置为有效, 并进入`wait_ready`状态, 等待slave就绪
> 2. 在`wait_ready`状态中, 同时检测slave的`ready`信号
>    1. 如果`ready`信号有效, 则握手成功, 返回`idle`状态
>    2. 如果`ready`信号无效, 则继续处于`wait_ready`状态等待

#### RTL分析

26030089

##### 1.  master状态

###### `idle`状态

- 无指令进入时，`IFU`是空闲，即`idle`状态，`valid`置为无效
- 有指令进入时，`IFU`将`valid`置为有效，并且进入`wait_ready`状态（疑问：这个状态需要写出来吗？）

###### `wait_ready`状态

- 收到有效`ready`信号，则变回`idle`状态
- 收到无效`ready`信号，继续保持`wait_ready`状态

##### 2.  slave信号变化

slave无状态变化，只负责发送`ready`信号

- 当前指令完成时（即每条指令结束后），置`ready`信号有效
- 当前指令未完成时，置`ready`信号无效

##### 3.  那其他模块不需要信号吗？

答：需要，但是信号类别或许不同？

- master(CPU)向slave(MEM)发送读地址`raddr`
- 下个周期slave向master回复数据`rdata`
- 上述行为每周期都发生



4）如何避免让IDU执行无效指令呢? 

答：回想一下处理器的状态机模型, 我们只要在指令无效时让处理器的状态保持不变即可. 在电路层次, 状态就是时序逻辑元件, 因此, 只需在指令无效时将时序逻辑元件的写使能置为无效即可.



### 系统总线

**访问只读存储器**

#### Q：评估单周期NPC的主频和程序性能

>  [!IMPORTANT]
>
> 在进一步修改NPC之前, 尝试通过你在预学习阶段中使用的`yosys-sta`项目来评估当前NPC的主频. 不过在评估之前, 你需要进行以下工作:
>
> 1. 先运行microbench的train规模测试, 记录其运行结束所需的周期数
> 2. 在RTL中注释通过DPI-C调用`pmem_read()`和`pmem_write()`的代码, 然后为取指和访存各自实例化一个存储器. 为了保持单周期的特性, 我们需要实例化的存储器需要当前周期就能返回读数据, 因此我们可以像寄存器堆那样通过触发器实现它. 如果你使用Verilog, 可以直接实例化`RegisterFile`模块, 当然你需要把端口正确连上. 为了统一测试结果, 我们约定实例化的存储器大小为256x32b, 即1KB, 共实例化两个这样的存储器, 总大小为2KB.
>
> 我们之所以这样修改, 是因为单周期NPC要求每个周期都完成一条指令完整的生命周期, 因此无法连接任何现实中的存储器, 只能连带两个类似寄存器堆的存储器一同评估主频. 修改后, 你就可以评估单周期NPC的主频了.
>
> 根据评估的主频和刚才记录的microbench执行的周期数, 就可以估算出将来NPC运行microbench需要多久了. 注意这并非仿真的耗时, 而是假设NPC在上述主频下运行程序的时间. 例如, yzh某个版本的NPC在`yosys-sta`项目默认提供的nangate45工艺下主频为51.491MHz, 因此可以算出microbench需要运行3.870s, 但仿真花费了19.148s.
>
> 当然, 这个估算结果其实并不准确, 而且还可以说是非常乐观的:
>
> - 这个单周期NPC距离可流片的配置还差很远, 例如我们刚才修改存储器的时候, 其实把I/O相关的部分都忽略了
> - 上述主频是综合后的主频, 布局布线之后引入的线延迟会进一步把主频拉低
> - 取指单元对应的存储器因为没有写操作, 被yosys优化掉了
> - 访存单元对应的存储器其实也远远装不下microbench. 要成功把train规模的测试运行起来, 数据需要占用1MB内存. 这个大小都已经远远超过实际处理器芯片设计中可以容纳的触发器数量了, 先不考虑EDA工具的处理时间, 光是在芯片上摆满这么多触发器, 从占用面积来估算线延迟就已经大得不得了了.
>
> 所以, 这个评估结果的参考意义其实很小, 就当作是给后续的评估练练手吧.

A：

下面是`nemu`的`ysyx-sta`，

![image-20260801105847175](https://cdn.jsdelivr.net/gh/Xuyang-Han/Piclist_imags@main/ysyx_imags/image-20260801105847175.jpg)

下面是`npc`进行DPIC实现访存的`ysyx-sta`，不准确，需要修改

![image-20260801152758655](https://cdn.jsdelivr.net/gh/Xuyang-Han/Piclist_imags@main/ysyx_imags/image-20260801152758655.jpg)



#### Q：支持SimpleBus的IFU

>  [!IMPORTANT]
>
> 根据上文, 让IFU支持SimpleBus协议. 对于存储器的取指部分, 你可以参考如下代码:
>
> ```verilog
> always @(posedge clock) begin
> ifu_rdata <= pmem_read(ifu_raddr);
> end
> ```
>
> 对于LSU的数据访问部分, 目前无需修改, 我们接下来再让它支持SimpleBus.
>
> 实现后, 尝试运行一些测试程序, 同时通过查看波形来确认NPC和存储器之间的通信过程是否符合预期. 原则上来说, 总线协议对上层程序是透明的, 因此之前能成功运行的程序, 实现SimpleBus后也应同样能成功运行.
>
> 不过由于此时存储器需要经过1周期才能读出数据, 这时候NPC已经不是一个严格意义上的单周期处理器了, 而是一个简单的多周期处理器:
>
> 1. 在第1个周期, IFU发出取指请求
> 2. 在第2个周期, IFU拿到指令, 并交给后续的模块译码并执行
>
> 如果你按照前文的建议重构了NPC, 你会发现将NPC改造成多周期处理器并不难实现.

##### （一）RTL分析

###### 1.  master状态

`idle`状态

- 无指令进入时，`IFU`是空闲，即`idle`状态，`valid`置为无效
- 有指令进入时，`IFU`将`valid`置为有效，并且进入`wait_ready`状态

`wait_ready`状态

- 收到有效`ready`信号，则变回`idle`状态
- 收到无效`ready`信号，继续保持`wait_ready`状态

###### 2.  slave信号变化

slave无状态变化，只负责发送`ready`信号

- 当前指令完成时（即每条指令结束后），置`ready`信号有效
- 当前指令未完成时，置`ready`信号无效

###### 3.  那其他模块不需要信号吗？

答：需要，但是信号类别或许不同？

- master(CPU)向slave(MEM)发送读地址`raddr`
- 下个周期slave向master回复数据`rdata`
- 上述行为每周期都发生

为了让NPC实现"等待存储器读出指令"的功能, 我们首先要让IFU得知当前处于取指令的哪个阶段, 并在不同的阶段采取不同的策略. 这种"在不同时候做不同事情"的功能, 可以通过数字电路的状态机来实现! 

具体地, 我们可以为<u>`IFU`</u>实现`idle`和`wait`两种状态:

- 在`idle`状态下, `valid`置为无效，需要将`ifu_raddr`设置为`pc`, 并跳转到`wait`状态
- 在`wait`状态下, `valid`置为有效，将`ifu_rdata`作为有效指令继续执行, 并跳转到`idle`状态



具体地:

1. 一开始处于空闲状态`idle`, 将`valid`置为无效
   1. 如果不需要发送消息, 则一直处于`idle`状态
   2. 如果需要发送消息, 则将`valid`置为有效, 并进入`wait_ready`状态, 等待slave就绪
2. 在`wait_ready`状态中, 同时检测slave的`ready`信号
   1. 如果`ready`信号有效, 则握手成功, 返回`idle`状态
   2. 如果`ready`信号无效, 则继续处于`wait_ready`状态等待



##### （二）Verilog语法问题

###### 1）如何实现master的2个状态？

`state`的实现可以利用类似于`nemu`里面的`cpu.state`,通过菜单来定义状态，在`verilog`内部使用`localparam`或是`parameter`来定义不同的状态（也可以使用`assign`，但`parameter`更好管理）。

当需要命名一个变量有非零初始值，且后续会随着不同情况变化更改，那么就命名为`localparam`或是`parameter`.电路里对应的组件类似于旋钮。

###### 2）为什么2个状态的赋值不能写在同一个`always`块内?

命名空闲状态`idle`和`wait_ready`状态时，需要放在同一个`always`块内部，如果是两个`always`块分别给同一个变量赋值不同的值，那么就会产生同一个变量被多次赋值的混乱错误。

###### 3）如何实现`valid`？

`valid`和`wait_ready`状态有相关性，`wait_ready`时，那么说明`master`在等待`slave`的信号，故此时`valid`一定有效，而且还有`ready`值来同时控制信号的有效性。

assign ifu_valid = (state == WAIT);





#### Q：让DiffTest适配多周期处理器

> [!IMPORTANT]
>
> 修改成多周期处理器后, NPC就并非每个周期都执行一条指令了. 为了让DiffTest机制可以正确工作, 你需要对检查的时机稍作调整. 为此, 你可能需要从仿真环境中读取RTL的一些状态, 来帮助你判断应该什么时候进行DiffTest的检查.

##### Difftest如何修改？

核心：cpp在发送出指令后，执行完成后，也就是下一条指令执行前，IFU变为`idle`状态之前，此时把指令送给`nemu`.

idle -> wait -> 下一条指令的idle

只要在   当前指令的 wait -> 下一条指令的idle  之间 进行difftest对比就可以

##### 另外发现寄存器脏数据问题

问题现象：

在`nemu`的`ref.c`内部`difftest_regcopy`中, 

如果是`memcpy(regs, cpu.gpr, sizeof(cpu.gpr));`就会出现报错溢出，此时打印`sizeof(cpu.gpr) = 128`,

如果是`memcpy(regs, cpu.gpr, 64);` 那么就可以全部覆盖`npc`的16个寄存器,

但是如果是`memcpy(regs, cpu.gpr, 16);`那么就会出现脏数据，只能覆盖前4个寄存器：

```shell
....
[nemu] reg[4] = 0x00007fff	[nemu] reg[5] = 0x9d687dda	[nemu] reg[6] = 0x000077ce	[nemu] reg[7] = 0x9de08000
[nemu] reg[8] = 0x000077ce	[nemu] reg[9] = 0x00000001	[nemu] reg[10] = 0x00000000	[nemu] reg[11] = 0x00000000
[nemu] reg[12] = 0x00000000	[nemu] reg[13] = 0xadb4d61e	[nemu] reg[14] = 0x000063e0	[nemu] reg[15] = 0xc3e102c4
```

原因就是`nemu`维护的是32个寄存器，无论`npc`是`riscv32`还是`riscv32e`，所以需要手动选择要使用16个寄存器还是32个寄存器：

```C
memcpy(regs, cpu.gpr, 16 * sizeof(word_t));
```



##### 发现bug

现象：`npc`的`pc`比`nemu`的`pc`慢一条，且`npc`的寄存器中内容均没有改变

结论：`npc`仅仅变化`pc`，但是不执行指令，问题应该出现在`ifu`是`wait`状态下模块`EXU`没有执行，所以`npc`的`pc`没有更新，也就导致比`nemu`的`pc`慢一条。

![image-20260809125237532](/home/Yang/.config/Typora/typora-user-images/image-20260809125237532.png)

IR始终没有变化，排查发现是pc_elc的问题问题分析

```CPP
static unsigned int pc_elc = (pc - 0x80000000);
```

- `static` 局部变量的 **初始化仅执行一次**（在程序启动时或第一次进入函数时，依赖于编译器实现）。
- 初始化表达式中的 `pc` 是函数参数，但在静态初始化时，`pc` 的值还未确定（未定义行为），通常编译器会将其视为 0，因此 `pc_elc` 被初始化为 `0 - 0x80000000` 的截断结果（由于无符号数溢出，结果为 0 或某个大数，你的平台可能是 0）。
- 后续再次调用 `pc_read` 时，**不会再执行初始化语句**，所以 `pc_elc` 永远保持第一次的垃圾值。

同样的问题也存在于你代码中的 `static unsigned int IR = 0;` 和 `static unsigned int pc = 0;`，虽然它们赋值为 0 暂时没出明显错误，但完全没有必要使用 `static`，去掉即可。

==**解决**==：整体逻辑是不太对的，需要深刻理解计算机是状态机，也就是什么时候做什么事情！按照讲义的思路重新改了一遍，问题在与控制`EXU`模块的执行，使用`wen`来控制寄存器的写入，不要在`ifu_idle`时写入寄存器，取消了`mem_ready`这个值来控制`sel`，`imm`等译码信息的赋值。



**更普遍的存储器**

#### Q：支持完整握手信号的SimpleBus协议

> [!IMPORTANT]
>
> 根据上文, 让IFU和LSU根据完整的握手信号来访问存储器.
>
> 实现后, 尝试运行一些测试程序, 同时通过查看波形来确认NPC和存储器之间的通信过程是否符合预期.
>
> 你可以对`reqReady`和`respReady`添加随机延迟, 来对总线的实现进行更充分的测试.

##### 我的bug

###### `Verilog`语法

- 不能有`input reg`
- 所有寄存器都要在`rst`时赋初值，不然会出现`x`值——我在三个模块内部都缺少

###### 状态机逻辑缺陷

① IFU 状态机

②`IFU`模块和`MEM`模块交互：`MEM`模块判断 `ifu_reqValid == 1`(表示`IFU`有内存请求) ，`IFU`模块判断`ifu_reqReady == 1` （表示`MEM`内存不忙）但是我没有考虑`IFU`有内存请求，但是内存忙的情况。

③`MEM` 状态机

- **记录为哪个模块服务**：MEM 没有记录当前服务的是 IFU 还是 LSU 请求。在 `WAIT` 状态，我同时检查 `ifu_respReady` 和 `lsu_respReady`，如果两个都为 1，无法知道应该置哪个 `reqReady`。

- **发送完`rdata`后撤销信号**：

  ```verilog
  WAIT：
  ifu_respValid <= 0;       // MEM模块 已经发送完rdata
  ```

- **(待定：未更改)`IFU`饥饿**：在 `IDLE` 状态，如果同时有 `lsu_reqValid` 和 `ifu_reqValid`，你的优先级判断是 `if(lsu_reqValid) ... else if(ifu_reqValid) ...`，这样 IFU 会被饿死。如果确实需要仲裁，可以设置固定优先级或轮询。目前简单设计下可以暂时接受，但需注意。



##### bug总结

主要问题集中在：

1. **端口类型错误**（`input reg`、缺少 `reg` 声明）。
2. **复位初始化缺失**。
3. **状态机中未覆盖所有分支，导致锁存或信号不更新**。
4. **MEM 缺少服务对象记录，导致响应混乱**。
5. **请求信号未正确撤销**。



```shell
----> 0x800000a0: 00054503    lbu      a0, 0(a0)
cpp: ISA层次 RAM_addr=0x8000049c,r_mask=0,op=0x6
mtrace: op=0x6,r_mask=0,M[0x49c]=RAM_rdata=0x54
❌ DiffTest FAIL at reg a0: NPC=0x8000049c, REF=0x00000054
```

分析：`lbu`指令取数据成功了，但是没有成功写入寄存器`a0`.



```shell
----> 0x80000034: 100007b7    lui      a5, 0x10000
[sv] imm_U = 00000000,IR = 00000413
[sv] reg写入 R[08]=00000000
❌ DiffTest FAIL at PC: NPC=0x80000004, REF=0x80000038
```

分析：`lui`指令取值的`IR`错误，没有及时更新对应的值。

```shell
----> 0x800000d0: f65ff0ef    jal      0x80000034
[sv] reg写入 R[01]=800000d4,sel = 0000000b
[sv] 时间 224,pc=0x800000d0,snpc=0x80000034,dnpc=0x80000034,IR=0xf65ff0ef
[sv] 时间 224,SimpleBus_pc_wen=0x1,en=0x1
----> 0x80000034: 100007b7    lui      a5, 0x10000
[sv] reg写入 R[08]=00000000,sel = 0000000c
[sv] 时间 230,pc=0x80000000,snpc=0x80000004,dnpc=0x80000004,IR=0x00000413
[sv] 时间 230,SimpleBus_pc_wen=0x1,en=0x1
❌ DiffTest FAIL at PC: NPC=0x80000004, REF=0x80000038
```

分析：`jal`指令是跳转得到的`dnpc`,但是`IR`不对应，上面的跳转指令执行都正确，所以可能是别的问题，查看波形`snpc<0x80000000`，和我设置更新`pc`的逻辑不对，修改：

![image-20260826162536384](https://cdn.jsdelivr.net/gh/Xuyang-Han/Piclist_imags@main/ysyx_imags/image-20260826162536384.jpg)

```verilog
if (rst == 1)
    pc <= 32'h80000000;   // 复位到内存起始地址
else if (en == 1 && snpc == 32'h00000000 ) // 原来：snpc < 32'h80000000
    pc <= 32'h80000000;
else if (en == 1 && SimpleBus_pc_wen)begin 
    pc <= snpc;
```





```shell
----> 0x800000d8: 00140413    addi     s0, s0, 1
[sv] reg写入 R[08]=8000049d,sel = 0000000c

----> 0x800000dc: 00178793    addi     a5, a5, 1
[sv] reg写入 R[15]=00000001,sel = 0000000c
----> 0x800000e0: 00f12023    sw       a5, 0(sp)
[cpp] ISA层次  RAM_addr=0x80008fb0,RAM_wdata=0x00000001,w_mask=0xf,op=0x5
当前是sw指令,M[0x00008fb0]=0x00000001
mtrace：写入内存 ISA M[0x80008fb0]=0x00000001

----> 0x800000e4: 00044503    lbu      a0, 0(s0)
cpp: ISA层次 RAM_addr=0x8000049c,  r_mask=0,  op=0x6
mtrace: op=0x00000006,r_mask=0x00000000,M[0x8000049c]=RAM_rdata=0x00000054
[sv] reg写入 R[10]=00000054, sel = 00000006
❌ DiffTest FAIL at reg a0: NPC=0x00000054, REF=0x00000052
```

分析：第1条和第3条的`s0`寄存器的内容不同，可能是`lbu`取的是`s0`的旧值，但是查看波形，发现`s0`的确是在执行`lbu`指令时就已经变成`80000089d`，查看`Mem_addr`形成，发现是自己把低`2bit`清零了.

```verilog
//内存的地址:load和store
assign Mem_addr = 
(sel==32'h4  || sel==32'h6 || sel==32'h5  || sel==32'h7  ||
 sel==32'h12 || sel==32'h13|| sel==32'h14 || sel==32'h15 ) 
    ? (Reg[rs1] + imm) & 32'hfffffffc : 32'h80000000; // 不能：低2bit清零
```

![image-20260826172010257](https://cdn.jsdelivr.net/gh/Xuyang-Han/Piclist_imags@main/ysyx_imags/image-20260826172010257.jpg)



```
----> 0x800002f4: 00f58023    sb       a5, 0(a1)
[cpp] ISA层次  RAM_addr=0x80008fc0,RAM_wdata=0x00000030,w_mask=0,op=0x7
mtrace：写入内存 ISA M[0x80008fc0]=0x00000030 
----> 0x800002f8: 000580a3    sb       zero, 1(a1)
[cpp] ISA层次  RAM_addr=0x80008fc1,RAM_wdata=0x00000000,w_mask=0x1,op=0x7
mtrace：写入内存 ISA M[0x80008fc1]=0x00000000 
----> 0x800002fc: 00100513    addi     a0, 1
[sv] reg写入 R[10]=00000001,sel = 0000000c
----> 0x80000300: 02410113    addi     sp, sp, 0x24
[sv] reg写入 R[02]=80008fb0,sel = 0000000c
----> 0x80000304: 00008067    jalr     
[sv] reg写入 R[00]=80000308,sel = 00000001
----> 0x80000200: 00a12423    sw       a0, 8(sp)
[cpp] ISA层次  RAM_addr=0x80008fb8,RAM_wdata=0x00000001,w_mask=0xf,op=0x5
当前是sw指令,M[0x00008fb8]=0x00000001
mtrace：写入内存 ISA M[0x80008fb8]=0x00000001 
----> 0x80000204: eea050e3    bge      a0, 0x800000e4
----> 0x80000208: 01010493    addi     s1, sp, 0x10
[sv] reg写入 R[09]=80008fc0,sel = 0000000c
----> 0x8000020c: 00a487b3    add      a5, s1, a0
[sv] reg写入 R[15]=80008fc1,sel = 00000003
----> 0x80000210: 00f12223    sw       a5, 4(sp)
[cpp] ISA层次  RAM_addr=0x80008fb4,RAM_wdata=0x80008fc1,w_mask=0xf,op=0x5
当前是sw指令,M[0x00008fb4]=0x80008fc1
mtrace：写入内存 ISA M[0x80008fb4]=0x80008fc1 
----> 0x80000214: 0004c503    lbu      a0, 0(s1)
cpp: ISA层次 RAM_addr=0x80008fc0,r_mask=0,op=0x6
mtrace: op=0x00000006,r_mask=0x00000000,M[0x80008fc0]=RAM_rdata=0x00000000
[sv] reg写入 R[10]=00000000,sel = 00000006
❌ DiffTest FAIL at reg a0: NPC=0x00000000, REF=0x00000030
(npc) m
请输入 ISA 内存地址 (hex): 0x80008fc0
len (dec): 5
读出内存地址 0x80008fc0 后 5 字节:
M[0x80008fc0]=0x00000000
M[0x80008fc4]=0x00000000
M[0x80008fc8]=0x00000000
M[0x80008fcc]=0x00000000
M[0x80008fd0]=0x00000000
(npc) q
退出仿真！
test list [1 item(s)]: dummy
[         dummy] PASS
```

目前是store和load的逻辑有问题，需要同时修改`difftest`访问对比内存的逻辑：
for循环对比当前的`RAM_addr`的低`2bit`清零后的，后4个字节的数组内容



#### Q：测试SimpleBus的实现

> [!IMPORTANT]
>
> 在存储器中添加随机延迟的功能, 来测试总线实现是否能在任意延迟下正确工作. 你可以按照从简单到复杂的顺序添加访存延迟:
>
> 1. 将存储器的访问延迟依次修改成5, 10, 20等
> 2. 在存储器模块中添加一个LFSR, 通过它来决定当前请求的延迟
> 3. 在IFU和LSU中也添加LFSR, 通过它来决定相应`valid`信号的延迟
>
> 如果NPC在充满LFSR的随机延迟下仍然能正确运行程序, 就能大大增强你对代码的信心.

A：在`IFU`和`MEM`中 添加随机延迟`LFSR`，但是`LSU`未添加成功，主要在与当同时请求内存时，`MEM`应该如何选择.


## Simple Bus协议 总结

主要是完整的理解模块利用总线是怎么交互的，`master`若是`IFU`模块，那么`ifu_reqValid`（发出访存请求）和`ifu_respReady`（已经准备好接收`Mem_rdata`）,同时`MEM`模块，即`slaver`，对应的是`ifu_reqReady`（内存空闲，可以处理读/写）和`ifu_respValid`（数据已经准备好）。

需要注意的是，当有2个模块同时向`MEM`模块发出访存请求时，`MEM`模块要如何处理，在进行随机延迟测试中，我并没有加入`LSU`模块的随机延迟，因为我的`MEM`模块是依赖一种巧合来选取的，而非握手信号，所以这里并未添加成功，看后面的那个总线协议是否有相关的定义。



# git基本操作

## 1）创建/切换/删除 分支

```bash
git branch               #查看所有分支/查看当前分支*
git status               #查看当前分支
git checkout 分支名       #切换到某个分支/Hash的前4位，即可创建一个对应的新分支
git checkout -b 分支名    #创建某个分支
git branch -D 11a4       #删除叫11a4的分支
```

## 2）操作分支

### a.查看分支内容

```bash
git log                          #查看新的提交信息
git status                       #查看文件有哪些变化
git diff                         #更直观的看有哪些变化
git show 11a4:semu.c >> 11a4.c   #查看11a4分支下的semu.c文件，并且存储到11a4.c文件里
```

==绝对不要==使用这个读档，==比这个存档新的所有记录都将被删除==，这意为着不能随便回到"将来"了.

```bash
git reset --hard b87c         #绝对不要使用这个读档
```

### b.给分支添加新文件

``` bash
git add (文件名)file.c         #把当前文件存到暂存区
git add .                     #把所有改动过的文件存到暂存区
git reset HEAD minirv32       #清除刚刚提交该分支的文件
git status                    #再次确认文件列表
git commit                    #把暂存区所有文件提交到永久区，会进入vim进行编辑
git commit --allow-empty      #这样允许提交没做任何修改的相同文件
git commit  文件名1 文件名2     #把暂存区的2个文件提交到永久区
git commit -m                 #把暂存区所有文件提交到永久区，不会进入vim，直接提交编辑内容
```

### c.合并分支（待定）（待测试）

```bash
git checkout master               #先切换到主分支
git merge 11a4（要合并的分支名）     #合并11a4到master，但不删除11a4
```

## 3）自己的常用git指令

```bash
git add 文件(夹)名  #把当前文件存到暂存区
git restore .      #丢弃主仓库当前所有未git commit的修改（就是红色的）
git restore --staged . #清除暂存区
git restore --staged homework/Two_way_switch/obj_dir/Vtop* #清除暂存区中某个特定的文件
git rm --cached -f nemu/tools/capstone/repo  #删除暂存区的某个文件夹
git ls-tree tracer-ysyx #查看分支tracer-ysyx的目录
git ls-tree tracer-ysyx:homework #查看tracer-ysyx分支下文件夹homework的目录
cat scripts/pdk/icsprout55.tcl  #获取该tcl文件
```



## 批量测试

```shell
make ARCH=riscv32e-npc run ALL="recursion crc32 if-else shift" -j
make ARCH=riscv32e-npc run ALL="unalign bit add hello-str bubble-sort" -j
make ARCH=riscv32e-npc run ALL="movsx leap-year add-longlong max quick-sort" -j
make ARCH=riscv32e-npc run ALL="fib shuixianhua div pascal mul-longlong" -j
make ARCH=riscv32e-npc run ALL="select-sort sum fact" -j
make ARCH=riscv32e-npc run ALL="wanshu dummy prime switch sub-longlong" -j
make ARCH=riscv32e-npc run ALL="goldbach load-store to-lower-case string mov-c" -j
make ARCH=riscv32e-npc run ALL="min3 matrix-mul mersenne" -j

make ARCH=riscv32-nemu run ALL="recursion crc32 if-else shift unalign bit add hello-str bubble-sort movsx leap-year add-longlong max quick-sort dummy" -j8
```

需要命令里面不要`-e $(ELF_FILE)`和`v`，以及关闭`sdb`，其他的无所谓：

```makefile
un: insert-arg
	$(MAKE) -C $(NPC_HOME) ISA=$(ISA) run ARGS="-t -d -w -b $(NPCFLAGS)" IMG=$(IMAGE).bin 
        #-e $(ELF_FILE)(ftrace) 
        #-v(vga) -t(itrace & mtrace) -w(wtrace) -b(no sdb) -d(difftest)
```

最大可以一次并行`-j`14个测试文件，但是15个会闪退，可能是内存上限，可以依靠`-j4`来规定最大并行数量。






## 业界中广泛使用的总线 - `AMBA`总线协议
