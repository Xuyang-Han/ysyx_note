# E4  从C代码到二进制程序

## 预处理

预处理属于正式编译之前的步骤, 其本质是文本处理. 

```C
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

从a.c的内容看：

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

```bash
gcc -E a.c >> a_c.txt
diff a.c a_c.txt >> diff.txt
```

```txt
1,6c1,819
< // a.c
< #include <stdio.h>
< #define MSG "Hello \
< World!\n"
< #define _str(x) #x
< #define _concat(a, b) a##b
...
```



### 2.`gcc`是如何找到头文件的?

### Q：如何寻找头文件

> [!IMPORTANT]
>
> 1. 尝试执行`gcc -E a.c --verbose > /dev/null`, 并在输出的结果中寻找和头文件相关的内容.
> 2. 在`man gcc`中搜索`-I`选项相关的说明并阅读.
>
> 了解后, 尝试创建一些`stdio.h`文件, 然后通过`-I`选项让`gcc`包含你创建的`stdio.h`, 而不是标准库中的`stdio.h`. 通过`-E`选项来检查预处理结果是否符合你的预期.

##### 1.下面的指令是什么意思？

```bash
gcc -E a.c --verbose > /dev/null
```

答：对 a.c 文件进行预处理，verbose显示详细的编译过程，并且把标准输出(stdout)重定向保存到一个特殊的地方叫/dev/null，且不理会标准错误(stdder)。

##### 2.设定自己的头文件，并使用`gcc`预处理

```bash
gcc   -I    自定义头文件路径    源文件.c
```

注意：编写自己的头文件，比如定义为stdio.h，有风险

结果：不利于-I会自动调用gcc自带的stdio.h，如果利用-I进行预处理，那么会显示按照自己编写的头文件进行显示。

### 3.  Q：观察预处理结果(2)

> [!IMPORTANT]
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



### 4.  Q：对比gcc和riscv64-linux-gnu-gcc的预定义宏

> [!IMPORTANT]
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

### Q:了解编译的过程

> [!IMPORTANT]
>
> 了解编译的过程
>
> 尝试查阅`man clang`, 阅读其中关于编译阶段的介绍, 从而大致了解编译过程.

编译过程如下：

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

语法分析的工作是按照C语言的语法将识别出的token组织成树状结构, 从而梳理出源程序的层次结构。

语法分析的结果通常通过抽象语法树(Abstrace Syntax Tree, AST)的方式呈现. 可以通过如下命令来查看结果:

```bash
clang -fsyntax-only -Xclang -ast-dump a.c
```

### 语义分析

语法分析的工作是按照C语言的语义确定AST中每个表达式的类型. 

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

```bash
clang a.c --analyze -Xanalyzer -analyzer-output=text 2>&1 | tee output.txt
```

这样可以将标准错误和标准输出都重定向到 `output.txt` 文件，并在终端显示。

### 中间代码生成

中间代码是一种由编译器定义的, 面向编译场景的ISA, 也称中间表示(Intermediate Representation, IR)或中间语言(Intermediate Language). 可以通过如下命令来查看`clang`生成的中间代码:

```bash
clang -S -emit-llvm a.c
cat a.ll
```



### 编译优化

编译优化是现代软件构建过程中的重要步骤, 通过编译优化, 开发者可以将精力集中在程序业务逻辑的开发中, 而不必在开发阶段过多考虑程序的性能。

#### 编译优化正确性的定义

我们可以从程序行为的角度来理解编译优化: 如果两个程序在某种意义上"一致", 就可以用"简单"的替代"复杂"的。    

只要优化后仍然满足程序可观测行为的一致性, 这种优化都是"正确"的. 在这个条件下, 如果优化后的程序变量更少, 或者语句更少, 可以预期程序的性能表现就会更优.

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

##### 1.可以进一步优化吗?

> [!NOTE]
>
> 上例优化结果保留了函数调用`f()`, 能否进一步移除`f()`? 为什么?

答：我感觉不能优化掉  f()  ,  因为如果函数f()调用结束后直接返回了呢，如果移除掉，就无法得到正确的ISA

##### 2.可以进一步优化吗? (2)

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
> 上例优化结果的`f2(x)`仍然在循环中, 能否进一步将`f2(x)`提到循环之前进行计算? 为什么?

答：不可以，`x`可以提到循环体外部是因为`x=a+2=f1() + 2;`所有循环执行完`f1()`只调用了一次，但y在每一次循环都需要执行`f2()`,如果`f2()`内部有递增部分，`y`就会受到循环次数的影响，如果调到循环外部就会改变运行结果。

##### 3.可以进一步优化吗? (3)

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
> 尝试在`int x = 10, y = 20;`之前添加`volatile`关键字, 并重新生成`-O1`的中间代码. 和之前生成的中间代码对比, 你发现此时的中间代码有何不同?

```bash
diff a_01.ll a_01_v.ll >> diff_v.txt
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
> 阅读`clang`交叉编译得到的riscv64汇编代码, 并尝试指出哪一段汇编代码是由哪一段C代码编译得到的.

```bash
clang -S a.c --target=riscv64-linux-gnu
clang -S -O0 a.c --target=riscv64-linux-gnu -o a_O0.s
```

如下：

第一个`printf`调用：

```
.Lpcrel_hi0:
	auipc	a0, %pcrel_hi(.L.str)
	addi	a0, a0, %pcrel_lo(.Lpcrel_hi0)
	call	printf
```

```C
printf(MSG /* "hi!\n" */);
```



第二个`printf`调用：

```
.Lpcrel_hi1:
	auipc	a0, %pcrel_hi(.L.str.1)
	addi	a0, a0, %pcrel_lo(.Lpcrel_hi1)
	call	printf
```

```C
#ifdef __riscv
  printf("Hello RISC-V!\n");
#endif
```



第三个`printf`调用

```
.Lpcrel_hi2:
	auipc	a0, %pcrel_hi(.L.str.2)
	addi	a0, a0, %pcrel_lo(.Lpcrel_hi2)
	call	printf
```

