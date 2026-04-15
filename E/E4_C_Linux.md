# E4_在Linux中学习C语言



## 练习1：启用编译器

### 写入.c文件

写入ex1.c代码，有些字符在bash中有特殊函数，比如“#”，“（”，“{}”.......，所以不能使用echo写入，需要使用其他方法，比如cat和'EOF'组合输入ex1.c：

```bash
home:~$ cat > ex1.c << 'EOF'
int main(int argc, char *argv[])
{
    puts("Hello world.");
    return 0;
}
EOF
```

### 附加题

- 在你的文本编辑器中打开`ex1`文件，随机修改或删除一部分，之后运行它看看发生了什么。

  A：可能会报错或者编译失败

- 再多打印5行文本或者其它比`"Hello world."`更复杂的东西。

- 执行`man 3 puts`来阅读这个函数和其它函数的文档。



## 练习2：用Make来代替Python

### 一.使用make来创建.c文件

```
$ make ex1
# or this one too
$ CFLAGS= "wall" make ex1
```

#### 1）make ex1  和  touch  ex1.c  有什么区别？

- "make ex1" 是make根据Makefile里面的名为"ex1"的规则构建对应操作，（此时Makefile里面的ex1是创建ex1.c）；
- "touch ex1.c" 是修改或创建一个ex1.c文件.

#### 2）CFLAGS=  "wall" make ex1 是什么意思？

```bash
home=~$ "CFLAGS="-Wall" make ex1" 
```

是设置编译标志为显示所有警告，然后构建ex1（此时Makefile里面的ex1是创建ex1.c）。



### 二.创建Makefile

Makefile内容如下：

```bash
CFLAGS=-Wall -g  #在shell需要双引号，但在Makefile里面通常不需要
all: ex1
ex1: ex1.c
	gcc -o ex1 ex1.c  # 读取在当前目录下的 ex1.c并且编译链接为可执行文件
clean:
	rm -f ex1  # 删除ex1.out
```

`Makefile`向你展示了make的一些新功能。首先我们在文件中设置`CFLAGS`，所以之后就不用再设置了。并且，我们添加了`-g`标识来获取调试信息。`clean`部分用来清理.out文件。

#### 1.Makefile的作用

一个快捷的工具，比如执行`make clean`，可以直接执行Makefile里面的clean；但要注意，==<u>Makefile里面不能写ex1.c的内容，只能写bash语言</u>==.

##### 1）all:ex1的作用

执行`make`，默认执行all，即执行Makefile里面的ex1，去生成可以执行文件ex1.out；

```bash
# Makefile 内容：
第一行： all: myprogram

# 第1行：告诉 make "当我只输入 make 时，请帮我编译 myprogram"
all: myprogram

# 第x行： myprogram
myprogram: main.c utils.c
    gcc main.c utils.c -o myprogram
```

##### 2）后缀文件记忆技巧：

- `-E` → **E**xpand（展开，预处理）
- `-S` → **S**ource（汇编源代码）
- `-c` → **c**ompile to object（编译为目标文件）
- `-o` → **o**utput（输出可执行文件）



### 附加题

- 创建目标`all:ex1`，可以以单个命令`make`构建`ex1`

  ```bash
  home:~$ make
  gcc -o ex1 ex1.c 
  ```

- 阅读`man make`来了解关于如何执行它的更多信息

  （待定）

- 阅读`man cc`来了解关于`-Wall`和`-g`行为的更多信息

  （待定）

- 在互联网上搜索Makefile文件，看看你是否能改进你的文件

  （待定）

- 在另一个C语言项目中找到`Makefile`文件，并且尝试理解它做了什么

  个人理解是Makefile是一个调用入口，负责编辑规则，而`make`则负责调用规则，可以简化命令行的使用，只需要编辑Makefile即可。



## 练习3：格式化输出

这一节我学到的是，如何使用Makefile操作.c文件——添加依赖。

### 1）all

all后面写不写ex3都可以，在出现相同名称时all起作用，或者运行`make`，此处先不深究；

### 2）添加依赖

规则ex3后面必须添加.c文件的名称，注意是名称，即不一定叫xxx.c，而是这个.c文件的名称，`make ex1`执行Makefile里面叫`ex1`的规则——读取在当前目录下的ex1.c，并编译链接为可执行文件叫 ex1；

