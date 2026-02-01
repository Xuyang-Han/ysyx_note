

# Verilog语法

关于写练习题时遇到的语法疑惑或者错误

## 一.语句

verilog主要有2种语句：assign和always

### 1.assign语句

一个assign只定义一个输出，并且分号结束。

#### 1)实现和if-else等价的语句

两种写法：

一种是只有true和false:

```verilog
assign  min=(sel==1) ? b : a ; #如果sel为真，则min=b,否则min=a
assign  min=(condition) ? true : false;
```

一种是n元(n>2)，或者二元的条件不再是true和false:

```verilog
assign q=
 (sel==2'b00) ? d :
 (sel==2'b01) ? q1 :
 (sel==2'b10) ? q2 :
 (sel==2'b11) ? q3 :
  d;     # MUX的实现
```

#### 2)简约算子

可以实现减少式子长度，输出为1bit.

同时可以组合使用，比如NAND（与非门）。

```
& a[3:0]     // AND: a[3]&a[2]&a[1]&a[0]. Equivalent to (a[3:0] == 4'hf)
| b[3:0]     // OR:  b[3]|b[2]|b[1]|b[0]. Equivalent to (b[3:0] != 4'h0)
^ c[2:0]     // XOR: c[2]^c[1]^c[0]
～& a[3:0]   // NAND: 等价为  邻位取与然后再取非，或者邻位取与非
```

#### 3)assign实现for循环

构建generate需要注意的点：

在generate内部不能使用assign进行赋值；

genvar是常数，不能使用assign进行赋值修改，genvar不能在循环体内部进行修改，只能在for(genvar)内部进行修改，并且只能有一个genvar；

使用generate实现，其中`begin: adder`表示每个循环生成一个名为adder的块。注意，在generate循环中，每个循环实例化一个块，并且需要给块命名（这里是adder）；

```verilog
genvar i;
generate
    for(i=0;i<100;i=i+1)begin:adder           
        bcd_fadd ex1(.a(a[i*4+3:i*4]),.b(b[i*4+3:i*4]),
                     .cin(cout_i[i]),.cout(cout_i[i+1]),
                     .sum(sum[i*4+3:i*4]));  

    end
endgenerate
```

