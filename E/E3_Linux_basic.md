# E3_Linux入门

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

## 1.课程概览与shell

为了查看指定目录下包含哪些文件，我们使用 `ls` 命令

`mv`（用于重命名或移动文件）、 `cp`（拷贝文件）以及 `mkdir`（新建文件夹）

请试试 `man` 这个程序。它会接受一个程序名作为参数，然后将它的文档（用户手册）展现给您。注意，使用 `q` 可以退出该程序。

```bash
missing:~$ man ls
```

为了查看指定目录下包含哪些文件，我们使用 `ls` 命令

最简单的重定向是 `< file` 和 `> file`。这两个命令可以将程序的输入输出流分别重定向到文件：

```bash
missing:~$ echo hello > hello.txt
missing:~$ cat hello.txt
hello
missing:~$ cat < hello.txt
hello
missing:~$ cat < hello.txt > hello2.txt
missing:~$ cat hello2.txt
hello
```



```bash
cat missing/semester      # 查看内容
wc -l missing/semester    # 查看行数（应该是2行）
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

### 课后习题一

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

### （待定）shell 是如何知晓这个文件需要使用 `sh` 来解析呢？

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
missing:~$ ls -l /home
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

```
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

```
echo "hello world" | awk '{print $1}'  # 输出第一列：hello
echo "42000" | awk '{print $1/1000}'  # 42000/1000 = 42
awk '{printf "%.1f°C\n", $1/1000}')   # %.1f 表示保留 1位小数
awk '{printf "%.1f°C\n", $1/1000}') | column -t  #column -t：自动对齐，智能调整列宽
```

## 2.Shell 工具和脚本

### Shell 脚本

#### 1.变量赋值&引用  

在 bash 中为变量赋值的语法是 `foo=bar`，访问变量中存储的数值，引用foo的语法为 `$foo`；即，赋值——变量1=变量2；引用变量1——$变量1 ；

注意：`foo = bar` （使用空格隔开）是不能正确工作的，因为解释器会调用程序 `foo` 并将 `=` 和 `bar` 作为参数。 总的来说，在 shell 脚本中使用空格会起到分割参数的作用，有时候可能会造成混淆；

#### 2.Bash 中的 `'` 和 `"` 

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

#### 3.Bash中的函数

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

###### 1）source运行脚本是什么方式，为啥不能使用它来运行？

答：source运行原理：相当于把脚本复制粘贴到shell运行，不建立子shell，只在当前的shell成立，所以source可以使用来更新函数 ；

如果使用source来运行脚本，那么脚本里的内容就会影响当前shell——如果脚本里有exit，那么就会使整个shell退出，出现“不返回”；

###### 2）exit 1是什么作用？为什么不能加？

 答：exit的作用是退出当前shell，并返回执行结果的参数：

```
exit 0   # 正常退出，表示成功
exit 1   # 非正常退出，表示失败
exit 2   # 其他错误
```

exit 1可以加，但是不能是source运行脚本，要利用./来运行脚本，

原理：./运行脚本是为当前脚本创建一个子shell，而我调用./脚本的shell叫做父shell，子shell里面有exit 1最终只是退出子shell而不是父shell，所以可以正常执行并且返回父shell。

###### 3）fi是if的结束标志吗？

答：是的

###### 4）命令 tee - 从标准输入读入并写往标准输出和文件

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

  

### Shell 工具

#### 1.查看命令如何使用

（练习4）本节课我们讲解的 `find` 命令中的 `-exec` 参数非常强大，它可以对我们查找的文件进行操作。但是，如果我们要对所有文件进行操作呢？例如创建一个 zip 压缩文件？我们已经知道，命令行可以从参数或标准输入接受输入。在用管道连接命令时，我们将标准输出和标准输入连接起来，但是有些命令，例如 `tar` 则需要从参数接受输入。这里我们可以使用 [`xargs`](https://man7.org/linux/man-pages/man1/xargs.1.html) 命令，它可以使用标准输入中的内容作为参数。 例如 `ls | xargs rm` 会删除当前目录中的所有文件。

您的任务是编写一个命令，它可以递归地查找文件夹中所有的 HTML 文件，并将它们压缩成 zip 文件。注意，即使文件名中包含空格，您的命令也应该能够正确执行（提示：查看 `xargs` 的参数 `-d`，译注：MacOS 上的 `xargs` 没有 `-d`，[查看这个 issue](https://github.com/missing-semester/missing-semester/issues/93)）

如果您使用的是 MacOS，请注意默认的 BSD `find` 与 [GNU coreutils](https://en.wikipedia.org/wiki/List_of_GNU_Core_Utilities_commands) 中的是不一样的。你可以为 `find` 添加 `-print0` 选项，并为 `xargs` 添加 `-0` 选项。作为 Mac 用户，您需要注意 mac 系统自带的命令行工具和 GNU 中对应的工具是有区别的；如果你想使用 GNU 版本的工具，也可以使用 [brew 来安装](https://formulae.brew.sh/formula/coreutils)。

（练习5-进阶）编写一个命令或脚本递归的查找文件夹中最近修改的文件。更通用的做法，你可以按照最近的修改时间列出文件吗？