```bash
ex1: ex1.c
	gcc -o ex1 ex1.c  # 读取ex1.c并且编译链接为可执行文件
```

### 3）规则的顺序

Makefile中规则的顺序不影响执行结果。

```bash
all: ex1 ex3
ex1: ex1.c
	gcc -o ex1 ex1.c
ex3: ex3.c	# 这里添加依赖
	gcc -o ex3 ex3.c	
clean:
	rm -f ex1
	rm -f ex3
```



#### 附加题

- 找到尽可能多的方法使`ex3`崩溃

  A：定义int，但是使用%s输出会让ex3崩溃，即数据和输出类型不符合；

  ​       输出的角标超过数组的最大值，也让ex3崩溃。

- 执行`man 3 printf`来阅读其它可用的'%'格式化占位符。如果你在其它语言中使用过它们，应该看着非常熟悉（它们来源于`printf`）

  A：在练习6总结过

- 将`ex3`添加到你的`Makefile`的`all`列表中。到目前为止，可以使用`make clean all`来构建你所有的练习

  A：(疑问，待定)这是什么意思呢？

- 将`ex3`添加到你的`Makefile`的`clean`列表中。当你需要的时候使用`make clean`可以删除它

  ```C
  clean:
  	rm -f ex1
  	rm -f ex3
  ```

  



## 练习4 & 练习5

#### 利用Valgrind 分析 C程序的结构

下载valgrind和分析C程序，Valgrind 是命令行程序，为了更加方便，我在Vim里面下载了插件shellcheck来进行对bash代码的检查。



## 练习1~5总结

主要是说：

1. 如何利用shell建立并写入.c文件，再编译链接为可执行文件.out；
2. 如何通过`make`配合Makefile执行对应操作，可以理解为Makefile是规则集合体，使用`make`就可以对当前目录文件进行规则调用。



## 练习6：变量类型

### 1.其他printf占位符

一些其他的printf占位符的具体用法如下：

- long long 对应的是"%lld"；%f默认保留6位小数；% e 显示科学计数法；

- 左对齐（-）：在指定了宽度的情况下，默认是右对齐，使用-标志可以使其左对齐。
  例如：printf("%-10d", 123); // 输出 "123                   "（左对齐，宽度10）
- 显示正负号（+）：强制在正数前显示加号，负数前显示减号。
  例如：printf("%+d", 123); // 输出 "+123"
  printf("%+d", -123); // 输出 "-123"
- 空格：在正数前加空格，负数前显示减号。如果同时使用+标志，则+标志会覆盖空格。
  例如：printf("%   d", 123); // 输出 "   123"
  printf("%  d", -123); // 输出 "-123"
- \#：对于八进制和十六进制，显示前缀（0和0x）。对于浮点数，强制输出小数点（即使没有小数部分）。
  例如：printf("%#x", 255); // 输出 "0xff"
  printf("%#o", 8); // 输出 "010" ------是==<u>字母“O”</u>==
  printf("%#g", 10.0); // 输出 "10.0000"（注意，对于g/G，会保留尾随零，直到指定的精度）
- 0：使用0而不是空格进行填充。注意，如果同时使用-标志（左对齐），则0标志会被忽略。另外，对于整数，如果指定了精度（precision），0标志也会被忽略。
  例如：printf("%05d", 123); // 输出 "00123"



## 练习7：更多变量和一些算术

相关语法：

1.对于C来说，一个“字符”同时也是一个整数，这意味着可以对字符做算术运算，以特殊的语法`'\0'`声明了一个字符，这样创建了一个“空字节”字符，实际上是数字0.

```C
// this makes no sense, just a demo of something weird
    char nul_byte = '\0';
    int care_percentage = bugs * nul_byte;
    printf("Which means you should care %d%%.\n",//两个%%，为了打印出“0%”
            care_percentage);
```

### 如何使它崩溃

像之前一样，向`printf`传入错误的参数来使它崩溃。对比`%c`，看看当你使用`%s`来打印`nul_byte`变量时会发生什么。做完这些之后，在`Valgrind`下运行它看看关于你的这次尝试会输出什么：

改为 `%c`，不会报错，也什么都不会输出，因为`'\0'`是空字符，

```
home:~$ ./ex7
Which means you should care %.
```

