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

### c.合并分支（待测试）



```bash
git checkout master               #先切换到主分支
git merge 11a4（要合并的分支名）     #合并11a4到master，但不删除11a4
```

## 3）自己的常用git指令

```bash
git add 文件(夹)名  #把当前文件存到暂存区
git switch tracer-ysyx  # 切换仓库
git restore .      #丢弃主仓库当前所有未git commit的修改（就是红色的）
git restore --staged . #清除暂存区
git restore --staged homework/Two_way_switch/obj_dir/Vtop* #清除暂存区中某个特定的文件
git rm --cached -f nemu/tools/capstone/repo  #删除暂存区的某个文件夹
git ls-tree tracer-ysyx #查看分支tracer-ysyx的目录
git ls-tree tracer-ysyx:homework #查看tracer-ysyx分支下文件夹homework的目录
cat scripts/pdk/icsprout55.tcl  #获取该tcl文件

#提交错分支的话，这样可以纠正
git switch tracer-ysyx  # 切换到最终需要提交到的分支下
git cherry-pick b6eb88(错误提交的Hash编号) 
```



# 批处理测试

```shell
make ARCH=riscv32e-npc run ALL="recursion crc32 if-else shift" -j
# 所有测试程序的集合 -j8
make ARCH=riscv32e-npc run ALL="recursion crc32 if-else shift unalign bit add hello-str bubble-sort movsx leap-year add-longlong max quick-sort fib shuixianhua div pascal mul-longlong select-sort sum fact wanshu dummy prime switch sub-longlong goldbach load-store to-lower-case string mov-c min3 matrix-mul mersenne" -j8
make ARCH=riscv32-nemu run ALL="recursion crc32 if-else shift unalign bit add hello-str bubble-sort movsx leap-year add-longlong max quick-sort fib shuixianhua div pascal mul-longlong select-sort sum fact wanshu dummy prime switch sub-longlong goldbach load-store to-lower-case string mov-c min3 matrix-mul mersenne" -j8
```

需要命令里面不要`-e $(ELF_FILE)`和`-v`，以及关闭`sdb`，其他的无所谓：

```makefile
run: insert-arg
	$(MAKE) -C $(NPC_HOME) ISA=$(ISA) run ARGS="-t -d -w -b $(NPCFLAGS)" IMG=$(IMAGE).bin 
        #-e $(ELF_FILE)(ftrace) 
        #-v(vga) -t(itrace & mtrace) -w(wtrace) -b(no sdb) -d(difftest)
```

最大可以一次并行`-j`14个测试文件，但是15个会闪退，可能是内存上限，可以依靠`-j4`或者`-j8`来规定最大并行数量。

------



# B2-`SoC`计算机系统

我们先给出`ysyxSoC`包含的外围设备和相应的地址空间.

| 设备           | 地址空间                  |
| -------------- | ------------------------- |
| CLINT          | `0x0200_0000~0x0200_ffff` |
| SRAM           | `0x0f00_0000~0x0fff_ffff` |
| UART16550      | `0x1000_0000~0x1000_0fff` |
| SPI master     | `0x1000_1000~0x1000_1fff` |
| GPIO           | `0x1000_2000~0x1000_200f` |
| PS2            | `0x1001_1000~0x1001_1007` |
| MROM           | `0x2000_0000~0x2000_0fff` |
| VGA            | `0x2100_0000~0x211f_ffff` |
| Flash          | `0x3000_0000~0x3fff_ffff` |
| ChipLink MMIO  | `0x4000_0000~0x7fff_ffff` |
| PSRAM          | `0x8000_0000~0x9fff_ffff` |
| SDRAM          | `0xa000_0000~0xbfff_ffff` |
| `ChipLink MEM` | `0xc000_0000~0xffff_ffff` |
| Reserved       | 其他                      |