```C
_concat(pr, intf)(_str(RISC-V));
```



开头和结尾：

 `main`函数的整体结构和`return 0;`



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

A：当使用 `-O1` 优化级别编译 C 代码为 RISC-V 汇编时，生成的指令序列会发生显著变化，其核心目标是**减少指令数量、提高执行效率，同时保持代码功能不变**。相较于未优化（`-O0`）的代码，`-O1` 生成的汇编更加简洁、高效，并与 C 代码的逻辑建立更直接的执行流映射。



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

但一般文本编辑器无法正确读出目标文件的内容. 为了查看内容, 需要二进制文件的解析工具. 

例如, 可用`binutils(Binary Utilities)`工具包中的`objdump`工具来解析目标文件,从目标文件中重新解析出汇编代码的过程称为"==<u>反汇编</u>=="(disassemble):

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

我的bug：

对于汇编和反汇编一开始搞混了，汇编是高级语言进行编译汇编为x86或rv32；返汇编是根据可执行程序反汇编出x86或rv32.



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

```bash
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

(gdb) break main #在main函数处设置断点
Breakpoint 1 at 0x1144

(gdb) run
Starting program: /home/yylinux/sum.out 
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
Breakpoint 1, 0x0000555555555144 in main ()

(gdb) break exit #在exit函数处设置断点
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

延伸知识点：gdb 命令序列

`(gdb) bt`追踪程序当前状态,以调用函数(帧)为单位：

```bash
(gdb) bt
#0  0x000056468844813b in actual_calc (a=13, b=0) at test.c:3
#1  0x0000564688448171 in calc () at test.c:12
#2  0x000056468844818a in main () at test.c:17
```

`(gdb f n)`检查某个帧；`(gdb list)`逐个检查每个帧；

```bash
(gdb) f 2 #检查帧2
#2  0x000055fa2323318a in main () at test.c:17
17    calc();

(gdb) list #逐个检查每个帧
12    actual_calc(a, b);
13    return 0;
14  }
15  
16  int main(){
17    calc();
18    return 0;
19  }
```

`(gdb) p 变量名`，打印该变量的值：

```bash
(gdb) f 1
#1  0x000055fa23233171 in calc () at test.c:12
12    actual_calc(a, b);
(gdb) list
7   int calc(){
8     int a;
9     int b;
10    a=13;
11    b=0;
12    actual_calc(a, b);
13    return 0;
14  }
15  
16  int main(){...}

(gdb) p a
$1 = 13
(gdb) p b
$2 = 0
(gdb) p c
No symbol "c" in current context.
(gdb) p a/b
Division by zero
```



```
(gdb) starti                    # 在第一条指令暂停
(gdb) break exit                # 在exit函数处设置断点
(gdb) break main                # 在main函数处设置断点
(gdb) break abort               # 在 abort() 函数处断点

(gdb) continue                  # 继续执行，直到结束
(gdb) run                       # 继续执行，直到下一个断点
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

####  Q：阅读官方手册(RTFM)

> [!CAUTION]
>
> RTFM
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

Q: 为什么不使用`int8_t`?

A: C语言标准规定, 有符号数溢出是未定义行为, 但无符号数不会溢出



sEMU要用C语言语句实现指令的语义, 需要编写一个函数`inst_cycle()`实现如下功能：

- 取指 - 直接根据`PC`索引内存`M`, 即可取出一条指令
- 译码 - 通过C语言的位运算抽取出指令的`opcode`字段, 并检查属于哪一条指令; 然后根据指令格式抽取出操作数字段, 并获得相应的操作数
- 执行 - 若执行的指令不是`bner0`, 则将结果写回目的寄存器; 否则, 根据判断情况决定是否进行跳转
- 更新PC - 若不跳转, 则让`PC`加1

实现`inst_cycle()`后, 只需要让sEMU不断调用它即可:

```c
while (1) { inst_cycle(); }
```

以"只运行数列求和程序"为前提, 来讨论运行时**环境支持**：

考虑用sISA计算`1+2+...+10`时提供的运行时环境

1. 在程序执行开始前

   - 加载程序：在程序执行开始前: 一个需要考虑的问题是如何加载程序. 但因为sEMU只打算运行数列求和这一个程序, 我们可以直接对`M`进行初始化:

     ```c
     uint8_t M[16] = { ... };
     ```

     由于目前数列求和程序的运行过程中并没有涉及参数, 故运行时环境可暂

   - 暂时不支持程序参数的传递

2. 在程序执行过程中

   - 没有库函数调用，故暂时不考虑库函数

3. 在程序执行结束后

   - 通过死循环结束 - `bner0`指令实现

#### Q：实现sEMU

> [!IMPORTANT]
>
> 根据上述思路, 用C代码实现sEMU, 并运行之前的数列求和程序. 由于数列求和程序本身不会结束, 你可以修改`while`语句的循环条件, 在循环一定的次数后退出循环, 然后检查数列求和的结果是否符合预期.

以下是  求和数列   的指令序列和机器码：

```
10001010    # 0: li r0, 10   # 这里是十进制的10
10010000    # 1: li r1, 0
10100000    # 2: li r2, 0
10110001    # 3: li r3, 1
00010111    # 4: add r1, r1, r3
00101001    # 5: add r2, r2, r1
11010001    # 6: bner0 r1, 4
11011111    # 7: bner0 r3, 7
```

##### 我的bug：

###### 1.怎么译码？

即，如何实现取PC的某几位

```C
 uint32_t op =M[PC_elc] << 25 >> 25 ;//取低7位
//一共32位，先左移25bit，再右移25bit，可以实现高25位清零，留下低7bit
```

可以利用这个特点取得任意nbit

###### 2.调用函数的定义要在主函数之前

定义的函数`inst_cycle()`必须写在main()之前，一般是从上之下加载，先写就先加载未定义的inst_cycle()，后写在main()就会无法识别`inst_cycle()`，所以必须先写在函数定义.

###### 3.头文件的使用

如果使用`printf()`，需要使用头文件`#include <stdio.h>`;

如果使用`uint8_t`，需要使用头文件`#include <stdint.h>`.