改为 `%s`，Valgrind会显示：`%s`期望的是字符串指针，但传递的却是`int`

```C
char nul_byte = '\0';
	int care_percentage = bugs * nul_byte;
	printf("Which means you should care %s%%.\n", //改为 %s
	care_percentage);
```

### 附加题

- 这些巨大的数字实际上打印成了什么？

  A：在存储时会发生溢出，打印出来的是溢出后的值。具体值取决于类型（有符号或无符号）以及溢出后的模运算结果。

- 试着自己解释（在下个练习之前）为什么`char`可以和`int`相乘。

  A：`char`可以解释为ASSIC码，每个ASSIC码可以解释为一个整数，所以可以和`int`相乘。

- 把为`universe_of_defects`赋值的数改为不同的大小，观察编译器的警告。

  

- 将`long`改为`unsigned long`，并试着找到对它来说太大的数字。

  

- 上网搜索`unsigned`做了什么

  



## 练习8：大小和数组

使用了`sizeof`关键字来问C语言这些东西占多少个字节

```bash
printf("The size of an int: %d",sizeof(int));
The size of an int: 4  #输出
```

### 如何使它崩溃

使这个程序崩溃非常容易，只需要尝试下面这些事情：

- 将`char full_name[]`最后的`'\0'`去掉，并重新运行它，在`valgrind`下再运行一遍。现在将`full_name`的定义从`main`函数中移到它的上面，尝试在`Valgrind`下运行它来看看是否能得到一些新的错误。有些情况下，你会足够幸运，不会得到任何错误。

  ```bash
  home:~$ ./ex8
  The size of full_name (char[]): 11
  The number of chars: 11
  name="Zed" and full_name="Zed A. Shaw"
  ```

- 将`int areas[0]`改为`areas[10]`并打印，来看看`Valgrind`会输出什么。

  输出：数组的最大长度是5

  ![image-20260204185401673](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260204185401673.png)

- 尝试上述操作的不同变式，也对`name`和`full_name`执行一遍。

  报错和`int areas[]`一样

### 附加题

- 尝试使用`areas[0] = 100;`以及相似的操作对`areas`的元素赋值。

  ```C
  
  ```

  

- 尝试对`name`和`full_name`的元素赋值。

- 尝试将`areas`的一个元素赋值为`name`中的字符。

  ```C
  areas[3]=name[6]；
  ```

- 上网搜索在不同的CPU上整数所占的不同大小。



## 练习9：数组和字符串

```bash
home:~$ ./ex9
numbers: 0 0 0 0 #初始为0
name each: a   #初始为空
name: a
numbers: 1 2 3 4
name each: Z e d 
name: Zed
another: Zed
another each: Z e d 
```

相关知识点：

