# E4  从C代码到二进制程序

## 预处理

预处理属于正式编译之前的步骤, 其本质是文本处理. 预处理主要包含以下工作:

- 头文件包含
- 宏替换
- 去掉注释
- 连接因断行符(行尾的`\`)而拆分的字符串
- 处理条件编译`#ifdef`/`#else`/`#endif`
- 处理字符串化操作符`#`
- 处理标识符连接操作符`##`

```
// a.c
#include <stdio.h>
#define MSG "Hello \
World!\n"
#define _str(x) #x
#define _concat(a, b) a##b
int main() {
  printf(MSG /* "hi!\n" */);
#ifdef __riscv
  printf("Hello RISC-V!\n");
#endif
  _concat(pr, intf)(_str(RISC-V));
  return 0;
}
```

从a.c的内容看，它应该输出一些东西。让我分析一下：

- \#include <stdio.h> 标准输入输出头文件。
- \#define MSG "Hello \ World!\n" 注意这里有换行符，但被反斜杠转义了，所以MSG是"Hello World!\n"，其中反斜杠用于续行。
- \#define _str(x) #x 这个宏将参数转换为字符串字面量。
- \#define _concat(a, b) a##b 这个宏连接两个标识符。

在main函数中：

- printf(MSG /* "hi!\n" */); 这里打印MSG，而注释部分被忽略，所以输出"Hello World!\n"。
- \#ifdef __riscv 检查是否定义了__riscv宏。如果定义了，就打印"Hello RISC-V!\n"。这通常是在RISC-V架构上编译时定义的。
- _concat(pr, intf)(_str(RISC-V)); 这个宏展开：_concat(pr, intf) 变成 printf，因为pr和intf连接成printf。然后 _str(RISC-V) 变成 "RISC-V"，所以这一行相当于 printf("RISC-V");。

### 1.观察预处理结果

> [!IMPORTANT]
>
> 观察预处理结果
>
> 尝试运行上述`gcc`命令, 然后对比预处理结果和源文件的区别.

多了很多我看不懂的字符，预处理内容要比源文件多得多。

### 2.`gcc`是如何找到头文件的? 

> [!IMPORTANT]
>
> 如何寻找头文件
>
> 1. 尝试执行`gcc -E a.c --verbose > /dev/null`, 并在输出的结果中寻找和头文件相关的内容.
> 2. 在`man gcc`中搜索`-I`选项相关的说明并阅读.
>
> 了解后, 尝试创建一些`stdio.h`文件, 然后通过`-I`选项让`gcc`包含你创建的`stdio.h`, 而不是标准库中的`stdio.h`. 通过`-E`选项来检查预处理结果是否符合你的预期.

#### 1.下面的指令是什么意思？

```
gcc -E a.c --verbose > /dev/null
```

答：对 a.c 文件进行预处理，verbose显示详细的编译过程，并且把标准输出(stdout)重定向保存到一个特殊的地方叫/dev/null，且不理会标准错误(stdder)。

#### 2.设定自己的头文件，并使用`gcc`预处理

```
gcc      -I    自定义头文件路径    源文件.c
```

注意：编写自己的头文件，比如定义为stdio.h，有风险

结果：不利于-I会自动调用gcc自带的stdio.h，如果利用-I进行预处理，那么会显示按照自己编写的头文件进行显示。

### 3.观察预处理结果(2)

> [!IMPORTANT]
>
> 观察预处理结果(2)
>
> 尝试安装面向RISC-V架构的`gcc`, 并用其进行预处理:
>
> ```bash
> apt-get install g++-riscv64-linux-gnu
> riscv64-linux-gnu-gcc -E a.c
> ```
>
> 查看此时的预处理结果, 你发现有什么新的变化?

#### 1.安装面向RISC-V架构的`gcc`

出错，需要使用sudo模式才能进行安装。

完整的安装步骤：

```bash
sudo apt update  #1.更新软件包列表（先获取最新信息）
sudo apt install g++-riscv64-linux-gnu   # 2.安装 RISC-V 交叉编译器
riscv64-linux-gnu-g++ --version   #3.验证安装
```

#### 2.使用RISC-V架构的`gcc`预处理

```
riscv64-linux-gnu-gcc -E a.c
```

结果和使用gcc预处理对比，多了一行输出：

```
# 7 "a.c"
int main() {
  printf("Hello World!\n" );

  printf("Hello RISC-V!\n");  #gcc预处理没有的，使用rv-gcc预处理后出现的

  printf("RISC-V");
  return 0;
}
```

可以通过以下命令查看所有预定义宏:

```bash
echo | gcc -dM -E - | sort
```

上述命令让`gcc`对一个空文件进行预处理, 然后打印出这个过程中的所有定义的宏, 并将其排序.

### 4.对比gcc和riscv64-linux-gnu-gcc的预定义宏

> [!IMPORTANT]
>
> 对比gcc和riscv64-linux-gnu-gcc的预定义宏
>
> 尝试对比`gcc`和`riscv64-linux-gnu-gcc`的预定义宏, 从而了解两者在预处理时的差异. 你只需要简单了解这些差异即可, 无需深入了解每一个宏的具体含义.
>
> Hint:
>
> - 使用`diff`或者相关的命令能帮助你快速找到两个文件的不同之处
> - 如果你想了解一些宏的含义, 可以查阅[`gcc`的相关手册](https://gcc.gnu.org/onlinedocs/cpp/Macros.html)

看了，是有点不一样。

diff指令（代整理）

## 编译

Q:了解编译的过程

> [!IMPORTANT]
>
> 了解编译的过程
>
> 尝试查阅`man clang`, 阅读其中关于编译阶段的介绍, 从而大致了解编译过程.



```
源代码 (.c/.cpp) 
    ↓
├─▶ 预处理 (Preprocessing)
    ↓
├─▶ 词法分析 (Lexical Analysis)
    ↓
├─▶ 语法分析 (Syntax Analysis)
    ↓
├─▶ 语义分析 (Semantic Analysis)
    ↓
├─▶ 抽象语法树生成 (AST Generation)
    ↓
├─▶ 中间代码生成 (IR Generation) → LLVM IR
    ↓
├─▶ 优化 (Optimization)
    ↓
├─▶ 代码生成 (Code Generation)
    ↓
├─▶ 汇编 (Assembly)
    ↓
└─▶ 链接 (Linking)
```

### 词法分析

词法分析的工作是识别并记录源文件中的每一个token, 包括标识符, 关键字, 常数, 字符串, 运算符, 大括号, 分号等. 若遇到非法的token(如`@`), 则报告错误. 可以通过如下命令来查看词法分析的结果:

```bash
clang -fsyntax-only -Xclang -dump-tokens a.c
```

可以看到, 词法分析的结果还通过`文件名:行号:列号`的格式记录了每一个token的位置.

### 语法分析

语法分析的工作是按照C语言的语法将识别出的token组织成树状结构, 从而梳理出源程序的层次结构, 从文件, 函数, 到语句, 表达式, 变量等. 若遇到语法错误(如漏了分号), 则报告错误. 语法分析的结果通常通过抽象语法树(Abstrace Syntax Tree, AST)的方式呈现. 可以通过如下命令来查看语法分析的结果:

```bash
clang -fsyntax-only -Xclang -ast-dump a.c
```

### 语义分析

“语法分析的工作是按照C语言的语义确定AST中每个表达式的类型. 在这个过程中, 相容的类型将根据C语言标准进行类型转换(如算术类型提升). 而对于不符合语义的情况, 则报告错误. 一些符合语法但不符合语义的情况包括未定义的引用, 运算符的操作数类型不匹配(如`struct mytype a; int b = a + 1;`), 函数调用参数的类型和数量不匹配等.”

语义分析：找出不符合语义的情况，然后报错，可以利用lint工具来进行。

类似地, 也可以调用`clang`的lint工具来分析上述程序:

```bash
clang a.c --analyze -Xanalyzer -analyzer-output=text
```

> [!CAUTION]
>
> 重视lint工具的作用
>
> 类似上文的代码造成的问题非常隐蔽, 软件很可能在运行很长一段时间后突然崩溃, 要调试是非常困难的. 因此, 大型项目通常都会充分利用lint工具, 来尽可能提升项目的质量.

```
clang a.c --analyze -Xanalyzer -analyzer-output=text 2>&1 | tee output.txt
```

这样可以将标准错误和标准输出都重定向到 `output.txt` 文件，并在终端显示。

### 中间代码生成

中间代码是一种由编译器定义的, 面向编译场景的ISA, 也称中间表示(Intermediate Representation, IR)或中间语言(Intermediate Language). 可以通过如下命令来查看`clang`生成的中间代码:

```bash
clang -S -emit-llvm a.c
cat a.ll
```

编译的工作主要是将C程序的状态机翻译成ISA的状态机, 也即, 将C程序的变量翻译成寄存器或内存, 将C程序的语句翻译成指令序列. 

那为什么不直接翻译到目标语言, 即处理器相关的ISA呢? ——减少编译器开发的工作量

一方面, 处理器相关的ISA有很多, 如果直接翻译到处理器相关的ISA, 并且要要对程序进行优化, 就要将一个优化技术分别实现到不同的ISA上, 增加了编译器的维护成本; 如果先翻译到中间代码, 再翻译到处理器相关的ISA, 就只需要将优化技术实现到中间代码上. 

另一方面, 中间代码还能作为多种源语言(如C语言, Fortran, Haskell等) 和多种目标语言(如x86, ARM, RISC-V等)之间的桥梁: 假设有种源语言和种目标语言, 如果直接翻译到目标语言, 就需要实现个翻译模块; 如果引入中间代码, 以中间代码为边界, 将编译器的流程分为前端(frontend)和后端(backend), 就只实现个翻译模块, 其中个前端模块分别负责将种源语言翻译到中间代码, 个后端模块分别负责将中间代码翻译到种目标语言.

### 编译优化

编译优化是现代软件构建过程中的重要步骤, 通过编译优化, 开发者可以将精力集中在程序业务逻辑的开发中, 而不必在开发阶段过多考虑程序的性能, 编译器通常能提供一个还不错的性能下限.       真实项目普遍都使用编译优化技术, 将来你也会在自己设计的处理器上运行各种经过编译优化的程序. 因此, 了解一些常见的优化技术, 知道编译器为什么会生成相应的指令序列, 将有助于将来开展调试和体系结构优化等工作.

#### 编译优化正确性的定义

我们可以从程序行为的角度来理解编译优化: 如果两个程序在某种意义上"一致", 就可以用"简单"的替代"复杂"的。    只要优化后仍然满足程序可观测行为的一致性, 这种优化都是"正确"的. 在这个条件下, 如果优化后的程序变量更少, 或者语句更少, 可以预期程序的性能表现就会更优.

#### 编译优化技术举例

- 消除冗余操作 - 对于那些没有被读出就被覆盖的赋值操作, 可将其移除，如下图：

```C
//          优化前              |            优化后
  int a;                       |    int a;
  a = 3;                       |    f();
  a = f();                     |    a = 10;
  a = 7;                       |
  a = 10;                      |
```

##### 可以进一步优化吗?

> [!NOTE]
>
> 可以进一步优化吗?
>
> 上例优化结果保留了函数调用`f()`, 能否进一步移除`f()`? 为什么?

答：我感觉不能优化掉  f()  ,  因为如果函数f()调用结束后直接返回了呢，如果移除掉，就无法得到正确的ISA

##### 可以进一步优化吗? (2)

```C
//          优化前              |            优化后
  int a = f1();                |    int x = f1() + 2;
  for (i = 0; i < 10; i ++) {  |    for (i = 0; i < 10; i ++) {
    int x = a + 2;             |      int y = f2(x);
    int y = f2(x);             |      sum += y + i;
    sum += y + i;              |    }
  }                            |
```

> [!NOTE]
>
> 可以进一步优化吗? (2)
>
> 上例优化结果的`f2(x)`仍然在循环中, 能否进一步将`f2(x)`提到循环之前进行计算? 为什么?

答：不可以，`x`可以提到循环体外部是因为`x=a+2=f1() + 2;`所有循环执行完`f1()`只调用了一次，但y在每一次循环都需要执行`f2()`,如果`f2()`内部有递增部分，`y`就会受到循环次数的影响，如果调到循环外部就会改变运行结果。

##### 可以进一步优化吗? (3)

- 死代码消除 - 对于不可达(unreachable)的代码或不再使用的变量, 可将其移除。

```C
//          优化前              |            优化后
  int f1(int x, int y) {       |    int f1(int x, int y) {
    return x + y;              |      return x + y;
  }                            |    }
  int f2(int x) {              |    int f2(int x) {
    return f1(x, 3);           |      return x + 3;
  }                            |    }
```

> [!NOTE]
>
> 可以进一步优化吗? (3)
>
> 上例中, 假设`f1(x, 3)`是在该源文件中对`f1()`的唯一调用, 能否对优化结果中的`f1()`应用死代码消除技术将其移除? 为什么?

答：不可以，因为是否有其他源文件引用该全局函数`f1(x, y)`,除非声明`f1(x, y)`是static（内部链接），不被其他源文件引用。

##### Q:对比编译优化的结果

可以在`clang`的命令行中给出`-O1`选项来开启更多的编译优化工作:

```bash
clang -S -emit-llvm -O1 a.c
cat a.ll #输出a.ll到shell
```

> [!IMPORTANT]
>
> 对比编译优化的结果
>
> 尝试对比添加`-O1`前后所生成的中间代码, 你发现添加`-O1`后, 生成的中间代码有何不同?

以下是部分截取，可以看出第一处不同是`a.ll`的第8～22行 被  `a_01.ll`  的第10行替换，可以看出a.c被优化了。

```bash
hoem:~$ diff a.ll a_01.ll >> diff.txt #对比两个中间生成代码文件
8,22c8,10
< ; Function Attrs: noinline nounwind optnone uwtable
...
> ; Function Attrs: nofree nounwind uwtable
26 c 14,15
...
28,29c17,18
...
31,32c20,21
...
38,39c27
...
```

##### Q:对比编译优化的结果(2)

> [!IMPORTANT]
>
> 对比编译优化的结果(2)
>
> 尝试在`int x = 10, y = 20;`之前添加`volatile`关键字, 并重新生成`-O1`的中间代码. 和之前生成的中间代码对比, 你发现此时的中间代码有何不同?

```bash
home:~$ diff a_01.ll a_01_v.ll >> diff_v.txt
```

a_01.ll：一个被优化（直接计算常量）的版本:常量传播和死代码消除等优化；

a_01_v.ll：另一个由于`volatile`而无法实现优化。

### 目标代码生成

目标代码生成的工作是将优化后的中间代码翻译成目标代码, 也即处理器相关的ISA.

可以通过如下命令来查看`clang`生成的目标代码，该`clang`命令会默认生成与本地环境相同ISA的汇编代码, 如x86：

```bash
clang -S a.c
cat a.s
```

 也可以向`clang`提供编译选项`--target=xxx`来生成对应的汇编代码, 例如, 可以通过`--target=riscv64-linux-gnu`来让`clang`生成riscv64的汇编代码:

```bash
clang -S a.c --target=riscv64-linux-gnu
```

这种生成与本地环境不同的ISA汇编代码的编译过程, 称为"交叉编译"(cross-compilation).

#### Q:理解C代码与riscv指令序列的关联

> [!IMPORTANT]
>
> 理解C代码与riscv指令序列的关联
>
> 阅读`clang`交叉编译得到的riscv64汇编代码, 并尝试指出哪一段汇编代码是由哪一段C代码编译得到的.

```bash
clang -S a.c --target=riscv64-linux-gnu
clang -S -O0 a.c --target=riscv64-linux-gnu -o a_O0.s
```

(待定)

#### Q:理解C代码与riscv指令序列的关联(2)

> [!IMPORTANT]
>
> 理解C代码与riscv指令序列的关联(2)
>
> 添加`-O1`并重新编译得到riscv64汇编代码, 你发现生成的汇编代码有何不同? 它如何与C代码建立关联?

```bash
clang -S a.c --target=riscv64-linux-gnu  #未优化00
clang -S -O0 a.c --target=riscv64-linux-gnu -o a_O0.s  #未优化00
clang -S -O1 a.c --target=riscv64-linux-gnu -o a_O1.s  #优化编译01
```

## 二进制文件的生成和执行

### 汇编 & 反汇编

汇编(assemble)需要将汇编代码转变成指令的二进制编码，完成这一工作的工具称为汇编器,即a.s 变为 a.o 

反汇编(disassemble)即 从机器码（目标文件或可执行文件中的二进制指令）  变为 a_dis.o

#### X86

可以通过如下命令来让`clang`生成汇编后的==<u>目标文件</u>==(x86):

```bash
clang -c a.c  #-c只编译，但不链接
ls a.o
```

但文本编辑器或Vim都无法正确读出目标文件的内容. 为了查看目标文件中的内容, 要一些面向二进制文件的解析工具. 例如, 我们可以利用`binutils(Binary Utilities)`工具包中的`objdump`工具来解析目标文件,从目标文件中重新解析出汇编代码的过程称为"==<u>反汇编</u>=="(disassemble):

```bash
objdump -d a.o
objdump -d a.o>>a_dis_x86.s>>a_dis_x86.txt
```

解析x86目标文件，得到反汇编文件：

```bash
home:~$ objdump -d a.o>>a_dis_x86.s>>a_dis_x86.txt
a.o:     file format elf64-x86-64
Disassembly of section .text:
0000000000000000 <main>:
   0:	55                   	push   %rbp
   1:	48 89 e5             	mov    %rsp,%rbp
   4:	48 83 ec 10          	sub    $0x10,%rsp
   8:	c7 45 fc 00 00 00 00 	movl   $0x0,-0x4(%rbp)
   f:	c7 45 f8 0a 00 00 00 	movl   $0xa,-0x8(%rbp)
  16:	c7 45 f4 14 00 00 00 	movl   $0x14,-0xc(%rbp)
  1d:	8b 45 f8             	mov    -0x8(%rbp),%eax
  20:	03 45 f4             	add    -0xc(%rbp),%eax
  23:	89 45 f0             	mov    %eax,-0x10(%rbp)
  26:	8b 75 f0             	mov    -0x10(%rbp),%esi
  29:	48 8d 3d 00 00 00 00 	lea    0x0(%rip),%rdi        # 30 <main+0x30>
  30:	b0 00                	mov    $0x0,%al
  32:	e8 00 00 00 00       	call   37 <main+0x37>
  37:	31 c0                	xor    %eax,%eax
  39:	48 83 c4 10          	add    $0x10,%rsp
  3d:	5d                   	pop    %rbp
  3e:	c3                   	ret
```

#### riscv64

为了得到riscv64的==<u>目标文件</u>== ，我们需要进行交叉编译，之后再==<u>反汇编</u>==：

```bash
clang -c a.c --target=riscv64-linux-gnu
riscv64-linux-gnu-objdump -d a.o>>a_dis_rv64.s>>a_dis_rv64.txt
```

或者用LLVM的工具链来进行==<u>反汇编</u>==, 它可以自动识别目标文件对应的ISA架构, 用起来更方便:

```bash
llvm-objdump -d a.o  # 反汇编：x86和RISC-V的目标文件均支持 
```

类似地, 我们也可以用`gcc`来生成==<u>目标文件</u>==:

```bash
gcc -c a.c  #X86汇编
riscv64-linux-gnu-gcc -c a.c  #rv64汇编
```

##### Q:查看riscv64目标文件的反汇编结果

> [!IMPORTANT]
>
> 查看riscv64目标文件的反汇编结果
>
> 根据上述命令, 查看`riscv64`目标文件的反汇编结果, 并将其与编译器生成的汇编文件进行对比.

生成反汇编a_dis_rv64.s，并输出到a_dis_rv64.txt：

```bash
home:~$ clang -c a.c --target=riscv64-linux-gnu
riscv64-linux-gnu-objdump -d a.o>>a_dis_rv64.s>>a_dis_rv64.txt

a.o:     file format elf64-littleriscv
Disassembly of section .text:
0000000000000000 <main>:
   0:	7179                	addi	sp,sp,-48
   2:	f406                	sd	ra,40(sp)
   4:	f022                	sd	s0,32(sp)
   6:	1800                	addi	s0,sp,48
   8:	4501                	li	a0,0
   a:	fca43c23          	sd	a0,-40(s0)
   e:	fea42623          	sw	a0,-20(s0)
  12:	4529                	li	a0,10
  14:	fea42423          	sw	a0,-24(s0)
  18:	4551                	li	a0,20
  1a:	fea42223          	sw	a0,-28(s0)
  1e:	fe842503          	lw	a0,-24(s0)
  22:	fe442583          	lw	a1,-28(s0)
  26:	9d2d                	addw	a0,a0,a1
  28:	fea42023          	sw	a0,-32(s0)
  2c:	fe042583          	lw	a1,-32(s0)

0000000000000030 <.Lpcrel_hi0>:
  30:	00000517          	auipc	a0,0x0
  34:	00050513          	mv	a0,a0
  38:	00000097          	auipc	ra,0x0
  3c:	000080e7          	jalr	ra # 38 <.Lpcrel_hi0+0x8>
  40:	fd843503          	ld	a0,-40(s0)
  44:	70a2                	ld	ra,40(sp)
  46:	7402                	ld	s0,32(sp)
  48:	6145                	addi	sp,sp,48
  4a:	8082                	ret
```

汇编a.rv64.s，如下：

```RISC-V汇编语言
	.text
	.attribute	4, 16
	.attribute	5, "rv64i2p1_m2p0_a2p1_f2p2_d2p2_c2p0_zicsr2p0"
	.file	"a.c"
	.globl	main                            # -- Begin function main
	.p2align	1
	.type	main,@function
main:                                   # @main
	.cfi_startproc
# %bb.0:
	addi	sp, sp, -48
	.cfi_def_cfa_offset 48
	sd	ra, 40(sp)                      # 8-byte Folded Spill
	sd	s0, 32(sp)                      # 8-byte Folded Spill
	.cfi_offset ra, -8
	.cfi_offset s0, -16
	addi	s0, sp, 48
	.cfi_def_cfa s0, 0
	li	a0, 0
	sd	a0, -40(s0)                     # 8-byte Folded Spill
	sw	a0, -20(s0)
	li	a0, 10
	sw	a0, -24(s0)
	li	a0, 20
	sw	a0, -28(s0)
	lw	a0, -24(s0)
	lw	a1, -28(s0)
	addw	a0, a0, a1
	sw	a0, -32(s0)
	lw	a1, -32(s0)
.Lpcrel_hi0:
	auipc	a0, %pcrel_hi(.L.str)
	addi	a0, a0, %pcrel_lo(.Lpcrel_hi0)
	call	printf
                                        # kill: def $x11 killed $x10
	ld	a0, -40(s0)                     # 8-byte Folded Reload
	ld	ra, 40(sp)                      # 8-byte Folded Reload
	ld	s0, 32(sp)                      # 8-byte Folded Reload
	addi	sp, sp, 48
	ret
.Lfunc_end0:
	.size	main, .Lfunc_end0-main
	.cfi_endproc
                                        # -- End function
	.type	.L.str,@object                  # @.str
	.section	.rodata.str1.1,"aMS",@progbits,1
.L.str:
	.asciz	"z = %d\n"
	.size	.L.str, 8

	.ident	"Ubuntu clang version 18.1.3 (1ubuntu1)"
	.section	".note.GNU-stack","",@progbits
	.addrsig
	.addrsig_sym printf
```

反汇编a_dis_rv64.s   VS    汇编a.rv64.s

- 反汇编得到的汇编代码与原始汇编代码不完全相同，一些符号和注释信息丢失l；
- 反汇编是静态分析工具，不能完全恢复原始源代码的所有结构（如函数、变量名等，除非有调试信息）

### 链接

链接的工作是将多个目标文件合并成最终的可执行文件. 

可以通过如下命令来让`clang`生成链接后的==可执行文件==(x86):

```bash
clang a.c
ls a.out
```

可执行文件同样可以通过`objdump`进行反汇编.

```bash
llvm-objdump -d a.out
```

不过你会发现, 和链接前的目标文件(`a.o`)相比, 链接后的可执行文件中多出了不少内容. 为了进一步了解这些内容来自哪里, 我们可以查看`clang`命令执行时的日志:

```bash
clang a.c --verbose
```

在日志末尾可以看到链接相关的命令, 在该命令中还包含了若干命名形如`crt*.o`的目标文件. 此处的`crt`是`C runtime`的缩写, 表示C程序的运行时环境. 也即, 链接过程会将`a.c`编译和汇编后得到的目标文件, 和已有的C程序运行时环境相关的目标文件进行合并, 最终生成可执行文件. 可以预见, 可执行文件之所以要包含这些和运行时环境相关的目标文件, 是为了向可执行文件的执行提供必要的支持.

类似地, 我们也可以用`gcc`来生成可执行文件:

```bash
gcc a.c
```

也可以通过交叉编译来生成`riscv64`的可执行文件:

```bash
clang a.c --target=riscv64-linux-gnu
riscv64-linux-gnu-gcc a.c
```

#### Q: 查看riscv64可执行文件的反汇编结果

> [!IMPORTANT]
>
> 查看riscv64可执行文件的反汇编结果
>
> 尝试生成`riscv64`的可执行文件, 查看其反汇编结果, 并将其与链接前的目标文件进行对比.

```bash
clang a.c --target=riscv64-linux-gnu
riscv64-linux-gnu-gcc a.c   #生成rv64的可执行文件a.out
riscv64-linux-gnu-objdump -d a.out >> a_dis_out_rv64.s #生成反汇编并保存
clang a.c --verbose  #查看clang日志
```

运行`clang a.c --verbose >> verbose_txt`，但输出没有写入文件。是因为`--verbose`的输出是写入标准错误流（stderr）而不是标准输出流（stdout），而`>>`重定向只能捕获标准输出。因此，需要将标准错误流也重定向到文件。

解决方案：将标准错误流重定向到标准输出流，然后一起写入文件，

**`2>&1` 的核心思想**：先定标准输出的去向，再让错误输出跟上，让错误信息跟着正常输出去同一个地方。

```bash
home:~$ clang a.c --verbose >> verbose_txt 2>&1

Ubuntu clang version 18.1.3 (1ubuntu1)
Target: x86_64-pc-linux-gnu
Thread model: posix
InstalledDir: /usr/bin
Found candidate GCC installation: /usr/bin/../lib/gcc/x86_64-linux-gnu/13
Selected GCC installation: /usr/bin/../lib/gcc/x86_64-linux-gnu/13
Candidate multilib: .;@m64
Selected multilib: .;@m64
 "/usr/lib/llvm-18/bin/clang" -cc1 -triple x86_64-pc-linux-gnu -emit-obj -mrelax-all -dumpdir a- -disable-free -clear-ast-before-backend -disable-llvm-verifier -discard-value-names -main-file-name a.c -mrelocation-model pic -pic-level 2 -pic-is-pie -mframe-pointer=all -fmath-errno -ffp-contract=on -fno-rounding-math -mconstructor-aliases -funwind-tables=2 -target-cpu x86-64 -tune-cpu generic -debugger-tuning=gdb -fdebug-compilation-dir=/home/yylinux -v -fcoverage-compilation-dir=/home/yylinux -resource-dir /usr/lib/llvm-18/lib/clang/18 -internal-isystem /usr/lib/llvm-18/lib/clang/18/include -internal-isystem /usr/local/include -internal-isystem /usr/bin/../lib/gcc/x86_64-linux-gnu/13/../../../../x86_64-linux-gnu/include -internal-externc-isystem /usr/include/x86_64-linux-gnu -internal-externc-isystem /include -internal-externc-isystem /usr/include -ferror-limit 19 -fgnuc-version=4.2.1 -fskip-odr-check-in-gmf -faddrsig -D__GCC_HAVE_DWARF2_CFI_ASM=1 -o /tmp/a-d820c1.o -x c a.c
clang -cc1 version 18.1.3 based upon LLVM 18.1.3 default target x86_64-pc-linux-gnu
ignoring nonexistent directory "/usr/bin/../lib/gcc/x86_64-linux-gnu/13/../../../../x86_64-linux-gnu/include"
ignoring nonexistent directory "/include"
#include "..." search starts here:
#include <...> search starts here:
 /usr/lib/llvm-18/lib/clang/18/include
 /usr/local/include
 /usr/include/x86_64-linux-gnu
 /usr/include
End of search list.
 "/usr/bin/ld" -z relro --hash-style=gnu --build-id --eh-frame-hdr -m elf_x86_64 -pie -dynamic-linker /lib64/ld-linux-x86-64.so.2 -o a.out /lib/x86_64-linux-gnu/Scrt1.o /lib/x86_64-linux-gnu/crti.o /usr/bin/../lib/gcc/x86_64-linux-gnu/13/crtbeginS.o -L/usr/bin/../lib/gcc/x86_64-linux-gnu/13 -L/usr/bin/../lib/gcc/x86_64-linux-gnu/13/../../../../lib64 -L/lib/x86_64-linux-gnu -L/lib/../lib64 -L/usr/lib/x86_64-linux-gnu -L/usr/lib/../lib64 -L/lib -L/usr/lib /tmp/a-d820c1.o -lgcc --as-needed -lgcc_s --no-as-needed -lc -lgcc --as-needed -lgcc_s --no-as-needed /usr/bin/../lib/gcc/x86_64-linux-gnu/13/crtendS.o /lib/x86_64-linux-gnu/crtn.o
```

### 执行

对于x86, 编译出可执行文件后, 通过以下命令来执行:

```bash
home:~$ ./a.out
z=30
```

#### Q: 对比编译优化前后的性能差异

> [!IMPORTANT]
>
> 对比编译优化前后的性能差异
>
> 我们之前介绍了各种编译优化的选项, 现在你可以来体会一下这些选项的威力了. 以之前介绍的数列求和程序为例, 你可以测量不同编译优化等级下程序的运行时间, 从而体会不同优化等级对程序性能的影响.
>
> 不过之前的数列求和程序的末项为`10`, 为了方便测量出性能的区别, 你需要调整程序的末项, 以增加其执行时间. 如果程序的末项较大, 你还可以将`sum`变量的类型修改为`long long`. 你可以通过`time`命令来测量一条命令的执行时间, 如`time ls`将报告`ls`命令的执行时间.
>
> 调整数列的末项后, 分别在`-O0`, `-O1`, `-O2`下编译并测量程序的运行时间.
>
> 如果你感兴趣, 你还可以通过反汇编来查看相应的汇编代码, 并尝试根据汇编代码理解: 为什么会得到相应的性能提升? 编译器可能应用了哪些编译优化技术? 不过为了回答这些问题, 你可能需要通过RTFM或STFW来了解一些汇编指令的功能.

1~1000的数列求和C程序sum.c：

```C
#include <stdio.h> #数列求和
   int main() {
   long long sum = 0;
   int i = 1;
   do {
     sum = sum + i;
     i = i + 1;
   } while (i <= 1000);
   printf("sum = %d\n", sum);
   return 0;
}
```

O0优化即不进行任何编译优化，于是分别进行O1,O2优化：

```bash
clang -S -emit-llvm -O0 sum.c

clang -S -emit-llvm -O1 sum.c

clang -S -emit-llvm -O2 sum.c
```

延伸Q：clang命令进行编译优化O1和O2有啥区别呢？以及O3呢？

答：区别在于编译优化的力度，O1 < O2 < O3，他们的编译时间逐个递增，优化强度逐个递增，一般O2是平衡的优化选择。



测试程序的运行时间：

```bash
sudo apt install llvm   #安装 lli（如果还没有），已安装

time lli sum_O0.ll  #直接解释执行 LLVM IR
time lli sum_O1.ll
time lli sum_O2.ll
```

运行了三次取平均值，其中real为真实运行时间，user为用户态运行时间，sys为内核态运行时间，可以看出O1的real运行时间最短，说明该程序O1优化最好：

```bash
sum = 500500
real	0m0.020s
user	0m0.012s
sys	0m0.011s

sum = 500500
real	0m0.015s
user	0m0.008s
sys	0m0.007s

sum = 500500
real	0m0.018s
user	0m0.011s
sys	0m0.006s
```



#### Q：程序真的从main() 开始执行 / 返回后结束 的吗?（1）（2）

> [!IMPORTANT]
>
> （1）程序真的从main()开始执行吗?
>
> （2）程序真的从main()返回后结束吗?
>
> 尝试用strace或gdb验证你的想法.
>
> Hint: gdb可使用`starti`命令, 让程序在第一条指令处暂停.

```
clang sum.c -o sum #编译并链接，生成.out文件
```

启用gdb交互，`gdb ./可执行文件名称`：

```bash
home:~$ gdb ./sum.out

成功启动gdb后启用交互...

(gdb) starti
Starting program: /home/yylinux/sum.out 

Program stopped.
0x00007ffff7fe4540 in _start () from /lib64/ld-linux-x86-64.so.2
```

答1：程序实际上是在从动态链接器（[ld-linux-x86-64.so](https://ld-linux-x86-64.so/).2）开始，函数`in _start ()`开始，而不是`main()`。

答2：程序实际上是在 **`exit_group()` 系统调用** 中真正结束的，这是在 C 库的 `exit()` 函数中调用的，而 `exit()` 是在 `main()` 返回后被调用的。

```bash
home:~$ gdb ./sum.out
GNU gdb (Ubuntu 15.0.50.20240403-0ubuntu1) 15.0.50.20240403-git
...(启动gdb交互)

(gdb) break main
Breakpoint 1 at 0x1144

(gdb) run
Starting program: /home/yylinux/sum.out 
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
Breakpoint 1, 0x0000555555555144 in main ()

(gdb) break exit #在exit设置断点
Breakpoint 2 at 0x7ffff7c47ba0: file ./stdlib/exit.c, line 137.

(gdb) continue
Continuing.
sum = 500500

Breakpoint 2, __GI_exit (status=0) at ./stdlib/exit.c:137
warning: 137	./stdlib/exit.c: No such file or directory

(gdb) where  #显示程序暂停的位置
#0  __GI_exit (status=0) at ./stdlib/exit.c:137
#1  0x00007ffff7c2a1d1 in __libc_start_call_main (main=main@entry=0x555555555140 <main>, argc=argc@entry=1, 
    argv=argv@entry=0x7fffffffdcf8) at ../sysdeps/nptl/libc_start_call_main.h:74
#2  0x00007ffff7c2a28b in __libc_start_main_impl (main=0x555555555140 <main>, argc=1, argv=0x7fffffffdcf8, 
    init=<optimised out>, fini=<optimised out>, rtld_fini=<optimised out>, stack_end=0x7fffffffdce8)
    at ../csu/libc-start.c:360
#3  0x0000555555555075 in _start ()
```

延伸知识点：其他gdb 命令序列

```
(gdb) starti                    # 在第一条指令暂停
(gdb) x/i $pc                   # 查看当前指令
(gdb) backtrace                 # 查看调用栈
(gdb) info files                # 查看入口点信息
(gdb) break main                # 在main函数设置断点
(gdb) continue                  # 继续执行
(gdb) backtrace                 # 查看是否在main中
(gdb) q                         # 退出
```



## 手册行为和编码规范

以[C99](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf)为例, 来了解C语言标准中的一些定义和概念。

### 标准规范的实现

标准规范的本质是一些定义和约定, 通常以手册作为载体呈现.

### 程序执行的语义

回顾计算机系统的状态机模型, C程序执行的过程就是通过C程序的语句改变程序状态的过程. 

函数的调用顺序和书写顺序无关，这个程序的输出是什么?

```c
#include <stdio.h>
int f() { printf("in f()\n"); return 1; }
int g() { printf("in g()\n"); return 2; }
int h() { printf("in h()\n"); return 3; }
int main () {
  int result = f() + g() * h();
  return 0;
}
```

事实上, 这个程序可能输出任意的函数调用顺序, **输出可能是任意的顺序，但数学结果总是7**，

这是因为, 在同一个表达式语句中, 多个函数调用之间是不确定序的, 因此它们可以按任意顺序调用. 如果程序的行为依赖于某种调用顺序, 程序的执行结果将可能不符合预期.

### 未指定行为(Unspecified Behavior)

C99手册对Unspecified Behavior的定义如下:

```text
use of an unspecified value, or other behavior where this International
Standard provides two or more possibilities and imposes no further requirements
on which is chosen in any instance
```

也即, 对于这类行为的结果, C语言标准提供了多种选择, 但没有规定选择哪一种, 具体实现可以从中选择一种.

C99手册中举了如下示例:

```text
An example of unspecified behavior is the order in which the arguments to a
function are evaluated
```

也即, ==<u>函数调用时参数求值顺序是未指定的</u>==. 我们可以通过以下程序来验证:

```c
// a1.c
#include <stdio.h>
void f(int x, int y) {
  printf("x = %d, y = %d\n", x, y);
}
int main() {
  int i = 1;
  f(i ++, i ++);
  return 0;
}
```

在yzh的系统中分别通过`gcc`和`clang`编译并运行程序, 结果如下:

```bash
$ gcc a.c && ./a.out
x = 2, y = 1
$ clang a.c && ./a.out
x = 1, y = 2
```

可以看到, 即使是同一份程序代码, 使用不同的编译器进行编译, 程序的运行结果会有所不同.

#### Q：体验未指定行为

> [!IMPORTANT]
>
> 体验未指定行为
>
> 尝试在你的系统中编译运行上述程序, 观察程序的结果.

不同结果的运行原理：

对于`i++`是先赋值后自增，所以在`f(i++,i++)`中，显示的是`i`赋值前的值，若先执行`x`，则`x=1,y=2,f(1,2)`(不自增)，若先执行`y`,则为`x=2,y=1,f(2,1)`.

进行修改成无论怎么编译都不会改变的代码：

```C
int i = 1;
  int x = i ++;
  int y = i ++;
  f(x, y);
```

### 实现定义行为(Implementation-defined Behavior)

这类行为是一类特殊的未指定行为, 但具体实现需要将行为的选择写进相关文档中.

### 区域特定行为(Locale-specific Behavior)

这类行为的结果依赖于国家地区, 文化和语言的本土习惯, 具体实现需要将行为的结果写入文档中, 是一类特殊的实现定义行为.

### 未定义行为(Undefined Behavior)

这是一类程序或数据不符合标准的错误行为, 但C语言标准对这种行为的结果不作任何约束, 也即, 无论结果如何, 都符合C语言标准. 通俗来说, 就是"一切皆有可能". C99手册列举了一些可能的结果:

这些结果包括:

- 编译时或执行时报错并退出
- 按照具体实现的文档要求来处理, 可能不报警告
- 完全无法预料的结果

一个未定义行为的例子是缓冲区溢出. 考虑如下程序:

```c
// a2.c
#include <stdio.h>
int main() {
  int a[10] = {0};
  printf("a[10] = %d\n", a[10]);
  return 0;
}
```

####  Q：体验未定义行为

> [!IMPORTANT]
>
> 体验未定义行为
>
> 尝试在你的系统中编译并多次运行上述程序, 观察程序的结果.

```bash
home:~$ ./a2
a[10] = 1231198440
home:~$ ./a2
a[10] = -85786344
home:~$ ./a2
a[10] = -1403205784
```

可以观察到，程序本身没什么错误，但运行多次的结果都不一样，这个时候就可以考虑：该程序存在未定义行为了.

### 应用程序二进制接口(Application Binary Interface)

程序, 编译器, 操作系统, 库函数, ISA这些概念, 它们作为一个计算机系统的整体, 彼此之间存在一定的关联. 这种关联一般通过约定和规范来呈现, 这就是[应用程序二进制接口](https://en.wikipedia.org/wiki/Application_binary_interface), 简称ABI, 也即, ABI是程序在二进制层面与上述概念之间的接口规范.

目前只需简单认识ABI即可, 将会在D阶段以RISC-V为例, 再展开介绍ABI的具体内容.

####  阅读官方手册(RTFM)

> [!CAUTION]
>
>  RTFM
>
> 当然, 不仅是[C语言标准](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf)和以[C99](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf)的手册, 我们也推荐大家在茶余饭后阅读其他手册, 包括[RISC-V手册](https://github.com/riscv/riscv-isa-manual/releases/download/20240411/unpriv-isa-asciidoc.pdf), [Verilog手册](http://staff.ustc.edu.cn/~songch/download/IEEE.1364-2005.pdf), [RISC-V的ABI手册](https://github.com/riscv-non-isa/riscv-elf-psabi-doc/blob/master/riscv-cc.adoc)等, 它们能为你理解相关细节带来最全面, 最权威的帮助. 如果你发现阅读它们非常吃力, 不必担心, 当你随着学习的深入积累越来越多的基础知识后, 再回过头来阅读它们, 就会越来越顺利.

## 指令集模拟器 - 可以执行程序的程序

要用C程序的状态机实现ISA的状态机, 我们需要开发一个包含如下功能的C程序:

- 用C程序的状态实现ISA的状态, 也即, 用C程序的变量实现ISA的PC, GPR和内存
- 用C程序的状态转移规则实现ISA的状态转移规则, 也即, 用C语言语句实现指令的语义

这种仅仅从ISA层面实现指令行为的模拟器, 称为**指令集模拟器**. 类似地, 还有架构模拟器和电路模拟器, 不过我们暂时不会接触它们.

### sEMU的基本实现

下面以sISA为例说明如何实现指令集模拟器, 我们称相应的指令集模拟器为sEMU(simple EMUlator). 关于sISA的细节, 请查阅之前的讲义.

sEMU要用C程序的变量实现ISA的PC, GPR和内存, 这并不难:

```c
#include <stdint.h>
uint8_t PC = 0;
uint8_t R[4];
uint8_t M[16];
```

其中, 虽然在ISA层面中PC位宽为4位, 但C语言中不存在4位的基础数据类型, 故此处使用`uint8_t`; 此外, PC的位宽只有4位, 说明sISA的程序最多只能包含16条指令, 因此用于实现内存的数组`M`的大小只需设置为`16`即可.