#### Q：实现输出功能

> [!IMPORTANT]
>
> 根据上文, 在sEMU中实现`out`指令, 并修改数列求和程序, 使得在计算出结果后通过`out`指令输出计算结果. 如果你的实现正确, 你应该能看到终端上显示`55`.

![image-20260117094504609](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260117094504609.png)

##### 设计out指令

指令out的操作码是01，rs2占最低2bit，中间使用0填充，机器码为01000010B=42H.

```
10001010    # 0: li r0, 10   # 这里是十进制的10
10010000    # 1: li r1, 0
10100000    # 2: li r2, 0
10110001    # 3: li r3, 1
00010111    # 4: add r1, r1, r3
00101001    # 5: add r2, r2, r1
11010001    # 6: bner0 r1, 4
01000010    # 7: out rs2
11011111    # 8: bner0 r3, 7
```

##### A：实现加入out指令版本

程序名：semu_out.c

程序备注：semu_out.c———加入out版本

已经提交到分支tracer-ysyx下的homework/sEMU/semu_out.c



#### Q：实现参数化的数列求和

> [!IMPORTANT]
>
> 修改数列求和程序, 使其从`r0`中获取数列的末项. 然后修改sEMU, 来将用户输入的参数放置在`r0`中. 实现后, 简单测试你的实现, 例如, 键入`./semu 10`, 程序应输出`55`; 键入`./semu 15`, 程序应输出`120`. 我们假设用户的输入不会导致计算过程溢出, 因此你不必考虑如何处理结果溢出的情况.

A：

##### 我的bug：

###### `uint_8`对应的`scanf`是`%hhd`；

###### 如何实现输入“./semu 10”，正确输出sum？

如果是写`scanf`是无法实现的，因`scanf`只能识别输入流，所以必须输入” ./semu  换行  10“才能正常识别.

所以要更改`int main(int argc, char *argv[])`，实现的几种办法：

- 方法1：头文件`<stdio.h>`，`sscanf(argv[1],"%hhd",&R[0]);`看成是从左往右进行转化，输入`char argv[1]`，转化为`%hhd`，最后存到`&R[0]`中.

- 方法2：头文件`<stdlib.h>`，`R[0] = (uint8_t)atoi(argv[1]);`，`(目标类型uint)atoi(初始类型char)`

  ```C
  #include <stdio.h>
  #include <stdint.h>
  //#include <stdlib.h>
  
  int main(int argc, char *argv[]){
      if(argc > 1)
      //方法2：
          // R[0] = (uint8_t)atoi(argv[1]);//需要头文件<stdlib.h>
  	//方法1
          sscanf(argv[1],"%hhd",&R[0]);  
      
     	printf("当前sum = %d\n", R[2]);//输出sum
      return 0;
  }
  ```



##### A：运行结果

```bash
home:~$ ./semu 10                                   
当前sum = 55                                                             
home:~$ ./semu 15      
当前sum = 120     
```

代码名称：semu_num.c

代码备注：//semu_num.c

已经提交到分支tracer-ysyx下的homework/sEMU/semu_num.c



### minirvEMU的实现

#### 以下的3道题同时实现：

#### Q：实现两条指令的minirvEMU

> [!IMPORTANT]
>
> 支持`addi`和`jalr`指令的minirvEMU, 并让其运行之前在Logisim上运行过的那个两条指令的测试程序.

#### Q：实现完整的minirvEMU

> [!IMPORTANT]
>
> 为minirvEMU添加剩余的6条指令, 并更新加载程序的方式, 然后运行之前在Logisim上运行过的`sum`和`mem`两个程序. 为了判断程序是否成功结束运行, 你可以参考之前在Logisim上的判断方式.

#### Q：实现程序结束的自动判断

> [!IMPORTANT]
>
> 根据上述运行时环境的约定, 在minirvEMU中添加并实现`ebreak`指令, 然后修改程序的指令序列, 使其在结束时执行`ebreak`指令. 如果你的实现正确, 你会看到程序自动结束并通过minirvEMU输出结束信息.

A：

##### 我的设计：

1）首先，指令实现：先根据F6的笔记把八条指令分别利用C语言实现；再寻找`halt`的位置，进行手动添加ebreak和bne指令的机器码；

2）其次，实现文件读入功能：根据不同文件名载入对于的ebreak位置；

3）最后，测试mem和sum：若成功返回“hit good trap”，否则返回“hit good trap”.



##### 我的bug

###### ebreak和bne添加在哪里？

按照提示，应该添加到`halt`附近，并且指令的条数不能改变，不然跳转地址全错了，所以只能修改原来的指令。改为：判断a0=0是否成立，成立则退出循环，所以原本的循环可以改成：判断a0=0，成立则ebreak。

###### ebreak指令——sel=8

查询手册我找到ebreak，如下：

![img](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/39e8f6bda7b86c19f39b6866084eb29c.png)

![img](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/11.png)

###### bne指令——sel=9

对于判断指令，我选择了bne，即不相等就跳转：

![img](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/155c04ed547b1ddc323bacb34c8b61fa.png)

![img](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/4d29a1dc6c68c5017e21c381bd48b373.png)

译码操作：

读rs1的值 和 读rs2的值，进行比较，

若相等，则pc = pc+“1” ;

若不相等，则pc = pc+imm .



下面是进行修改mem，sum和vga同理：

```
//mem
00001218 <halt>:
    1218:	00001237          	lui		tp,0x1
    121c:	21820213          	addi	tp,tp,536 # 1218 <halt>
    1220:	00020067          	jalr	zero,0(tp) # 0 <_start>
    
00001218 <halt>:
    1218:	00051463          	bne		a0,zero，0x8    #若a0不为零，则跳转
    121c:	90000002          	ebreak	
    1220:	00020067          	jalr	zero,0(tp) # 0 <_start>
```



###### 实现文件读入功能

利用`FILE *fp = fopen("mem.bin", "rb");`，`fopen(filename，读入类型)`读入叫“mem.bin”的文件，文件类型是二进制。