- 初始化 `int  num[]`时，如果只提供了某一个元素`int  num[2]=3`，剩下的都会为`0`；
- 初始化`char names[]`，如果只提供了某一个元素`char names[2]='a'`，剩下的前后都会为`'\0'`，是特殊字符而不会显示；所以首次打印`names`，只打印出了`"a"`，`'a'`字符之后的空间都用`'\0'`填充，是以`'\0'`结尾的有效字符串；
- 创建一个字符串的两种语法：`char name[4] = {'a'}`，或者char *another = "name"`。前者不怎么常用，后者更常用。

### 如何使它崩溃

C中所有bug的大多数来源都是忘了预留出足够的空间，或者忘了在字符串末尾加上一个`'\0'`.

### 附加题

- 将一些字符赋给`numbers`的元素，之后用`printf`一次打印一个字符，你会得到什么编译器警告？

  ```C
  int numbers[4] = {'a','d','f','A'};
  for(int i=0;i<4;i++){
      printf("numbers[%d]=%c\n",i,numbers[i]);}
  ```

  编译成功，我没有收到任何警告，神奇！

  ```bash
  home:~$ make ex9
  gcc -o ex9 ex9.c
  home:~$ ./ex9
  numbers[0]=a
  numbers[1]=d
  numbers[2]=f
  numbers[3]=A
  ```

  

- 对`names`执行上述的相反操作，把`names`当成`int`数组，并一次打印一个`int`，`Valgrind`会提示什么？

  ```C
   char name[4] = {12,37,99,100};
   for(int j=0;j<4;j++){
   printf("name[%d]=%c\n",j,name[j]);}
  ```

  没有提示，但是输出很奇怪

  ```bash
  home:~$ make ex9
  gcc -o ex9 ex9.c
  home:~$ ./ex9
  name[0]=
  
  name[1]=%
  name[2]=c
  name[3]=d
  
  ```

  

- 有多少种其它的方式可以用来打印它？

  A：`int`可以使用%c来打印，`char *`可以使用`%s`来打印

- 如果一个字符数组占四个字节，一个整数也占4个字节，你可以像整数一样使用整个`name`吗？你如何用黑魔法实现它？

  A：可以使用char转int来实现

  ```C
  #include<string.h>
  char name[4];
  int num;
  memcpy(&num,name,sizeof(int));
  ```

  

- 拿出一张纸，将每个数组画成一排方框，之后在纸上画出代码中的操作，看看是否正确。

- 将`name`转换成`another`的形式，看看代码是否能正常工作。



## 练习10：字符串数组和循环

```bash
home:~$ make ex10
gcc -o ex10 ex10.c
home:~$ ./ex10
state 0: California
state 1: Oregon
state 2: Washington
state 3: Texas
```

### 你会看到什么

ex10.c部分截取：

```C
#include <stdio.h>
int main(int argc, char *argv[])
{
    int i = 0;
    for(i = 1; i < argc; i++) {
        printf("arg %d: %s\n", i, argv[i]);
    }
    ...
```

如果不给main传入任何参数，那么`i < argc`= false，那么就不会执行第一个`for`循环.

### 如何使它崩溃

- 使用你喜欢的另一种语言，来写这个程序。传入尽可能多的命令行参数，看看是否能通过传入过多参数使其崩溃。

  A：程序会在命令行参数的总大小超过进程栈大小时崩溃。在Linux中，单个参数的最大长度是128KB，而参数总长度（包括所有参数和环境变量）的限制是2MB，而栈大小是8MB，所以可能无法通过命令行参数达到栈溢出。

- 将`i`初始化为0看看会发生什么。是否也需要改动`argc`，不改动的话它能正常工作吗？为什么下标从0开始可以正常工作？

  ```bash
  home:~$ ./ex10
  arg 0: ./ex10
  state 0: California
  state 1: Oregon
  state 2: Washington
  state 3: Texas
  ```

  A：难道是因为arge=1吗？——是的，不传参数的话，argc=1是默认的；

  ​       如果传入参数呢？——argc=1开始，逐个累加参数个数，传入三个参数，argc=1+3=4

  ```bash
  home:~$ ./ex10 hello world 123
  arg 0: ./ex10
  arg 1: hello
  arg 2: world
  arg 3: 123
  ```

- 将`num_states`改为错误的值使它变大，来看看会发生什么。——后面会输出乱码

  ```bash
  home:~$ ./ex10
  state 0: California
  state 1: Oregon
  state 2: Washington
  state 3: Texas
  state 4: �p���]
  Segmentation fault (core dumped)
  ```



### 附加题

- `for`循环的每一部分可以放置什么样的代码?

- 查询如何使用`','`（逗号）字符来在`for`循环的每一部分中，`';'`（分号）之间分隔多条语句。

  #### 初始化部分 (Initialization)

  a) 简单变量初始化

  ```c
  for (int i = 0; ...; ...) {...}
  ```

  b) 多个变量初始化（用逗号分隔）

  ```c
  for (int i = 0, j = 10, k = 100; ...; ...) {...}
  ```

  c) 赋值表达式

  ```c
  for (i = get_input(); ...; ...) {  // 调用函数初始化
  for (ch = 'A'; ...; ...) {        // 字符赋值
  for (ptr = array; ...; ...) {     // 指针赋值
  ```

  d) 空语句（什么都不放）

  e) 复杂表达式

  ```c
  for (int i = (printf("开始循环\n"), 0); ...; ...) {...}
  ```

  #### 条件部分 (Condition)

  a) 比较表达式

  ```c
  for (int i = 0; i < 10; i++) {      // i < 10
  for (int i = 0; i != 10; i++) {     // i != 10
  for (int i = 10; i > 0; i--) {      // i > 0
  for (int i = 0; i <= 100; i++) {    // i <= 100
  ```

  b) 复合条件

  ```c
  for (int i = 0, j = 10; i < 10 && j > 0; i++, j--) {...}
  for (int i = 0; i < 10 || condition_func(); i++) {...}
  ```

  c) 函数调用

  ```c
  for (int i = 0; is_valid(i); i++) {...}
  for (int i = 0; !feof(file); i++) {...}
  ```

  d) 赋值表达式（有副作用）

  ```c
  int ch;
  for (ch = getchar(); ch != EOF; ch = getchar()) {...}
  
  int i = 0;
  for (; (i = i + 1) < 10; ) {...}
  ```

  e) 空条件（无限循环）——需要内部使用`break`跳出循环

  ```c
  for (int i = 0; ; i++) {  // 条件永远为真
      if (i >= 10) break;   // 需要在循环体内break
  }
  
  for (;;) { ...}
  ```

  #### 更新部分 (Update)

  a) 递增/递减/乘法/加法/减法

  ```c
  for (int i = 0; i < 10; i++) {     // 后置递增
  for (int i = 0; i < 10; ++i) {     // 前置递增
  for (int i = 10; i > 0; i--) {     // 递减
  for (int i = 0; i < 10; i += 2) {  // 步长为2
  ```

  b) 多个更新表达式

  ```c
  for (int i = 0, j = 10; i < j; i++, j--) {...}
  for (int i = 0; i < 10; i++, printf("i=%d\n", i)) {...}
  ```

  c) 函数调用

  ```c
  for (int i = 0; i < 10; update_counter(&i)) {...}
  ```

  d) 复杂表达式

  ```c
  for (int i = 0; i < 10; i = (i * 2) + 1) {...}
  ```

  e) 空更新

  ```c
  for (int i = 0; i < 10; ) {
      // 在循环体内更新i
      i = get_next_value();
  }
  ```

  

- 查询`NULL`是什么东西，尝试将它用做`states`的一个元素，看看它会打印出什么。

  `Null`意思是不存在，

  ```bash
  #设置 "NULL"
  home:~$ ./ex10
  ...
  state 4: NULL
  
  #设置NULL
  home:~$ ./ex10
  ...
  state 4: (null)
  ```

- 看看你是否能在打印之前将`states`的一个元素赋值给`argv`中的元素，再试试相反的操作。

  ```C
  #include <stdio.h>
  int main(int argc, char *argv[])
  {
      int i = 0; 
      char *states[] = {
          "California", "Oregon",
          "Washington", "Texas",NULL
      };
      
      //交换赋值
      argv[1]=states[2]; 
      states[3]=argv[2];
      int num_states = 5;
      
      for(i = 1; i < argc; i++) {
          printf("arg %d: %s\n", i, argv[i]);}
      for(i = 0; i < num_states; i++) {
          printf("state %d: %s\n", i, states[i]);}
      return 0;
  }
  ```

  运行结果：可以看出交换成功！

  ```bash
  home:~$  ./ex10 hello world 123
  arg 1: Washington
  arg 2: world
  arg 3: 123
  state 0: California
  state 1: Oregon
  state 2: Washington
  state 3: world
  state 4: (null)
  ```

## 练习11：While循环和布尔表达式









# Vim的使用

不能有空格，必须是Tab，不然报错。

vim执行完：lop后里面报错的location_list保存哪里

简短版，只讲操作。

------

**1️⃣ 普通模式（Normal）**

- 复制一行

```
yy
```

- 复制3行

```
3yy
```

- 粘贴到光标后

```
p
```

- 粘贴到光标前

```
P
```

------

**2️⃣ 可视模式（Visual）**

进入可视模式：

```
v
```

选中后复制：

```
y
```

粘贴：

```
p
```

------

**3️⃣ 行可视模式（Visual Line）**

进入：

```
V
```

复制整行块：

```
y
```

粘贴：

```
p
```

------

**4️⃣ 插入模式（Insert）**

👉 不能直接复制

先按：

```
Esc
```

回到普通模式，再用 `p`

------

**系统剪贴板（可选）**

复制到系统剪贴板：

```
"+y
```

粘贴系统剪贴板：

```
"+p
```

------

**核心记忆**

```
选中 → y
粘贴 → p
```

需要我再讲 **删除并粘贴(d / x)** 或 **多寄存器复制** 再说。
