# E5 从RTL代码到可流片版图

## RTL仿真(Simulation) - 功能验证

### RTL仿真的基本原理

本质是通过软件程序来模拟硬件电路的行为. 因此, 要实现RTL仿真, 就是要考虑如何用C程序的状态机实现数字电路的状态机.

### STFW + RTFM搭建Verilator仿真环境

#### Q：安装Verilator

> [!IMPORTANT]
>
> 在官网中找到安装Verilator的步骤, 然后按照从git安装的相应步骤进行操作. 我们之所以不采用`apt-get`安装, 是因为其版本较老. 我们推荐安装`stable`版本. 为此, 你还需要进行一些简单的git操作, 如果你对此感到生疏, 你可能需要寻找一些git教程来学习. 另外, 你最好在`ysyx-workbench/`之外的目录进行这一操作, 否则git将会追踪到Verilator的源代码, 从而占用不必要的磁盘空间.
>
> 安装成功后, 运行以下命令来检查安装是否成功, 以及版本是否正确.
>
> ```bash
> verilator --version
> ```

git方式安装：

```bash
home:~/verilator$ verilator --version
Verilator 5.045 devel rev v5.044-189-ge0c626e48
```



#### Q：运行示例

> [!IMPORTANT]
>
> Verilator手册中包含一个C++的示例, 你需要在手册中找到这个示例, 然后按照示例的步骤进行操作.
>
> 你已经学习过C语言, 为了使用Verilator, 你并不需要了解复杂的C++语法, 你只需要了解一些类(class)的基本使用方法就可以了. 单从这一点来看, 网上的很多资料都可以满足你的需求.

按照C++方式编译运行：

```bash
home:~/E5/test_our$ verilator --cc --exe --build -j 0 -Wall sim_mai
n.cpp our.v                                                                   
make: Entering directory '/home/Yang/E5/test_our/obj_dir'                     
g++  -I.  -MMD -I/home/Yang/snap/verilator/include -I/home/Yang/snap/verilator
/include/vltstd -DVERILATOR=1 -DVM_COVER
...（编译信息）
home:~/E5/test_our$ ./obj_dir/Vour
Hello World
- our.v:2: Verilog $finish
```

按照SC方式编译运行（硬件仿真程序）：

```bash
home:~/test_our_sc$ verilator --sc --exe -Wall sc_main.cpp our.v
- V e r i l a t i o n   R e p o r t: Verilator 5.045 devel rev v5.044-189-ge0c
626e48                                                                        
- Verilator: Built from 0.027 MB sources in 2 modules, into 0.025 MB in 5 C++ 
files needing 0.000 MB                                                        
- Verilator: Walltime 0.004 s (elab=0.000, cvt=0.002, bld=0.000); cpu 0.004 s 
on 1 threads; allocated 19.438 MB 

home:~/test_our_sc$ make -j -C obj_dir -f Vour.mk Vour          
make: Entering directory '/home/Yang/E5/test_our_sc/obj_dir'
...
make: Leaving directory '/home/Yang/E5/test_our_sc/obj_dir'

home:~/test_our_sc$ obj_dir/Vour
        SystemC 2.3.4-Accellera --- Apr 22 2024 14:54:19
        Copyright (c) 1996-2022 by all Contributors,
        ALL RIGHTS RESERVED
Hello World
- our.v:4: Verilog $finish
```

### 示例: 双控开关(组合逻辑电路)

#### Q：对双控开关模块进行仿真

> [!IMPORTANT]
>
> 尝试在Verilator中对双控开关模块进行仿真. 由于顶层模块名与手册中的示例有所不同, 你还需要对C++文件进行一些相应的修改. 此外, 这个项目没有指示仿真结束的语句, 为了退出仿真, 你需要键入`Ctrl+C`.

更改cpp后，使用Makefile（C++方式）来编译运行，得到可运行文件Vtop，我的bash是Ctrl+Z暂停。

运行成功，提交和   “必答题：一键仿真”  是同一个

测试文件夹：~/ysyx-workbench/npc

Two_way_switch内有测试.cpp复制到npc主目录

已经提交至：tracer-ysyx/npc

```bash
a = 1, b = 1, f = 0
a = 0, b = 1, f = 1
...
```



### 打印并查看波形

#### Q：生成波形并查看

> [!IMPORTANT]
>
> Verilator手册中已经介绍了波形生成的方法, 你需要阅读手册找到相关内容, 然后按照手册中的步骤生成波形文件, 并通过
>
> ```bash
> sudo apt-get install gtkwave
> ```
>
> 安装GTKWave来查看波形.

##### 我的bug：

###### 如何生成.vcd?

这个问题想了很久，我的思路是直接使用Verilator生成.cpp，再由.cpp生成.vdc，因为这就是Verilator的作用，但其实不是这样的，cpp是需要手写的，Verilator可以生成vcd，前提是已经有对应的.v和.cpp

我试着看官网文档和例子的Makefile，还是有点晕，大部分的名称和写法感觉很陌生，加上对C++不如C熟，最后百度了一下，找到一份运行成功的ysyx笔记，看了博客里的代码我才知道自己想歪了，而且写的.cpp也不对。我按照这篇博客修改了自己的代码（可以说没怎么改，因为电路很简单，所以主体很一致），参考笔记来源链接：https://blog.csdn.net/Xuanx007/article/details/146460440

```bash
verilator --trace --cc --exe --build -j 0 -Wall sim_main_trace.cpp top.v 
./obj_dir/Vtop #运行可执行文件，以支持显示波形
gtkwave wave.vcd
```

![image-20260214214317469](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260214214317469.png)



#### Q：一键仿真

> [!IMPORTANT]
>
> 反复键入编译运行的命令是很不方便的, 尝试为`npc/Makefile`编写规则`sim`, 实现一键仿真, 如键入`make sim`即可进行上述仿真.

运行成功，为了区分功能，sim规则用于提交git跟踪，run用于编译运行，并且调用规则sim，结果和上2道题一样，已经git commit

测试文件夹：~/ysyx-workbench/npc

Two_way_switch内有测试.cpp复制到npc主目录

Makefile位置：~/npc/Makefile



==<u>注意</u>==，如果是完成了NVBoard，下载了oss-cad-suite，再返回来做这个实验，一定要注意.bashrc里面的路径设置：

```bash
export VERILATOR_ROOT=/home/Yang/snap/verilator
export PATH=$VERILATOR_ROOT/bin:$PATH
# export PATH="/home/Yang/snap/oss-cad-suite/bin:$PATH"
export PATH="$PATH:/home/Yang/snap/oss-cad-suite/bin"
```

==<u>`$PATH`必须在最前面，因为在oss-cad-suite里面也有Verilator，会优先访问先设置的Verilator路径.</u>==

如果不这么设置，就会无法因为无法找到正确的Verilator路径而无法正确编译运行。



#### Q：运行NVBoard示例

> [!IMPORTANT]
>
> 阅读NVBoard项目的介绍, 尝试运行NVBoard项目中提供的示例.

除了按照README的操作，还有几个步骤需要注意，之前必答题其实也遇到过，但是自己一直没有在意，这里栽根头了，

安装软件：

```bash
sudo apt update    #安装任何软件前，都要更新软件包
sudo apt-get install libsdl2-dev libsdl2-image-dev libsdl2-ttf-dev
```

若修改了 ~/.bashrc

```bash
source ~/.bashrc  # 需要运行一遍
```

最后运行成功加载出来了

![image-20260216114202713](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260216114202713.png)



#### Q：在NVBoard上实现双控开关

> [!IMPORTANT]
>
> 阅读NVBoard项目的说明, 然后仿照该示例下的C++文件和Makefile, 修改你的C++文件, 为双控开关的输入输出分配引脚, 并修改`npc/Makefile`, 使其连接到NVBoard上的开关和LED灯

##### A：example_tws（包含npc/Makefile）的代码

代码夹名称：example_tws

已经提交到分支tracer-ysyx 下的  homework/example_tws

Makefile的运行命令，加了git跟踪，`make run` 启动。

![image-20260216173939079](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260216173939079.png)

![image-20260216174002062](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260216174002062.png)



#### Q：将流水灯接入NVBoard

> [!IMPORTANT]
>
> 编写流水灯模块, 然后接入NVBoard并分配引脚. 如果你的实现正确, 你将看到灯从右端往左端依次亮起并熄灭.

