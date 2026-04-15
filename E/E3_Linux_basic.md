# E3_Linux入门

## PA0

### 1 Preparation

> [!IMPORTANT]
>
> 安装一个Linux操作系统
>
> 我们复用PA讲义的内容, 请大家根据[PA0](https://ysyx.oscc.cc/docs/ics-pa/PA0.html)安装Linux操作系统.



安装教程参考了：

【Ubuntu 24.04 安装教程之一：制作启动U盘】 https://www.bilibili.com/video/BV1KRTYzgE2y/?share_source=copy_web&vd_source=bd67740674966eb4f3ece57a63fd3148

【Ubuntu 24.04 安装教程之二：安装系统】 https://www.bilibili.com/video/BV1esTfzdEea/?share_source=copy_web&vd_source=bd67740674966eb4f3ece57a63fd3148

【联想笔记本进bios方法】 https://www.bilibili.com/video/BV1Xf4y1o7sV/?share_source=copy_web&vd_source=bd67740674966eb4f3ece57a63fd3148

我的电脑是联想小新Air14，经过各种尝试进入BIOS的方式：开机键+狂按F2



### 2  First Exploration with GNU/Linux

Ubuntu 占用了多少磁盘空间

```bash
df -h #查看ubuntu使用了多少磁盘可
```



### 3  Installing Tools

我使用的是Ubuntu24.04,需要格外记录下，因为和讲义Ubuntu22.04不一样。

#### Setting APT source file

##### creat a new superuser

```bash
sudo su                     #  0. 进入当前的root，即管理者
sudo adduser yy_user  #1. 创建用户yy_user，并设置密码
sudo adduser yy_user sudo   #  2. 用 yylinux 添加权限，把yy_user加入sudo组
groups yy_user              #  3. 检查是否添加成功
sudo whoami                 #  4. 检查yy_user权限，回答“root”
```

##### Setting APT source file

```
sed -i "s/archive.ubuntu.com/mirrors.tuna.tsinghua.edu.cn/g" /etc/apt/sources.list.d/ubuntu.sources  #必须在superuser下进行操作，否则无法成功
（ubuntu24.04对应apt文件是/etc/apt/sources.list.d/ubuntu.sources）
```

#### Updating APT package information

```bash
sudo apt-get update    #修改apt文件，必须在superuser下进行
# 遇到could not resolve 'cn.mirrors.tuna.tsinghua.edu.cn'的错误
```

是因为你在安装Ubuntu时默认源选择了国内镜像, 导致默认的sources.list被修改造成的. 为了修复这个问题, 你需要额外运行以下命令:

```bash
sed -i "s/cn.mirrors.tuna/mirrors.tuna/g"  /etc/apt/sources.list.d/ubuntu.sources
（ubuntu24.04对应apt文件是/etc/apt/sources.list.d/ubuntu.sources）
apt-get update   #然后尝试重新运行这条指令
```



### 4 Configuring vim

在Shell的学习中 “编辑器 (Vim)”有更详细的总结。

点击跳转：



### 5 More Exploration

#### 在Makefile中写C程序

在E4_C_Liunx中的练习0～5写过，故不再重复。

####  GDB的基本使用

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

#### （待定）一个实际应用

点击跳转     [E4 程序真的从main()开始执行的吗？]()    E4  Q：程序真的从main() 开始执行 / 返回后结束 的吗?（1）（2）

```
(gdb) starti                    # 在第一条指令暂停
(gdb) break exit                # 在exit函数处设置断点
(gdb) break main                # 在main函数处设置断点
(gdb) break abort               # 在 abort() 函数处断点

(gdb) continue                  # 继续执行，直到结束
(gdb) run                       # 继续执行，直到下一个断点
(gdb) q                         # 退出
```

####  `tmux`的基本使用

```bash
tmux                         #启动tmux
tmux                         # 启动新会话
tmux ls                      # 列出会话
tmux kill-server             # 终止所有会话

在tmux内部，前缀键（默认为 Ctrl+b）
  + [      # 打开浏览模式，q退出
  + c      # 新建窗口
  + d      # 关闭所有会话
  + %      # 垂直分割
  + "      # 水平分割
  + 方向键  # 切换窗格
  长按 + 方向键  # 调整窗格大小
  + x      # 关闭当前的窗格
  + z      # 最大化/恢复窗格
  
#感觉不如快捷键
tmux attach -t <name>        # 附加到会话
tmux new -s <name>           # 启动命名会话
tmux kill-session -t <name>  # 终止会话
```



### 6  Getting Source Code for PAs

#### (待定)Q:在github上添加ssh key

> [!IMPORTANT]
>
> 在获取框架代码之前, 首先请你在github上添加一个ssh key, 具体操作请STFW.

按照百度，连接成功！

```bash
Yang@yyl-Ubuntu:~$ ssh -T git@github.com
Hi Xuyang-Han! You've successfully authenticated, but GitHub does not provide shell access.
```



#### git快速入门

在实验中, 我们使用 `git` 进行版本控制. 下面简单介绍如何使用 `git` .

##### （一）安装git

首先你得安装 `git` :

```bash
sudo apt-get install git
```

进行配置工作. 在终端里输入以下命令，经过这些配置, 你就可以开始使用 `git` 了.

```bash
git config --global user.name "Zhang San"		# your name
git config --global user.email "zhangsan@foo.com"	# your email
git config --global core.editor vim			# your favourite editor
git config --global color.ui true
```

在实验中, 你会通过 `git clone` 命令下载我们提供的框架代码, 里面已经包含一些 `git` 记录, 因此不需要额外进行初始化. 如果你想在别的实验/项目中使用 `git` , 你首先需要切换到实验/项目的目录中, 然后输入

```bash
git init  # 进行初始化.
```



##### （二）git基本操作

##### 1）创建/切换/删除 分支

```bash
git branch               #查看所有分支/查看当前分支*
git status               #查看当前分支
git checkout 分支名       #切换到某个分支/Hash的前4位，即可创建一个对应的新分支
git checkout -b 分支名    #创建某个分支
git branch -D 11a4       #删除叫11a4的分支
```

##### 2）操作分支

###### a.查看分支内容

```bash
git log                          #查看新的提交信息
git status                       #查看文件有哪些变化
git diff                         #更直观的看有哪些变化
git show 11a4:semu.c >> 11a4.c   #查看11a4分支下的semu.c文件，并且存储到11a4.c文件里
```

绝对不要使用这个读档，==比这个存档新的所有记录都将被删除==，这意为着不能随便回到"将来"了.

```bash
git reset --hard b87c         #绝对不要使用这个读档
```

###### b.给分支添加新文件

``` bash
git -f add (文件名)file.c      #把当前文件存到暂存区
git reset HEAD minirv32       #清除刚刚提交该分支的文件
git status                    #再次确认文件列表
git commit                    #把暂存区所有文件提交到永久区，会进入vim进行编辑
git commit --allow-empty      #这样允许提交没做任何修改的相同文件
git commit  文件名1 文件名2     #把暂存区的2个文件提交到永久区
git commit -m                 #把暂存区所有文件提交到永久区，不会进入vim，直接提交编辑内容
```

###### c.合并分支（待定）（待测试）

```bash
git checkout master               #先切换到主分支
git merge 11a4（要合并的分支名）     #合并11a4到master，但不删除11a4
```

##### 3）自己的常用git指令

```bash
git restore --staged . #清除暂存区
git rm --cached -f nemu/tools/capstone/repo  #删除暂存区的某个文件夹
git ls-tree tracer-ysyx #查看分支tracer-ysyx的目录
git ls-tree tracer-ysyx:homework #查看tracer-ysyx分支下文件夹homework的目录
cat scripts/pdk/icsprout55.tcl  #获取该tcl文件
```



#### 实验报告要求

> [!IMPORTANT]
>
> 实验报告内容
>
> 你必须在实验报告中描述以下内容:
>
> - 实验进度. 简单描述即可, 例如"我完成了所有内容", "我只完成了xxx".
>   缺少实验进度的描述, 或者描述与实际情况不符, 将被视为没有完成本次实验.
> - 必答题.
>
> 你可以自由选择报告的其它内容. 你不必详细地描述实验过程, 但我们鼓励你在报告中描述如下内容:
>
> - 你遇到的问题和对这些问题的思考
> - 对讲义中蓝框思考题的看法
> - 或者你的其它想法, 例如实验心得, 对提供帮助的同学的感谢等
>
> 认真描述实验心得和想法的报告将会获得分数的奖励; 蓝框题为选做, 完成了也不会得到分数的奖励, 但它们是经过精心准备的, 可以加深你对某些知识的理解和认识. 因此当你发现编写实验报告的时间所剩无几时, 你应该选择描述实验心得和想法. 如果你实在没有想法, 你可以提交一份不包含任何想法的报告, 我们不会强求. 但请不要
>
> - 大量粘贴讲义内容
> - 大量粘贴代码和贴图, 却没有相应的详细解释(让我们明显看出来是凑字数的)
>
> 来让你的报告看起来十分丰富, 编写和阅读这样的报告毫无任何意义, 你也不会因此获得更多的分数, 同时还可能带来扣分的可能.



## 如何解决无法git跟踪？

在ysyx-workbench里面，输入：

```bash
echo $YSYX_HOME
# 我的输出为“not found”，说明git找不到路径，所以无法正确提交
```

在/.bashrc里面添加路径：

```bash
export YSYX_HOME=/home/Yang/ysyx/ysyx-workbench
```

在终端刷新：

```bash
source  ~/.bashrc
```

测试ysyx-workbench/Makefile里面的.git_commit是否工作：

```bash
 make .git_commit MSG="test"
 # 输出正常，且有git log，说明是npc的问题
```

在npc/Makefile里面（或者是想要git跟踪的文件夹下的Makefile），确保可以包含上一级Makefile：

```makefile
include ../Makefile # 提交后可以调用上一级的Makefile里面的.git_commit
```

在npc/Makefile里面（或者是想要git跟踪的文件夹下的Makefile），还要确保：

```Makefile
#确保在Makefile的某个规则下有
$(call git_commit, "sim RTL")  # 编写其他规则后，可调用该规则，实现git跟踪
```



## Q:关于“如何提问”读后感

> [!IMPORTANT]
>
> 独立解决问题是作为码农的一项十分重要的生存技能. 往届有同学在群里提出如下问题:
>
> - su认证失败是怎么回事?
> - grep提示no such file or directory是什么意思?
> - 请问怎么卸载Ubuntu?
> - C语言的xxx语法是什么意思?
> - ignoring return vaule of 'scanf'是什么意思?
> - 出现curl: not found该怎么办?
> - 为什么strtok返回NULL?
> - 为什么会有Segmentation fault这个错误?
> - 什么是busybox?
>
> 请仔细阅读[提问的智慧](https://github.com/ryanhanwu/How-To-Ask-Questions-The-Smart-Way/blob/master/README-zh_CN.md)和[别像弱智一样提问](https://github.com/tangx/Stop-Ask-Questions-The-Stupid-Ways/blob/master/README.md) (这篇文章很短, 1分钟就能看完)这两篇文章, 结合自己在大一时提问和被提问, 以及完成PA0的经历, 写一篇不少于800字的读后感, 谈谈你对"好的提问"以及"通过STFW和RTFM独立解决问题"的看法.
>
> Hint: 我们设置这道题并不是为了故意浪费大家的时间, 也不是为了禁止大家提出任何问题, 而是为了让大家知道"什么是正确的". 当你愿意为这些"正确的做法"去努力, 并且尝试用专业的方式提出问题的时候, 你就已经迈出了成为"成为专业人士"的第一步.



## Shell

> [!IMPORTANT]
>
> 学习Linux基本使用
>
> 我们给大家墙裂推荐MIT的Linux工具使用系列课程: [The Missing Semester of Your CS Education](https://missing-semester-cn.github.io/)中文版. 通过学习这些课程, 你将会了解到如何使用Linux中的工具来方便地完成各种任务, 这将大大提升你的工作效率.
>
> **必做题**:
>
> - 课程概览与shell
> - Shell工具和脚本
> - 编辑器 (Vim)
> - 数据整理
> - 命令行环境
> - 版本控制(Git)
>
> 包括阅读讲义并完成课后习题. 此外, [B站上](https://www.bilibili.com/video/BV1x7411H7wa)有这门公开课的视频供大家参考.



### 1.课程概览与shell

为了查看指定目录下包含哪些文件，我们使用 `ls` 命令

`mv`（用于重命名或移动文件）、 `cp`（拷贝文件）以及 `mkdir`（新建文件夹）

最简单的重定向是 `< file` 和 `> file`。这两个命令可以将程序的输入输出流分别重定向到文件：

```bash
man ls                    #查看指定目录下包含哪些文件
echo hello > hello.txt
cat missing/semester      # 查看内容
```



管道`|` 操作符允许我们将一个程序的输出和另外一个程序的输入连接起来：

```bash
missing:~$ ls -l / | tail -n1
drwxr-xr-x 1 root  root  4096 Jun 20  2019 var
missing:~$ curl --head --silent google.com | grep --ignore-case content-length | cut --delimiter=' ' -f2
219
```

常见的输入输出指令：

`head` # 显示开头部分；`tail` # 显示结尾部分；

输入指令：`ls`     `stat`    `cat`

输出指令：`echo`        

个人感觉不要拘泥于输入还是输出指令，反正它们都可以和head搭配利用管道和>，把字节写入文件中。

其他：`grep`   # 搜索文本；`sort`   # 排序；`uniq`   # 去重；`cut`  # 剪切列；`tr` # 字符替换



#### 课后习题一

1.本课程需要使用类 Unix shell，例如 Bash 或 ZSH。如果您在 Linux 或者 MacOS 上面完成本课程的练习，则不需要做任何特殊的操作。

2.在 `/tmp` 下新建一个名为 `missing` 的文件夹。

答：不知道是指home本身就有的文件夹tmp，我选择新建一个文件夹tmp

```bash
home:~$ mkdir tmp
home:~$ cd tmp
home:~/tmp$ mkdir missing
```

3.用 `man` 查看程序 `touch` 的使用手册。

```bash
home:~$ man touch 
```

4.用 `touch` 在 `missing` 文件夹中新建一个叫 `semester` 的文件。

```bash
home:~$ touch tmp/missing/semester
```

5.将以下内容一行一行地写入 `semester` 文件：

```
 #!/bin/sh
 curl --head --silent https://missing.csail.mit.edu
```

注意点：

1）使用单引号来进行写入带有#的字符串

第一行可能有点棘手， `#` 在 Bash 中表示注释，而 `!` 即使被双引号（`"`）包裹也具有特殊的含义。 单引号（`'`）则不一样，此处利用这一点解决输入问题。更多信息请参考 [Bash quoting 手册](https://www.gnu.org/software/bash/manual/html_node/Quoting.html)；

2）这个符号 > 是写入，即 echo 写入内容  >  文件路径/文件名，但是针对第一行写入，后面要是还用>写入，就会被覆盖，就是一直写的都是第一行；

3）写入第二行乃至后续行，需要使用 >>，其余不变。

```bash
home:~$ echo '#!/bin/sh' > tmp/missing/semester
home:~$ echo curl --head --silent https://missing.csail.mit.edu >>tmp/missing/semester

home:~$ cat tmp/missing/semester  #查看semester内容
#!/bin/sh
home:~$ curl --head --silent https://missing.csail.mit.edu
```

6.尝试执行这个文件。例如，将该脚本的路径（`./semester`）输入到您的 shell 中并回车。如果程序无法执行，请使用 `ls` 命令来获取信息并理解其不能执行的原因。

```bash
home:~$ tmp/missing/semester
bash: tmp/missing/semester: Permission denied  #没有操作权限
```

7.查看 `chmod` 的手册(例如，使用 `man chmod` 命令)

答：chmod指令全称为"change file mode bits"，改变文件模式位；有三种模式，分别是读+r，写+w，执行+x，分别对应数字4、2、1。

`+x`：给文件所有者、所属组和其他用户都加上执行权限（如果只想给所有者，可以用`u+x`）.

```
chmod +x 文件路径/文件名  #表示修改文件的执行权限：可以被所有用户执行
```

另外通常使用`./`来运行当前目录下的可执行文件，例如：`./tmp/missing/semester`；

8.使用 `chmod` 命令改变权限，使 `./semester` 能够成功执行，不要使用 `sh semester` 来执行该程序。您的 shell 是如何知晓这个文件需要使用 `sh` 来解析呢？更多信息请参考：[shebang](https://en.wikipedia.org/wiki/Shebang_(Unix))

```bash
home:~$ chmod +x tmp/missing/semester
home:~$ tmp/missing/semester
HTTP/1.1 200 Connection established

HTTP/2 200 
server: GitHub.com
content-type: text/html; charset=utf-8
last-modified: Sun, 25 Jan 2026 16:06:09 GMT
access-control-allow-origin: *
etag: "69763f71-274c"
expires: Sun, 01 Feb 2026 14:05:13 GMT
cache-control: max-age=600
x-proxy-cache: MISS
x-github-request-id: 2B73:6C21C:C03AA6:C4BA4F:697F5B40
accept-ranges: bytes
age: 0
date: Sun, 01 Feb 2026 13:55:13 GMT
via: 1.1 varnish
x-served-by: cache-nrt-rjtf7700080-NRT
x-cache: MISS
x-cache-hits: 0
x-timer: S1769954114.736836,VS0,VE177
vary: Accept-Encoding
x-fastly-request-id: 8983a552983af946cdf2b2630cb4ebda9ac55a40
content-length: 10060
```

#### （待定）shell 是如何知晓这个文件需要使用 `sh` 来解析呢？

答：



9.使用 `|` 和 `>` ，将 `semester` 文件输出的最后更改日期信息，写入主目录下的 `last-modified.txt` 的文件中。

1）`ls -l`获取文件的最后修改时间；`stat`获取文件的详细信息，包括最后修改时间；`stat -c %y`获取最后修改时间。

```bash
home:~$ ls -l tmp/missing/semester
-rwxrwxr-x 1 yylinux yylinux 61 Feb  1 21:54 tmp/missing/semester

home:~$ stat -c %y tmp/missing/semester
2026-02-01 21:54:29.480537579 +0800
```

额外的知识：这行代码是啥意思？

`-rwx rwx r-x 1 yylinux yylinux 61 Feb  1 21:54 tmp/missing/semester`

答：-指普通文件；rwx指文件所有人yylinux可以读写执行，所在同一用户组的其他成员也可以读写执行；r-x指其他用户只能读和执行，不能写；1指该文件拥有一个硬链接数；第一个yylinux是文件所有人；第二个yylinux是文件所有人所在的用户组，一般同名；61指文件大小是61KB；后面是文件最后的修改时间是2月1号21：54；文件所在位置。

下面是课程里的一个例子，很类似：

```bash
home:~$ ls -l /home
d rwx r-x r-x 1 missing  users  4096 Jun 15  2019 missing
```

`-l`这个参数可以更加详细地列出目录下文件或文件夹的信息。

- 首先，本行第一个字符 `d` 表示 `missing` 是一个目录。

- 然后，接下来的九个字符，每三个字符构成一组。 

  （`rwx`）. 它们分别代表了文件所有者(`missing`)，用户组(`users`)以及其他所有人具有的权限。

  其中 `-` 表示该用户不具备相应的权限。从上面的信息来看，只有文件所有者可以修改（`w`）`missing` 文件夹 （例如，添加或删除文件夹中的文件）。

为了进入某个文件夹，用户需要具备该文件夹以及其父文件夹的“搜索”权限（以“可执行”：`x`）权限表示。为了列出它的包含的内容，用户必须对该文件夹具备读权限（`r`）。对于文件来说，权限的意义也是类似的。注意，`/bin` 目录下的程序在最后一组，即表示所有人的用户组中，均包含 `x` 权限，也就是说任何人都可以执行这些程序。

**文件类型**

`d`：目录（directory）`-`：普通文件`l`：符号链接（软链接）`b`：块设备文件`c`：字符设备文件`s`：套接字文件`p`：管道文件

人话：目录文件`d`就是文件夹.



2）利用管道`|`和`>`输出符号写入：

个人感觉不要拘泥于输入还是输出指令，反正它们都可以和head搭配利用管道和>，把字节写入文件中。

```bash
home:~$ stat -c %y tmp/missing/semester | head -n 1 > ./last-modified.txt
home:~$ ls -l tmp/missing/semester | > ./last-modified.txt
```

 管道`|` 操作符允许我们将一个程序的输出和另外一个程序的输入连接起来：

```bash
missing:~$ ls -l / | tail -n1
drwxr-xr-x 1 root  root  4096 Jun 20  2019 var
missing:~$ curl --head --silent google.com | grep --ignore-case content-length | cut --delimiter=' ' -f2
219
```



10.写一段命令来从 `/sys` 中获取笔记本的电量信息，或者台式机 CPU 的温度。注意：macOS 并没有 sysfs，所以 Mac 用户可以跳过这一题。

1）获取电量

```bash
home:~$ cat /sys/class/power_supply/BAT*/capacity 2>/dev/null
100
home:~$ cat /sys/class/power_supply/BAT*/status 2>/dev/null
Full
```

2）读出CPU 的温度

```bash
# 详细查看所有thermal_zone以及温度
home:~$ paste <(cat /sys/class/thermal/thermal_zone*/type 2>/dev/null) \
              <(cat /sys/class/thermal/thermal_zone*/temp 2>/dev/null | 
              awk '{printf "%.1f°C\n", $1/1000}') | column -t
INT3400       Thermal  20.0°C
SEN1          40.0°C   
SEN2          38.0°C   
SEN3          36.0°C   
SEN6          36.0°C   
TCPU          40.0°C   
iwlwifi_1     40.0°C   
x86_pkg_temp  40.0°C   
```

a.`paste`命令：合并文件操作

```bash
paste file1.txt  file2.txt  #可以得到两个文件按列合并为一个文件
paste  <(echo "line1") <(echo "line2")  #从标准输入合并
paste  <(cat /sys/class/thermal/thermal_zone*/type 2>/dev/null)  \
       <(cat /sys/class/thermal/thermal_zone*/temp 2>/dev/null 
#从cat指令读到数据后合并
#\为连续符，意思为指令未完，增加可读性，无其他作用
```

b.`awk`命令：读取文件的某一行的  字符  /   数据   ， 再进行操作

`$1,$2...`表示第一列，第二列......

`column -t`：自动对齐，智能调整列宽，独立指令，与awk无关.

```bash
echo "hello world" | awk '{print $1}'  # 输出第一列：hello
echo "42000" | awk '{print $1/1000}'  # 42000/1000 = 42
awk '{printf "%.1f°C\n", $1/1000}')   # %.1f 表示保留 1位小数
awk '{printf "%.1f°C\n", $1/1000}') | column -t  #column -t：自动对齐，智能调整列宽
```



### 2.Shell 工具和脚本

#### Shell  脚本

##### 1.变量赋值&引用

在 bash 中为变量赋值的语法是 `foo=bar`，访问变量中存储的数值，引用foo的语法为 `$foo`；即，赋值——变量1=变量2；引用变量1——$变量1 ；

注意：`foo = bar` （使用空格隔开）是不能正确工作的，因为解释器会调用程序 `foo` 并将 `=` 和 `bar` 作为参数。 总的来说，在 shell 脚本中使用空格会起到分割参数的作用，有时候可能会造成混淆；

##### 2.Bash 中的 `'` 和 `"`

Bash 中的字符串通过 `'` 和 `"` 分隔符来定义，但是它们的含义并不相同。以 `'` 定义的字符串为原义字符串，其中的变量不会被转义，而 `"` 定义的字符串会将变量值进行替换。

```bash
home:~$ foo=bar
home:~$ echo $foo
bar
home:~$ echo '$foo' 
$foo
home:~$ echo "$foo"
bar
```

可以看出直接打印$变量，和打印带`'`的效果是一样的。



[习题解答](https://missing-semester-cn.github.io/missing-notes-and-solutions/2020/solutions//shell-tools-solution)

（练习1）阅读 [`man ls`](https://man7.org/linux/man-pages/man1/ls.1.html) ，然后使用 `ls` 命令进行如下操作：

- 所有文件（包括隐藏文件）

```bash
# 所有文件（包括隐藏文件）
home:~$ ls -a
.              .bash_logout  .dotnet         .gnupg    .pam_environment  .ssh                       .viminfo    文档
..             .bashrc       Downloads       .lesshst  .pki              .sudo_as_admin_successful  .vscode
a.c            .cache        E3_Makefile_c   .local    .profile          .sys1og.conf               .wget-hsts
.a.c.swp       .config       E4              .npm      Screenshots       .test.swp                  .xwechat
.bash_history  Documents     .empty_desktop  .nvm      snap              tmp                        ysyx_note
```

- 文件打印以人类可以理解的格式输出 (例如，使用 454M 而不是 454279954)

```bash
home:~$ ls -lh
total 44K
-rw-rw-r-- 1 yylinux yylinux   10 Feb  3 12:13 123456789.txt
-rw-rw-r-- 1 yylinux yylinux  136 Jan 31 16:43 a.c
drwx------ 3 yylinux yylinux 4.0K Jan 31 11:36 Documents
drwxr-xr-x 2 yylinux yylinux 4.0K Feb  2 20:54 Downloads
drwxrwxr-x 2 yylinux yylinux 4.0K Feb  2 21:25 E3_Makefile_c
drwxrwxr-x 6 yylinux yylinux 4.0K Feb  3 01:05 E4
drwxrwxr-x 3 yylinux yylinux 4.0K Feb  2 20:15 Screenshots
drwx------ 8 yylinux yylinux 4.0K Feb  2 21:29 snap
drwxrwxr-x 3 yylinux yylinux 4.0K Feb  1 21:20 tmp
drwx------ 3 yylinux yylinux 4.0K Feb  3 12:17 ysyx_note
drwx------ 3 yylinux yylinux 4.0K Jan 26 13:20 文档
```

- 文件以最近修改顺序排序

```bash
home:~$ ls -lt
total 44
-rw-rw-r-- 1 yylinux yylinux   10 Feb  3 12:13 123456789.txt
drwx------ 3 yylinux yylinux 4096 Feb  3 12:11 ysyx_note
drwxrwxr-x 6 yylinux yylinux 4096 Feb  3 01:05 E4
drwx------ 8 yylinux yylinux 4096 Feb  2 21:29 snap
drwxrwxr-x 2 yylinux yylinux 4096 Feb  2 21:25 E3_Makefile_c
drwxr-xr-x 2 yylinux yylinux 4096 Feb  2 20:54 Downloads
drwxrwxr-x 3 yylinux yylinux 4096 Feb  2 20:15 Screenshots
drwxrwxr-x 3 yylinux yylinux 4096 Feb  1 21:20 tmp
-rw-rw-r-- 1 yylinux yylinux  136 Jan 31 16:43 a.c
drwx------ 3 yylinux yylinux 4096 Jan 31 11:36 Documents
drwx------ 3 yylinux yylinux 4096 Jan 26 13:20 文档
```

- 以彩色文本显示输出结果

```bash
home:~$ ls --color
home:~$ ls --color=auto    #只有在终端显示颜色
home:~$ ls --color=always  #始终显示颜色
home:~$ ls --color=never  #永不显示颜色
```

![image-20260203123049428](https://gitee.com/brownie145810/ysyx_pic/raw/master/ysyx_pic/image-20260203123049428.png)

##### 3.Bash中的函数

（练习2）编写两个 bash 函数 `marco` 和 `polo` 执行下面的操作。 每当你执行 `marco` 时，当前的工作目录应当以某种形式保存，当执行 `polo` 时，无论现在处在什么目录下，都应当 `cd` 回到当时执行 `marco` 的目录。 为了方便 debug，你可以把代码写在单独的文件 `marco.sh` 中，并通过 `source marco.sh` 命令，（重新）加载函数。

```bash
#marco.sh
marco(){
     #保存当前目录 作为环境变量，pwd是获取当前工作目录的命令
     export marco_path=$(pwd);
}
polo(){
    cd "$marco_path"; #双引号引用，进行变量替换
}
```

`pwd`(printf working directory)是获取当前工作目录的命令；

`$(pwd)`将获取的工作目录保存为字符变量；

`export`设置当前变量为环境变量；

进行调用，注意一定要在当前shall下执行 `source marco.sh` ，加载函数：

```bash
home:~$ source marco.sh
home:~$ cd E4
home:~/E4$ cd E4_3
home:~/E4/E4_3$ marco #调用marco，保存当前工作目录
home:~/E4/E4_3$ cd
home:~$ polo
home:~/E4/E4_3$ home #返回成功
```



（练习3）假设您有一个命令，它很少出错。因此为了在出错时能够对其进行调试，需要花费大量的时间重现错误并捕获输出。 编写一段 bash 脚本，运行如下的脚本直到它出错，将它的标准输出和标准错误流记录到文件，并在最后输出所有内容。 加分项：报告脚本在失败前共运行了多少次。

```bash
 #!/usr/bin/env bash

 n=$(( RANDOM % 100 ))

 if [[ n -eq 42 ]]; then
    echo "Something went wrong"
    >&2 echo "The error was using magic numbers"
    exit 1
 fi

 echo "Everything went according to plan"
```

进行脚本编写：

```bash
#!/usr/bin/env bash

# 设置日志文件
LOG_FILE="count_wrong.txt"

# 清空日志文件
:> "$LOG_FILE"  #最前面加上true或者：
#否则报错——该命令行的重定向操作符 > 前面没有命令

count=0

while true ;do #没有固定次数的循环使用while，不使用for
	count=$((count+1))
	n=$(( RANDOM % 100 ))
	
if [[ $n -eq 42 ]]; then #注意双[[]]内部的空格
	# exec >> count_wrong.txt 2>&1
	echo "Something went wrong" >>  count_wrong.txt 
	echo "The error was using magic numbers">> count_wrong.txt  
	break;	
fi
	echo "Everything went according to plan,成功运行$count次" >> count_wrong.txt 

done
echo "脚本在失败前，成功运行$((count-1))次"	>>  count_wrong.txt  
cat "$LOG_FILE" #注意“$变量名”，$在内部

exit 1
```

##### 疑问以及解答

###### 1）`source`运行脚本是什么方式，为啥不能使用它来运行？

答：`source`运行原理：相当于把脚本复制粘贴到shell运行，不建立子shell，只在当前的shell成立，所以`source`可以使用来更新函数 ；

如果使用`source`来运行脚本，那么脚本里的内容就会影响当前shell——如果脚本里有`exit`，那么就会使整个shell退出，出现“不返回”；

###### 2）`exit 1`是什么作用？为什么不能加？

 答：`exit`的作用是退出当前shell，并返回执行结果的参数：

```bash
exit 0   # 正常退出，表示成功
exit 1   # 非正常退出，表示失败
exit 2   # 其他错误
```

`exit 1`可以加，但是不能是`source`运行脚本，要利用`./`来运行脚本，

原理：`./`运行脚本是为当前脚本创建一个子shell, 而我调用`./`脚本的shell叫做父shell, 子shell里面有`exit 1`最终只是退出子shell而不是父shell，所以可以正常执行并且返回父shell。

###### 3）`fi`是`if`的结束标志吗？

答：是的

###### 4）命令 `tee` 从标准输入读入并写往标准输出和文件

-a, --append追加到给出的文件，而不是覆盖

```bash
echo "Something went wrong" |tee -a count_wrong.txt  # 同时显示和追加到文件 	
```

###### 5）脚本可以是任何语言？

是的，例如脚本的第一行`#!/usr/bin/env bash`，在 `shebang` 行中使用 [`env`](https://man7.org/linux/man-pages/man1/env.1.html) 命令是一种好的实践，它会利用环境变量中的程序来解析该脚本的语言，提高了脚本的可移植性。`env` 会利用 `PATH` 环境变量来进行定位，上述代码就把脚本定义为了bash.

##### 错误总结

- 一般不使用`source`运行脚本，因为i脚本中的命令会直接影响当前Shell环境，所以脚本里不能有`exit 1`；

- `exit`用于退出整个脚本（如果是`source`运行，则退出当前Shell）,`exit`可以和`./`联合使用；

- `fi`是`if`语句的结束标记

- | tee -a 文件名：利用管道和tee实现即在终端输出又实现存入文件；

  

#### Shell 工具

##### 1.查看命令如何使用

（练习4）本节课我们讲解的 `find` 命令中的 `-exec` 参数非常强大，它可以对我们查找的文件进行操作。但是，如果我们要对所有文件进行操作呢？例如创建一个 zip 压缩文件？我们已经知道，命令行可以从参数或标准输入接受输入。在用管道连接命令时，我们将标准输出和标准输入连接起来，但是有些命令，例如 `tar` 则需要从参数接受输入。这里我们可以使用 [`xargs`](https://man7.org/linux/man-pages/man1/xargs.1.html) 命令，它可以使用标准输入中的内容作为参数。 例如 `ls | xargs rm` 会删除当前目录中的所有文件。

您的任务是编写一个命令，它可以递归地查找文件夹中所有的 HTML 文件，并将它们压缩成 zip 文件。注意，即使文件名中包含空格，您的命令也应该能够正确执行（提示：查看 `xargs` 的参数 `-d`)

```bash
#!/usr/bin/env bash
find . -name "*.html" -type f | while IFS= read -r file; do
    zip -j "${file%.html}.zip" "$file"
done
ls -l *.zip
exit 1
```

###### `zip -j`压缩命令

`-j` 选项- 不存储路径信息

- `j` 代表 "junk paths"（丢弃路径）
- 只存储文件本身，不存储目录结构
- 例如：`./dir/subdir/file.html` 会被存储为 `file.html`，而不是 `dir/subdir/file.html`



###### `IFS` - 内部字段分隔符

- **IFS** = Internal Field Separator（内部字段分隔符）
- 默认值：空格、制表符、换行符
- **`IFS=`**：设置为空，禁用分词（word splitting）



输出：

```bash
home:~$ ./find.sh
... # 执行信息
-rw-rw-r-- 1 Yang Yang 174 Feb  8 10:07  1.zip
-rw-rw-r-- 1 Yang Yang 180 Feb  8 10:07 '2  1.zip'
-rw-rw-r-- 1 Yang Yang 174 Feb  8 10:08  3.zip
-rw-rw-r-- 1 Yang Yang 182 Feb  8 10:08 '4   3.zip'
-rw-rw-r-- 1 Yang Yang 174 Feb  8 10:08  5.zip
```



（练习5-进阶）编写一个命令或脚本递归的查找文件夹中最近修改的文件。更通用的做法，你可以按照最近的修改时间列出文件吗？

``` bash
find type -f exec ls -lm {}+ 
```

###### 解析`find   exec {} +`：

`exec ... {}`中的`{}`会被替换成`find`的搜索结果，

结束符号：`+`表示一次性列出所有文件结果；`/；`表示列出所有文件结果的详细信息.



输出

```bash
160b14_0
-rw------- 1 Yang Yang   144102 Feb  5 14:35  ./snap/yelmusic/Cache/95a8588717323fb9_0
-rw------- 1 Yang Yang   425371 Feb  5 14:35  ./snap/yelmusic/Cache/9f032dfe456a65ad_0
-rw------- 1 Yang Yang     5857 Feb  5 14:35  ./snap/yelmusic/Cache/f836eff4dac8578d_0
-rw------- 1 Yang Yang      223 Feb  5 14:35 './snap/yelmusic/Code Cache/js/beb81e82b6e3debc_0'
-rw-rw-rw- 1 Yang Yang       76 Feb  5 14:35  ./snap/yelmusic/config.json
-rw------- 1 Yang Yang      335 Feb  5 14:35  ./snap/yelmusic/IndexedDB/http_localhost_27232.indexeddb.leveldb/LOG
...
```



### 3.编辑器 (Vim)

#### 课后练习

[习题解答](https://missing-semester-cn.github.io/missing-notes-and-solutions/2020/solutions//editors-solution)

1. 完成 `vimtutor`。备注：它在一个 [80x24](https://en.wikipedia.org/wiki/VT100)（80 列，24 行） 终端窗口看起来效果最好。

   ```bash
   home:~$ vimtutor zh  # 执行后会出现中文版vim使用手册
   ```

   

2. 下载我们提供的 [vimrc](https://missing-semester-cn.github.io/2020/files/vimrc)，然后把它保存到 `~/.vimrc`。 通读这个注释详细的文件 （用 Vim!）， 然后观察 Vim 在这个新的设置下看起来和使用起来有哪些细微的区别。

   已经安装，并且按照自己的习惯修改了一些配置

   ```.vimrc
   " 启用语法高亮
   syntax on
   " 基本显示
   set number              " 显示行号
   "set relativenumber      " 显示相对行号
   set cursorline          " 高亮当前行
   ```

   

3. 安装和配置一个插件： [ctrlp.vim](https://github.com/ctrlpvim/ctrlp.vim).

   1. 用 `mkdir -p ~/.vim/pack/vendor/start` 创建插件文件夹

   2. 下载这个插件： `cd ~/.vim/pack/vendor/start; git clone https://github.com/ctrlpvim/ctrlp.vim`

   3. 阅读这个插件的 [文档](https://github.com/ctrlpvim/ctrlp.vim/blob/master/readme.md)。 尝试用 CtrlP 来在一个工程文件夹里定位一个文件，打开 Vim, 然后用 Vim 命令控制行开始 `:CtrlP`.

   4. 自定义 CtrlP：添加 [configuration](https://github.com/ctrlpvim/ctrlp.vim/blob/master/readme.md#basic-options) 到你的 `~/.vimrc` 来用按 Ctrl-P 打开 CtrlP

      

4. ~~练习使用 Vim, 在你自己的机器上重做 [演示](https://missing-semester-cn.github.io/2020/editors/#demo)。~~

5. ~~下个月用 Vim 完成 *所有的* 文件编辑。每当不够高效的时候，或者你感觉 “一定有一个更好的方式”时， 尝试求助搜索引擎，很有可能有一个更好的方式。如果你遇到难题，可以来我们的答疑时间或者给我们发邮件。~~

6. ~~在其他工具中设置 Vim 快捷键 （见上面的操作指南）。~~

7. ~~进一步自定义你的 `~/.vimrc` 和安装更多插件。~~

   A：4～7在学习前面的内容已经完成

   

8. （高阶）用 Vim 宏将 XML 转换到 JSON ([例子文件](https://missing-semester-cn.github.io/2020/files/example-data.xml))。 尝试着先完全自己做，但是在你卡住的时候可以查看上面 [宏](https://missing-semester-cn.github.io/2020/editors/#macros) 章节。

   A：(待定)



### 4.数据整理

主要内容：不断地对数据进行处理，直到得到我们想要的最终结果

#### 课后练习

[习题解答](https://missing-semester-cn.github.io/missing-notes-and-solutions/2020/solutions//data-wrangling-solution)

1. 学习一下这篇简短的 [交互式正则表达式教程](https://regexone.com/).

2. 统计 words 文件 (`/usr/share/dict/words`) 中包含至少三个 `a` 且不以 `'s` 结尾的单词个数。这些单词中，出现频率前三的末尾两个字母是什么？ `sed` 的 `y` 命令，或者 `tr` 程序也许可以帮你解决大小写的问题。共存在多少种词尾两字母组合？还有一个很 有挑战性的问题：哪个组合从未出现过？

3. 进行原地替换听上去很有诱惑力，例如： `sed s/REGEX/SUBSTITUTION/ input.txt > input.txt`。但是这并不是一个明智的做法，为什么呢？还是说只有 `sed` 是这样的? 查看 `man sed` 来完成这个问题

4. 找出您最近十次开机的开机时间平均数、中位数和最长时间。在 Linux 上需要用到 `journalctl` ，而在 macOS 上使用 `log show`。找到每次起到开始和结束时的时间戳。在 Linux 上类似这样操作：

   ```
   Logs begin at ...
   ```

   和

   ```
   systemd[577]: Startup finished in ...
   ```

   

5. 查看之前三次重启启动信息中不同的部分(参见 `journalctl` 的 `-b` 选项)。将这一任务分为几个步骤，首先获取之前三次启动的启动日志，也许获取启动日志的命令就有合适的选项可以帮助您提取前三次启动的日志，亦或者您可以使用 `sed '0,/STRING/d'` 来删除 `STRING` 匹配到的字符串前面的全部内容。然后，过滤掉每次都不相同的部分，例如时间戳。下一步，重复记录输入行并对其计数(可以使用 `uniq` )。最后，删除所有出现过 3 次的内容（因为这些内容是三次启动日志中的重复部分）。

6. 在网上找一个类似 [这个](https://stats.wikimedia.org/EN/TablesWikipediaZZ.htm) 或者 [这个](https://ucr.fbi.gov/crime-in-the-u.s/2016/crime-in-the-u.s.-2016/topic-pages/tables/table-1) 的数据集。或者从 [这里](https://www.springboard.com/blog/free-public-data-sets-data-science-project/) 找一些。使用 `curl` 获取数据集并提取其中两列数据，如果您想要获取的是 HTML 数据，那么 [`pup`](https://github.com/EricChiang/pup) 可能会更有帮助。对于 JSON 类型的数据，可以试试 [`jq`](https://stedolan.github.io/jq/)。请使用一条指令来找出其中一列的最大值和最小值，用另外一条指令计算两列之间差的总和。



