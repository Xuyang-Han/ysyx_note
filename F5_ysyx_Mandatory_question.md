# F5必答题

## 支持数列求和的简单处理器

### 1.实现取值功能

![image-20260116165707491](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260116165707491.png)

以下是2位ROM的组成，

![image-20260116170437554](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260116170437554.png)

以下是求和数列的指令序列：

```
0: li r0, 10   # 这里是十进制的10
1: li r1, 0
2: li r2, 0
3: li r3, 1
4: add r1, r1, r3
5: add r2, r2, r1
6: bner0 r1, 4
7: bner0 r3, 7
```

以下是上面指令序列对应的机器码：

```
10001010    # 0: li r0, 10
10010000    # 1: li r1, 0
10100000    # 2: li r2, 0
10110001    # 3: li r3, 1
00010111    # 4: add r1, r1, r3
00101001    # 5: add r2, r2, r1
11010001    # 6: bner0 r1, 4
11011111    # 7: bner0 r3, 7
```

#### 设计思路：

已知求和程序的指令序列对应的机器码8行，所以设置PC为3bit，所以设置3-8_decoder，这样可以翻译出8种情况，对应这8行；

每条机器码有8bit，相当于GPR是8位，于是设置或门8输入，最终输出8bit。

设计输入机器码，ROM是不可修改，所以设计成常数。

#### 设计电路图：

此时图中8×8_ROM输出了PC=0的机器码。

![image-20260116183032792](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260116183032792.png)

### 2.实现GPR及其写入功能

![image-20260116180949526](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260116180949526.png)

#### 设计思路：

题中提示GPR是8bit，所以宽度设置为8，即设置八组D_flip-flop，又因为1到10累加之和的指令序列有八行，所以深度设置为8.

#### 设计电路图：

下面这个是错的，也是我后面才排查出来的错误，因为D_flip-flop的WE应该和与门相连，而不是和译码器。

![image-20260117204906479](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260117204906479.png)

更正后，改为了4×8_RAM，原理不变。

![image-20260117205419108](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260117205419108.png)

### 3~5.实现支持计算1-10之和的sCPU

![image-20260116204249423](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260116204249423.png)

![image-20260116222314976](C:/Users/Lenovo/AppData/Roaming/Typora/typora-user-images/image-20260116222314976.png)

![image-20260117190825504](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260117190825504.png)

#### 设计思路

##### 取指

根据指令序列翻译为机器码，再将翻译好的机器指令写成常数存到ROM中，封装成子电路PC_Register,为了区分设置为sCPU_ROM_1.

```
10001010    # 0: li r0, 10
10010000    # 1: li r1, 0
10100000    # 2: li r2, 0
10110001    # 3: li r3, 1
00010111    # 4: add r1, r1, r3
00101001    # 5: add r2, r2, r1
11010001    # 6: bner0 r1, 4
11011111    # 7: bner0 r3, 7
```

![image-20260117212629912](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260117212629912.png)

##### 译码

###### 设计思路：

要根据每种指令的含义，来利用位选择器进行位识别，这样一共就有三种不同的识别，不同的识别连接MUX，选择端口连接2-4译码器的输出端，这样就可以达到对应的指令进行对应的电路处理。

![image-20260117094504609](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260117094504609.png)

利用位选择器识别最高两位，识别是哪一种指令。再使用2-4译码器，来选择进入哪一种指令的操作。

![image-20260117205653873](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260117205653873.png)

##### 执行

###### 1.如何处理add和li？

![image-20260117205817343](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260117205817343.png)

如果我自己想的话，卡在了add操作需要2个时钟周期，不像li指令只需要一个时钟周期，所以我不知道如何在一个时钟周期完成add指令，而题中给的提示是修改RAM的规格，可以在一个时钟周期完成同时读和同时写，改完的RAM_Pro如下，

![image-20260117210051194](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260117210051194.png)

![image-20260117210117407](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260117210117407.png)

![image-20260117210138218](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260117210138218.png)

###### 2.如何实现指令bner0？

![image-20260117212955980](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260117212955980.png)

###### 3.如何实现三种指令的选择输出？

利用选择器，选择器的选择端我与2-4译码器的输出端相连

