# F4必答题

## 计算机系统的状态机模型

### 1.模拟执行汇编程序

![image-20260114170256222](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260114170256222.png)

```
PC r0 r1 r2 r3
(0, 0, 0, 0, 0)   # 初始状态
(1, 10, 0, 0, 0)  # 执行PC为0的指令后, r0更新为10, PC更新为下一条指令的位置
(2, 10, 0, 0, 0)  # 执行PC为1的指令后, r1更新为0, PC更新为下一条指令的位置
(3, 10, 0, 0, 0)  # 执行PC为2的指令后, r2更新为0, PC更新为下一条指令的位置
(4, 10, 0, 0, 1)  # 执行PC为3的指令后, r3更新为1, PC更新为下一条指令的位置
(5, 10, 1, 0, 1)  # 执行PC为4的指令后, r1更新为r1+r3, PC更新为下一条指令的位置
(6, 10, 1, 1, 1)  # 执行PC为5的指令后, r2更新为r2+r1, PC更新为下一条指令的位置

(4, 10, 1, 1, 1)  # 执行PC为6的指令后, 因r1不等于r0, 故PC更新为4
(5, 10, 2, 1, 1)  # 执行PC为4的指令后, r1更新为r1+r3, PC更新为下一条指令的位置
(6, 10, 2, 3, 1)  # 执行PC为5的指令后, r2更新为r2+r1, PC更新为下一条指令的位置

(4, 10, 2, 3, 1)  # 执行PC为6的指令后, 因r1不等于r0, 故PC更新为4
(5, 10, 3, 3, 1)  # 执行PC为4的指令后, r1更新为r1+r3, PC更新为下一条指令的位置
(6, 10, 3, 6, 1)  # 执行PC为5的指令后, r2更新为r2+r1, PC更新为下一条指令的位置
……
```

执行过程中R0=10，R3=1始终不变，当R1≠10时，执行到6时就会一直执行jump  4，即R1++，直到R1=10=R0时，会继续往下执行7，但是bneqr0 R3,7始终成立，所以会一直执行7.

处理器会一直执行PC=7，处于死循环状态。

数列的求和结果会存在R2中。



### 2.计算10以内的奇数之和

![image-20260114170430804](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260114170430804.png)

```
0: li r0, 9   # 这里是十进制的9
1: li r1, 1
2: li r2, 1
3: li r3, 2
4: add r1, r1, r3
5: add r2, r2, r1
6: bner0 r1, 4
7: bner0 r3, 7
```



### 3.在线运行C程序

![image-20260115114006972](C:/Users/Lenovo/AppData/Roaming/Typora/typora-user-images/image-20260115114006972.png)

![image-20260115114120818](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260115114120818.png)



### 4.继续执行C程序

![image-20260115114156610](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260115114156610.png)

![image-20260115113930457](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260115113930457.png)

最后i=11，sum=55，跳出循环，执行return 0结束，不会进入死循环。



### 5.从状态机视角理解数列求和电路的工作过程

![image-20260115115908134](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260115115908134.png)

设4bit计数器为A存储i，8位寄存器存储sum.

```
 i sum  
(0, 0)   # 初始状态
(1, 0)   #sum=sum+i=1+0=1
(2, 1)   #i每一次脉冲都+1
(2, 3)   #sum=sum+i=1+2=3
(3, 3)   #i每一次脉冲都+1
(3, 6)   #sum=sum+i=3+3=6
……
(10, 45)   #i每一次脉冲都+1
(10, 55)   #sum=sum+i=45+10=55
```

![image-20260115123137791](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260115123137791.png)



### 6.结合数列求和的例子理解编译器的工作

![image-20260115145829671](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260115145829671.png)

![image-20260115130904786](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260115130904786.png)

汇编语言里的寄存器和主存，相当于是C语言里的变量，也是ISA里的时序逻辑电路——C语言设int sum=0和int i=1，对应指令序列中利用li指令对寄存器R0，R1，R2，R3进行赋值；

而指令相当于C语言的语句，与汇编语言实现效果一致，也是ISA里的组合逻辑电路：

1）C语言利用while循环来实现累加，循环条件设置为 i ≤10，当i=11时跳出循环，对应指令序列里面bner0 r1,4（若≠r0.则跳转到PC=4），若不符合条件，就会循环执行PC=4,5,6；

2）C语言中的sum=sum+i和i=i+1，对应指令序列的add

3）不同的是C语言不会陷入死循环，但右面的汇编程序最后会反复执行PC=7.

另外，指令由组合逻辑电路实现，寄存器和主存由时序逻辑电路实现。



### 7.在Compiler Explorer中理解编译器的工作

![image-20260115130917030](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260115130917030.png)

![image-20260115152054755](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260115152054755.png)



## 题外话：

### 1.关于指令集和处理器的几个误区

![image-20260115154841714](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260115154841714.png)

最后一个问题，竟然可以！所以还是处理器设计和实现是重点，指令集并非是源头核心。



### 2.状态机角度理解计算机

![image-20260115155416995](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260115155416995.png)