构建100bit的BCD码加法器，点击跳转：
[BCD码的加法运算 ](####2.BCD码的加法运算)——若无法跳转查找目录“三.其他知识点2.BCD码的加法运算”

```verilog
input [399:0] a, b,
    input cin,
    output cout,
    output [399:0] sum );
    
    wire [100:0]cout_i;
    assign cout_i[0]=cin;
    assign cout=cout_i[100];
    genvar i;

    generate
        for(i=0;i<100;i=i+1)begin:adder           
            bcd_fadd ex1(.a(a[i*4+3:i*4]),.b(b[i*4+3:i*4]),
                         .cin(cout_i[i]),.cout(cout_i[i+1]),
                         .sum(sum[i*4+3:i*4]));  
          
        end
    endgenerate
   // assign cout=cout_i[101];

endmodule
```



### 2.always语句

在always语法下实现的语法，和C语言很类似。

#### 1）实现case语句

```verilog
  always@(*) begin // This is a combinational circuit
       case(sel)
          3'b000:out= data0;
          3'b001:out= data1;
          ..........
         default: out= 4'b0000;
       endcase  #  注意：case(变量)    endcase结尾
  end
```

##### 优先编码器的类别

优先编码器有高位优先和低位优先，看定义；在标准的优先编码器设计（如74LS148）中，通常是高位优先，并且是低电平有效。

##### case-z语句

该语句可以实现优先编码器少编写语句，z字母可以代表任意1bit二进制。

#### 2）现实if-else语句

里面的begin和end相当于C语言里面的{}，用法和C语言一样，如果if-else只有一条语句可以不加begin-end.

```verilog
always@(*) begin 
        if(condition)  begin
          ..........;
        else
          ..........;
        end
  end
```

#### 3)实现for循环语句

基本语法和C语言很像，但是一个区别是对整数变量的定义integer，并且要注意必须要在always语句块里面进行intrger初始化，在外面初始化会被忽略，到了always里面等于没有初始化。

##### a.逆转

逆转可以只使用一个变量，而不是两个。

```verilog
   integer i,j; # 需要定义为integer
    always @(*) begin
        for(i=0,j=99;i<100;i++,j--) begin
            out[i]=in[99-1];  #1个变量即可
            out[i]=in[j];     #2个变量
        end
    end
```

##### b.累加

通过累加后再给output  out赋值：不要利用out累加，引入一个整型变量count进行累加，最后再赋值给out.

引入累加，再赋值的原因：

- out是wire类型，不能在always块中赋值，所以引入一个中间变量count.
- 即使是在always块外面，使用assign对out赋值，最后输出out的话——形成了两次对output out赋值，这样是非法的，所以引入第三个变量最好。

```verilog
   integer i;
   integer count;  // 不要在这里写初始化，因为综合工具会忽略，而且我们会在always块中初始化
   assign out=0;  //会出现2次对out赋值，非法
   always @(*) begin $\textcolor{Blue}{}$

       '''count = 0;  // 重要：在组合逻辑中，每次执行都要初始化
       out=0;  //在always语句块内对wire赋值非法，必须使用assign赋值
       for(i=0;i<255;i++) begin
           if(in[i]==1)
               count = count + 1;  // 或者count++
               out++;  //最后非法
       end
   end
   
   assign out = count[7:0];  // 取count的低8位，因为count最多255，所以低8位就是正确结果
```

正确如下：

```verilog
   integer i;
   integer count;    
   always @(*) begin 
       count = 0;
       for(i=0;i<255;i++) begin
           if(in[i]==1)
               count = count + 1;  // 或者count++
       end
   end
   
   assign out = count[7:0];  
```

##### c.实现100bit波形加法器

我的bug：

- 如何确定每一次的cout[i]，需要三个部分共同决定：a[i],b[i],couy[i-1]
- 一开始的cin不一定为0,所以也必须考虑

正确如下：

```verilog
    input [99:0] a, b,
    input cin,
    output [99:0] cout,
    output [99:0] sum );
    
    integer i;
    always @(*) begin      
        cout[0]=(a[0]&b[0]) | (a[0]&cin) | (b[0]&cin); 
        sum[0]=a[0]+b[0]+cin;
        for(i=1;i<100;i++) begin            
            sum[i]=a[i]+b[i]+cout[i-1];           
            cout[i]=(a[i]&b[i]) | (a[i]&cout[i-1]) | (b[i]&cout[i-1]);                       
        end    
    end

endmodule
```

## 二.定义变量

关于一些变量的定义和赋值的要点

#### 1）Vector向量

定义向量：input/output wire ［高位：低位］vec_name
*   可以是［7:0］也可以是［0：7］，即大小可以反过来，命名不同，线本身不变
*   意思为 定义了 一个8bit的 输入/输出 电线，叫vec

定义：input/output wire b 0:3 向量不得随意改变bit编号大小顺序
*   b3:0 是非法的, “向量部分选择的方向必须与声明的方向一致”
*   人话：同一个向量的bit编号必须一致从大到小，或从小到大

定义一般变量 input/output x
* 默认是1bit


#### 2）关于赋值

##### 1.把一个8bit向量赋值给另一个1bit输出

*   x=veci

##### 2.把一个8bit向量赋值给另一个8bit向量输出（向量赋值向量）

*   非法：高位：低位 vec_out=input——vec_out前面不能加维度，只能加在后面
*   合法： vec_out =input ——直接赋值即可，前提是input和vec_out的维度匹配

##### 3.把一个8bit向量 截断 赋值给另一个4bit的向量输出

*   非法：高位：低位 vec_out=input —— vec_out前面不能加维度，只能加在后面
*   合法： vec_out a：b =input3：0 —— 意思是vec_out a =input{7]，vec_out b =input0

##### 4.拼起来赋值

*   拼向量：assign {w,x,y,z7:2} = {a,b,c,d,e,f};
*   拼常量： assign z1:0={2'b11};——写法：n'b10101(n是要赋值的比特数，b表示比特，后面是常数)
*   拼多个相同的常量
    *   assign ={2{2'b11}}
    *   assign ={ { 2{2'b11} } , in} ——如有封装且还有其他拼接元素，必须带一个大括号
*   拼多个相同的变量
    *   assign ={ 2{a , b , c } }
    *   assign ={ { 2{a , b , c } } , in}

#### 3）wire和reg的区别

* 一般来说，wire是连线，reg是有存储功能的部件  

*   reg不是任何时候都是寄存器，只有在clk存在时才是寄存器
    ```
    reg q; // 这会被综合成寄存器（有时钟）
    always @(posedge clk) q <= d; // 时序逻辑 → 寄存器 // 这不会综合成寄存器（无时钟）
    
    reg y; // 组合逻辑 → 只是连线(无时钟)
    always @(*) y = a & b; 
    ```

## 三.其他知识点

#### 1.奇偶校验

奇偶校验的基本思想是：通过添加一个额外的位（称为奇偶校验位），使得整个数据（包括校验位）中1的个数为奇数（奇校验）或偶数（偶校验）。

偶校验位= 每一邻位进行异或.

奇校验位= 每一邻位进行异或，再取反.

#### 2.BCD码的加法运算

##### **1)BCD码**

把十进制数利用n个4bit二进制表示出来.（此刻不仅仅讨论4bit的BCD码，而是nbit的BCD码）

把十进制数分个十百千位利用n个4bit二进制表示出来。比如，如果是2位十进制数，那么就需要两个4bit的二进制，分别来表示十位和个位. 若是a+b=3+8=11，11>9则需要拆分为10+1,二进制表示为：1010  0001 B

##### 2)**BCD码的加法**

先只讨论一个4bit的BCD码

1.范围：因为本质是十进制数，所以范围限制是由十进制的特性决定的

一个4bit二进制加数范围：[0,9]，所以和范围：[0,18].

2.加法：

先化为十进制数，然后再相加，根据十进制和是否超过9，决定是否对二进制和进行+6校验；

若十进制和>9：对二进制和进行+6校验；

若十进制和<9：不对二进制和进行校验；

此处cout为1bit，所以只是讨论20以内的加法。

##### 3)**构建100bit的BCD码加法器**

需要注意的点：

- 给出子模块BCD加法器，可以默认该加法器已经实现2个BCD码加法功能；
- 由于generate的特点，不能有两个genvar常量，所以必须要  genvar  i  可以同时控制——进位链cout_i和加法器的位置选取，如下代码：

```verilog
input [399:0] a, b,
    input cin,
    output cout,
    output [399:0] sum );
    
    wire [100:0]cout_i;
    assign cout_i[0]=cin;
    assign cout=cout_i[100];
    genvar i;

    generate
        for(i=0;i<100;i=i+1)begin:adder           
            bcd_fadd ex1(.a(a[i*4+3:i*4]),.b(b[i*4+3:i*4]),
                         .cin(cout_i[i]),.cout(cout_i[i+1]),
                         .sum(sum[i*4+3:i*4]));  
          
        end
    endgenerate
endmodule
```