![image-20260117210321666](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260117210321666.png)

#### 设计电路：

![image-20260117210451665](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260117210451665.png)

经过测试：

R2最后得数位55，

![image-20260117212534319](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260117212534319.png)

PC=7.

![image-20260117212607209](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260117212607209.png)

#### 重点——遇到的bug:

##### 1）输出决定输入

1.处理指令bner0时，需要输出的PC来反过来决定下一次的输入PC，这里就出现了爆红——原因是因为电路判断，这两条电路时同时修改的，所以电路里的值会起冲突，形成了反馈电路，所以只要打破反馈电路，不是同时出现修改就不会出现爆红，故加入D_flip-flop，果然解决啦！

2.还有一种解决办法，就是加入多路选择器MUX，比如指令add最后需要将结果再一次写入RAM，也是输出决定输入的情况，但是加入了MUX就不会爆红，打破了反馈电路。

##### 2）RAM连错

1.RAM内部连错了，见这节的“实现GPR及其写入功能”，D触发器的WE应该和与门相连，但是我连接了译码器的输出端。

##### 3）MUX爆红解决

如果MUX选中的输入什么都没有，那么输出就会爆红，但是比如bner0指令，此时WEN=0，所以输入什么都可以，可以随便写个常数，避免后续爆红。



### 6.和数列求和电路进行对比

![image-20260117190850022](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260117190850022.png)

#### 数列求和电路：

优点：实现比较容易；缺点：只能完成一种求和运算，遇到其他需求的指令就需要重新设计电路。

#### sCPU:

优点：可以运行该sISA内的所有指令序列，不同功能不同需求，只有指令属于sISA；缺点：实现更加复杂，需要用心设计。



### 7.计算10以内的奇数之和

![image-20260117190915050](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260117190915050.png)

#### 设计思路

##### 取指

以下是奇数求和指令序列翻译为机器码，和前面一样写成常数封装在PC_Register当中，为了区分我设置为sCPU_ROM_2.

```
10001001    # 0: li r0, 9
10010001    # 1: li r1, 1
10100001    # 2: li r2, 1
10110010    # 3: li r3, 2
00010111    # 4: add r1, r1, r3
00101001    # 5: add r2, r2, r1
11010001    # 6: bner0 r1, 4
11011111    # 7: bner0 r3, 7
```

##### 译码与执行

均保持不变

#### 设计电路图：

最后R2=25，即第三行，二进制为000011001，

![image-20260117172324001](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260117172324001.png) 

PC=7.   

![image-20260117215226164](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260117215226164.png)                                          

#### 对“存储程序”思想的认识

存储程序可以实现自动执行指令序列，不需要全过程人工干预执行，极大地提高了执行效率，只要在同一个ISA中，实现不同功能的程序只需要更换PC寄存器的机器码，不需要修改电路。

### 8.添加新指令

![image-20260117172359464](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260117172359464.png)

#### 设计思路：

##### 设计sISA

指令out的操作码是01，rs占最低2bit，中间使用0填充。

##### 设计指令序列：

我设计功能为计算1到5的奇数之和并且结束后一直显示结果，指令序列如下:

```
10000101    # 0: li r0, 5
10010001    # 1: li r1, 1
10100001    # 2: li r2, 1
10110010    # 3: li r3, 2
00010111    # 4: add r1, r1, r3
00101001    # 5: add r2, r2, r1
11010001    # 6: bner0 r1, 4
01000010    # 7: out r2
```

#### 设计电路图

##### 如何可以循环执行out r2？

有一处与前面不同，就是在PC=7时，如何可以循环执行out r2，前面是依靠bner0来实现死循环，但是如果在此处也同样的方式，七段数码管会频繁变换，而不是一直显示，于是我在PC进行跳转的位置进行了更改，一开始PC变换有两种情况：

1.当add与li时，PC+1；2.当为指令bner0时，PC=4；

要是想实现一直反复执行指令out，也就是PC+0，于是在PC+1的情况中扩展一下，可以利用多路选择器加一条加法情况，当为指令out时，进行PC+0，当add与li时，PC+1，这样就可以实现一直显示结果。

结果R2=9，PC=7.

![image-20260117222400368](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260117222400368.png)