利用`fread(&b, size, num, fd) == num `来实现不断的读入文件所有内容，`fread(读出后存入的地址, 一次读出每个元素的大小 字节为单位, 一次读出的data包含多少个元素, 从哪里读文件fd) == 和num有关 `，直到读不出数据则跳出循环。

```C
int i=0;
while (i < SIZE && fread(&b, size, num, fd) == 4 ) {
	M[i]  = b;
	i++;}   
fclose(fd);
```



要是想实现动态读入文件名，那么就需要传入参数，故改`int main()`为`int main(int argc,char* argv[])`，`argv[1]`就是文件名，如下：

```C
//伪代码
FILE* fd=fopen(argv[1],"rb");
if(strcmp(argv[1], "mem.bin") == 0){
	M[(raddr)]=0x00000000; //bne的位置
	M[(raddr)]=0x00000000; //ebreak的位置
}else if(strcmp(argv[1], "sum.bin") == 0){
	M[(raddr)]=0x00000000; //bne的位置
	M[(raddr)]=0x00000000; //ebreak的位置 }
```

另外，可以使用`strcmp(str1,str2)`来比较字符串是否相等，相等则返回`0`，头文件是`include <string.h>`.

这里遇到的bug：这里输入的时候一开始我并没有输入文件名，直接运行./semu，结果报错，因为输入不全，所以系统直接报错。

​	

###### 有关C语言的类型转化

1）无符号数的%ud吗

A：不是，是%u

2）`imm`的加减实现

`raddr`是有符号数和无符号数运算后得到的，那么它是什么类型？

```C
uint32_t raddr,R[16];
int32_t imm_s;
raddr=(R[rs1]+imm_s)/4;
```

A: 如果是`uint_32`和`int_32`则向`uint_32`看齐，但是是这样，那么`(R[rs1]+imm_s)`就会自动变为无符号加减，导致计算结果不正确，因为`imm`可以是负数，所以需要类型转化。

变为：

```C
raddr=((int32_t)R[rs1]+imm_s)/4;
```

如果是`int`和`unsigned int`：当无符号数和有符号数一起运算时： 如果 `int` 能表示所有无符号数的值，则无符号数转换为 int 否则，两者都转换为 `unsigned int`.



###### hex变二进制bin

讲义里面的，积累一下

```bash
tail -n +2 sum.hex | sed -e 's/.*: //' -e 's/ /\n/g' | sed -e 's/\(..\)\(..\)\(..\)\(..\)/\4 \3 \2 \1/' | xxd -r -p > sum.bin
```



###### 实现ebreak指令

可以利用返回值来判断，比如结束`ebreak`后`return 2;`其他指令返回`0`，主函数的循环来判断返回的值是多少，若是`2`则`break`，若是`0`则继续循环。



###### 已经实现末2bit清零

对于设计计算`sel`的指令，比如`sb`和`lbu`指令，有一个重大的误区，就是我进行的`/4`整除运算已经给计算的地址清零，再赋给`sel`就会一直是`0`；

```C
raddr=(((int32_t)R[rs1]+imm)/4) << 30 >> 30 ;
sel=radd;
```

所以求`sel`必须要取余运算，对于`raddr`不再进行位移：

```C
sel=((int32_t)R[rs1]+imm) % 4;
raddr=(((int32_t)R[rs1]+imm)/4) ;//已经实现末2bit清零
```

#### A：运行结果

```
当前指令是ebreak，PC=0x121c
hit good trap
```

代码夹名称：minirv32

核心代码：/minirv32/semu.c

已经提交到分支tracer-ysyx下的homework/minirv32.c



### 支持GUI输入输出的程序

#### Q：运行超级玛丽

> [!IMPORTANT]
>
> 阅读PA1的`在开始愉快的PA之旅之前->NEMU是什么?`开头部分的讲义内容, 按照讲义指示尝试运行超级玛丽, 并完成画面, 按键和声音的检查.



##### 我的bug：

###### 1.哪个文件夹放哪个文件夹里？

在 AM 体系里，角色是分开的： 

`abstract-machine/` → 提供底层库和框架，即可以理解为是AM的环境配置

`fceux-am-master/ `→ 一个基于 AM 的应用工程，即运行在AM环境的app

所以它们俩是同级，而且工作目录在应用工程，即`fceux-am-master`里面的`Makefile`



###### 2.`AM_HOME`是什么，以及在那里？

```bash
home:~./fceux-am-master$ make ARCH=native mainargs=balloon run
Makefile:12: /home/Yang/ysyx-workbench/abstract-machine/Makefile: No such file or directory
make: *** No rule to make target '/home/Yang/ysyx-workbench/abstract-machine/Makefile'. Stop.

home:~$ echo $AM_HOME  #输出AM_HOME
/home/Yang/ysyx-workbench/abstract-machine
```

我出现的这个错误是因为AM_HOME不匹配：

AM_HOME 必须指向 abstract-machine 仓库目录，是在构建阶段，告诉 Makefile 去哪里找 AbstractMachine 的源码和构建规则，即依赖库源码。

构建系统需要两类东西：1)当前项目源码；2)依赖库源码

AM_HOME 就是把“依赖库源码路径”显式声明出来。

如果不声明，Makefile 不知道去哪 include，就会报错。

------

对AM_HOME进行修改：

```bash
home:~$ export AM_HOME=/home/Yang/ysyx/ysyx-workbench/abstract-machine
```

小记：该构建很依赖系统，如果可以自行判断abstract-machine 在哪里就好了，不然移动仓库位置就需要相应的改变AM_HOME.

所以永久生效，避免每次都 export：

```bash
vim ~/.bashrc #执行
#在.bashrc最后加一行
export AM_HOME=/home/Yang/ysyx/ysyx-workbench/abstract-machine
source ~/.bashrc #再执行
```

------

3.如何运行？

以下摘录自`fceux-am-master/README`：

运行方式：

将游戏ROM放置在`nes/rom/`目录下, 并命名为`xxx.nes`, 如`nes/rom/mario.nes`.
然后可通过`mainargs`选择运行的游戏, 如运行超级玛丽mario:

