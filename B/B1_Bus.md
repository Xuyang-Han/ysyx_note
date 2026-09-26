# B1 总线

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

#### Q：（待定）评估单周期NPC的主频和程序性能

>  [!IMPORTANT]
>
>  在进一步修改NPC之前, 尝试通过你在预学习阶段中使用的`yosys-sta`项目来评估当前NPC的主频. 不过在评估之前, 你需要进行以下工作:
>
>  1. 先运行microbench的train规模测试, 记录其运行结束所需的周期数
>  2. 在RTL中注释通过DPI-C调用`pmem_read()`和`pmem_write()`的代码, 然后为取指和访存各自实例化一个存储器. 为了保持单周期的特性, 我们需要实例化的存储器需要当前周期就能返回读数据, 因此我们可以像寄存器堆那样通过触发器实现它. 如果你使用Verilog, 可以直接实例化`RegisterFile`模块, 当然你需要把端口正确连上. 为了统一测试结果, 我们约定实例化的存储器大小为256x32b, 即1KB, 共实例化两个这样的存储器, 总大小为2KB.
>
>  我们之所以这样修改, 是因为单周期NPC要求每个周期都完成一条指令完整的生命周期, 因此无法连接任何现实中的存储器, 只能连带两个类似寄存器堆的存储器一同评估主频. 修改后, 你就可以评估单周期NPC的主频了.
>
>  根据评估的主频和刚才记录的microbench执行的周期数, 就可以估算出将来NPC运行microbench需要多久了. 注意这并非仿真的耗时, 而是假设NPC在上述主频下运行程序的时间. 例如, yzh某个版本的NPC在`yosys-sta`项目默认提供的nangate45工艺下主频为51.491MHz, 因此可以算出microbench需要运行3.870s, 但仿真花费了19.148s.
>
>  当然, 这个估算结果其实并不准确, 而且还可以说是非常乐观的:
>
>  - 这个单周期NPC距离可流片的配置还差很远, 例如我们刚才修改存储器的时候, 其实把I/O相关的部分都忽略了
>  - 上述主频是综合后的主频, 布局布线之后引入的线延迟会进一步把主频拉低
>  - 取指单元对应的存储器因为没有写操作, 被yosys优化掉了
>  - 访存单元对应的存储器其实也远远装不下microbench. 要成功把train规模的测试运行起来, 数据需要占用1MB内存. 这个大小都已经远远超过实际处理器芯片设计中可以容纳的触发器数量了, 先不考虑EDA工具的处理时间, 光是在芯片上摆满这么多触发器, 从占用面积来估算线延迟就已经大得不得了了.
>
>  所以, 这个评估结果的参考意义其实很小, 就当作是给后续的评估练练手吧.

A：

下面是`nemu`的`ysyx-sta`，