运行结果符合题目：

##### A：example_running_light（包含npc/Makefile）的代码

代码夹名称：example_running_light

已经提交到分支tracer-ysyx 下的  homework/example_running_light

Makefile的运行命令，为了git跟踪，由`make run` 改为了 `make sim` 

![image-20260216215810936](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260216215810936.png)



### 静态代码检查

#### Q：通过Verilator进行静态代码检查

> [!IMPORTANT]
>
> 尝试使用Verilator检查你的代码, 并尽最大可能修复所有警告.

我们建议你将来总是开启Verilator的静态代码检查功能. 一方面, 这有助于你养成良好的编码习惯, 从而编写出更高质量的代码. 另一方面, 尽早发现代码中潜在问题, 也有利于节省不必要的调试工作: 随着代码规模的增加, 将来你很可能因为某个信号的位宽错误而调试好几天, 而Verilator的警告可以让你马上注意到这个问题, 从而轻松地排除相应的错误.

```bash
# 检查命令
verilator --lint-only -Wall <我的顶层模块文件名>.v
```



## Verilog的仿真行为和编码风格

Verilog语言在仿真过程中的语义是由[Verilog标准手册](http://staff.ustc.edu.cn/~songch/download/IEEE.1364-2005.pdf)定义的。

截取自讲义：  

“ 总结: Verilog语言成分的语义都和事件有关联:

- 仿真的过程就是按某种正确的顺序处理这些事件的过程
  - 包括更新事件和求值事件
- 在处理事件的过程中, 电路中对象的状态会发生改变
- 我们期望这种改变符合电路的预期行为, 从而实现对电路行为的仿真”



### Verilog代码的执行

Verilog语言用来从各个抽象层次来描述硬件的行为(包括系统级, 行为级, RTL级, 门级, 晶体管级).

### 赋值操作的事件调度

#### Q：用事件模型分析Verilog代码的行为

> [!IMPORTANT]
>
> 考虑以下代码, 假设在`t`时刻有`a = 1`, `b = 2`, `c = 3`, `d = 4`, `e = 5`. 尝试利用 事件模型分析在`t+1`时刻, 变量的值各为多少.
>
> ```verilog
> always @(posedge clk) begin
>   b  = a;
>   c <= b;
>   d  = c;
>   e <= d;
>   a  = e;
> end
> ```

阻塞赋值是计算右侧，立刻更新左侧；非阻塞赋值是计算右侧，最结束后赋值左侧。

所以结果如下：

```verilog
b=a=1  # b 立刻被赋值
c <=b=1 # 未结束，最后赋值
d=c=3    #d立刻被赋值
e <=d=c=3
a=e=5 #a 立刻被赋值
#结束后，c=b=1，e=c=3
```



### 事件处理顺序

截取讲义，4条规则：

![image-20260217210259477](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260217210259477.png)



### 仿真器和仿真程序

#### Q：理解Verilator生成的仿真程序的行为

> [!IMPORTANT]
>
> 用Verilator编译流水灯电路, 尝试理解生成的C++代码的行为.

A：生成的C++代码就是根据verilog语言的事件执行顺序来进行设计生成的。根据上一小节提到的四条规则，然后推出事件执行顺序，最后根据顺序生成相应的C++代码。



### 数据竞争

对于无法使用那4种规则的情况，当出现多个激活事件的处理，它们处理顺序是任意的，即多个`always`语句块，这时有些像C语言里面的未定义情况，执行结果是无法预测的。

Q：用事件模型分析Verilog代码的行为(2)

> [!IMPORTANT]
>
> 将上述代码中的阻塞赋值改成非阻塞赋值, 尝试重新分析可能的事件处理顺序及其结果. 修改后的代码还存在数据竞争吗? 为什么?
>
> ```verilog
> always @(posedge clk or negedge rstn) begin
>   if (!rstn) a <= 1'b0;
>   else a <= b;
> end
> 
> always @(posedge clk or negedge rstn) begin
>   if (!rstn) b <= 1'b1;
>   else b <= a;
> end
> ```

A：修改以后就不存在数据竞争了（所有模拟结果有冲突就叫数据竞争），

分析——只可能出现两组情况，

- 1）满足`!rstn` ，故 `a <= 1'b0`且`b <= 1'b1`，a和b的值互不影响；
- 2）不满足`!rstn`，故 `a <= b`且`b <= a`，由于是非阻塞赋值，所以最后结果是a和b得到的是彼此的旧值，即a和b数值交换，无数据竞争。



#### Q：用事件模型分析Verilog代码的行为(3)

> [!IMPORTANT]
>
> 将上述代码中的`$display`改成`$strobe`, 尝试重新分析可能的事件处理顺序及其结果. 修改后的代码还存在数据竞争吗? 为什么?
>
> ```verilog
> always @(posedge clk or negedge rstn) begin
>   if (!rstn) a = 1'b0;
>   else a = 1;
> end
> 
> always @(posedge clk) begin
>   $strobe("a = %d", a);
> end
> ```

A：不会。`strobe`是输出寄存器最终的结果，而`$display`是输出执行过程中的值，所以使用`strobe`不会产生数据竞争。



#### Q：我会写Verilog不就行了吗? 为什么要知道这些?

> [!IMPORTANT]
>
> 作为一个小小的测试, 下面有若干条Verilog的编码建议或描述, 但其中有一些是不正确的, 请尝试找出它们:
>
> 1. 使用`#0`可以将赋值操作强制延迟到当前仿真时刻的末尾.
> 2. 在同一个`begin`-`end`语句块中对同一个变量进行多次非阻塞赋值, 结果是未定义的.
> 3. 用`always`块描述组合逻辑元件时, 不能使用非阻塞赋值.
> 4. 不能在多个`always`块中对同一个变量进行赋值.
> 5. 不建议使用`$display`系统任务, 因为有时候它无法正确输出变量的值.
> 6. `$display`无法输出非阻塞赋值语句的结果.

A：2不对，5和6不严谨

2）在同一个`begin`-`end`语句块中对同一个变量进行多次非阻塞赋值, 结果是未定义的.

在同一个`begin`-`end`语句块中是顺序执行的，最后的结果一定是有定义的。

什么是非阻塞赋值？

- 非阻塞赋值如上述代码，阻塞赋值是`=`；非阻塞赋值：在时钟上升沿时计算新值，在块结束后才进行赋值。

  ```verilog
  always @(posedge clk) begin
      a <= b;
      b <= a;
  end
  ```

5）不建议使用`$display`系统任务, 因为有时候它无法正确输出变量的值

A：`$display` 总是输出执行时刻的当前值，因此可能显示旧值。

6）`$display`无法输出非阻塞赋值语句的结果

A：`$display` 可以输出非阻塞赋值的结果，但输出的是赋值前的值，而非更新后的值。严格来说，它并非“无法输出”，而是输出的不是用户期望的更新后结果。



## 逻辑综合(Logic Synthesis) - 从RTL代码到网表

ysyx提供了基于开源EDA的综合和评估项目. 可以通过以下命令克隆该项目:

```bash
git clone git@github.com:OSCPU/yosys-sta.git
```

### Q：尝试使用综合器

> [!IMPORTANT]
>
> 克隆上述项目后, 尝试对上文的流水灯项目作为示例进行综合. 你可以在上述项目中通过`make syn`命令进行综合, 但你需要修改一些配置或参数, 具体操作方式请阅读项目中的README.

修改Makefile：

```makefile
O ?= $(PROJ_PATH)/result
DESIGN ?= counter # 修改成流水灯的counter.v
SDC_FILE ?= $(PROJ_PATH)/scripts/default.sdc
RTL_FILES ?= $(shell find $(PROJ_PATH)/example -name "*.v")
```

额外修改git跟踪，添加了`sim_syn`和`sim_sta`，分别对应命令为`make  syn`和`make sta`.



运行结果符合题目：

##### A：

代码名称：yosys-sta/Makefile

已经提交至    tracer_ysyx/子模块yosys-sta



`make syn`和`make sta`的区别？

- `make sta` 需要 `make syn`生成的网表，所以如果直接执行`make sta`，那么会默认先执行`make syn`.

此处提供[yosys的官方手册](https://yosyshq.readthedocs.io/projects/yosys/en/latest/), 供需要时查阅.



### 细化(Elaboration)

```bash
yosys  #切换至yosys的交互界面
yosys> read_verilog example/counter  #读入需要分析的.v文件
yosys> hierarchy -check -top counter #细化
```



#### 中间代码生成

##### Q：查看结构图

> [!IMPORTANT]
>
> 通过`show`命令查看当前的结构图, 从而认识RTLIL.

```bash
#接着
yosys> dump  # yosys将当前设计的RTLIL以文本形式输出到终端 
#或者执行可以保存到文件查看   write_rtlil counter.rtlil
yosys> show  #安装dot工具后执行，可以看到结构图
```

![image-20260218000118645](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260218000118645.png)

注意：文件默认保存在`~/.yosys_show.dot`, 后面多次执行会被覆盖，可以另存.



### 粗粒度综合(Coarse-grain synthesis)

#### 将过程描述转换为粗粒度表示

##### Q：查看结构图(2)

> [!IMPORTANT]
>
> 通过`show`命令查看当前的结构图, 对比执行`proc`命令前后的不同.

```bash
#接着
yosys> proc
yosys> show
```

结构图如下：

![image-20260218001313862](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260218001313862.png)

可以看出，这一次的结构图把PROC部分细分成了三部分：2个`$mux`单元和1个`$dff`单元。

#### 优化

##### Q：查看结构图(3)

> [!IMPORTANT]
>
> 通过`show`命令查看当前的结构图, 对比执行`opt`命令前后的不同.

```bash
yosys> opt
yosys> show
```

![image-20260218002236750](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260218002236750.png)

合并了2个相同的单元`$mux`；对于D触发器的常量优化和时钟复位信号合并，把`dff`优化为了`$sdffe`；移除了重复的线`2'01`，移除了`count[1:0]`.



#### 细粒度综合(Fine-grain synthesis)

t

##### Q：查看结构图(4)

> [!IMPORTANT]
>
> 通过`show`命令查看当前的结构图, 对比执行`techmap`命令前后的不同.

```bash
yosys> proc     #粗粒度
yosys> techmap  #细粒度
```

![image-20260218162854669](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260218162854669.png)

可以看出很多地方进行了更加细致的划分，并且将当前设计的单元替换成yosys内部的门级单元库中的单元实现。



### 工艺映射(Technology mapping)

Q：查看结构图(5)

> [!IMPORTANT]
>
> 通过`show`命令查看当前的结构图, 对比执行`dfflibmap`命令前后的不同.

![image-20260218163857767](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260218163857767.png)

A：不同的是，门级单元`$_SDFFE_PP0P_`被替换成`DFF`和一些`$_MUX_`, 其中`DFF`就是标准单元库`cell.lib`中的标准单元. 



#### Q：查看结构图(6)

> [!IMPORTANT]
>
> 通过`show`命令查看当前的结构图, 对比执行`abc`命令前后的不同.
>
> `abc`命名将会调用一个外部工具`ABC`来进行组合逻辑单元的工艺映射工作. 

![image-20260218164328135](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260218164328135.png)

A：不同——所有门级单元都被替换成标准单元库`cell.lib`中的标准单元，并且通过`clean`命令清除无用的单元和连线, 得到最终的网表, 从而完成从RTL代码到网表的转换.



### 网表和报告生成

#### Q：通过yosys的日志文件了解综合过程

> [!IMPORTANT]
>
> 你已经对流水灯项目进行综合了. 尝试查看`yosys-sta/result`目录下的yosys日志文件, 结合上述文字了解综合过程.

```bash
yosys> write_verilog netlist.v #将网表写入到文件
yosys> stat -liberty cell.lib  #输出所用标准单元的信息
```

解析文件(read) —>  细化(hierarchy -check -top counter)  —>  粗粒度综合(proc)   —>   优化(opt)   —>   细粒度综合(techmap)   —>  拆分线网和端口(splitnets -ports)    —>   工艺映射：1）对时序逻辑(dfflibmap -liberty cell.lib)；2）对组合逻辑(abc -liberty cell.lib)



> 为了避免反复键入yosys的命令, `make syn`通过脚本驱动yosys进行综合, 具体可查阅`yosys-sta/scripts/yosys.tcl`. 如果你想进一步了解yosys的命令, 可以查阅yosys的官方手册, 或者在yosys的命令行中输入`help xxx`来查看`xxx`命令的相关信息.

![image-20260218165715619](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260218165715619.png)

### Verilog的RTL综合语义

#### Q：RTFM

> [!IMPORTANT]
>
> 阅读 [Verilog RTL综合标准手册](https://0x04.net/~mwk/vstd/ieee-1364.1-2002.pdf)的第5章`Modeling hardware elements`, 了解什么样的Verilog代码被综合成什么样的电路.
>
> 这部分内容只有10页左右, 但比其他所有Verilog学习资料都要权威, 而且提供了大量的代码示例和说明, 甚至连`x`和`z`在哪些使用场景下是可综合, 都有详细的说明.
>
> 事实上, 上文引用的若干Verilog编码建议, 其中有一些建议就是依据Verilog RTL综合标准手册的规范提出的.

#### Q：用事件模型分析Verilog代码的行为(4)

> [!IMPORTANT]
>
> 尝试用事件模型分析上述代码为什么存在数据竞争.
>
> ```verilog
> always @(posedge clock) begin
>   a = 0;
>   a = 1;
> end
> 
> always @(posedge clock)
>   b = a;
> ```

A：第二个always块的执行时间：可能在第一个块执行之前、之间或之后执行。所以两个always块可以类比两个进程，是同时调用随即执行的。

所以，若是在执行完a=0后。执行b=a，则`b=0`；若是在执行完a=1后。执行b=a，则`b=1`.



#### Q：RTFM(2)

> [!IMPORTANT]
>
> 阅读Verilog RTL综合标准手册的附录B, 了解还有哪些情况会造成综合网表和仿真行为的功能不匹配.

A：

1）对于模拟来说，如果` en = 2'b11`，模拟将按顺序执行第一个符合条件的 `case`项语句，跳过第二个，最后`en_mem=1'b1`和`en_cpu=1'b0`；

2）对于综合工具，则会默认所有的分支都是互斥的，但下述代码并非如此，所以将执行前两个` case `项语句，会同时驱动 `en_mem=1'b1`和`en_cpu=1'b1`.

综上所述，这将导致综合前后的模拟结果不匹配。

```verilog
module endec_pc (output reg en_mem, en_cpu, en_io,input [1:0] en);
   always @* begin
		en_mem=1'b0;
		en_cpu=1'b0;
		en_io =1'b0;
(* synthesis, parallel_case *)
	casez (en)
		2'b1?: en_mem=1'b1;
		2'b?1: en_cpu=1'b1;
		default: en_io =1'b1;
	endcase
  end
endmodule
```



## 用开源EDA工具评估电路

### 性能评估

#### Q：评估电路的性能

> [!IMPORTANT]
>
> 尝试通过`yosys-sta`项目评估电路的性能, 并阅读静态时序分析的报告, 了解目标电路能运行的最大频率.
>
> 目前我们不要求你了解报告中的所有细节, 我们会在B阶段介绍更多STA的内容. 如果你现在对报告的细节感兴趣, 可以参考[这个教程](https://ieda.oscc.cc/tools/ieda-tools/ista.html), 或者在上网查询时序报告的阅读教程. 其他工具也可以生成时序报告, 虽然格式可能有所不同, 但其中的大部分概念是相通的.
>

在`~/ics55_LLSC_H7C_typ_tt_1p2_25_nldm.lib`下评估：

对流水灯counter.v进行分析，运行`make sta`，得到~/result/counter-500MHz/counter.rpt，打开搜索Freq可以定位最大帧率：

![image-20260219175010591](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260219175010591.png)

##### 如何找到最大帧率？

1. 在报告里表格列出了所有路径的时序信息，其中最后一列就是 `Freq(MHz)`，表示帧率；
2.  “Delay Type” 为 `max` 的路径，即建立时间检查，决定最高频率；
3. 在所有 `max` 路径中，找最小的 `Freq(MHz)` 值 即当前设计能达到的最大频率，因为受限于最慢的路径。

所以，当前流水灯的最大帧率为5331.883 MHZ.



### 功耗评估

#### Q：评估电路的功耗

> [!IMPORTANT]
>
> 尝试通过`yosys-sta`项目评估电路的性能, 并阅读功耗分析的报告, 了解目标电路的功耗信息.

在`~/ics55_LLSC_H7C_typ_tt_1p2_25_nldm.lib`下评估：

分析功耗需要使用iEDA里面的iPower工具，所以运行`make sta`，得到~/result/counter-500MHz/counter.pwr，如下：

![image-20260219181418988](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260219181418988.png)

`Internal Power`是内部功耗. `Switch Power`是翻转功耗. `Leakage Power`是漏电功耗.



## PDK和标准单元库

### 工艺视角的芯片结构

#### Q：了解ICsprout55的金属层

> [!IMPORTANT]
>
> ICsprout55的工艺LEF文件, 其中包含多少层金属层? 并尝试根据线宽和走线间距, 推测每层金属层的作用.

打开~/ysyx-workbench/yosys-sta/pdk/icsprout55/prtech/techLEF/N551P6M.lef查看里面的金属层定义。

##### A：判断依据

###### 1）判断金属层依据：

如果一个 `LAYER` 的定义中包含 `TYPE ROUTING`，那么该层就是金属层。搜索可得： `MET1`～`MET5`、`T4M2`、`RDL` 都属于金属层，一共7层。

###### 2）判断金属层作用：

LEF里面的金属层，按作用分类一共有4种：

局部互连：底层连线，需要高密度，所以线宽和走线间距最小，MET1 (0.09μm)；

中间信号：线宽和走线间距适中，MET2 ~ MET5 (0.10μm)

全局/电源：线宽和走线间距较高，电阻较小，T4M2 (0.40μm)；

再分布层 RDL：用于把芯片扩大/搬家，线宽和走线间距最大，电阻较小，RDL (3.0μm)；



### 标准单元的属性

LIB文件中各个字段的具体含义, 可以查阅[`Liberty Timing File`的文件格式手册](https://media.c3d2.de/mgoblin_media/media_entries/659/Liberty_User_Guides_and_Reference_Manual_Suite_Version_2017.06.pdf).

#### Q：根据CDL文件画出晶体管结构

> [!IMPORTANT]
>
> ```bash
> cd yosys-sta/pdk/icsprout55/IP/STD_cell/ics55_LLSC_H7C_V1p10C100/
> vim ics55_LLSC_H7CL/liberty/ics55_LLSC_H7CL_typ_tt_1p2_25_nldm.lib
> ```
>
> 尝试根据上述CDL描述, 画出标准单元`NAND2X1H7L`的晶体管结构, 并检查其功能与与非门是否一致.

CDL文件描述部分截取：

```CDL
.SUBCKT NAND2X1H7L A B VDD VSS Y  #有5个端口
*.PININFO A:I B:I Y:O VDD:B VSS:B  # *开头为注释
MMN0 Y B net6 VSS nm1p2_lvt_lp W=210n L=60n m=1
晶体管名称，D，G，S，衬底，衬底类型，沟道宽度，沟道长度，晶体管数量 
#各个变量的含义
```

故标准单元`NAND2X1H7L`的晶体管结构，如下：

![img](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/6ded65ca11b7cd3604883f93f51cc0b7.png)

经过测试，完全符合两输入与非门。



### 标准单元的分类

#### Q：理解复杂逻辑门单元的功能

> [!IMPORTANT]
>
> ICsprout55的LIB文件中还存在命名类似`OAI22X1H7L`的标准单元, 其功能并不直观. 尝试查阅标准单元`OAI22X1H7L`相关的属性, 了解其功能.

A：经过定位可以找到，分别在两个LIB文件中：

```bash
home:~/ysyx/ysyx-workbench/yosys-sta$ grep -r "OAI22X1H7L"
pdk/icsprout55/IP/STD_cell/ics55_LLSC_H7C_V1p10C100/ics55_LLSC_H7CL/liberty/ics55_LLSC_H7CL_typ_tt_1p2_25_nldm.lib:  cell (OAI22X1H7L) {
pdk/icsprout55/IP/STD_cell/ics55_LLSC_H7C_V1p10C100/ics55_LLSC_H7CL/liberty/ics55_LLSC_H7CL_typ_tt_1p2_25_nldm.lib:    cell_footprint : "OAI22X1H7L";
pdk/icsprout55/IP/STD_cell/ics55_LLSC_H7C_V1p10C100/ics55_LLSC_H7CL/liberty/ics55_LLSC_H7CL_ss_rcworst_1p08_125_nldm.lib:  cell (OAI22X1H7L) {
pdk/icsprout55/IP/STD_cell/ics55_LLSC_H7C_V1p10C100/ics55_LLSC_H7CL/liberty/ics55_LLSC_H7CL_ss_rcworst_1p08_125_nldm.lib:    cell_footprint : "OAI22X1H7L";
```

标准单元`OAI22X1H7L`相关的属性：

- **类型**：2-2 输入(即2组输入)的或与非门（OR-AND-INVERT）
- **表达式**：`Y = !((A0 | A1) & (B0 | B1))`
- **输入引脚**：A0, A1, B0, B1
- **输出引脚**：Y

截取自讲义，先取或，再取和，最后取反的逻辑表达式：

```
(!A0 * !A1) + (!B0 * !B1) = !(A0 + A1) + !(B0 + B1) = !((A0 + A1) * (B0 + B1))
```



#### Q：理解OAI22的晶体管结构

> [!IMPORTANT]
>
> 在CDL文件中查阅`OAI22X1H7L`的晶体管结构, 它有多少根晶体管? 尝试理解如何通过其晶体管结构实现`OAI22X1H7L`的逻辑表达式.

A：位置上一题已经搜索到，打开对应的CDL文件截取源码，

```
.SUBCKT OAI22X1H7L A0 A1 B0 B1 VDD VSS Y
*.PININFO A0:I A1:I B0:I B1:I Y:O VDD:B VSS:B
MNM4 net8 A1 VSS VSS nm1p2_lvt_lp W=210n L=60n m=1
MNM3 net8 A0 VSS VSS nm1p2_lvt_lp W=210n L=60n m=1
MMN5 Y B0 net8 VSS nm1p2_lvt_lp W=210n L=60n m=1
MNM5 Y B1 net8 VSS nm1p2_lvt_lp W=210n L=60n m=1
MMP5 Y A1 net046 VDD pm1p2_lvt_lp W=270n L=60n m=1
MPM3 net046 A0 VDD VDD pm1p2_lvt_lp W=270n L=60n m=1
MPM5 Y B1 net049 VDD pm1p2_lvt_lp W=270n L=60n m=1
MPM4 net049 B0 VDD VDD pm1p2_lvt_lp W=270n L=60n m=1
.ENDS
```

一共有八个晶体管，四个nMOS和四个pMOS，把这些晶体管的结构都画出来，然后写出或与非的真值表，看是否符合，如果符合就可以实现逻辑表达式子。



#### Q：理解驱动能力

> [!IMPORTANT]
>
> 尝试查阅`NAND2X1H7L`, `NAND2X2H7L`和`NAND2X4H7L`的相关属性, 对比其面积和功耗.

A：依旧是上述的LIB文件中，搜索部件名称后可得，

`NAND2X1H7L`面积是1.12μm²，漏电功率是0.434974pw

```
cell (NAND2X1H7L) {
    area : 1.12;
    cell_footprint : "NAND2X1H7L";
    cell_leakage_power : 0.434974;
```

`NAND2X2H7L`面积是1.96μm²，漏电功率是0.750363pw

```
cell (NAND2X2H7L) {
    area : 1.96;
    cell_footprint : "NAND2X2H7L";
    cell_leakage_power : 0.750363;
```

`NAND2X4H7L`面积是3.36μm²，漏电功率是1.50065pw

```
cell (NAND2X4H7L) {
    area : 3.36;
    cell_footprint : "NAND2X4H7L";
    cell_leakage_power : 1.50065;
```



###### 延伸问题：CDL和LIB分别记录着什么信息？

CDL文件：描述了某个单元是由哪些晶体管构成的（物理网表）；

LIB文件：描述了某个单元在工作时的表现怎么样（时序和功耗模型）。



#### Q：了解I/O单元的尺寸

> [!IMPORTANT]
>
> 尝试查阅ICsprout55中的相关文件, 了解一个二输入与非门的尺寸以及一个I/O单元的尺寸, 并对比它们.

A：I/O单元一般在一个单独的文件夹中，故在ICsprout55中进行寻找，找到了IO文件夹，在`~/ysyx-workbench/yosys-sta/pdk/icsprout55/IP/IO/ICsprout_55LLULP1233_IO_251013/liberty`中查阅`PAD`：

```
cell ("P65_1233_PAR") {
		cell_leakage_power : 0.000000e+00;
		area : 8450.000000;
		pad_cell : true;
		pin (PAD) {
			direction : "inout";
			drive_current : 4.000000;
			is_pad : true;
			fall_capacitance : 1.325000;
			capacitance : 2.512000;
			rise_capacitance : 2.738000;
		}
```

这是其中一个I/O设备，可以看到面积为 8450μm²，而上一道题的二输入与非门面积区间在(1,3.5)，可以看出I/O单元的尺寸要比二输入与非门大得多。



#### Q：复杂单元的全定制电路

> [!IMPORTANT]
>
> 以`ADDHX1H7L`为例, 尝试从标准单元库的相关文件中找到这个标准单元的面积和晶体管结构. 假设某标准单元库不提供半加器的标准单元, 需要通过若干基本逻辑门的标准单元来搭建半加器, 请计算此时所需的面积和晶体管数量.

检索得：

```bash
Yang@yyl-Ubuntu:~/ysyx/ysyx-workbench/yosys-sta$ grep -r "ADDHX1H7L"
pdk/icsprout55/IP/STD_cell/ics55_LLSC_H7C_V1p10C100/ics55_LLSC_H7CL/liberty/ics55_LLSC_H7CL_typ_tt_1p2_25_nldm.lib:  cell (ADDHX1H7L) {
pdk/icsprout55/IP/STD_cell/ics55_LLSC_H7C_V1p10C100/ics55_LLSC_H7CL/liberty/ics55_LLSC_H7CL_typ_tt_1p2_25_nldm.lib:    cell_footprint : "ADDHX1H7L";
pdk/icsprout55/IP/STD_cell/ics55_LLSC_H7C_V1p10C100/ics55_LLSC_H7CL/liberty/ics55_LLSC_H7CL_ss_rcworst_1p08_125_nldm.lib:  cell (ADDHX1H7L) {
pdk/icsprout55/IP/STD_cell/ics55_LLSC_H7C_V1p10C100/ics55_LLSC_H7CL/liberty/ics55_LLSC_H7CL_ss_rcworst_1p08_125_nldm.lib:    cell_footprint : "ADDHX1H7L";
#以上是lib文件，以下是cdl文件
pdk/icsprout55/IP/STD_cell/ics55_LLSC_H7C_V1p10C100/ics55_LLSC_H7CL/cdl/ics55_LLSC_H7CL.cdl:* Cell Name:    ADDHX1H7L
pdk/icsprout55/IP/STD_cell/ics55_LLSC_H7C_V1p10C100/ics55_LLSC_H7CL/cdl/ics55_LLSC_H7CL.cdl:.SUBCKT ADDHX1H7L A B CO S VDD VSS
```

在lib文件中搜索得到：

```
cell (ADDHX1H7L) {
    area : 5.6;
    cell_footprint : "ADDHX1H7L";
    cell_leakage_power : 4.01715;
    pg_pin (VDD) {
      pg_type : primary_power;
      voltage_name : "VDD";
    }
```

在cdl文件里面搜索得到：

```
.SUBCKT ADDHX1H7L A B CO S VDD VSS
*.PININFO A:I B:I CO:O S:O VDD:B VSS:B
XI13 BN B net45 net_25 VDD VSS / TG pl=6E-08 pw=3.4E-07 nl=6E-08 nw=2.4E-07
XXI12 B BN net034 net_25 VDD VSS / TG pl=6E-08 pw=3.4E-07 nl=6E-08 nw=2.4E-07
XI12 A VDD VSS net45 / INV pl=6E-08 pw=3.4E-07 nl=6E-08 nw=2.4E-07
XI14 net_25 VDD VSS S / INV pl=6E-08 pw=3.4E-07 nl=6E-08 nw=2.4E-07
XXI11 net45 VDD VSS net034 / INV pl=6E-08 pw=2.8E-07 nl=6E-08 nw=2E-07
XI9 B VDD VSS BN / INV pl=6E-08 pw=3.4E-07 nl=6E-08 nw=2.4E-07
XI11 net_7 VDD VSS CO / INV pl=6E-08 pw=3.4E-07 nl=6E-08 nw=2.4E-07
XI10 B A VDD VSS net_7 / NAND2 pl=6E-08 pw=2E-07 nl=6E-08 nw=2E-07
.ENDS
```

从以上文件可知：

- `ADDHX1H7L`面积为5.6μm²；
- 该半加器调用了以下子模块：2个传输门 (TG)——(XI13、XXI12)；5个反相器 (INV)——(XI12、XI14、XXI11、XI9、XI11)；1个二输入与非门 (NAND2)——XI10；
  - 根据CMOS的一般定义：1个传输门TG需要2个晶体管；一个反相器 (INV)需要2个晶体管；1个二输入与非门需要4个晶体管。

A1：故`ADDHX1H7L`面积为5.6μm²，且需要18个晶体管。



若是没有半加器的定义，那么构建半加器的组建仍然为：2个传输门 (TG)；5个反相器 (INV)；1个二输入与非门 (NAND2)；总面积计算需要查阅lib文件里面定义基本门电路的面积，再进行类加。

```
#并未找到TG的定义
但是TG和INV都是2个晶体管构成，所以可以它俩使用一个面积

#由于ADDHX1H7L功率较小，所以选取了功率较小的INV和NAND2
cell (INVX0P5H7L) {
    area : 0.84;
    cell_footprint : "INVX0P5H7L";
    cell_leakage_power : 2.0707;
    pg_pin (VDD) {
      pg_type : primary_power;
      voltage_name : "VDD";
    }
 
 cell (NAND2BX0P7H7L) {
    area : 1.4;
    cell_footprint : "NAND2BX0P7H7L";
    cell_leakage_power : 3.01433;
```

A2：共18个晶体管，总面积=`0.84+0.84*5+1.4=6.44`μm²



#### Q：了解的所有标准单元

> [!IMPORTANT]
>
> 尝试结合PDK的相关文件, 进一步了解ICsprout55提供的标准单元. 你可以查看相应的注释和功能属性了解相关标准单元的作用. 了解后, 你将会对你的RTL代码如何被综合器处理有一个简单的认识.

A：综合器如何处理 RTL 代码：

1. **读取库信息**：综合器首先会读入 `.lib` 文件，获取所有可用标准单元的逻辑功能 (`function`)、面积 (`area`) 和时序信息。
2. **逻辑映射**：当你的 RTL 代码描述了一个 `assign Y = ~(A & B);` 的语句时，综合器会在库中搜索 `function` 属性匹配 `Y = (A & B)'` 的单元，并找到 `NAND2X1H7L`。
3. **优化与选择**：综合器会根据你设定的约束（如面积、速度、功耗），从不同驱动版本的单元（如 `NAND2X1`、`X2`、`X4`）中选择最合适的一个。选择 `X2` 能驱动更大负载但面积和功耗也更大。
4. **输出网表**：最终，综合器会生成一个门级网表，其中只实例化了标准单元库中提供的单元。后续的布局布线工具则会读取 `.lef` 文件来放置这些单元并连接它们 。



#### Q：尝试不同PVT角的评估结果

> [!IMPORTANT]
>
> ICsprout55提供了不同PVT角的LIB文件(位于`yosys-sta/pdk/ICsprout55/IP/STD_cell/ics55_LLSC_H7C_V1p10C100/ics55_LLSC_H7CL/liberty`目录下). 尝试为`yosys-sta`项目更换不同PVT角的LIB文件(在`yosys-sta/scripts/pdk/icsprout55.tcl`中指定), 然后重新评估电路的性能, 对比不同PVT角下的评估结果.

##### A：代码提交

代码已经提交到分支tracer-ysyx 下的  homework/icsprout55.tcl

对流水灯counter.v进行分析，修改：`yosys-sta/scripts/pdk/icsprout55.tcl`，后运行`make sta`，

##### 1）分析最大帧率

得到~/result/counter-500MHz/counter.rpt，打开搜索Freq可以定位最大帧率：

在`~/ics55_LLSC_H7C_ss_rcworst_1p08_125_nldm.lib`下评估：

![image-20260221212604863](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260221212604863.png)

在`~/ics55_LLSC_H7C_typ_tt_1p2_25_nldm.lib`下评估（之前评估使用的）：

![image-20260219175010591](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260219175010591.png)

所以，原先分析流水灯的最大帧率为5331.883 MHZ，更换PVT角度后最大帧率为2927.563 MHZ.



##### 2）分析电路功耗信息

得到~/result/counter-500MHz/counter.pwr，如下：

在`~/ics55_LLSC_H7C_ss_rcworst_1p08_125_nldm.lib`下评估：

![image-20260221213534412](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260221213534412.png)

在`~/ics55_LLSC_H7C_typ_tt_1p2_25_nldm.lib`下评估（之前评估使用的）：

![image-20260219181418988](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260219181418988.png)

`Internal Power`是内部功耗. `Switch Power`是翻转功耗. `Leakage Power`是漏电功耗.

可以看出，漏电功耗变大了，总功耗变小了，内部功耗也变小了。



##### 延伸：2个PVT角度有什么区别？

PVT指的是Process（工艺）、Voltage（电压）、Temperature（温度）；

- 在典型条件typ_tt_1p2_25中，假设工艺典型、供电电压1.2V、温度25°C，是最常见的标称条件.
- 在ss_rcworst_1p08_125中，ss代表慢工艺（slow-slow，即NMOS和PMOS都慢），rcworst代表互连线阻容最差情况，电压1.08V（较低），温度125°C（高温）。

总结：可以理解为typ_tt_1p2_25是正常情况下评估电路，ss_rcworst_1p08_125是非常极端恶劣的情况下评估。

现象：高温会增大漏电功耗，低压会降低内部功耗，慢工艺会降低最大帧率，所以以上的变化符合2个不同的PVT角度下的评估。



### 阈值电压

根据晶体管的工作原理, 栅极电压与源极电压之间的差值必须达到某个阈值, 晶体管才能导通, 否则晶体管截止。

ICsprout55提供了HVT, RVT和LVT三种不同阈值电压的标准单元.LVT（低阈值，高速高漏电）, RVT（常规阈值，平衡性能与漏电）, HVT（高阈值，低速低漏电）。

#### Q：尝试不同的阈值电压

> [!IMPORTANT]
>
> todo

A：通过百度定位阈值电压设置的位置，在`yosys-sta/pdk/ICsprout55/IP/STD_cell/ics55_LLSC_H7C_V1p10C100`下有2个阀值电压，分别是LVT和RVT，但没有HVT，应该是yosys-sta下没有设置：

```
...
│   └── STD_cell                             # Standard cell library
│       └── ics55_LLSC_H7C_V1p10C100         # 55nm LLSC H7C standard cell library version 1.10
│           ├── ics55_LLSC_H7CL              # LVT standard cells   
│           └── ics55_LLSC_H7CR              # RVT standard cells     
```

 修改：`yosys-sta/scripts/pdk/icsprout55.tcl`，后运行`make sta`，

```
#设置阀值电压 LVT 和 RVT
# set VTS [list L]  
 set VTS [list R]
```

在RVT中只提供了经典情况`typ_tt_1p2_25`，得到~/result/counter-500MHz/中的counter.rpt和counter.pwr，如下：

1）在RVT（常规阈值，平衡性能与漏电）下，且经典情况`typ_tt_1p2_25`，此时最大帧率为455.395 MHZ：

![image-20260223125617147](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260223125617147.png)

对比分析：在LVT（低阈值，高速高漏电）下，且也为经典情况`typ_tt_1p2_25`，

![image-20260219175010591](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260219175010591.png)

LVT（低阈值，高速高漏电）要比RVT（常规阈值，平衡性能与漏电）的最大帧率大。



2）在RVT（常规阈值，平衡性能与漏电）下，且经典情况`typ_tt_1p2_25`，此时电路功耗信息为 ：

![image-20260223125655561](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260223125655561.png)

对比分析：在LVT（低阈值，高速高漏电）下，且也为经典情况`typ_tt_1p2_25`，![image-20260219181418988](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260219181418988.png)

可以看出，LVT（低阈值，高速高漏电）的内部功耗和总功率   比   RVT（常规阈值，平衡性能与漏电）的内部功耗和总功率都大，LVT的漏电功率变大了，符合高速高漏电的特性。



### 轨道数

金属层的`PITCH`属性描述了该层的最小走线间距. 为了方便EDA工具开展布局布线工作, 通常会让标准单元的高度取M1金属层的`PITCH`属性的整数倍, 这个倍数就是标准单元的轨道数.

#### Q：理解ICsprout55的轨道数

> [!IMPORTANT]
>
> 上文在介绍ICsprout55时已经提到过其轨道数. 尝试在相关文件中找到需要的参数, 计算ICsprout55标准单元的轨道数, 检查计算所得的轨道数是否与上文所介绍的轨道数一致.

与金属层有关，故寻找 在文件  ~/ysyx-workbench/yosys-sta/pdk/icsprout55/prtech/techLEF 的 N551P6M.lef 中，

查询`MET1`，得知垂直方向轨道间距=0.2μm：

```
LAYER MET1
  TYPE ROUTING ;
  DIRECTION HORIZONTAL ; #水平方向
  PITCH 0.2 0.2 ;
 ....
END MET1
```

 查询`SIZE`，得知单元高度：

```
SITE core7 
    SIZE 0.200 BY 1.400 ;
    CLASS CORE ;
    SYMMETRY Y  ;
END core7

SITE core9
    SIZE 0.200 BY 1.800 ;
    CLASS CORE ;
    SYMMETRY Y  ;
END core9
```

计算轨道数：

1.4/0.2=7  ———  7T   

1.8/0.2=9  ———  9T

说明：

若金属名称有“H7”说明轨道数为7，并且所有的标准单元必须使用一个轨道数。经过观察只有“H7”，所以采用的数7T标准单元。与上文使用的6T标准单元并不相同。



## 物理设计(Physical Design) - 从网表到可流片版图



### Q：填充单元的尺寸

> [!IMPORTANT]
>
> 标准单元库通常提供不同尺寸的填充单元, 用于填充芯片中没有摆放标准单元的空白位置. 以ICsprout55为例, 尝试在相关文件中找到最小填充单元的尺寸, 这个尺寸和标准单元的`SITE`属性有什么关联? 为什么?

#### 1）如何找到填充单元？

A：填充单元一般叫`FULL CELL`,在ics55_LLSC_H7CL/lef/进行搜索`FULL`：

```bash
cd ~/ysyx/ysyx-workbench/yosys-sta/pdk/icsprout55/IP/STD_cell/ics55_LLSC_H7C_V1p10C100/ics55_LLSC_H7CL/lef/
grep -i "FILL" *.lef
```

找到带有`FULL`的标准单元中`SIZE`最小的，如下图：

```
ACRO FILLER1H7L
  CLASS CORE ;
  ORIGIN 0 0 ;
  FOREIGN FILLER1H7L 0 0 ;
  SIZE 0.2 BY 1.4 ;  # 宽度 0.2 μm，高度 1.4 μm
  SYMMETRY X Y ;
  SITE core7 ;
  ...
```

该单元就是最小填充单元的尺寸，宽度 0.2 μm，高度 1.4 μm。

#### 2）最小填充单元的尺寸和标准单元的`SITE`属性有什么关联? 为什么?

A：

- 高度必须一致：填充单元的高度`BY`必须严格等于标准单元的 `SITE` 高度。这样才能完美地放置在标准单元的行内 ，此时`BY=1.4μm`.
- 宽度是 `SITE` 宽度的整数倍：填充单元的宽度通常是 `SITE` 宽度（0.2 μm）的整数倍。最小填充单元的宽度往往就是 `SITE` 的宽度（即 0.2 μm），用于填补细小的缝隙。



## 若干代码风格和规范

简单的来说，就是不要使用`always`描述电路，不利于构建电路思维，正常情况下选择连线+实例化来描述电路.

### Q：尝试用negedge综合

> [!IMPORTANT]
>
> 以上述模块为例, 尝试评估以下模块的时序:
>
> ```verilog
> module test(input clk, input rst, input in, output out);
>   wire t0, t1;
>   Reg r1(clk, rst, in, t0, 1'b1);
>   Reg r2(clk, rst, t0, t1, 1'b1);
>   Reg r3(clk, rst, t1, out, 1'b1);
> endmodule
> ```
>
> 然后单独将`r2`修改为时钟下降沿触发, 并重新评估时序. 对比修改前后的时序报告, 你发现有什么不同之处? 如果你发现自己无法理解其中的细节, 不必担心, 我们会在B阶段进一步介绍时序分析的细节, 到时候将会重新讨论这个问题.

A：将 `r2` 单独改为下降沿触发后，时序分析结果会出现明显差异，主要体现在**数据路径的可用时间减半**，导致建立时间（setup time）更紧张，可能引发时序违例。



## 完成数字电路实验

### Q：借助NVBoard完成数字电路实验

> [!IMPORTANT]
>
> 我们首先推荐南京大学的[数字电路与计算机组成实验](https://nju-projectn.github.io/dlco-lecture-note/index.html).
>
> 南京大学开展教学改革, 将"数字电路"与"计算机组成原理"两门课程进行融合, 其实验内容贯穿从数字电路基础到简单的处理器设计. 有了NVBoard之后, 你就可以把它当作FPGA来使用, 用它来实现需要FPGA支持的实验内容.
>
> 你需要完成以下**必做内容**:
>
> - 实验二 译码器和编码器
> - 实验三 加法器与ALU
> - 实验六 移位寄存器及桶形移位器
> - 实验七 状态机及键盘输入
> - 实验八 VGA接口控制器实现
>
> 如果你打算使用Chisel来完成上述数字电路实验, 你只需要把编译出的Verilog代码接入Verilator和NVBoard就可以了.

A：五个实验如下，

#### 1)实验二  译码器和编码器

实验内容：实现8-3优先编码器，并且将显示结果显示到七段数码管上，二进制利用LED显示。

8-3优先编码器的基本设计：

使能端en接SW8；输入的8bit数据接SW7-SW0；sign标志是否有数据，接LD4；编码后的结果利用LED显示二进制，利用seg显示十进制，范围在十进制0～7. 

已经提交到分支tracer-ysyx 下的  homework/example_83encoder：

代码文件名为：example_83encoder

git log为  NVBoard_83encoder

实验结果：

![image-20260305220102226](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260305220102226.png)



#### 2)实验三 加法器与ALU

##### 实验内容：

实现4bit补码加法，与，或，取反，异或，比较的操作，额外增加了负零的判断，当操作数为1,000B时，当成零来进行处理.

##### 基本设计：

利用sel进行功能选择，操作数A连接SW3-SW0，操作数B连接SW7-SW4，sum连接LD3-LD0，判断sum是否为零连接LD5；判断sum溢出连接LD4；判断进位连LD6；按钮LCR选择ALU操作——000加法，001减法，010取反，011与，100或，101异或，110比大小，111判相等。

##### 我的bug：

- 补码的负零比较特殊，需要当作零来计算，需要格外写一下；

- 比较大小时，两个负数补码可以当作无符号数来比较，最后结果不变，即当A和B同号时，可以直接比较大小.

- 4bit带符号补码加法和减法，最高位有进位，等同于溢出吗？

  - 不等同：

  - | 1111 + 0001 = 0000 | -1+1=0 | 1    | 0    | 有进位，但结果正确，无溢出 |
    | ------------------ | ------ | ---- | ---- | :------------------------: |


### 4bit有符号补码运算示例

| 序号 | 运算 | a (二进制) | a (十进制) | b (二进制) | b (十进制) | 结果 (二进制) | 实际十进制值 | 进位/借位 Cout*C**o**u**t* | 溢出  | 说明                                  |
| :--- | :--- | :--------- | :--------- | :--------- | :--------- | :------------ | :----------- | :------------------------- | :---- | :------------------------------------ |
| 1    | 加法 | 0011       | +3         | 0010       | +2         | 0101          | +5           | 0                          | 0     | 正+正，结果在范围内，无进位无溢出     |
| 2    | 加法 | 0101       | +5         | 0100       | +4         | 1001          | -7（应为+9） | 0                          | **1** | 正+正，结果超出 7，产生溢出，无进位   |
| 3    | 加法 | 0110       | +6         | 1101       | -3         | 0011          | +3           | **1**                      | 0     | 正+负，结果正确，有进位但无溢出       |
| 4    | 加法 | 1100       | -4         | 1110       | -2         | 1010          | -2           | **1**                      | 0     | 负+负，结果正确，有进位无溢出         |
| 5    | 加法 | 1011       | -5         | 1100       | -4         | 0111          | +7（应为-9） | **1**                      | **1** | 负+负，结果超出 -8，有进位且溢出      |
| 6    | 减法 | 0111       | +7         | 1110       | -2         | 1001          | -7（应为+9） | 0                          | **1** | 7 - (-2) = 9，正-负得负，溢出，有借位 |
| 7    | 减法 | 1100       | -4         | 0101       | +5         | 0111          | +7（应为-9） | 0                          | **1** | -4 - 5 = -9，负-正得正，溢出，无借位  |

##### 实验结果：

下面是加法，A=1,001  和   B=0,100

![image-20260305224414699](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260305224414699.png)



##### 提交：

已经提交到分支tracer-ysyx下的homework/example_ALU：

代码文件名为：example_ALU

git log为  NVBoard_ALU



#### 3)实验六 移位寄存器及桶形移位器

##### 实验内容：

实现了8bit简单的随机数发生器,先使用logisim画图，然后再使用Verilog语言描述电路，对零的单独处理，避免出现遇到零停住的情况.

##### 我的bug：

- 不能同时进行阻塞赋值和非阻塞赋值：虽然上面学到过，但是确实是只有上手自己写才能切身体会到；
- 对零单独处理：
  - 当随机数的变化过程为0时，就选择输出1，避免出现遇到零就停住的情况。
  - 注意：rst复位后依旧是0，而不是1，我考虑到可以自己输入想要的初始数字

##### 设计思路：

依照我在logisim设计的电路图，再利用Verilog语言描述出来。对于NVBoard上，最开始输入的8bit数字a接SW7~SW0，rst接BTNC，选择器的选择位sel接SW8；七段数码管的低2位放未变化前的数，seg4-seg3放变化后的数，高4位全灭；en接SW9,可以实现单步一次生成一个随机数，算模拟Logisim上CLK的功能。

###### 设计的电路图：

![image-20260225233858870](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260225233858870.png)

##### 操作过程：

输入初始值SW7～SW0；然后按下再松开SW9（即en）写入寄存器；再按下SW8,可以看到右移一次后的值已经写入寄存器，此时再按下一次SW9，就可以得到一个随机数。

##### 实验结果：

已经提交到分支tracer-ysyx下的homework/example_Shifter：

代码文件名为：example_Shifter

git log为  NVBoard_Shifter

![image-20260227214356179](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260227214356179.png)

#### 4)实验七 状态机及键盘输入

##### 实验内容&实现情况：

- 实现：
  - 七段数码管低两位显示当前按键的键码，中间两位显示对应的ASCII码（引入ROM，设置了@键码 ASCII码）；
  - 七段数码管的高两位显示按键的总次数。按住不放只算一次按键。只考虑顺序按下和放开的情况，不考虑同时按多个键的情况.

- 未能实现：
  - 当按键松开时，七段数码管的低四位全灭————目前只能按下和松开都显示键盘码，直到按下另一个键码，才会更改显示；

##### 实验结果：

按下键码`a`，seg3-seg2显示ASCII为`61`；seg7-seg6显示按键次数，当前为4次；seg1-seg0显示键码为`1C`.

![image-20260227220121455](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260227220121455.png)

已经提交到分支tracer-ysyx下的homework/example_kbd：

代码文件名为：example_kbd

git log为  NVBoard_kbd



#### 5)实验八 VGA接口控制器实现。

##### 实验内容&实现：

- 利用NVBoard，在显示器上显示一张静态图片————本质上我只是修改了NVBoard的读取图片地址的参数，其余的都是借用原有的NVBoard上的默认设置
- 将图片转化为hex文件————利用deepseek写了一个python可以将png转化为hex，设定size为640*480,颜色数据为24bit，长和宽位之和不得超过19.

##### 我的bug：

1）由于NVBoard的图片位置的大小是640x480，所以 转化hex文件时就设定size为640x480：

```bash
Yang@yyl-Ubuntu:~$ ./img2mif.py  city.png city.mif --size 640x480 --mode rgb
原始图片尺寸: 3840x2160
图片已缩放至: 640x480
MIF 文件已保存至: city.mif
HEX 文件已保存至: city.hex
最终尺寸: 640x480, 数据位宽: 24, 深度: 307200
```

2）由于NVBoard初始的读取图片的地址是 列优先，且是(x,y)拼接起来的，高度必须是2的幂次方，我转化的hex文件并不符合这个读取规则，于是需要更改如下：

```verilog
//assign vga_data = vga_mem[{h_addr[9:0], v_addr[8:0]}];
wire [18:0] addr = v_addr * 19'd640 + {9'b0, h_addr}; // 图片宽度640或任意
assign vga_data = vga_mem[addr];
```

上述转化规则是：行优先，且可以是任意宽度。由于NVBoard的图片设置默认为640x480，所以设置宽度为640.

##### 实验结果：

未改变地址读取方式前：

![image-20260226171348159](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260226171348159.png)

改变地址读取方式后：

![image-20260226171427618](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260226171427618.png)

已经提交到分支tracer-ysyx下的homework/example_vga：

代码文件名为：example_vga

git log为  NVBoard_vga



## 用RTL实现简单处理器

### Q：用RTL实现sCPU

> [!IMPORTANT]
>
> 尝试把sCPU作为NPC的设计目标, 根据之前你用Logisim设计的sCPU, 用RTL重新设计它, 用于计算`1+2+...+10`. 为了看到输出效果, 你可以在sCPU中添加`out rs`指令, 用于通过NVBoard的七段数码管显示数列求和的计算结果.
>
> 和sEMU不同, 我们暂时不要求你实现更强大的运行时环境, 目前只需要让这个处理器支持`1+2+...+10`的计算即可. 我们会在D阶段介绍运行时环境的更多相关内容.

##### 设计思路：

###### 1）分层设计：

分为了几个部分和一个顶层调用`top.v` ：译码`Decoder.v`,PC寄存器`pc.v`，执行`Execute.v`，结果显示`seg.v`；

并且设置常数num=10，其实也可以用Reg[0]表示，但是又想到可维护代码，故把num单独拿了出来，但其实可以不用，最后改的话其实需要更改ROM里的指令。

###### 2）NVBoard连接设计：

seg1-seg0显示当前PC；SW0是en端；seg4-seg2显示sum的十六进制，LD0表示当前clk，LD1表示当前的PC是否需要跳转。



##### 我的bug：

###### 1）bner0指令的处理

这个最麻烦，我一开始就在想是在`pc.v`里面进行PC变换，还是在`Execute.v`里进行。我先在`Execute.v`里面进行尝试，后面感觉这样就失去分层在PC寄存器`pc.v`的意义了，所以就让在`Execute.v`判断bner0,传出一个`jump_en`给`pc.v `,来表示当前是否需要跳转，如果`jump_en=1`,那么就选择`pc_jump`,否则就选择`pc_next+1`

###### 2）如何实现自动执行？

这个问题在上述问题中，选择“pc+1”部分，必须是` pc_next <= pc_next +4'h1` ，否则的话pc初始值为1,后面就停在pc=1不动了。

```verilog
pc_next <= pc_next +4'h1 ; //对
pc_next <= pc +4'h1 ;      //错
```



##### 实验结果：

###### 1）在终端测试，所监控变量的变化：

![img](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/b28af18d77188395115c04f2640b44d8.png)

可以明显看到`sum`的变化，从`1+2+…+10`累加的结果变化.

###### 2）在NVBoard的显示结果

运行后，可以看到seg1-seg0显示当前PC，再按下SW0,即en端，就可以在seg4-seg2看到sum的十六进制的显示，LD0表示clk，LD1表示当前的PC是否需要跳转：

![img](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/885eeb89760d7f68c5ad7894a4ead085.png)

##### 代码提交：

已经提交到分支tracer-ysyx下的homework/example_sCPU：

代码文件名为：example_sCPU

git log为  NVBoard_sCPU





### Q：对比sEMU和sCPU

> [!IMPORTANT]
>
> 针对sISA, 你已经分别实现了sEMU和sCPU. 尝试对比一下, 它们之间有什么不同?

#### A1：对比sEMU和sCPU

- 实现语言：sEMU是使用C语言实现的，sCPU是使用verilog实现的；
- 实现环境：sEMU如果加上图形功能，可以在AM环境下实现；sCPU未加入任何特殊环境；
- 实现角度：
  - 1）sEMU是完全拿C语言来模拟指令的功能，并不是拿电路的角度来设计处理器，更像是是用C语言描述指令功能；
  - 2）sCPU则是站在电路的角度来设计的，虽然不是完全描述我之前在logisim设计的sCPU电路，但是很多细节和bug的处理都是站在电路的角度处理的，大体的设计思路也是在遵循电路的设计思路；



#### A2：额外讨论——sCPU在RTL实现和Logisim实现的区别和联系？

​	对比完全使用C语言设计，以及利用Logisim完全模拟实际电路设计，个人认为RTL设计兼备了两者的优点。

- 首先，RTL实现的运行速度远高于Logisim，这样很大程度上提高了设计的效率；

- 其次，画电路图 VS 修改verilog参数，肯定是修改参数简单，连电路非常容易连错，而且每次位数不匹配都需要重新修改，元器件受画图软件的限制，对比起来修改语言更容易也更高效；

- 最后，C语言设计的sEMU是无法流片的，只能说明是利用C语言模拟出来了和处理器一样的功能，但是无法落地；但是RTL实现的sCPU是可以进入流片的，并且评估出设计芯片的性能。

​	可以这么说，RTL设计处理器，比设计实际电路板更高效，也更节约成本，也可以在没有拿到芯片时就能估算出芯片的性能，并且作出调整。个人所见，RTL设计是完全面向设计实体芯片而诞生的。



### Q：评估sCPU的性能

> [!IMPORTANT]
>
> 尝试通过`yosys-sta`对你设计的sCPU进行评估.

选择了LVT（低阈值，高速高漏电）下，且也为经典情况`typ_tt_1p2_25`，可以在`~/yosys-sta/scripts/pdk/icsprout55.tcl`更改。

我把sCPU/vsrc放到了`~/yosys-sta/example`下，命名为`sCPU_vsrc`，之后更改`~/yosys-sta/Makefile`：

```
...
DESIGN ?= top  #第一处：修改顶层文件名称top.v
SDC_FILE ?= $(PROJ_PATH)/scripts/default.sdc 
RTL_FILES ?= $(shell find $(PROJ_PATH)/example/sCPU_vsrc -name "*.v") #第二处： example/sCPU_vsrc
...
```

第一处会找到顶层文件top.v；第二处会读取所有的.v文件，在终端运行`make sta`即可，在`~/yosys-sta/result/top-500MHz`中找到`top.pwr`和`top.rpt`:

根据`top.rpt`可以看出最大帧率为：1151.912 MHZ 

![img](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/c1627aa742fa851098a6fb7cef257b2d.png)

根据`top.pwr`可以看出，内部功耗为1.142e-3，占99.99%；  漏电功耗为8.455e-08 ，仅占不大0.01% ；总功耗为1.142e-03 W.

![img](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/832cd0185a033e6e949a44f749b41f3b.png)

由于并为要求作分析，所以就不分析了。







(nemu) p  1+2+)3+4(
[src/monitor/sdb/expr.c:358 eval] 没有找到运算符，表达式非法
[src/monitor/sdb/sdb.c:212 cmd_p] result= 0x3 
(nemu) p  (1+2+)3+4()
[src/monitor/sdb/expr.c:358 eval] 没有找到运算符，表达式非法
[src/monitor/sdb/expr.c:358 eval] 没有找到运算符，表达式非法
[src/monitor/sdb/sdb.c:212 cmd_p] result= 0 





state