```bash
make ARCH=native run mainargs=mario
```

------

注意：在rom文件下只能有一个.nes文件，不然会无法识别运行哪一个而报错

```bash
home:~/ysyx/ysyx-workbench/fceux-am-master$ make ARCH=native mainargs=mario run
# Building fceux-run [native]
+ CXX src/emufile.cpp
+ CC nes/gen/mario.c
# Building am-archive [native]
# Building klib-archive [native]
# Creating image [native]
+ LD -> build/fceux-native.elf
/usr/bin/ld: /home/Yang/ysyx/ysyx-workbench/fceux-am-master/build/native/src/emufile.o:(.data.rel+0x8): undefined reference to `rom_balloon_nes'
/usr/bin/ld: /home/Yang/ysyx/ysyx-workbench/fceux-am-master/build/native/src/emufile.o:(.data.rel+0x10): undefined reference to `rom_balloon_nes_len'
/usr/bin/ld: /home/Yang/ysyx/ysyx-workbench/fceux-am-master/build/native/src/emufile.o:(.data.rel+0x38): undefined reference to `rom_mario3_nes'
/usr/bin/ld: /home/Yang/ysyx/ysyx-workbench/fceux-am-master/build/native/src/emufile.o:(.data.rel+0x40): undefined reference to `rom_mario3_nes_len'
collect2: error: ld returned 1 exit status
make: *** [/home/Yang/ysyx/ysyx-workbench/abstract-machine/Makefile:139: /home/Yang/ysyx/ysyx-workbench/fceux-am-master/build/fceux-native.elf] Error 1
```



#### Q：体验时钟功能

> [!IMPORTANT]
>
> 通过如下命令运行时钟测试程序:
>
> ```bash
> cd am-kernels/tests/am-tests
> make ARCH=native mainargs=t run
> ```
>
> 你会发现程序在终端上每隔1s输出一句话. 此外, 程序还会弹出一个画面全黑的新窗口, 但在当前程序中无任何功能, 目前你不必关心它.
>
> 体验上述功能后, 尝试阅读`am-kernels/tests/am-test/src/tests/rtc.c`, 理解上述功能是如何实现的. 其中, 代码`io_read(AM_TIMER_UPTIME).us`将获得程序运行以来经过的时间, 单位是us.



###### 理解时钟测试原理：

```C
void rtc_test() {
  AM_TIMER_RTC_T rtc;
  int sec = 1;
  while (1) {
    while(io_read(AM_TIMER_UPTIME).us / 1000000 < sec) ;
    rtc = io_read(AM_TIMER_RTC);
    printf("%d-%d-%d %02d:%02d:%02d GMT (", rtc.year, rtc.month, rtc.day, rtc.hour, rtc.minute, rtc.second);
    ...
    sec ++;
}
```

以上只截取核心部分，可以看出，这是第一个循环是死循环，即只要不退出黑框就会一直执行；第二个循环的条件是当前获取的时间是us转化为s，如果小于当前sec，则执行循环体。

所以第二个循环是以秒为最小单位来获取时间，每一次条件不满足就会跳出循环，更新`rtc`。

###### 测试结果：

```bash
home:~/ysyx/ysyx-workbench/am-kernels/tests/am-tests$ make ARCH=native mainargs=t run
# Building amtest-run [native]
# Building am-archive [native]
# Building klib-archive [native]
/home/Yang/ysyx/ysyx-workbench/am-kernels/tests/am-tests/build/amtest-native.elf
2026-2-12 16:31:28 GMT (1 second).
2026-2-12 16:31:29 GMT (2 seconds).
2026-2-12 16:31:30 GMT (3 seconds).
...
```



#### Q：体验按键功能

> [!IMPORTANT]
>
> 通过如下命令运行按键测试程序:
>
> ```bash
> cd am-kernels/tests/am-tests
> make ARCH=native mainargs=k run
> ```
>
> 你会发现程序弹出一个画面全黑的新窗口, 在新窗口中按下按键, 你将会看到程序在终端输出相应的按键信息, 包括按键名, 键盘码, 以及按键状态.
>
> 体验上述功能后, 尝试阅读`am-kernels/tests/am-test/src/tests/keyboard.c`, 理解上述功能是如何实现的(目前你可以忽略代码中`uart`相关的功能). 其中, 代码`io_read(AM_INPUT_KEYBRD)`将获得一个按键事件`ev`, `ev.keycode`表示按键的编码, `ev.keydown`表示按键为按下还是释放. 按键的编码值可查阅`abstract-machine/am/include/amdev.h`, 它们均以`AM_KEY_`为前缀, 如`A`键的编码为`AM_KEY_A`. 特别地, `AM_KEY_NONE`表示无按键事件.



###### 理解按键测试原理：

```C
 if (has_kbd) {
    while (1) {
      AM_INPUT_KEYBRD_T ev = io_read(AM_INPUT_KEYBRD);
      if (ev.keycode == AM_KEY_NONE) break;
      printf("Got  (kbd): %s (%d) %s\n", names[ev.keycode], ev.keycode, ev.keydown ? "DOWN" : "UP");
    }
  }
```

以上只截取核心部分，可以看出，这是第一个循环是死循环，即只要不退出黑框就会一直执行，在循环体内定义了一个叫ev的事件来表示键盘操作，接着判断当前是否有按下任意键盘，

1）如果没有则跳出循环。跳出主函数main后依旧会持续检测，也就是会再一次进入该循环；

```C
void keyboard_test() {
 ...
  while (1) {
    drain_keys();
...
```

2）如果有ev，则输出是：x键，按下x键，松开x键。



###### 测试结果：

```bash
home:~/ysyx/ysyx-workbench/am-kernels/tests/am-tests$ make ARCH=native mainargs=k run
....(构建信息)
# Building am-archive [native]
# Building klib-archive [native]
# Creating image [native]
+ LD -> build/amtest-native.elf
/home/Yang/ysyx/ysyx-workbench/am-kernels/tests/am-tests/build/amtest-native.elf
Try to press any key (uart or keyboard)...
Got  (kbd): RETURN (54) DOWN
Got  (kbd): RETURN (54) UP
Got  (kbd): Y (34) DOWN
Got  (kbd): Y (34) UP
Got  (kbd): R (32) DOWN
Got  (kbd): R (32) UP
...
```