![image-20260801105847175](https://cdn.jsdelivr.net/gh/Xuyang-Han/Piclist_imags@main/ysyx_imags/image-20260801105847175.jpg)

下面是`npc`进行`DPIC`实现访存的`ysyx-sta`，不准确，需要修改

![image-20260801152758655](https://cdn.jsdelivr.net/gh/Xuyang-Han/Piclist_imags@main/ysyx_imags/image-20260801152758655.jpg)



#### Q：支持SimpleBus的IFU

>  [!IMPORTANT]
>
>  根据上文, 让IFU支持SimpleBus协议. 对于存储器的取指部分, 你可以参考如下代码:
>
>  ```verilog
>  always @(posedge clock) begin
>  ifu_rdata <= pmem_read(ifu_raddr);
>  end
>  ```
>
>  对于LSU的数据访问部分, 目前无需修改, 我们接下来再让它支持SimpleBus.
>
>  实现后, 尝试运行一些测试程序, 同时通过查看波形来确认NPC和存储器之间的通信过程是否符合预期. 原则上来说, 总线协议对上层程序是透明的, 因此之前能成功运行的程序, 实现SimpleBus后也应同样能成功运行.
>
>  不过由于此时存储器需要经过1周期才能读出数据, 这时候NPC已经不是一个严格意义上的单周期处理器了, 而是一个简单的多周期处理器:
>
>  1. 在第1个周期, IFU发出取指请求
>  2. 在第2个周期, IFU拿到指令, 并交给后续的模块译码并执行
>
>  如果你按照前文的建议重构了NPC, 你会发现将NPC改造成多周期处理器并不难实现.

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

- `IFU`饥饿：在 `IDLE` 状态，如果同时有 `lsu_reqValid` 和 `ifu_reqValid`，你的优先级判断是 `if(lsu_reqValid) ... else if(ifu_reqValid) ...`，这样 IFU 会被饿死。如果确实需要仲裁，可以设置固定优先级或轮询。目前简单设计下可以暂时接受，但需注意。



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
....
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



### Simple Bus协议 总结

主要是完整的理解模块利用总线是怎么交互的，`master`若是`IFU`模块，那么`ifu_reqValid`（发出访存请求）和`ifu_respReady`（已经准备好接收`Mem_rdata`）,同时`MEM`模块，即`slaver`，对应的是`ifu_reqReady`（内存空闲，可以处理读/写）和`ifu_respValid`（数据已经准备好）。

需要注意的是，当有2个模块同时向`MEM`模块发出访存请求时，`MEM`模块要如何处理，在进行随机延迟测试中，我并没有加入`LSU`模块的随机延迟，因为我的`MEM`模块是依赖一种巧合来选取的，而非握手信号，所以这里并未添加成功，看后面的那个总线协议是否有相关的定义。



## 业界中广泛使用的总线 - `AMBA`总线协议

### `AXI`总线协议

`AXI4-Lite` 的 5 个通道及其握手信号如下，

#### 五个通道的握手信号

`AXI4-Lite` 的 5 个通道各自包含一组信号，**每对 `VALID` / `READY` 信号组成了该通道独立的握手机制**。当 `VALID` 和 `READY` 在同一个时钟上升沿同时为高时，表示该通道的一次数据传输成功完成。

| 通道                | 方向  | 关键信号                                                     | 说明                     |
| :------------------ | :---- | :----------------------------------------------------------- | :----------------------- |
| **读地址通道 (AR)** | 主→从 | **`ARVALID`** (主发), **`ARREADY`** (从发), `ARADDR`(主发)   | 主机发送读请求地址       |
| **读数据通道 (R)**  | 从→主 | **`RVALID`** (从发), **`RREADY`** (主发), `RDATA`(从发), `rresp`(读响应，从发) | 从机返回读出的数据及响应 |
| **写地址通道 (AW)** | 主→从 | **`AWVALID`** (主发), **`AWREADY`** (从发), `AWADDR` (主发)  | 主机发送写请求地址       |
| **写数据通道 (W)**  | 主→从 | **`WVALID`** (主发), **`WREADY`** (从发), `WDATA`(主发), `WSTRB`(主发) | 主机发送要写入的数据     |
| **写响应通道 (B)**  | 从→主 | **`BVALID`** (从发), **`BREADY`** (主发), `BRESP`(从发)      | 从机返回写操作完成状态   |

> **注意**：读事务通过 **R通道** 中的 `RRESP` 信号返回读响应，因此没有单独的“读响应通道”。

### 握手机制要点

1.  `VALID` 与 `READY` 的独立性：`VALID` 信号由数据发送方控制，表示数据有效；`READY` 信号由数据接收方控制，表示准备好接收。两者可以以任意顺序先后拉高，传输仅发生在二者同时为高的时钟周期。

2.  写事务的通道依赖：写事务需要先完成写地址（AW）和写数据（W）通道的握手，从机才会通过写响应（B）通道返回 `BVALID` 响应。

3.  读事务的通道依赖：读事务中，主机先在 AR 通道完成地址握手，从机随后通过 R 通道返回数据。



摘录于https://www.cnblogs.com/amxiang/p/16847919.html#1


下面把`AXI4_lite`的所有信号罗列出来：

| 写地址 | AW_ADDR  | ADDR_WIDTH-1 ：0     |                  |
| ------ | -------- | -------------------- | ---------------- |
|        | AW_VALID |                      |                  |
|        | AW_READY |                      |                  |
|        | AW_PORT  | 1 : 0                | 写通道保护信号   |
| 写数据 | W_DATA   | DATA_WIDTH-1 : 0     |                  |
|        | W_STRB   | (DATA_WIDTH/8)-1 : 0 | 写字节有效位控制 |
|        | W_VALID  |                      |                  |
|        | W_READY  |                      |                  |
| 写回应 | B_RESP   | 1：0                 |                  |
|        | B_VALID  |                      |                  |
|        | B_READY  |                      |                  |
| 读地址 | AR_ADDR  | ADDR_WIDTH-1 : 0     |                  |
|        | AR_VALID |                      |                  |
|        | AR_READY |                      |                  |
|        | AR_PORT  | 1：0                 | 读通道保护信号   |
| 读数据 | R_DATA   |                      |                  |
|        | R_RESP   | 1：0                 |                  |
|        | R_VALID  |                      |                  |
|        | R_READY  |                      |                  |

生成于`chatGDP`各个信号含义：

| 信号      | 谁产生    | 含义               |
| --------- | --------- | ------------------ |
| `ARVALID` | `IFU/LSU` | 我要发读地址       |
| `ARREADY` | `MEM`     | 我能接受读地址     |
| `RVALID`  | `MEM`     | 我已经准备好读数据 |
| `RREADY`  | `IFU/LSU` | 我能接受读数据     |
| `AWVALID` | `LSU`     | 我要发写地址       |
| `AWREADY` | `MEM`     | 我能接受写地址     |
| `WVALID`  | `LSU`     | 我要发写数据       |
| `WREADY`  | `MEM`     | 我能接受写数据     |
| `BVALID`  | `MEM`     | 写操作已经完成     |
| `BREADY`  | `LSU`     | 我能接受写响应     |



#### Q: 避免握手的死锁和活锁

> [!IMPORTANT]
>
> 为了避免上述问题, `AXI`标准规范对握手信号的行为添加了一些约束. 你需要RTFM找到这些约束, 并正确理解它们.
>
> 注意你务必要查阅官方手册, 如果你参考了一些来源不够正规的资料, 你将会在接入`SoC`的时候陷入痛苦的调试黑洞.

A：不同通道的具体的依赖关系：

- **读事务 (Read Transaction)**:
  - Slave **可以**等待 `ARVALID` 有效后再置 `ARREADY`.
  - Slave **必须**等待 `ARVALID` 和 `ARREADY` 都有效后，才能开始返回读数据（置 `RVALID`）。
- **写事务 (Write Transaction)**:
  - Master **不能**等待Slave的 `AWREADY` 或 `WREADY` 有效后，才去置 `AWVALID` 或 `WVALID`。
  - Slave **可以**等待 `AWVALID` 或 `WVALID`（或两者都等待）后，再置 `AWREADY` 或 `WREADY`。
  - Slave **必须**等待 `WVALID` 和 `WREADY` 都有效后，才能置 `BVALID` 以返回写响应。

总之，避免死锁和活锁的两条核心准则就是：

1. **`VALID` 不依赖于 `READY`**（避免死锁）。
2. **`VALID` 一旦有效，必须保持到握手成功**（避免活锁）。

在设计 AXI 接口时，严格遵守这两条准则，就能避免上述两种锁死情况。



### 让`NPC`支持`AXI4-Lite`

####  Q：将`IFU`和`LSU`的访存接口改造成`AXI4-Lite`

> [!IMPORTANT]
>
> 你需要在master和slave两端都正确地用握手来实现`AXI4-Lite`总线协议, 具体地:
>
> 1. 将IFU和LSU的访存接口改造成AXI4-Lite
> 2. 将存储器的`ifu`和`lsu`两个SimpleBus接口分别改造成AXI4-Lite
>
> 由于IFU只会对存储器进行读操作, 不会写入存储器, 因此可以将IFU的`AW`, `W`和`B`三个通道的握手信号均置为0. 当然, 更好的做法是在握手信号的另一端通过`assert()`确保它们一直为0.
>
> 实现后, 尝试运行一些测试程序, 同时通过查看波形来确认NPC和存储器之间的通信过程是否符合预期. 如果`NPC`在充满`LFSR`的随机延迟下仍然能正确启动`RT-Thread,` 就能大大增强你对代码的信心.

A：

（待定） 其他bug总结

`AXI4-Lite` 中的`Resp`有四个值，目前只使用了2个：

```tex
00 → 成功 OKAY
10 → SLVERR，从设备内部发生错误(暂时不用)
11 → DECERR，地址/访问路径解码错误
01 → EXOKAY，AXI4-Lite （一般不用)
```

`DECERR`：根本找不到你要访问的设备
`SLVERR`：找到了设备，但设备没能正确完成操作



##### 加入随机延迟的要点

- 模块之间的延迟最好是排队延迟，而不是同时延迟，这样可以避免一个模块单独开始，导致信号错误



##### `printf`打印好几遍

- 原因是`pmem_write()`的时机不对，应该是在判断`bValid && rReady`之后再写入,如果是在更早时刻写入，此时还没有满足写入的条件，只在一次真正的写事务(B)握手时执行。



##### 无法正确启动`yield-os`

- 原因是`ecall`指令在写入`CSR`寄存器`mepc`的时机不对：

  ```verilog
  // 单独处理ecall
      if (csr_write_en && ecall_en) begin
          csr_write(12'h341,pc); //mepc:存储当前ecall的pc
          csr_write(12'h342,32'hb); //mcause = 11
      end
  ```

  这样会导致无法正确写入，因为有总线延迟，而非一个时钟周期内可以完成

- 改为

  ```verilog
  // 单独处理ecall
  	if (csr_write_en && ecall_en && SimpleBus_reg_wen) begin
          ...
      end
  ```

  

##### 无法正确启动`RT-Thread`

使用`difftest`:

```
----> 0x80017330: 00c50623    sb       a2, 0xc(a0)
[cpp] 电路层次 RAM_addr=0x21ffffff,RAM_wdata=0x00000049,w_mask=0,op=0x7
[cpp] ISA层次  RAM_addr=0x87fffffc,RAM_wdata=0x00000049,w_mask=0,op=0x7
[npc] 指令sb M[0x87fffffc] = 0x00000049
mtrace: 写入内存 ISA M[0x87fffffc]=0x00000049 
----> 0x80017334: 00178793    addi     a5, a5, 1
----> 0x80017338: fe6794e3    bne      a5, t1, 0x80017320
----> 0x80017320: 0005c603    lbu      a2, 0(a1)
mtrace: op=0x00000006,r_mask=0x00000001,M[0x80059e41]=0x0000004e
----> 0x80017324: 00f70533    add      a0, a4, a5
----> 0x80017328: 00158593    addi     a1, a1, 1
----> 0x8001732c: 02060663    beq      a2, 0x80017358
----> 0x80017330: 00c50623    sb       a2, 0xc(a0)
[cpp] 电路层次 RAM_addr=0x21ffffff,RAM_wdata=0x0000004e,w_mask=0x1,op=0x7
[cpp] ISA层次  RAM_addr=0x87fffffd,RAM_wdata=0x0000004e,w_mask=0x1,op=0x7
[npc] 指令sb M[0x87fffffd] = 0x0000004e
mtrace: 写入内存 ISA M[0x87fffffd]=0x0000004e 
make[1]: *** [Makefile:37: run] Segmentation fault (core dumped)
make[1]: Leaving directory '/home/Yang/disk_e/ysyx/ysyx-workbench/npc'
make: *** [/home/Yang/ysyx/ysyx-workbench/abstract-machine/scripts/platform/npc.mk:38: run] Error 2
```

原因：`lhu`和`lh`指令是从当前原始的内存地址取出16bit

同时修改了`sh`的逻辑，也是类似的。

其他bug总结：

| Bug                                    | 具体表现                                                     | 根本原因/修复                                                |
| -------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 1. `pmem_write` 被重复调用             | 加入随机延迟后，串口输出出现 `TTTRRRMMM...`、甚至重复字符    | 把持续多个周期的 `Valid` 当成了一次写请求；应该只在**一次**真正的写事务握手时执行 `pmem_write()` |
| 2. 延迟状态下重新仲裁                  | LSU 请求进入延迟后，IFU 又可能抢到 MEM                       | `DELAY` 状态不能重新仲裁；必须在进入延迟时锁存 `serving_which`.<br />**简单的来说，就是多个模块内部的延迟不能同时发生，必须一个接着一个** |
| 3. 延迟期间使用实时地址                | 请求已经等待几周期，但 MEM 仍直接使用 `ifu_arAddr/lsu_arAddr` | 地址必须在 AR 握手时锁存，后面使用锁存值<br />**可以理解为，读出rdata后立刻赋值给IR或者传给EXU模块。不要延迟，值会丢失。** |
| 4. `RVALID/RDATA` 时序不正确           | 加入延迟后 DiffTest/程序执行异常                             | `RVALID` 拉高后，`RDATA` 必须保持稳定，直到 `RVALID && RREADY` |
| 5. `BVALID` 不能随意打一拍             | LSU store 在延迟后出现事务异常                               | B 通道同样必须遵守 `VALID/READY` 握手，`BVALID` 应保持到握手完成 |
| 6. 直接把 `RESP=00` 当作完成           | 对 AXI 响应含义产生混淆                                      | `RESP=00` 只是 OKAY；真正完成要看 `VALID && READY`           |
| 7. ecall指令的difftest发生pc地址慢一拍 | 异常发生时，CSR 的 `mepc` 保存的是产生异常的那条指令的 PC，而不是异常处理后的 PC，也不是 `PC+4`。 | difftest检测位置不对                                         |

------



### 总线的仲裁

#### Q：实现`AXI4-Lite`仲裁器

> [!IMPORTANT]
>
> 让存储器保留一个`AXI4-Lite`接口, 编写一个`AXI4-Lite`仲裁器, 从`IFU`和`LSU`中选择一个master与存储器通信.
>
> Hint: 仲裁器本质上也是一个状态机, 而阻塞和转发的功能本质上是通过操作握手信号来实现的.

A：当前同时有`IFU`和`LSU`请求时，优先响应且只响应一次 `LSU` 访存请求:

```
             LSU请求？
             /       \
           有         无
           ↓          ↓
    LSU还能服务？    IFU请求？
      /     \        /    \
    能       不能   有      无
    ↓         ↓     ↓       ↓
  LSU       IFU    IFU     IDLE
```

在复杂系统中的仲裁器：

> 在复杂系统中, 调度策略还需要考虑
>
> - 避免饥饿: 任一个master都能在有限次仲裁后获得访问权
> - 避免死锁: 造成的阻塞不应使整个系统出现循环等待的现象
>
> 多周期处理器还很简单, 随着系统的复杂度上升, 大家就知道厉害了



#### Q: （待定）评估NPC的主频和程序性能

> [!IMPORTANT]
>
> 实现了`AXI4-Lite`之后, `NPC`就可以外接真实的存储器了, 我们将要评估的对象是带有一个`AXI4-Lite`接口的`NPC`, 其中包含刚才实现的`AXI4-Lite`仲裁器, 而通过`DPI-C`实现的`AXI4-Lite`接口的存储器模块则不在评估范围内.
>
> 按照同样的评估方式, `yzh`另一个版本的`NPC`在`yosys-sta`项目默认提供的`nangate45`工艺下主频为`297.711MHz`, 因此可以算出`microbench`需要运行`1.394s,` 但仿真花费了`29.861s`. 可以看到仿真时间增加了, 这是因为多周期NPC的IPC小于单周期NPC, 需要花费更多的周期数来执行程序. 虽然IPC下降了, 但因为主频大幅提升, 因此程序反而执行得更快了.
>
> 别忘了, 上面的单周期`NPC`评估结果是非常乐观的, 甚至是乐观到实际中不可行的程度. 但这个多周期`NPC`的评估结果就真实多了, 至少`1MB SRAM`是可以实现的. 不过这还是和我们将要流片的配置差别很大, 毕竟`1MB SRAM`的流片成本仍然很高. 接下来我们会接入`SoC`, 使得评估结果更接近流片场景.

A：需要修改的地方

- [x] `yosys-sta`项目`makefile`需要更改
  - [x] 改成读取`npc`的`vsrc`.
  - [x] 增加`npc.sdk`(直接复制模板的)
- [ ] `npc`的`pmem_write()`，`pread_write()`需要改成`RTL`形式（暂时不会）



### 多个设备的系统

真实的计算机系统中并不仅仅只有存储器,还有其他设备

*   回顾 - 内存映射`I/O`,通过不同的内存地址来指示不同的设备
    *   在仿真环境中,通过`pmem_read`()和`pmem_write`()实现
    *   在真实硬件中,通过crossbar(有时也写作`Xbar`)实现

```text
+-------+        +-----------+        +------+        +------+
| IFU   | -----> |           |        |      | -----> | UART |   [0x1000_0000, 0x1000_0fff]
+-------+        |           |        |      |  编号0  +------+
                 | Arbiter   | -----> | Xbar |
+-------+        |           |        |      |  编号1  +------+
| LSU   | -----> |           |        |      | -----> | SRAM |   [0x8000_0000, 0x80ff_ffff]
+-------+        +-----------+        +------+        +------+
```

`Xbar`根据请求地址将请求转发给不同的下游(设备或另一个`Xbar`)

*   `Xbar`发现目标地址无设备时,resp信号返回`decerr`错误(地址译码错)
*   地址译码 = 将请求的地址转换为下游的<u>编号</u>,是`Xbar`的核心功能

`Arbiter`和`Xbar`可合并成多进多出的`Xbar`(也称`Interconnect`,总线桥等)

#### RISC-V的内存访问检查机制

- 为每段地址空间添加若干权限属性(`RWX`等)
- 在`IFU`发出取指请求前，先检查请求的地址所属的地址空间是否可执行
  - 否，则抛出Access Fault异常

RISC-V提供两种物理内存检查机制

- [x] `PMA`(Physical Memory Attribute): 地址空间在系统中==固定==
  - 通过`RTL`实现权限表，在`RTL`设计时写入
- [ ] `PMP`(Physical Memory Protection): 地址空间动态分配(如`PCI-e`等)
  - 通过`CSR`实现权限表，在系统初始化时由软件写入
  - 如果支持虚拟内存，则能实现更细粒度的权限检查功能



#### 我的设计

实现多个设备的系统，我主要采用了总线桥`crossXbar` 和 固定内存地址检测的 `PMA`，

总线桥`crossXbar`：目前主设备`master`有2个，分别是`IFU`和`LSU`；从设备有3个分别是`MEM`，`Device_slave` 以及`CLINT`;

- [x] 修改top.v以及Xbar.v里面的name，Mem和device的name分别对应
- [x] 修改Xbar_slave判断：`Xbar`发现目标地址无设备时,resp信号返回`decerr`错误(地址译码错)



##### 一. `crossXbar`仲裁器状态判断

`IFU`、`LSU`同时请求时，`LSU`优先；但如果上一轮已经给了`LSU`，则让`IFU`。

那么：

```verilog
wire grant_lsu;
wire grant_ifu;

assign grant_lsu =
    lsu_req &&
    (!ifu_req || !lsu_serve_num);

assign grant_ifu =
    ifu_req &&
    (!lsu_req || lsu_serve_num);
```

| IFU  | LSU  | `lsu_serve_num` | 结果 |
| ---- | ---- | --------------- | ---- |
| 0    | 0    | 0/1             | 无人 |
| 1    | 0    | 0/1             | IFU  |
| 0    | 1    | 0/1             | LSU  |
| 1    | 1    | 0               | LSU  |
| 1    | 1    | 1               | IFU  |



##### 二. LSU反复发起访存请求

(1) bug原因：在`LSU`模块内部，我是在`IDLE`时根据`lsu_wen`和`lsu_ren`来判断是否要发出`LSU`模块的`Valid`，导致在还没有切换`IR`前就会一直发出访存请求，于是造成`LSU`发出读/写请求的时机不对。

​	如果是 `sw`，`addi`两条指令连续执行，但是当还没有取出`addi`的`IR`时，`sw`写入成功后返回到`IDLE`，此时判断依旧成立，就转到了`DELAY`状态，之后就卡在`DELAY`状态出不来了，因为后续2条指令是`addi`，`lbu`，当需要在`lsu`时发出读请求时，此时`LSU`不在`IDLE`状态，就无法进行发出`Valid`请求.

(2) 解决思路：`IR_new`  +  `lsu_req_sent` + 锁存当前指令的相关操作数据

​	Q：如何变化`lsu_req_sent` ？

​	A：`lsu_req_sent`在当前是首次申请才为1，后续如果还是当前指令则不进行改变，执行完后清零，再等待下一条指令

```tex
时钟            N       N+1       N+2       N+3       N+4
---------------------------------------------------------
IR             sw      sw        sw        addi      lbu
lsu_wen		    1       1         1         0         0
lsu_req_sent	1       0         0         0         1
           		↑                           ↑
        	第一次提交                      新请求
```

​	Q：何时变化`IR_new`和 `lsu_req_sent` ？

​	A：状态机如下，

- 如果当前指令是`store/load`（即`is_load_store == 1`），且`ifu_handshake_done == 1` 发生过，

  那么`IR_new <= 1`，表示当前IR已经更新成功；

- 如果`lsu_handshake_done == 1`发生过，且发生在`ifu_handshake_done == 1`之后，

  那么`IR_new <= 0`, 表示该指令执行完成，不再发出`Valid`.

这样就可以实现：在`IR_new <= 1`期间完成`store/load`指令，不会发起二次访存请求

```
时钟       T0       T1       T2       T3       T4       T5       T6
          ───────────────────────────────────────────────────────────

ifu_handshake
                   1
                   │
                   ▼
IR_new      0 ─────┴──────► 1 ─────── 1 ─────── 1 ─────── 1 ─────► 0
                              │
                              │ 这期间始终是同一条 IR
                              │
lsu_req_sent
            0 ─────────────► 1 ─────── 1 ─────── 1 ─────── 1 ─────► 0
                              │
                              │
                              └── 禁止重复提交 LSU request


lsu_handshake
                                      ...等待...
                                                        1
                                                        │
                                                        ▼
                                                  当前访存完成
```

实现代码：

```verilog
  always @(posedge clk) begin
    if (rst) begin
      lsu_req_sent <= 0;
      IR_new <= 0;
    end else begin
      // 第一次提交
      if (ifu_handshake_done && is_load_store) begin
        IR_new <= 1;
        lsu_req_sent <= 1;
      end

      if (lsu_handshake_done && is_load_store) begin
        if (IR_new) begin
          IR_new <= 0;
          lsu_req_sent <= 0;
        end
      end
```

（3）`PMA`改成了在每个模块内部进行判断，不是整体进行判断。

旧的判断逻辑：`PMA`整体判断，在`LSU`内部的`IDLE`判断`Mem_addr`是否在区间内部

新的判断逻辑：在每个模块内部进行`PMA`判断，只有需要LSU模块才需要判断

bug原因：如果是整体判断的话，那么就无法保证当前的`Mem_addr`是有效的。比如，`lbu`指令下一条是指令`addi`，此时的`Mem_addr = 0x1000_0000`，此时不需要访存，所以`Mem_addr`无效，但是由于`PMA`是检测当前的`Mem_addr`是否符合区间，而不在乎是什么指令，那么当前的`lbu`在读出`rdata`后就无法进入`IDLE`，所以`lbu`无法结束，而下一条`addi`指令就无法执行。



#### Q：实现`AXI4-Lite`接口的UART功能

> [!IMPORTANT]
>
> 编写一个`AXI4-Lite`接口的slave模块, 其中包含一个设备寄存器. 当往这个设备寄存器发送写请求时, 则将写入数据的低8位作为字符, 通过`$write()`或`printf()`输出. 为了方便测试, 这个设备寄存器的地址可以设置成与之前仿真环境中串口的地址相同. 实现后, 你还需要自己编写一个`Xbar`模块, 来将这个具备UART功能的模块接入系统中.
>
> 事实上, 我们并没有完整地用`RTL`来实现一个UART, 因为`$write()`或`printf()`仍然需要依赖仿真环境来实现字符的输出. 但作为一个总线的练习, 这已经足够了, 毕竟UART的实现还需要考虑很多电气细节. 不过我们很快就会接入`SoC`, 其中包含一个真实的`UART`控制器. 现在通过这个练习来测试总线的实现, 将来接入`SoC`的时候也会更顺利.

A：最麻烦的是实现crossXbar，实现后再创建一个从设备模块`Device_slave`，在该模块内部对串口地址`SERIAL_PORT`进行单独识别，再输出wdata即可，串口只支持写。



####  Q：实现`AXI4-Lite`接口的`CLINT`

> [!IMPORTANT]
>
> [CLINT(Core Local INTerrupt controller)](https://chromitem-soc.readthedocs.io/en/latest/clint.html)是RISC-V系统中较通用的中断控制器, 是一个用于维护时钟中断和软件中断的模块. 不过目前我们的系统还不需要中断功能, 因此我们先考虑时钟相关的功能即可.  
>
> 你需要实现一个`AXI4-Lite`接口的CLINT模块, 并将其接入系统. `CLINT`包含一个只读的设备寄存器`mtime`, 它会以一定的速率增长, 最简单的实现是每周期加1. 同样地, 为了方便测试, 其地址可以设置成与之前仿真环境中时钟的地址相同.
>
> 不过, `mtime`的流逝还不能直接反映时间的流逝, 它们之间相差一个系数, 需要由软件读出后进行处理. 在真实的处理器芯片中, 一般这个系数等于CLINT模块中的时钟频率, 从而可以让软件测量出真实的时间. 不过仿真环境中没有主频的概念, 如果这个系数等于仿真速率, 我们就可以在仿真环境中通过`mtime`的流逝计算出真实时间的流逝. 具体地, 你还需要修改`IOE`的相关代码, 让`AM_TIMER_UPTIME`返回的时间接近真实时间.
>
> 最后, 你还需要考虑`mtime`寄存器的位宽. 上述手册中定义的`mtime`是64位的, 这是为了避免在实际使用中发生溢出. 但目前`NPC`是32位的, 如果我们只读出`mtime`的低32位, 在一段时间之后, `mtime`将会发生溢出, 从而使系统的时间功能发生错误. 尽管你不太容易在仿真环境中运行到`mtime`溢出的时刻, 但如果`NPC`将来运行在`500MHz`的频率下, 将大概率会发生溢出. 因此, 运行在32位`NPC`上的软件需要依次读出`mtime`的低32位和高32位, 将其组合成一个64位的值, 供上层应用使用.

A：测试文件：``am-kernel/tests/am-tests`中的`real-time clock test`测试. 

```shell
make ARCH=riscv32-nemu run mainargs=t
make ARCH=riscv32e-npc run mainargs=t
```

创建一个新的从设备模块`clint_slave`，实现`mtime`寄存器来计算时钟周期总数，当识别到读地址为`RTC_ADDR` 或 `RTC_ADDR + 4`时，在`am/.../timer.c`内部读出时钟周期总数，再除以仿真主频得到秒，再换算为微秒。

`mcycle` 与 `mtime`十分类似，都是计算时钟周期数，但是前者是`CSR`寄存器，后者属于`CLINT`，用于中断。