#### Q：体验显示功能

> [!IMPORTANT]
>
> 通过如下命令运行显示测试程序:
>
> ```bash
> cd am-kernels/tests/am-tests
> make ARCH=native mainargs=v run
> ```
>
> 你会发现程序弹出一个新窗口并播放动画.
>
> 体验上述功能后, 尝试阅读`am-kernels/tests/am-test/src/tests/video.c`, 理解上述功能是如何实现的. 其中, 代码`io_write(AM_GPU_FBDRAW, x, y, pixels, w, h, true)` 表示向屏幕`(x, y)`坐标处绘制`w*h`的矩形图像. 图像像素按行优先方式存储在`pixels`中, 每个像素用32位整数以`00RRGGBB`的方式描述颜色.



###### 理解显示测试原理：





###### 测试结果：

```bash
home:~/ysyx/ysyx-workbench/am-kernels/tests/am-tests$ make ARCH=native mainargs=v run
# Building amtest-run [native]
# Building am-archive [native]
# Building klib-archive [native]
# Creating image [native]
+ LD -> build/amtest-native.elf
/home/Yang/ysyx/ysyx-workbench/am-kernels/tests/am-tests/build/amtest-native.elf
1001: FPS = 28
2002: FPS = 29
3003: FPS = 30
4004: FPS = 29
5005: FPS = 30
Exit code = 00h
```









#### Q：实现单种颜色的显示

> [!IMPORTANT]
>
> 实现上述的`draw()`函数, 它将弹出的窗口填充为参数`color`所指示的颜色, 窗口的分辨率为`400x300`.
>
> 实现后, 重新编译运行. 如果你的实现正确, 你应该看到弹出的窗口填充了蓝色.



A：较为简单，改了一下test/video.c



#### Q：实现颜色渐变效果

> [!IMPORTANT]
>
> 按照上述介绍, 实现颜色渐变效果. 你可以自行决定每轮渐变持续多长时间, 以及一轮渐变中的步数k.

我设置了k=300，每隔10秒变换

##### 我的bug：

###### 随机数

头文件：stdlib.h   ，int a = rand() % 51 + 13; //产生13~63的随机数

分析：取模即取余，rand()%51+13我们可以看成两部分：rand()%51是产生 0~50 的随机数，后面+13保证 a 最小只能是 13，最大就是 50+13=63。

最后给出产生 13~63 范围内随机数的完整代码：

版权声明：本文为CSDN博主「IUN_2930」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/m0_46613023/article/details/114778131



###### 别忘了乘号*

```C
 r_i=r0+(r_goal-r0)*(i/k);
 g_i=g0+(g_goal-g0)*(i/k);
 b_i=b0+(b_goal-b0)*(i/k);
```



###### 为什么canvas只显示三角形？

```C
 for (int i = k-1; i > 15 ; i-- )
    {
      // 更新下一条的颜色
      r_i = r0 + (r_goal - r0) * i / (k);
      g_i = g0 + (g_goal - g0) * i / (k);
      b_i = b0 + (b_goal - b0) * i / (k);
      color_i = ((uint32_t)r_i << 16) | 
          ((uint32_t)g_i << 8) | 
          ((uint32_t)b_i);
                
      // 染色第i条
      color_buf[i]= color_i; // 染色第x个条子的颜色
     }   
    
    //填充
 int x;
 for (x = 399; x > 0; x--){
     io_write(AM_GPU_FBDRAW, x * w, 0 * h, canvas, w, h, true); 
     // x*w是在屏幕的实际位置；先不立刻渲染
 }   
          
```



###### 求渐进线的公式需要精度，不然0～15就无法正确计算

![img](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/fac79eb94eb4e6ef3463e6d07c307fd9.png)

改为：

```C
r_i = r0 + (r_goal - r0) * i / k);
r_i = r0 + (int)((float)(r_goal - r0) * i / k);
//g和b同理
```

成功显示：

![image-20260213120453240](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260213120453240.png)

#### Q：添加按键效果

> [!IMPORTANT]
>
> 按照上述介绍, 为屏幕保护程序添加按键效果
>
> - 如果按下`ESC`键(按键编码为`AM_KEY_ESCAPE`), 则退出程序
> - 如果按下其他任意键, 则加快一轮渐变的时间; 释放按键后, 渐变时间恢复

##### 我的bug：

###### 循环捕获键盘动态

ev必须放在while里面，时刻捕获键盘动态，放在外面只能捕获一次，不会更新

```C
 while (1) {
    AM_INPUT_KEYBRD_T ev = io_read(AM_INPUT_KEYBRD);//必须在while里面，实时更新
```



#### A：以上有关Screensaver的必答题的代码

代码夹名称：screensaver

运行环境：AM

已经提交到分支tracer-ysyx 下的  homework/screensaver

运行视频链接：【一生一芯E4_screensaver_kbd成果展示】 https://www.bilibili.com/video/BV1XDZsBqE3R/?share_source=copy_web&vd_source=bd67740674966eb4f3ece57a63fd3148



### 为minirvEMU添加图形显示功能

#### Q：添加图形显示功能

> [!IMPORTANT]
>
> 根据上述思路, 为minirvEMU添加图形显示功能, 然后运行之前在Logisim上运行过的`vga`程序.
>
> 一些提示:
>
> - 为了使用AM提供的功能, 你可以参考上文屏幕保护程序的相关代码
> - 为了防止minirvEMU在显示图像后马上退出, 可以让它进入死循环



##### 设计思路：

按功能来梳理思路

###### 1. semu.c生成颜色数据

semu.c是之前没有显色功能的minirv32，在这个基础上改动sw指令。根据颜色范围可以生成绘图需要的颜色数据并存入  RGB二维数组  和颜色地址(y,x)，按照F6的提示，`x`的`raddr`的7～0 bit，`y`为`raddr`的第15～8 bit.

```C
//判断raddr是否在区间[0x20000000, 0x20040000]
if(raddr > 0x20040000/4 || raddr < 0x20000000/4){
    //不在该区间则存入M[]
	M[raddr]=R[rs2];
}else{
	x=(uint8_t)raddr;
	y=(uint8_t)(raddr>>8);
	RGB[y][x]=R[rs2];//在该区间的颜色数据 存入RGB
	i++;
}
```

bug：一定是(y,x)而不是(x,y)，不然绘图是镜象的，这个在F6同理。



###### 2. video.c用数据绘图

生成RGB数组后，调用video.c的绘图函数`draw()`，进行绘图但不立刻显示。

```C
// 按行渲染屏幕
for(int y = 0; y < hh ; y++){
 for (int x = 0; x < ww ; x++ ){    
  for(int r=0 ; r < block_size ;r++){
    block_buf[r]=RGB[y][x];}
  io_write(AM_GPU_FBDRAW,x * w,y * h,block_buf ,w,h,false);}    
 }  
  io_write(AM_GPU_FBDRAW, 0, 0, NULL, 0, 0, true); 
  // 行间延时
AM_TIMER_UPTIME_T t0 = io_read(AM_TIMER_UPTIME);
while (io_read(AM_TIMER_UPTIME).us - t0.us < delay_time_c);  // 50ms
}
```

按行显示的原理：绘图是以小格子为单位绘画的，但显示是为行为单位，每显示一行就设置一个停留时间，再显示下一行，相当于“等等再执行下一层循环”，可以依靠双层循环的第二层来设置时间间隔。

遇到的bug：

- 初始化窗口 `ioe_init()`并不需要单独运行，可以去掉该函数，一开始测试程序会报错重复初始化，但最后测试就没有这个问题：

  原因：调用 `io_write(AM_GPU_FBDRAW)` / `io_read(AM_GPU_CONFIG)` 也会触发初始化，窗口仍然会显示。

- 显色窗口无法铺满

  我想让窗口变为正方形，可以让logo完全铺满屏幕，我的思路就是把AM环境的窗口变为和我设计的窗口一样大就行了，但是不尽人意

  1.设置绘图为300✖300，把AM环境的配置窗口修改为300✖300，

  更改AM环境的默认窗口大小，检索`AM_GPU_CONFIG`：

  ```bash
  home:~/abstract-machine$ grep -r "AM_GPU_CONFIG"
  am/src/native/ioe/gpu.c:void __am_gpu_config(AM_GPU_CONFIG_T *cfg) {
  am/src/native/ioe/gpu.c:  *cfg = (AM_GPU_CONFIG_T) {
  am/src/native/ioe.c:void __am_gpu_config(AM_GPU_CONFIG_T *);
  am/src/native/ioe.c:  [AM_GPU_CONFIG  ] = __am_gpu_config,
  am/src/x86/qemu/ioe.c:static void gpu_config(AM_GPU_CONFIG_T *cfg) {
  am/src/x86/qemu/ioe.c:  *cfg = (AM_GPU_CONFIG_T) {
  am/src/x86/qemu/ioe.c:  [AM_GPU_CONFIG  ] = gpu_config,
  am/src/platform/nemu/ioe/ioe.c:void __am_gpu_config(AM_GPU_CONFIG_T *);
  am/src/platform/nemu/ioe/ioe.c:  [AM_GPU_CONFIG  ] = __am_gpu_config,
  am/src/platform/nemu/ioe/gpu.c:void __am_gpu_config(AM_GPU_CONFIG_T *cfg) {
  am/src/platform/nemu/ioe/gpu.c:  *cfg = (AM_GPU_CONFIG_T) {
  am/src/platform/logisim/ioe.c:void __am_gpu_config(AM_GPU_CONFIG_T *);
  am/src/platform/logisim/ioe.c:  [AM_GPU_CONFIG  ] = __am_gpu_config,
  am/src/platform/logisim/ioe.c:void __am_gpu_config(AM_GPU_CONFIG_T *cfg) {
  am/src/platform/logisim/ioe.c:  *cfg = (AM_GPU_CONFIG_T) {
  ```

  定位到`~/ysyx-workbench/abstract-machine/am/src/native/ioe/gpu.c`并且修改配置，保险起见都改成了300✖300：

  ```C
  //#define MODE_300x300
  #define WINDOW_W 300
  #define WINDOW_H 300
  #ifdef MODE_300x300
  const int disp_w = WINDOW_W, disp_h = WINDOW_H;
  #else
  const int disp_w = 300, disp_h = 300;
  #endif
  ```

  运行结果：

  ![image-20260215173455395](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260215173455395.png)

  结果运行窗口是变小了，但是还是无法铺满整个窗口。

  

  又再试了其他几种：

  - 如果我设计的窗口（比如400✖300）大于AM配置窗口（400✖300）

    这样就会产生不完整的图像，我猜是因为logo本来就是正方形，反而写在长方形肯定不行![](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260215173843627.png)如果我设计的窗口（比如800✖800）大于AM配置窗口（400✖400)，结果就会显示黑屏，这个原因是因为我有段代码如下：

    ```C
    // 分割屏幕
      int w = io_read(AM_GPU_CONFIG).width/ww; 
    // 查询当前显示屏幕的设备；把整个屏幕均匀切成 ww×hh 个小块
      int h = io_read(AM_GPU_CONFIG).height/hh;
      int block_size = w * h; //每个小块的大小
    ```

    其中的`ww`和`hh`就是我设计的logo尺寸，一旦我设计的尺寸大于实际尺寸，那么`w`和`h`就会为`0`,这样就无法进入`for`循环绘图.

​		或者会显示很虚的图像等等，总之这个问题并未解决，采用了第一版。

###### 反思：

反思一下自己，应该从原理入手反而自己瞎试，浪费时间也没解决问题，确实对这个C语言显示图像不熟系，在屏幕保护显示渐变色是因为y=0，我是以条状为单位来绘图显示的，但这里无法使用条状实现，所以抓瞎了。



###### 3. main.c显色

**实现循环播放**：在原来的主函数里面的`while()`外面再套一层`while()`来实现循环显示logo；

```C
int main{
	while(1){
		while(1){...}
	}
}
```

遇到的bug：一定要注意在一次新的循环开始前，必须清零，不然的话就会循环执行ebreak，因为PC在上一轮结束后指向最后一个指令ebreak。

清零所有的定义比如M[]，PC，RGB，最好是所有的全局变量都清零，我设置了`clean()`，实现简单的来说就是把数据和变量归零：

```C
//M,RGB,PC等清零
  PC=PC_elc=PC_yu=flag=0;
  PC=PC_elc=PC_yu=0;
  for (int i = 0; i < 0x22ac0/4 ; i++) { M[i] = 0;  }
    for (int x = 0; x < ww; x++) {...}
      for (int y = 0; y < hh; y++) {...}
  // 绘画清屏
for(int y = 0; y < hh ; y++{...}
  for (int x = 0; x < ww ; x++ ){...}    
    for(int r=0 ; r < block_size ;r++){...}
//颜色块清零
for(int r=0 ; r < ww*hh ;r++){block_buf[r]=0;}
```

另外，如果采用绘画的循环和渲染来清零画板，可以实现一行一行清屏的效果。



**取消输入文件名**：

取消输入文件名运行，只运行vga.bin文件；

```C
FILE* fd=fopen("vga.bin","rb");
```



**添加键盘功能：`Esc`退出；**

修改之前的 screensaver_kbd.c 即可，只要检测到`keycode==AM_KEY_ESCAPE`，就执行`return 0;`

```C
while(1)//持续检测键盘动态
{
    //键盘控制:Esc退出
    AM_INPUT_KEYBRD_T ev=io_read(AM_INPUT_KEYBRD);
    if(ev.keycode==AM_KEY_ESCAPE) return 0;
    ...
  }
```

小缺点：有点延迟，因为键盘检测是夹在函数调用和循环之间的，所以有些延迟，留个彩蛋，看以后有没有更好的实现办法。



###### 4. 重点——分层设计

之所以是重点，是因为花的时间较长，没有怎么写过这样分开的C语言，但是这样利于团体开发，自己调试也好找问题，还是值得学习的。

划分：分为main.c和semu.c和video.c，main.c为主程序，后两个对应于semu.h和video.h，再设置global.h为全局的变量，头文件和宏定义。

我的bug：

1）头文件引用

需要以下几个原则：

- 不能让非主程序的.c来引用其他的.h，可以引用自己的.h，或者全局global.h

​	原因：会报错 变量名被多次初始化，这是因为相互引用会被重复加载；

- 即使在全局global.h里面定义过的宏,变量和头文件，如果在非主程序中使用过，那么就需要在对应的.h再加载一次，但是变量的再次定义需要加上`extern`，表明非初次定义，而且初次定义必须在`extern`前面加载.

  例子：如果当前比如semu.c使用了变量`RGB`，虽然semu.c引用了global.h，但是还是会报错 没有该变量的定义，但是再一次定义就会报错 该变量 被多次初始化，所以需要利用`extern`，

  ```C
  //video.h
  extern uint32_t RGB[ww][hh]; //RGB[][]已经在别处定义过
  extern uint32_t RGB[ww][hh]={0}; //错误，extern不可初始化
  ```

  这样可以满足即在需要的.c里面定义了，也不会重复初始化。

  注意`extern`不要初始化，如果是初次定义就可以进行初始化。

- 设置global.h更加方便，个人感觉便于管理变量初值，改宏定义一定记得引用的.h文件全部要改.



2）遇到的最多报错

下面是我遇到的最多报错，是不是感觉很吓人，都不怎么认得，其实是双括号没有加全，导致函数识别到了外部.

```
/home/Yang/ysyx/ysyx-workbench/abstract-machine/klib/include/klib-macros.h:19:3: error: braced-group within expression allowed only inside a function
   19 |   ({ reg##_T __io_param; \
      |   ^
这个错误通常是因为你在函数外部（全局作用域）使用了 io_read() 或类似的宏。让我帮你解决。
错误分析
io_read() 是一个宏，它展开后包含一个语句块 ({ ... })，这种语法只能在函数内部使用，不能在全局作用域使用。

```



还有一个是：定义了但并为使用，符号是`^~~~~~~~~~~~`

```bash
if(ev.keycode==AM_KEY_ENTER) //Enter继续
      |                    ^~~~~~~~~~~~
      |                    AM_KEY_END
error: ‘AM_KEY_RETURE’ undeclared (first use in this function); did you mean ‘AM_KEY_RETURN’?
   59 |     if(ev.keycode==AM_KEY_RETURE) //Enter继续
      |                    ^~~~~~~~~~~~~
      |                    AM_KEY_RETURN
```

反思：

可以积累一下报错信息，发现编译器报错使用的是符号显示，虽然也有英文辅助描述，但是多积累下报错信息还是好的。如果遇到不太认识的报错，很有可能是括号问题，不一定是单一的报错信息，会引起其他报错。



3）函数返回后会去哪里？

这个自己写的时候忽然懵住了，不知道怎么函数返回了，百度了一下真想笑自己哈哈，学得都有点忘记了。

A：会返回调用函数的地方的下一条程序执行。

```C
int main{
...
while{
...
if(int a==0)
decode();
...  //这里函数返回
}}
```



#### A：minirv32_video的代码

代码夹名称：semu_video

运行环境：AM

已经提交到分支tracer-ysyx 下的  homework/semu_video

运行视频链接：【一生一芯E4_semu_video】 https://www.bilibili.com/video/BV1v2ZsB3EW2/?share_source=copy_web&vd_source=bd67740674966eb4f3ece57a63fd3148



## E4总结：

​	总的来说，利用程序设计处理器要比Logisim简单的多，毕竟动手连元器件，和写几行代码就可以实现的功能来比较的话，还是写程序更好实现，看了下E5感觉是硬件利用软件表示出来，加油！
