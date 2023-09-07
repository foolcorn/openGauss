## 前言

在学习一样东西之前，首先需要弄清楚这个东西有什么用，有什么使用价值，有什么实际意义，才不会迷失方向。

学习Shell之后，通过编写shell命令，可以喂入shell的解析器（某个极高权限的后台进程）与linux内核直接交互，并调用linux的内核函数操作计算机硬件（硬件CPU，内存，磁盘，显示器）。将多条Shell命令，变量，函数等，通过Shell程序语法组织在一起后，构成具备类似高级程序语言逻辑结构的Shell脚本，可一次性执行多条执行令并实现更加复杂的功能。

学会Shell以后，不论是读懂某个软件的编译脚本，编译出现错误去溯源，还是自己去编写脚本做些小测试，亦或是配置环境变量，这些都变得简单至极。

本文将按以下内容组织Shell编程相关知识：

- Shell入门
- Shell变量
- Shell常用命令
- Shell运算符和运算命令
- 流程控制语句：if else for
- Shell函数
- 重定向
- Shell工具（sed，awk）



##  （一）Shell 入门

### Shell解析器

一个linux系统会默认下载多个Shell解释器，这些Shell解释器功能各异，使用哪种Shell进行编程因个人喜好而异，本文内容主要针对bash 

可以通过以下命令查看自己linux系统支持的shell解析器。

```shell
cat /etc/shells
```

![image-20230723164600328](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230723164600328.png)

| 解析器        | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| /bin/sh       | Bourne Shell，1979年开发的原始Unix Shell，是Unix的默认解释器（《linux系统编程》有相关的历史） |
| /bin/bash     | Bourne Again Shell，简称bash，是原始Shell的兼容版，是目前其它Unix系列的默认Shell，提供命令补全，别名，条件表达式等等语法，具有灵活和强大的编辑接口，又有友好的用户界面，交互性很强。（旧的mac os系统也是默认bash，不过现在已经改Z shell了） |
| /usr/bin/bash | 在一些 Linux 发行版中，`/usr` 目录可能是一个单独的文件系统，可以被挂载为只读，而 `/bin` 目录是一个独立的文件系统，可以被挂载为可读写。这种分离的设计可以使系统在某些情况下更加灵活和可靠。`/bin/bash` 和 `/usr/bin/bash` 实际上是同一个 Bash 可执行文件的不同路径。无论你使用哪个路径，都可以启动 Bash Shell。 |
| /sbin/nologin | `/sbin/nologin` 是一个具有非交互性质的 Shell，它不提供任何命令行交互功能。它的主要目的是用于限制用户登录并防止他们执行任何交互式操作（关于登陆Shell环境和非登录Shell环境的区别会在后续章节进行说明）。这在一些情况下是有用的，例如当你希望创建一个系统账户，该账户只用于运行特定的服务或程序，而不允许用户以该账户登录并执行其他操作。 |
| /bin/dash     | Dash 是一个轻量级的 Unix Shell，它被设计为一个快速和高效的替代品，主要用于系统启动期间的初始化脚本和系统脚本,与 Bash Shell 相比，Dash Shell 不包含一些高级功能和扩展，如命令补全、历史命令管理和高级脚本编程功能，不适合用作交互式 Shell。 |
| /bin/csh      | C Shell 是由 Bill Joy 在 1979 年开发的一种 Unix Shell。它引入了许多新的特性，如命令行编辑、历史命令和 C 风格的语法。C Shell 在某些方面更加友好和易用，特别适用于编程和交互式使用。 |
| /bin/tcsh     | tcsh Shell 具有与 C Shell 类似的语法和命令，但增加了一些扩展功能，如命令行编辑、历史命令管理、别名扩展、通配符扩展、作业控制等。它还支持 C Shell 的脚本语法，并添加了一些额外的语法和控制结构，使得编写脚本更加灵活和强大。 |



通过以下命令可以查看系统默认的shell

```shell
echo $SHELL
```

- echo命令：打印输出数据到终端
- $SHELL：是一个Shell的环境变量



### Shell脚本

#### 1.文件名

以.sh为后缀

#### 2.首行固定语法

```shell
#!/bin/bash
```

- 设置Shell解释器的类型

#### 3. 注释

单行注释，'#'开头

```shell
# xxxxxxx comment conetent xxxxxxxx
```

多行注释，以 ':<!' 开头 '!' 结尾

```shell
:<!
xxxxxxx comment conetent1 xxxxxxxx
xxxxxxx comment conetent2 xxxxxxxx
xxxxxxx comment conetent3 xxxxxxxx
!
```

#### 4.执行脚本

三种方式：

``` shell
(1) sh xxx.sh #sh是指向默认Shell解释器的符号链接，实际是调用/bin/bash（默认）。
(2) bash xxx.sh #bash是指向Bash解释器的符号链接，实际是调用/bin/bash。
(3) ./xxx.sh #直接在当前路径下执行脚本，但是需要脚本有可执行权限，需要使用chomod +x xxx.sh命令更改权限。
```





## （二）Shell变量

shell中一共有三种变量类型

- 自定义变量
- 特殊符号变量
- 环境变量

### 1.自定义变量

#### （1）局部变量

只定义在一个脚本文件中的变量，变量可以由字母，数字和下划线组成，不能以数字开头。

##### ①定义局部变量

```shell
value_name=value                 #shell语法中"="两边一定不能有空格
"value name"=value               #若变量名内包含空格，则需要带双引号
```

变量的默认类型都是字符串，哪怕赋值为1,2,3...

##### ②查询局部变量

```shell
echo $value_name                 #直接使用变量名
echo ${value_name}               #使用花括号可以给value_name进行字符串拼接
```

##### ③删除局部变量

```shell
unset value_name
```



#### （2）常量

##### ①定义常量

```shell
readonly value_name              #将已经定义的变量名设置为常量
readonly value_name=value        #定义的同时设置为常量
```



#### （3）全局变量

##### ①父子Shell

和父子进程的概念类似，如果一个Shell脚本在执行过程中调用执行了另一个Shell脚本，则称其为父子Shell，

且子Shell可以访问父Shell定义的变量，只需将其设置为全局变量。

##### ②定义全局变量

```shell
export value_name               #将已经定义的变量名设置为全局变量
export value_name=value         #定义的同时设置为全局变量
```



###  2.特殊符号变量

####  （1）$n

用于接收脚本文件执行时传入的参数

```shell
$0                              #脚本文件名
$1~$9                           #输入参数第1~9
${10+}                          #输入参数第10及以上要加花括号           
```

执行脚本时传入参数

```shell
sh test.sh value1 value2 value3 ...
```

#### （2） $*、$@

一次性获取所有输入参数

```shell
echo $* 和 echo $@ 的结果一样
echo "$*"                       #加双引号的$*是拼接所有参数的字符串
for item in "$@"                #加双引号的$@会转变为字符串列表对象 
do
    echo $item
done
```

执行脚本时传入参数

```shell
sh test.sh value1 value2 value3 ...
```

#### （3）$#

获取输入参数的个数

#### （4）$$

用于获取当前Shell环境的进程id号



### 3.环境变量

环境变量存储在Shell的配置文件中，linux会加载该配置文件并将环境变量共享给所有Shell程序使用

#### （1）配置文件

①全局配置文件：

- /etc/profile
- /etc/profile.d/*sh
- /etc/bashrc

②个人用户配置文件(都是隐藏文件，用ls -a可查看)

- 用户目录/.bash_profile
- 当前用户/.bashrc

![image-20230723164707417](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230723164707417.png)

#### （2）系统级和用户级环境变量

系统级环境变量：存储在全局配置文件中，所有用户的Shell程序都共享。

用户级环境变量：存储在个人目录下的配置文件中，只有当前登陆的用户能共享。

env命令可以查询当前所有的环境变量（系统+用户）

```
env
```

![image-20230723164840511](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230723164840511.png)



set命令可以查询Shell当前的所有变量（环境变量+自定义变量+函数变量，这些都通过shell后台进程，存储在内存中，不以任何一个shell脚本为转移。）

```shell
set
```

![image-20230723164931958](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230723164931958.png)



##### 常用的系统级环境变量

| 变量名   | 含义                                                         |
| -------- | ------------------------------------------------------------ |
| PATH     | 与windows环境变量PATH功能一样，设置命令的搜索路径，以冒号分割 |
| HOME     | 当前用户的主目录                                             |
| SHELL    | 当前shell的解析器类型：/bin/bash                             |
| HISTFILE | 显示当前用户执行命令的历史列表文件：用户目录/.bash_history   |
| PWD      | 当前所在路径                                                 |
| OLDPWD   | 之前的路径                                                   |
| HOSTNAME | 当前的主机名                                                 |
| HOSTTYPE | 主机的架构：i386，i686 x86 x64...                            |
| LANG     | 当前系统的语言环境                                           |



##### 自定义系统环境变量

编辑/etc/profile文件，并在其中添加命令

```shell
export VALUE_NAME = xxxx
```

重新加载配置文件/etc/profile

```shell
source /etc/profile
```



#### （3）Shell登陆环境

shell登陆环境：需要用户名和密码登陆的shell环境，一般的ssh工具连接后都是进入到登陆的shell环境。

shell非登录环境：不需要用户名和密码进入的shell环境。

##### 两种登陆环境加载的配置文件

![image-20230723171412878](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230723171412878.png)

##### 查询目前的shell环境

```shell
echo $0                         #结果是"-bash"则是登陆环境，如果是"bash"则是非登录环境
```



##### 以登陆shell环境执行脚本

①以命令的形式

```shell
sh test.sh -l
sh test.sh --login
bash test.sh -l
bash test.sh --login
```

②使用ssh远程连接默认进入登陆环境

③切换用户进入登陆环境

```shell
su 用户名 -l/--login             #进入登陆环境
sh test.sh
```



##### 以非登陆shell环境执行脚本

①退出命令的形式

```shell
bash                            #执行该命令以退出登陆环境
sh test.sh
```

②切换用户进入登陆环境

```shell
su 用户名                        #进入非登陆环境
sh test.sh
```



##### 查询目前的shell环境

```shell
echo $0                         #结果是"-bash"则是登陆环境，如果是"bash"则是非登录环境
```



### 4.Shell常用变量类型

#### （1）字符串变量

##### 定义字符串变量

```shell
var1='abc'                      #使用单引号只能定义纯字符的字符串变量
var2="${var1}_test"             #使用双引号则会解析其中是否包含$变量，因此定义时可以利用${}拼接字符串
var3=abc def                    #什么都不加默认等同于""，但是不能包含空格，否则只能定义成abc不包括def
```

##### 获取字符串长度

```shell
echo ${#var1}
```

##### 字符串拼接

```shell
var1=abc                      
var2=def             
var3=${var1}${var2}             #不用双引号，直接写到一块即可拼接，但是不能包含空格
var4="${var1} ${var2}"          #使用双引号可以直接包含空格
var5=${var1}" "${var2}          #在中间使用一个''或""字符串从而粘贴两边，可以在中间字符串中包含空格
```

##### 字符串截取

```shell
${var1:start:length}            #从左边start下标字符开始（首位下标0）向右截取length个字符
${var1:start}                   #从左边start下标字符开始向右截取所有字符
${var1:0-start:length}          #从右边第start字符开始向右截取length个字符，0必须要有
${var1:0-start}                 #从右边第start字符开始向右截取所有字符
${var1#*chars}                  #左边首次匹配到chars串开始向右截取所有字符
${var1##*chars}                 #左边最后一次匹配到chars串开始向右截取所有字符
${var1%chars*}                  #右边第一次出现chars串开始向左截取所有字符
${var1%%chars*}                 #右边最后一次出现chars串开始向左截取所有字符
```

#### （2）索引数组

##### 定义索引数组

```shell
arr1=(ele1 els2 ele3)                  #按元素顺序初始化数组，用空格隔开
arr2=([0]=ele1 [1]=ele2 [9]=ele10)     #指定索引进行初始化，索引可以不用连续，中间的元素并不会被初始化，也不计入数组长度
arr3=(${arr1[*]} ${arr2[@]})           #也可以拼接定义好的数组
```

##### 获取数组元素

```shell
echo ${arr1[0]}                   #利用下标，和常见的编程语言无二，唯一区别是要用花括号{}
```

##### 获取所有数组元素

```shell
echo ${arr1[*]}
echo ${arr1[@]}                   #如果加了双引号可以转变为数组列表使用循环遍历
```

##### 获取数组长度

```shell
echo ${#arr1[*]}
echo ${#arr1[@]}                
```

##### 如果数组元素是字符串，可以获取其字符的长度

```shell
echo ${#arr1[0]}                 #利用下标先获取数组元素，再用#获取字符长度
```

##### 删除数组

```shell
unset arr1[0]                    #删除指定元素
unset arr1                       #删除整个数组
```



## （三）Shell常用命令

##### （1）type

用于查询某个指令是否是shell的内置指令，还是某些可执行的shell脚本。

![image-20230723222228324](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230723222228324.png)

cd是shell builtin即内置指令，ifconfig则是目录下的可执行脚本



##### （2）alias

给命令起别名，和C语言的#define类似。若不加任何参数，则显示当前shell环境中所有别名列表。

![image-20230723224540139](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230723224540139.png)

常见的像"ll"都是别名，比较坑的是cp指令，有的时候为了强制覆盖而不询问选择-f参数，然后默认的cp已经赋予了-i参数，会使-f参数失效

因为想要强制覆盖赋值时，需要使用\cp -f指令，绕过alias别名执行。



##### （3）bg

bg指令可以将作业（job）放到后台执行，作业是在当前 Shell 会话中运行的一个或多个进程。

当在前台运行一个作业时，它会占据 Shell 的控制，并需要等待该作业完成或者手动停止它。使用 bg指令，可以将一个在前台运行的作业切换到后台继续执行，这样你可以继续在 Shell 中执行其他任务。

以下是 `bg` 指令的基本语法：

```
bg [job_spec]
```

其中，`job_spec` 是作业的标识符，可以是作业号（job number）或作业名称（job name）。如果未提供 `job_spec`，`bg` 指令将默认将最近的一个停止的作业放到后台执行。

要使用 `bg` 指令，首先需要使用 `Ctrl+Z` 组合键将前台作业暂停（挂起）。然后，使用 `jobs` 指令查看当前的作业列表和标识符。最后，使用 `bg` 指令将特定的作业切换到后台执行。

这里给一个示例

![image-20230723225246387](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230723225246387.png)

```
$ sleep 60  # 在前台运行一个长时间的作业
^Z          # 使用 Ctrl+Z 暂停作业
[1]+  Stopped                 sleep 60
$ bg        # 将最近的一个停止的作业放到后台执行
[1]+ sleep 60 &
```

使用 `sleep` 睡眠60s，模拟一个长时间运行的作业，之后使用 `Ctrl+Z` 暂停了该作业。

此时使用 `bg` 指令可以将挂起的sleep操作（最近停止）切换到后台继续执行。



##### （4）bind

在 Shell 中，Readline 库提供了命令行编辑和历史记录功能，而 `bind` 命令用于配置 Readline 的键绑定。可以将特定的键盘序列映射到 Readline 函数或宏，从而实现自定义的键盘操作。

下面是一个示例，展示了如何使用 `bind` 命令将键盘序列绑定到一个 Readline 函数或宏：

```shell
bind '"\C-x\C-r": re-read-init-file'
```

在上述示例中，`\C-x\C-r` 是一个键盘序列，表示同时按下 `Ctrl` 和 `x`，然后再按下 `Ctrl` 和 `r`。该键盘序列被绑定到 `re-read-init-file` 函数，当用户按下该键序列时，Readline 会执行 `re-read-init-file` 函数的逻辑。



##### （5）break

`break` 用于跳出循环语句（如 `for`、`while`、`until`）的执行。

需要注意的是，`break` 关键字只能跳出最内层的循环。如果嵌套了多层循环，`break` 只会跳出当前层级的循环，而不会跳出外层的循环。



##### （6）builtin

内置命令是由 Shell 解释器内部提供的一些特殊命令。

然而在定义与shell内置函数同名的shell函数后，shell会默认执行用户定义的函数而非原来的内嵌命令，

使用 `builtin` 关键字则可以确保执行内嵌命令，利用该性质可以在函数中保留原来内置函数的功能。

写一个脚本：

```shell
#!/bin/bash

# 定义一个名为 echo 的自定义函数
echo() {
    builtin echo "This is a custom echo function"
}

# 调用 echo 函数
echo "Hello, World!"
```

测试结果：

![image-20230723231901844](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230723231901844.png)

由于脚本中定义了一个 `echo` 的自定义函数，因此绝不会执行内置命令echo去输出"helloworld"。在echo函数内部，由于使用了 `builtin echo`，此时不会再递归调用自身echo，而是真正执行内嵌echo并输出 "This is a custom echo function"。



##### （7）caller

`caller` [expr]命令返回当前活动的子程序调用的上下文，即调用堆栈信息，包括shell函数和内建命令source执行的脚本。

没有指定expr时，显示最近一次子程序调用的行号和源文件名。

如果expr是一个非负整数，显示对应栈深的子程序调用的行号、子程序名和源文件名。

如果shell没有子程序调用或者expr是一个无效的位置时，caller命令返回false。


写一个脚本：



```shell
#!/bin/bash

function test1() {
    echo "current test1"   #声明当前作用域
    caller                 #使用caller命令打印出调用堆栈信息
    caller 0               #使用caller命令打印出调用堆栈信息
    caller 1               #使用caller命令打印出调用堆栈信息
    caller 2               #使用caller命令打印出调用堆栈信息
}

function test2() {
    echo "current test2"  #声明当前作用域
    test1                 #调用test1
}

function test3() {
    echo "current test3"  #声明当前作用域
    test2                 #调用test2
}

test3
```

输出结果：

![image-20230724104853435](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230724104853435.png)

不加expr时，输出最近一次函数（test1）被调用的行号和shell脚本名

expr==0时，输出最近一次被压栈的子程序（test1）被调用的行号（13），并显示调用test1的程序名（test2）以及shell脚本名，

expr==1时，输出倒数第二次被压栈的子程序（test2）被调用的行号（18），并显示调用test2的程序名（test3）以及shell脚本名，

expr==2时，输出倒数第三次（首次）被压栈的子程序（test3）被调用的行号（21），并显示调用test3的程序名（main）以及shell脚本名。



##### （8）cd

用于切换路径，不填路径则切换成用户的主目录，

“cd -”可以切换到上一次访问的路径



##### （9）command

`command` 的用法和builtin类似，确保执行内置指令，但额外提供了一些功能。

`command` 命令的基本语法如下：

```
command [-p/v/V] [命令 [参数...]]
```

- -p：根据默认path搜索命令并执行
- -v：不执行命令，只打印路径信息
- -V：也不执行命令，打印更详细的信息

写一个示例：

```shell
#!/bin/bash

function ls() {
    echo "my ls"   #声明当前作用域
}

ls

command ls

command -p ls

command -v ls

command -V ls
```

测试结果：

![image-20230724112022107](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230724112022107.png)

第一行：直接执行ls会重定向到自定义函数

第二行：不加选项执行内嵌ls

第三行：-p选项，由于默认路径所以执行的也是内嵌ls

第四行：-v选项，由于在shell中自定义了ls，所以打印当前路径下的函数名ls

第五至九行：-V选项，打印自定义函数ls更详细的信息



##### （10）compgen

`compgen` 是一个用于生成命令补全列表的 Bash 内建命令。它可以帮助用户列出可用的命令、变量、别名和函数等，以及它们的补全选项。

`compgen` 命令的基本语法如下：

```
compgen [-c/a/v/k] [单词]
```

- -c：列出当前环境和单词相关的所有命令，如果没有单词则提供所有命令
- -a：列出当前环境和单词相关的所有别名，如果没有单词则提供所有别名
- -v：列出当前环境和单词相关的所有变量，如果没有单词则提供所有变量
- -k：列出当前环境和单词相关的所有函数，如果没有单词则提供所有函数

![image-20230724113727018](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230724113727018.png)





##### （11）complete

`complete` 可以自定义的自动补全规则，以便在命令行中按下 Tab 键时提供候选项。

`complete` 命令的基本语法如下：

```
complete [选项] [命令]
```

其中，`选项` 可以是以下之一：

- `-a`：指定补全的候选项来源为别名（aliases）。
- `-b`：指定补全的候选项来源为内置命令。
- `-c`：可以补全命令。
- `-d`：可以补全目录名。
- `-e`：可以补全环境变量。
- `-f`：可以补全文件名。
- `-j`：可以补全作业（jobs）。
- `-s`：可以补全信号（signals）。
- `-u`：可以补全用户。
- `-W` ：指定自动补全的候选项列表。它后面需要跟随一个用空格分隔的候选项列表。

以下是一个示例：

```bash
# 配置自动补全，使得在输入 "git " 后按下 Tab 键时，会自动补全 git 命令的子命令
complete -W "add commit push pull status" git
```

上述示例中，`complete -W "add commit push pull status" git` 命令配置了在输入 `git ` 后按下 Tab 键时，会自动补全 git 命令的子命令（例如 `add`、`commit`、`push` 等）。



##### （12）compopt

与（11）对应，compopt用于查询目前的自动补全选项。

`compopt` 命令的基本语法如下：

```
compopt [选项] [名称...]
```

其中，`选项` 可以是以下之一：

- `-o`：查询或设置自动补全选项。
- `-s`：查询或设置自动补全样式。

`名称` 是要查询或设置的自动补全选项或样式的名称。

以下是一些 `compopt` 命令的示例用法：

```bash
# 查询某个自动补全选项的状态
compopt -o nospace

# 设置某个自动补全选项为启用状态
compopt -o dirnames

# 查询某个自动补全样式的状态
compopt -s bashdefault

# 设置某个自动补全样式为启用状态
compopt -s default
```

上述示例中，`compopt` 命令用于查询或设置自动补全选项或样式的状态。`-o` 选项用于查询或设置自动补全选项，`-s` 选项用于查询或设置自动补全样式。`nospace`、`dirnames`、`bashdefault` 和 `default` 是自动补全选项或样式的名称。



##### （13）continue

`continue` 可以跳过当前循环中剩余的代码，并继续下一次循环的执行。

`continue` 的使用方式与循环类型有关，包括 `for`、`while` 和 `until` 循环。



##### （14）declare

`declare` 可以用于创建、修改和查询变量的类型、作用域和其他属性。

`declare` 命令的基本语法如下：

```
declare [选项] [变量名[=值]...]
```

`选项` 可以是以下之一：

- `-a`：声明变量为数组。
- `-i`：声明变量为整数。
- `-r`：声明变量为只读（不可修改）。
- `-x`：声明变量为环境变量。

`变量名` 是要声明或设置属性的变量名，`值` 是可选的初始值。

以下是一些 `declare` 命令的示例用法：

```bash
# 声明一个整数变量
declare -i count=10

# 修改变量的值
declare -i count=20

# 声明一个只读变量
declare -r name="John"

# 声明一个数组变量
declare -a fruits=("apple" "banana" "orange")

# 声明一个环境变量
declare -x PATH="/usr/local/bin:$PATH"
```



##### （15）dirs

`dirs` 可以显示当前会话中保存的目录堆栈（directory stack）的内容。目录堆栈是一个记录着先前访问的目录路径的列表。

基本语法如下：

```
dirs [选项]
```

`选项` 可以是以下之一：

- `-p`：以长格式显示目录堆栈，每个目录路径占一行。
- `-l`：以路径名的形式显示目录堆栈，路径之间用空格分隔。
- `-c`：清空目录堆栈，删除所有保存的目录路径。
- `+N`：显示目录堆栈中的第 N 个目录路径，其中 N 是一个非负整数。`+1` 表示最早访问的目录路径，`+2` 表示第二个，以此类推。如果 N 超出了目录堆栈的范围，将不会有输出。

目录堆栈对于在不同目录之间进行快速切换很有用。但需要配合其它指令使用， `pushd` 命令可以将目录路径添加到堆栈中， `popd` 命令从堆栈中弹出目录路径。当会话结束时，目录堆栈将被清空。



##### （16）disown

`disown` 将正在运行的作业（job）从当前 shell 的作业表中移除，使其成为一个与 shell 无关的后台作业。这样，即使在关闭终端或退出 shell 后，该作业仍将继续在后台运行。

`disown` 命令的基本语法如下：

```
disown [选项] [作业ID...]
```

`选项` 可以是以下之一：

- `-h`：将作业标记为无法被挂起（uninterruptible），即使收到 SIGHUP 信号也不会中止作业。
- `-r`：将作业标记为正在运行（running），即使作业已经停止。

`作业ID` 是要移除的作业的标识符。可以通过 `jobs` 命令查看当前 shell 的作业列表，并找到要移除的作业的 ID。

以下是一些 `disown` 命令的示例用法：

```bash
# 将当前作业从作业表中移除
disown

# 将作业ID为 1 的作业从作业表中移除
disown 1

# 将作业ID为 2 和 3 的作业从作业表中移除，并标记为正在运行
disown -r 2 3
```

使用 `disown` 命令将作业移除后，你将无法再使用 `fg` 命令将其切换到前台。作业将继续在后台运行，直到完成或被其他方式终止。



##### （17）echo

`echo` 用于在终端输出文本内容或变量的值。

`echo` 命令的基本语法如下：

```
echo [选项] [字符串...]
```

`选项` 可以是以下之一：

- `-n`：不输出末尾的换行符。
- `-e`：启用特殊字符的解释，例如`\n`表示换行符。
- `-E`：禁用特殊字符的解释（默认行为）。

`字符串` 是要输出的文本内容或变量的值。多个字符串之间用空格分隔，它们将按顺序输出在一行上。



##### （18）enable

`enable` 可以启用或禁用指定的 shell 内置命令或函数。

`enable` 命令的基本语法如下：

```
enable [选项] [命令...]
```

`选项` 可以是以下之一：

- `-n`：启用指定的命令或函数。
- `-a`：显示所有可用的命令和函数，并指示它们是否已启用。

`命令` 是要启用或禁用的内建命令或函数的名称。

以下是一些 `enable` 命令的示例用法：

```bash
# 启用指定的内建命令
enable -n echo

# 禁用指定的内建命令
enable echo

# 显示所有可用的命令和函数，并指示它们是否已启用
enable -a
```

通过启用或禁用内建命令或函数，可以控制它们在当前 shell 环境中的可用性。禁用一个命令后，尝试执行该命令将会失败。

`enable` 命令只能用于内建命令和函数，无法用于外部命令（例如可执行文件）。要禁用外部命令，可以使用 `alias` 命令创建一个与外部命令同名的别名，并将其指向一个无效的命令或空字符串。



##### （19）eval

`eval` 可以将字符串作为命令执行。

`eval` 命令的基本语法如下：

```
eval [字符串]
```

`字符串` 是要执行的命令或表达式，可以包含变量、命令替换、重定向等。

当 `eval` 命令被执行时，它会将传递给它的字符串作为 Bash 命令进行解析和执行。这对于动态生成和执行命令非常有用，因为它允许在运行时构建和执行字符串命令。



##### （20）exec

`exec` 是一个 Bash 内建命令，用于替换当前 shell 进程（或当前 shell 中的某个命令）为新的进程。

`exec` 命令的基本语法如下：

```
exec [选项] [命令 [参数...]]
```

`选项` 可以是以下之一：

- `-l`：将新进程作为登录 shell 启动。
- `-a`：指定新进程的参数列表。
- `-c`：将参数作为命令执行，而不是作为可执行文件。

`命令` 是要替换为的新进程或命令的名称。`参数` 是传递给新进程或命令的参数列表。

以下是一些 `exec` 命令的示例用法：

```bash
# 替换当前 shell 进程为新的进程
exec ls -l

# 替换当前 shell 进程为新的登录 shell 进程
exec -l bash

# 替换当前 shell 中的某个命令为新的进程
exec echo "Hello, World!"

# 替换当前 shell 中的某个命令为新的命令并传递参数
exec echo "Hello, World!" "Goodbye"
```

当使用 `exec` 命令时，当前 shell 进程会被替换为新的进程，而不是创建一个新的子进程。因此，一旦 `exec` 命令执行成功，原来的 shell 进程将不再存在，并且新进程将接管当前的 shell 环境。

`exec` 命令常用于在脚本中切换到不同的命令或进程，并且在新进程执行完毕后不返回原来的脚本。



##### （21）exit

`exit`用于终止当前 shell 进程或退出当前的脚本。

`exit` 命令的基本语法如下：

```
exit [退出状态]
```

`退出状态` 是一个整数值，用于表示脚本或程序的退出状态。默认情况下，如果省略退出状态，`exit` 命令将使用上一个命令的退出状态作为自己的退出状态。



##### （22）export

`export` 用于设置环境变量。使其在当前 shell 进程及其子进程中可见。

`export` 命令的基本语法如下：

```
export [变量名[=值]]
```

`变量名` 是要导出为环境变量的变量的名称，`值` 是要设置的变量的值。如果省略 `值`，则变量的值将保持不变。

以下是一些 `export` 命令的示例用法：

```bash
# 设置并导出环境变量
export PATH="/opt/gcc/bin"

# 导出现有变量为环境变量
export PATH

# 导出变量并将值附加到现有值
export PATH="$PATH:/usr/local/bin"
```

通过使用 `export` 命令，你可以将变量设置为环境变量，使其在当前 shell 进程及其子进程中可见。这对于设置常用的环境变量如 `PATH`、`JAVA_HOME` 等非常有用。

要注意的是，通过 `export` 导出的环境变量仅在当前 shell 进程及其子进程中可见。对于其他 shell 进程或新打开的终端窗口，需要重新设置或导出相应的环境变量。



##### （23）fc

`fc` 是一个 Bash 内建命令，用于编辑和重新执行之前执行过的命令。

`fc` 命令的基本语法如下：

```shell
fc [选项] [历史命令]
```

`选项` 可以是以下之一：

- `-l`：列出历史命令。
- `-n`：指定要编辑的历史命令的编号并进入编辑器，保存后会执行编辑后的命令。
- `-e`：指定编辑器(vi、vim、gedit)编辑最近一次历史命令，比如"fc -e vi"



##### （24）fg

`fg` 将后台运行的作业切换到前台运行。

`fg` 语法如下：

```shell
fg [作业标识]
```

`作业标识` 是一个可选参数，用于指定要切换到前台的作业。如果省略该参数，则默认切换到当前正在运行的最后一个后台作业。当然可以配合（29）jobs指令查询后台的作业号。



##### （25）getopts

`getopts` 是一个用于解析命令行选项和参数的 Bash 内建命令。它通常在 Bash 脚本中使用，用于处理命令行参数，使脚本能够根据不同的选项执行不同的逻辑。

`getopts` 命令的语法如下：

```bash
getopts optstring variable
```

- `optstring` 是一个包含期望的选项字符的字符串。每个字符代表一个选项，如果后面跟着冒号 `:`，表示该选项需要附加参数。例如，`ab:c` 表示期望选项 `-a`、`-b`，`-c`，但只有 `-b` 需要附加参数。
- `variable` 是一个用于存储解析结果的变量名。在 `getopts` 执行后，该变量会存储当前解析到的选项字符。

`getopts` 命令通常与 `while` 循环结合使用，以遍历所有的选项和参数。以下是一个示例：

```bash
while getopts "ab:c" opt; 
do
  case $opt in
    a)
      echo "选项 -a 被设置"
      ;;
    b)
      echo "选项 -b 被设置，参数为 $OPTARG" #总是存储原始$*中下一个要处理的元素
      ;;
    c)
      echo "选项 -c 被设置"
      ;;
    \?)
      echo "无效的选项: -$OPTARG"
      ;;
  esac
done
```

测试结果：

![image-20230724163434002](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230724163434002.png)

`getopts` 会解析输入参数 `"ab:c"`，`while` 循环会迭代每个选项，并将每次遍历的临时结果存储在变量 `$opt` 中。



##### （26）hash

`hash` 用于管理和显示已经被 Bash 解释器记住的命令的路径。

当你在 Bash 中执行一个命令时，Bash 会使用命令的名称搜索可执行文件的路径，并将找到的路径缓存起来，以便下次执行该命令时可以更快地找到它。这个缓存的过程由 `hash` 命令完成。

`hash` 命令有以下用法：

- `hash`：显示当前已经被记住的命令和它们的路径。
- `hash 命令`：将指定的命令从缓存中删除。
- `hash -r`：清除所有已记住的命令，使 Bash 重新搜索命令的路径。

![image-20230724163722685](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230724163722685.png)

 `hash` 命令可以在 Bash 中管理命令的路径缓存。这在你安装新的命令或更新已有命令的路径时特别有用，以确保 Bash 可以正确找到并执行这些命令。



##### （27）help

`help` 可以显示 Bash 内置命令的帮助信息，有以下用法：

- `help`：显示内建命令的帮助信息列表。
- `help 命令`：显示指定内建命令的详细帮助信息。



##### （28）history

`history` 可以显示当前会话中执行过的命令历史记录，和fc类似，但是没有编辑能力，只能查看。

`history` 命令有以下用法：

- `history`：显示当前会话中执行过的命令历史记录，按照从最新到最旧的顺序排列，并带有每个命令的编号。
- `history n`：仅显示最近执行的最多 n 条命令历史记录。
- `!n`：执行历史记录中编号为 n 的命令。

以下是一些示例：

![image-20230724164312203](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230724164312203.png)

你使用 `history` 命令时，它会将命令历史记录输出到终端。每个历史记录条目包含一个编号和相应的命令。你可以使用这些编号来执行特定的历史命令，只需在命令前面加上感叹号 `!` 和编号即可。



##### （29）jobs

使用 `jobs` 指令可以显示当前 Shell 会话中的作业及其相关信息，包括作业号。

![image-20230723225911385](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230723225911385.png)

上述示例中，`[1]` 和 `[2]` 就是作业号。作业号旁边的符号表示作业的状态，`+` 表示当前前台作业，`-` 表示上一个作业。

如果使用 `jobs -l` 命令，将显示更详细的信息，包括作业号、进程 ID、作业状态、命令和命令行参数。

![image-20230723230048934](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230723230048934.png)

如图，`12345` 和 `67890` 是作业的进程 ID（PID），与作业号相关联。



##### （30）kill

`kill` 是一个用于终止进程的命令。它可以通过发送信号来停止正在运行的进程。

`kill` 命令的常见用法是：

```bash
kill [选项] <进程ID或作业ID>
```

其中，`进程ID` 是要终止的进程的标识符。你可以使用其他命令（如 `ps`）来获取进程的 ID。`作业ID` 可以通过jobs获取，但是作业号需要一个前缀的'%'来标识。

以下是一些常用的选项：

- `-l`：列出所有可用的信号名称。
- `-s <信号>`：指定要发送的信号。默认情况下，`kill` 命令发送 `TERM` 信号（终止信号）。
- `-<信号>`：使用信号的缩写形式。例如，`-9` 表示发送 `KILL` 信号（强制终止信号）。

以下是一些示例：

```bash
# 终止进程ID为 1234 的进程
kill 1234

# 终止作业ID为 %1 的作业
kill %1

# 列出所有可用的信号名称
kill -l

# 使用 KILL 信号终止进程ID为 5678 的进程
kill -9 5678
```

![image-20230724164849878](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230724164849878.png)

这里展示了下使用kill杀死job。

需要注意的是，使用 `kill` 命令终止进程时，进程可能会收到一个信号来请求正常退出。如果进程没有对该信号做出响应，你可以使用 `-9` 选项发送 `KILL` 信号，强制终止进程。



##### （31）let

`let` 用于执行算术运算，和eval很像但又有所不同，eval理论上可以执行所有指令，let只关注算术运算。

`let` 命令的一般语法如下：

```bash
let <表达式>
```

其中，`表达式` 是要执行的算术表达式。表达式可以包含变量、常数和运算符。

以下是一些示例：

```bash
# 执行加法
let "result = 2 * 3" #使用双引号的字符串，*号不用转义，=两边也可以有空格

# 支持递增和递减操作
let "x++"
```



##### （32）local

`local` 命令是 Bash shell 中用于声明局部变量的关键字。它用于在函数内部创建和使用仅在函数范围内有效的变量。

当在函数内部使用 `local` 命令声明一个变量时，该变量的作用域仅限于该函数。这意味着该变量在函数外部是不可见的，并且不会与同名的全局变量发生冲突。

`local` 命令的语法如下：

```bash
local <变量名>[=<值>]
```

其中，`变量名` 是要声明的局部变量的名称。`值` 是可选的，用于初始化该局部变量。如果省略了值，变量将被初始化为一个空字符串。

以下是一个示例函数，演示了如何使用 `local` 命令声明局部变量：

```bash
function my_function() {
  local local_var="局部变量"
  global_var="全局变量"

  echo "在函数内部访问局部变量：$local_var"
  echo "在函数内部访问全局变量：$global_var"
}

my_function

echo "在函数外部访问全局变量：$global_var"
echo "在函数外部访问局部变量：$local_var"  # 报错，局部变量在函数外部不可见
```

输出结果如下：

![image-20230724165447607](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230724165447607.png)

可以看到，通过 `local` 命令声明的局部变量 `local_var` 只在函数内部可见，而在函数外部无法访问。而未使用 `local` 命令声明的变量 `global_var` 是一个全局变量，在函数内外都可见。



##### （33）logout

`logout` 是一个用于退出当前登录会话的命令。在 Bash shell 中，`logout` 命令通常用于退出当前用户的登录会话，关闭终端或退出远程登录会话。

当你在终端中输入 `logout` 命令并按下回车键时，Bash shell 将执行以下操作：

1. 结束当前 shell 进程：当前的 Bash shell 进程将被终止，将不再接受任何命令或输入。
2. 关闭终端窗口或退出远程会话：如果你是通过终端窗口登录的，那么执行 `logout` 命令将关闭该终端窗口。如果你是通过远程登录会话登录的，那么执行 `logout` 命令将退出该远程会话。

需要注意的是，`logout` 命令仅适用于登录会话，而不适用于子 shell 或脚本。如果你在子 shell 或脚本中使用 `logout` 命令，它将被视为一个普通的命令，并不会退出整个会话。



##### （34）mapfile

`mapfile` 用于将文本文件的内容读取到数组中。它可以一次性读取文件的多行内容，并将每行文本分配给数组中的元素。

`mapfile` 命令的基本语法如下：

```bash
mapfile [-n count] [-O origin] [-s count] [-t] [-u fd] [-C callback] [-c quantum] [-d delim] array
```

其中，`array` 是要存储文件内容的数组名。

`-n` 选项指定要读取的行数，

`-O` 选项指定数组的起始索引，

`-s` 选项指定要跳过的行数，

`-t` 选项用于去除每行末尾的换行符，

`-u` 选项指定要读取的文件描述符，

`-C` 选项指定回调函数，在每次读取行时执行，

`-c` 选项指定一次读取的行数，`-d` 选项指定行分隔符。

以下是一个示例，演示了如何使用 `mapfile` 命令读取文件内容到数组中：

```bash
mapfile -t lines < file.txt

# 输出数组中的每行内容
for line in "${lines[@]}"; do
  echo "$line"
done
```

在上面的示例中，`mapfile -t lines < file.txt` 将文件 `file.txt` 的内容读取到名为 `lines` 的数组中。每行文本将分配给数组 `lines` 中的一个元素。

然后，使用 `for` 循环遍历数组 `lines`，并使用 `echo` 命令输出每行的内容。

需要注意的是，`mapfile` 命令从文件中读取的每一行都会保留行尾的换行符。如果不希望包含换行符，可以使用 `-t` 选项去除它们。



##### （35）popd

`popd` 是一个 Bash shell 内置命令，用于从目录堆栈中弹出（移除）并切换到上一个目录。

在 Bash shell 中，可以使用 `pushd` 命令将当前目录推入（添加）目录堆栈，并切换到指定的目录。而 `popd` 命令则用于从目录堆栈中弹出并切换到上一个目录。

`popd` 命令没有任何参数。每次执行 `popd` 命令时，它会从目录堆栈中移除最近添加的目录，并将当前工作目录切换到上一个目录。



##### （36）printf

`printf` 是一个用于格式化输出的常见命令，它在许多编程语言和操作系统中都存在。在 Bash shell 中，`printf` 命令用于格式化并输出文本。

`printf` 命令的基本语法如下：

```bash
printf format [arguments...]
```

其中，`format` 是格式字符串，用于指定输出的格式。`arguments` 是要输出的参数，可以是一个或多个。

`printf` 命令使用格式字符串中的占位符来表示要输出的值。以下是一些常用的占位符：

- `%s`：用于输出字符串。
- `%d`：用于输出十进制整数。
- `%f`：用于输出浮点数。
- `%c`：用于输出字符。
- `%b`：用于输出带转义序列的字符串。

以下是一个示例，演示了如何使用 `printf` 命令格式化输出：

```bash
name="Alice"
age=25
balance=1234.56

printf "Name: %s\n" "$name"
printf "Age: %d\n" "$age"
printf "Balance: %.2f\n" "$balance"
```

测试结果：

![image-20230724170542938](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230724170542938.png)

`printf` 命令还支持更多的格式化选项，如对齐、填充字符、宽度等。你可以在 Bash 的文档或其他资源中查找更多关于 `printf` 命令的详细信息和示例。



##### （37）pushd

`pushd` 是一个 Bash shell 内置命令，可以看作是cd的升级版，除了切换目录，还能将当前目录推入（添加）目录堆栈。

最后调用dirs指令展示当前的目录栈

`pushd` 命令的基本语法如下：

```bash
pushd [directory]
```

其中，`directory` 是要推入目录堆栈的目录路径。如果未指定 `directory`，则 `pushd` 命令将切换到主目录（通常是用户的家目录）。

以下是一个示例：

![image-20230724171302777](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230724171302777.png)

一般来说，需要配合 `dirs` 命令，以及 `popd` 命令一起使用。



##### （38）pwd

`pwd` 可以显示当前工作目录的路径。



##### （39）readarray

`readarray` 用于将文本文件的内容读取到数组中。

`readarray` 命令的基本语法如下：

```bash
readarray [-t] array_variable < file
```

其中，`array_variable` 是要存储文本行的数组变量的名称，`file` 是要读取的文本文件的路径。

默认情况下，`readarray` 命令会保留每行的换行符。如果希望删除每行末尾的换行符，可以使用 `-t` 选项。

由于`readarray` 命令可以使用 `mapfile` 命令来替代，因此不多赘述。



##### （40）readonly

`readonly` 用于将变量设置为只读，防止其被修改。



##### （41）return

`return` 用于从函数中返回一个值或退出函数。

以下示例演示了如何使用 `return` 命令：

```bash
function my_function() {
  echo "Inside function"
  return 42
}

my_function
echo "Return value: $?"
```

测试结果：

![image-20230724171850100](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230724171850100.png)


注意，`return` 命令只能用于函数中，而不能在脚本的主体部分使用。在脚本的主体部分，可以使用 `exit` 命令来退出整个脚本。



##### （42）set

`set` 是一个 Bash shell 内置命令，用于设置或显示 shell 的各种属性和选项。

`set` 命令的使用方式有多种，可以用于显示当前 shell 的设置，修改 shell 的行为，或者用于调试脚本。

以下是 `set` 命令的一些常见用法：

1. 显示当前 shell 的设置：

   ```bash
   set
   ```

   上述命令将显示当前 shell 的各种属性和选项，包括环境变量、函数、位置参数等。

2. 设置选项：

   ```bash
   set -option
   ```

   上述命令用于设置特定的选项。例如，`set -e` 可以使脚本在遇到错误时立即退出。

   一些常用的选项包括：

   - `-e`：在命令执行失败时立即退出脚本。
   - `-u`：对未定义的变量进行严格检查，如果使用了未定义的变量，将会引发错误。
   - `-x`：在执行命令之前打印出命令及其参数，用于调试脚本。

3. 设置位置参数：

   ```bash
   set -- arg1 arg2 arg3
   ```

   上述命令用于设置位置参数。位置参数是指在脚本或函数中传递给它们的参数。使用 `set --` 可以重新设置位置参数，将其替换为指定的参数列表。

4. 显示选项的值：

   ```bash
   set -o option
   ```

   上述命令用于显示指定选项的当前值。例如，`set -o errexit` 可以显示 `-e` 选项的当前值。

以上只是 `set` 命令的一些常见用法，还有其他更多的用法和选项。你可以查阅 Bash 的文档或使用 `help set` 命令获取更详细的信息和用法示例。



##### （43）shift

`shift` 用于对位置参数进行移位操作。

在 Bash 脚本中，位置参数是指传递给脚本或函数的参数。`shift` 命令可以将位置参数列表向左移动，丢弃最左边的参数，并将参数列表中的每个参数向前移动一个位置。

`shift` 命令的基本语法如下：

```bash
shift [n]
```

其中，`n` 是一个整数值，表示要移动的位置数。如果省略 `n`，则默认为 1，即移动一个位置。

以下是一个示例，演示了如何使用 `shift` 命令：

```bash
#!/bin/bash

echo "Script name: $0"
echo "First argument: $1"
echo "Second argument: $2"

shift 2

echo "After shifting:"
echo "First argument: $1"
echo "Second argument: $2"
```

在上面的示例中，我们首先打印出脚本的名称和前两个位置参数的值。

然后，我们使用 `shift 2` 命令对位置参数进行移位操作，将其向左移动两个位置。这意味着原来的第三个参数现在成为了第一个参数，原来的第四个参数现在成为了第二个参数。

最后，我们再次打印出移位后的位置参数的值。

假设我们运行该脚本并传递三个参数，例如：`./script.sh arg1 arg2 arg3`，则输出将会是：

```
Script name: ./script.sh
First argument: arg1
Second argument: arg2
After shifting:
First argument: arg3
Second argument:
```

可以看到，经过 `shift 2` 命令后，原来的 `arg3` 成为了第一个参数，原来的第二个参数为空。

`shift` 命令通常用于处理多个位置参数的情况，通过移位操作，可以逐个处理位置参数，而不需要手动访问每个参数。



##### （44）shopt

`shopt` 用于显示和设置 shell 的选项。`shopt` 命令的使用方式如下：

1. 显示当前 shell 的选项：

   ```bash
   shopt
   ```

   上述命令将显示当前 shell 的各种选项，包括开启和关闭的选项。

2. 显示指定选项的状态：

   ```bash
   shopt -p option
   ```

   上述命令将显示指定选项的状态，包括该选项是否开启或关闭。

3. 设置选项的状态：

   ```bash
   shopt -s option
   shopt -u option
   ```

   上述命令用于分别开启或关闭指定的选项。`-s` 选项用于开启选项，`-u` 选项用于关闭选项。

   例如，要开启 `extglob` 选项，可以使用 `shopt -s extglob`。

一些常见的 `shopt` 选项包括：

- `extglob`：启用扩展的模式匹配功能，允许使用更强大的模式匹配语法。
- `globstar`：启用递归的通配符匹配，允许在目录层级中进行模式匹配。
- `nullglob`：如果没有匹配的文件时，将通配符展开为空而不是原样输出。
- `nocaseglob`：在文件名匹配时忽略大小写。

你可以使用 `shopt` 命令来查看当前 shell 的选项，并根据需要开启或关闭特定的选项。

请注意，`shopt` 命令只适用于 Bash shell，不适用于其他 shell（如 sh、ksh 等）。



##### （45）source

`source` 是一个 Bash shell 内置命令，也可以使用点号（`.`）来表示。它用于在当前 shell 环境中执行指定脚本文件，并将其中的命令和变量直接应用于当前的 shell。

`source` 命令的语法如下：

```bash
source filename
. filename
```

其中，`filename` 是要执行的脚本文件的路径。

当你使用 `source` 命令或点号来执行脚本时，脚本中的命令和变量将在当前 shell 环境中直接生效。这意味着脚本中定义的变量将在当前 shell 中可用，并且脚本中执行的命令将直接影响当前 shell。

与直接运行脚本不同，使用 `source` 命令执行脚本可以实现以下效果：

1. 导入环境变量：脚本中的变量定义将在当前 shell 中生效，而不仅仅在子进程中生效。这对于设置环境变量或修改当前 shell 的行为非常有用。

2. 执行函数：脚本中定义的函数将在当前 shell 中可用，并可以直接调用。

3. 修改当前目录：脚本中的 `cd` 命令将直接影响当前 shell 的工作目录。

以下是一个示例，演示了如何使用 `source` 命令来执行脚本：

```bash
#!/bin/bash

# script.sh
name="John Doe"
echo "Hello, $name!"
```

在终端中执行以下命令：

```bash
source script.sh
```

或者：

```bash
. script.sh
```

执行上述命令后，脚本中的命令将在当前 shell 中执行。在这个示例中，脚本会打印出 `Hello, John Doe!`。

需要注意的是，`source` 命令只适用于 Bash shell，不适用于其他 shell（如 sh、ksh 等）。



##### （46）suspend

`suspend` 是一个 Bash shell 内置命令，用于将当前的 shell 进程挂起（暂停），让其进入后台运行，并返回到调用该 shell 的父进程或终端。

当你在终端中执行 `suspend` 命令时，当前 shell 进程将被挂起，你将回到调用该 shell 的父进程或终端。这样你就可以在父进程或终端中执行其他任务，而不是等待当前 shell 的命令执行完毕。

在执行 `suspend` 命令后，你可以使用 `fg` 命令将挂起的 shell 进程重新置于前台运行，继续执行。

以下是一个示例，演示了如何使用 `suspend` 命令：

1. 打开终端并执行一个长时间运行的命令，例如：

   ```bash
   sleep 60
   ```

   这个命令会让 shell 进程休眠 60 秒。

2. 在命令执行过程中，按下 `Ctrl + Z` 组合键，执行 `suspend` 命令。

   这会将当前的 shell 进程挂起，并返回到父进程或终端。

3. 在终端中执行其他任务。

4. 要恢复挂起的 shell 进程，可以使用 `fg` 命令：

   ```bash
   fg
   ```

   这会将挂起的 shell 进程重新置于前台运行，并继续执行。

需要注意的是，`suspend` 命令只适用于交互式的 Bash shell，而不适用于在脚本中使用。



##### （47）test

`test` 是一个 Bash shell 内置命令，用于在条件表达式中进行测试。它可以用于检查文件属性、比较值、判断字符串等。

`test` 命令的一般语法如下：

```bash
test condition
```

或者使用方括号表示：

```bash
[ condition ]
```

其中，`condition` 是要测试的条件表达式。

`test` 命令会根据条件表达式的结果返回一个退出状态码，用于判断条件是否为真。通常，退出状态码为 0 表示条件为真，非零值表示条件为假。

以下是一些常见的 `test` 命令用法示例：

1. 检查文件属性：

   ```bash
   test -f filename  # 检查文件是否存在且为普通文件
   test -d dirname   # 检查目录是否存在
   test -r file      # 检查文件是否可读
   test -w file      # 检查文件是否可写
   test -x file      # 检查文件是否可执行
   ```

2. 比较值：

   ```bash
   test $var1 -eq $var2    # 检查两个变量是否相等
   test $var1 -ne $var2    # 检查两个变量是否不相等
   test $var1 -lt $var2    # 检查 var1 是否小于 var2
   test $var1 -le $var2    # 检查 var1 是否小于等于 var2
   test $var1 -gt $var2    # 检查 var1 是否大于 var2
   test $var1 -ge $var2    # 检查 var1 是否大于等于 var2
   ```

3. 判断字符串：

   ```bash
   test -z $string     # 检查字符串是否为空
   test -n $string     # 检查字符串是否非空
   test $string1 = $string2   # 检查两个字符串是否相等
   test $string1 != $string2  # 检查两个字符串是否不相等
   ```

这些只是 `test` 命令的一些常见用法示例。`test` 命令支持更多的条件测试，包括文件权限、字符串匹配等。

另外，你也可以使用方括号 `[ ]` 来代替 `test` 命令，两者在功能上是等价的。

需要注意的是，在使用 `test` 命令时，条件表达式中的各个元素需要正确地进行空格分隔，以确保命令的正确解析。



##### （48）times

`times` 是一个 Bash shell 内置命令，用于显示当前 shell 进程和其子进程的累计执行时间统计信息。

`times` 命令会显示以下统计信息：

- `user`：用户态执行时间，表示 CPU 在用户态下花费的时间。
- `sys`：内核态执行时间，表示 CPU 在内核态下花费的时间。
- `child user`：子进程用户态执行时间的总和。
- `child sys`：子进程内核态执行时间的总和。

这些时间值通常以秒为单位，可以用于了解命令或脚本的执行时间。

以下是一个使用 `times` 命令的示例：

```bash
#!/bin/bash

echo "Start"

times

sleep 3

echo "End"

times
```

在上面的示例中，我们使用 `times` 命令两次，分别在脚本的开始和结束位置。执行脚本后，`times` 命令会显示当前 shell 进程的执行时间统计信息。

注意，`times` 命令只能显示当前 shell 进程及其子进程的执行时间，而不能用于其他进程。



##### （49）trap

`trap` 是一个 Bash shell 内置命令，用于捕获和处理信号。

在操作系统中，信号是用于通知进程发生某个事件的一种机制。例如，当用户按下 Ctrl+C 键时，操作系统会向前台进程发送一个中断信号（SIGINT），用于请求进程终止。

`trap` 命令可以用来定义在接收到指定信号时要执行的操作。它的一般语法如下：

```bash
trap command signals
```

其中，`command` 是要执行的命令或脚本代码，`signals` 是一个或多个信号名称或信号编号，用空格分隔。

以下是一些 `trap` 命令的常见用法示例：

1. 捕获信号并执行命令：

   ```bash
   trap 'command' signal
   ```

   这将在接收到指定信号时执行 `command` 命令。

2. 忽略信号：

   ```bash
   trap '' signal
   ```

   这将忽略接收到的指定信号，不执行任何操作。

3. 恢复默认信号处理：

   ```bash
   trap - signal
   ```

   这将恢复接收到的指定信号的默认处理方式。

4. 显示当前的信号处理设置：

   ```bash
   trap -l
   ```

   这将列出当前的信号处理设置，包括信号名称和对应的处理方式。

以下是一个示例，演示如何使用 `trap` 命令在接收到中断信号时执行自定义操作：

```bash
#!/bin/bash

# 定义信号处理函数
cleanup() {
    echo "Received interrupt signal. Cleaning up..."
    # 添加自定义的清理操作
    # ...
    exit 1
}

# 捕获中断信号并执行 cleanup 函数
trap cleanup SIGINT

# 主程序逻辑
echo "Running..."
# ...
# ...

# 等待用户按下 Ctrl+C
while true; do
    sleep 1
done
```

在上面的示例中，我们定义了一个名为 `cleanup` 的函数作为信号处理函数，并使用 `trap` 命令捕获中断信号（SIGINT）。当用户按下 Ctrl+C 时，将执行 `cleanup` 函数中的自定义操作。



##### （51）typeset

`typeset` 是一个 Bash shell 内置命令，用于声明和设置变量的属性。

`typeset` 命令在较新的 Bash 版本中已经被 `declare` 命令取代，两者功能相似。



##### （52）ulimit

`ulimit` 是一个用于设置和显示 shell 运行时限制的命令。它可以用来控制和管理进程的资源使用，例如可以限制进程的最大打开文件数、最大可用内存等。

`ulimit` 命令的一般语法如下：

```bash
ulimit [options] [limit]
```

其中，`options` 是可选的参数，用于指定命令的行为。常用的选项包括：

- `-a`：显示所有限制的当前值。
- `-c`：设置或显示核心文件的最大字节数。
- `-n`：设置或显示最大打开文件数。
- `-u`：设置或显示最大用户进程数。
- `-m`：设置或显示最大可用内存大小。
- `-s`：设置或显示最大堆栈大小。

`limit` 是可选的参数，用于设置相应的限制。它可以是一个数字，表示限制的具体值，也可以是一个软限制和硬限制的组合，以 `soft_limit:hard_limit` 的形式表示。

以下是一些 `ulimit` 命令的示例用法：

1. 显示当前的最大打开文件数限制：

   ```bash
   ulimit -n
   ```

   这将显示当前 shell 的最大打开文件数限制。

2. 设置最大打开文件数限制为 1000：

   ```bash
   ulimit -n 1000
   ```

   这将将当前 shell 的最大打开文件数限制设置为 1000。

3. 显示所有限制的当前值：

   ```bash
   ulimit -a
   ```

   这将显示所有限制的当前值，包括最大打开文件数、最大堆栈大小等。

需要注意的是，`ulimit` 命令所设置的限制只对当前 shell 及其子进程有效，并不会影响其他 shell 实例或系统级别的限制。

##### （53）umask

`umask` 是一个用于设置文件创建权限掩码的命令。它决定了在创建新文件或目录时，默认的权限设置。

在 Linux 系统中，每个文件和目录都有一组权限，用于确定谁可以读取、写入和执行它们。权限由三个组成部分组成：所有者权限、组权限和其他用户权限。

`umask` 命令用于设置掩码，它会从新创建的文件和目录的权限中去除特定权限位。对于目录而言，创建时系统允许的最高权位为777，对于文件而言，创建时系统允许的最高权位为666，因为linux不允许文件在创建时就有可执行权限。

以下是 `umask` 命令的一些常见用法：

1. `umask`：不带任何参数运行 `umask` 命令会显示当前的掩码值。

2. `umask <mask>`：将 `<mask>` 替换为一个三位八进制数，用于设置新的掩码值。例如，`umask 027` 将设置掩码为 027，表示新创建的文件权限为 666-027=640（负值按0算），目录权限为 777-027=750。

3. 以目录为例，设置umask为027后，目录权限为750。意义如下：

   - 所有者权限：保留写入权限（2），执行权限（1）和读取权限（4）。

   - 组权限：保留读取权限（4）和执行权限（1），禁止写入权限（2）。

   - 其他用户权限：禁止所有权限。

`umask` 命令的设置是针对当前会话的，并不是永久性的。它会影响在当前会话中创建的新文件和目录的默认权限。如果你希望永久更改默认权限，可以将 `umask` 命令添加到 shell 配置文件（如 `~/.bashrc`）中。

##### （54）unalias

`unalias`和 `alias`命令相反，用于取消已定义的别名（alias）。

使用 `unalias` 命令的语法如下：

```
unalias [选项] [别名...]
```

其中，选项可以是以下之一：

- `-a`：取消所有已定义的别名。
- `-v`：显示取消的别名的详细信息。

如果不指定别名参数，则默认取消所有已定义的别名。

以下是一些示例：

1. 取消单个别名：

```
unalias ll
```

上述命令将取消名为 `ll` 的别名。

2. 取消多个别名：

```
unalias ls ll la
```

上述命令将取消名为 `ls`、`ll` 和 `la` 的别名。

3. 取消所有别名：

```
unalias -a
```

上述命令将取消所有已定义的别名。

请注意，取消别名只在当前会话中生效。如果要永久取消别名，可以将 `unalias` 命令添加到 shell 配置文件（如 `~/.bashrc` 或 `~/.bash_profile`）中。



##### （55）unset

`unset` 命令用于取消已定义的变量或函数。它可以用来删除已经存在的变量或函数，以及取消变量的赋值。

使用 `unset` 命令的语法如下：

```
unset [选项] [变量...]
```

其中，选项可以是以下之一：

- `-v`：显示取消的变量的详细信息。

如果不指定变量参数，则默认取消所有已定义的变量。

以下是一些示例：

1. 取消单个变量：

```
unset my_var
```

2. 取消多个变量：

```
unset var1 var2 var3
```

3. 取消所有变量：

```
unset -v
```

请注意，取消变量只在当前会话中生效。如果要永久取消变量，可以将 `unset` 命令添加到 shell 配置文件（如 `~/.bashrc` 或 `~/.bash_profile`）中。



##### （56）wait

`wait` 命令用于等待后台进程的完成。它会暂停当前 shell 进程，直到指定的后台进程全部完成为止。

使用 `wait` 命令的语法如下：

```
wait [选项] [后台进程ID...]
```

其中，选项可以是以下之一：

- `-n`：等待后台进程中的任意一个完成。
- `-p`：指定要等待的后台进程组的进程ID。

如果不指定后台进程ID参数，则 `wait` 命令会等待当前 shell 中的所有后台进程完成。

以下是一些示例：

1. 等待单个后台进程完成：

```
command1 &
wait $!
```

上述命令中的 `command1` 是一个后台进程的示例命令。`$!` 是一个特殊变量，表示最近一个后台进程的进程ID。`wait $!` 命令将等待该后台进程完成。

2. 等待多个后台进程完成：

```
command1 &
command2 &
command3 &
wait
```

上述命令中的 `command1`、`command2` 和 `command3` 都是后台进程的示例命令。`wait` 命令将等待所有后台进程完成。

3. 等待后台进程组完成：

```
set -m  # 启用作业控制
command1 &
command2 &
command3 &
wait -p
```

上述命令中的 `command1`、`command2` 和 `command3` 都是后台进程的示例命令。`-p` 选项用于指定要等待的后台进程组的进程ID。

请注意，`wait` 命令只能等待当前 shell 进程中的后台进程。如果需要等待其他 shell 进程中的后台进程，可以使用进程间通信机制（如管道或共享文件）来实现。



##### （57）wc

`wc` 用于统计文件或输入流中的字节数、字数和行数。它是 "word count"（单词计数）的缩写。

使用 `wc` 命令的基本语法如下：

```
wc [选项] [文件...]
```

其中，选项可以是以下之一：

- `-c`：统计字节数。
- `-w`：统计字数（以空格、制表符或换行符分隔的单词数）。
- `-l`：统计行数。

如果不指定文件参数，则 `wc` 命令会从标准输入读取数据并进行统计。

以下是一些示例：

1. 统计文件的字节数、字数和行数：

```
wc file.txt
```

上述命令将统计名为 `file.txt` 的文件的字节数、字数和行数。

2. 统计多个文件的总字节数、总字数和总行数：

```
wc file1.txt file2.txt file3.txt
```

上述命令将统计名为 `file1.txt`、`file2.txt` 和 `file3.txt` 的文件的总字节数、总字数和总行数。

3. 从标准输入读取数据并统计：

```
echo "Hello, world!" | wc
```

上述命令将统计从标准输入读取的数据的字节数、字数和行数。在这个例子中，输入是字符串 "Hello, world!"。

`wc` 命令还可以与其他命令结合使用，例如使用管道将输出重定向到 `wc` 命令进行统计。






## （四）Shell运算符和运算命令



### 算术运算符

| 运算符 | 表达式                                              |
| ------ | --------------------------------------------------- |
| +      | expr $a + $b  #无需echo，执行expr后结果会输出到终端 |
| -      | expr $a - $b                                        |
| *      | expr $a \\* $b   #'*'要转义                         |
| /      | expr $a % $b                                        |
| %      | expr $a + $b                                        |
| =      | a=$b                                                |

如果有括号也需要转义，且使用expr时每个变量和运算符之间都必须有空格

![image-20230724180752377](http://foolcorn-image.oss-cn-shenzhen.aliyuncs.com/img/image-20230724180752377.png)

最后测试结果是14。





### 整数比较运算符

使用[]时，必须要有空格隔开每个token，使用(())则不能有空格

| 运算符 | 说明     | 表达式        |
| ------ | -------- | ------------- |
| -eq    | 相等     | [ $a -eq $b ] |
| -ne    | 不等     | [ $a -ne $b ] |
| -gt    | 大于     | [ $a -gt $b ] |
| -lt    | 小于     | [ $a -lt $b ] |
| -ge    | 大于等于 | [ $a -ge $b ] |
| -le    | 小于等于 | [ $a -le $b ] |
| >      | 大于     | (($a>$b))     |
| <      | 小于     | (($a<$b))     |
| >=     | 大于等于 | (($a>=$b))    |
| <=     | 小于等于 | (($a<=$b))    |
| ==     | 等于     | (($a==$b))    |
| !=     | 不等于   | (($a!=$b))    |

注意逻辑表达式的结果true是返回0，false返回1，和C语言不同



### 字符串比较运算符

字符串比较可以用[[]]和[]两种形式

| 运算符 | 说明               | 表达式                         |
| ------ | ------------------ | ------------------------------ |
| ==或=  | 相等               | [ $a == $b ] 或 [[ $a == $b ]] |
| !=     | 不等               | [ $a != $b ] 或 [[ $a != $b ]] |
| <      | 小于               | [ $a \\< $b ] 或 [[ $a < $b ]] |
| >      | 大于               | [ $a \\> $b ] 或 [[ $a > $b ]] |
| -z     | 检查字符串是否为空 | [ -z $a ]                      |
| -n     | 检查字符串是否非空 | [ -n $a ]                      |
| $      | 检查字符串是否为空 | [ $a ]                         |

使用[[ $a < $b && $a == $b ]]来实现<=，因为shell的true是0，false是1，故使用&&而非||

##### []和[[]]的区别

- []会识别带空格的字符串并拆分成多个字符串后比较，[[]]不会拆分空格
- []的>和<需要转义，[[]]不需要



### 布尔运算符

布尔运算符只能用[]，没有[[]]

| 运算符 | 说明   | 表达式           |
| ------ | ------ | ---------------- |
| !      | 取反   | [ ! exp1 ]       |
| -o     | 或运算 | [ exp1 -o exp2 ] |
| -a     | 与运算 | [ exp1 -a exp2 ] |



### 逻辑运算符

&&和||只能在[[]]和(())中生效。

!只能在[]和[[]]中生效，(())中不生效。

| 运算符 | 说明   | 表达式               |
| ------ | ------ | -------------------- |
| !      | 非     | [[ ! exp1 ]]         |
| &&     | 逻辑与 | [[ exp1 && exp2 ]]   |
| \|\|   | 逻辑或 | [[ exp1 \|\| exp2 ]] |



### 文件测试运算符

#### linux文件类型

-：普通文件

d：目录文件

l：链接文件

b：块设备文件

c：字符设备文件

文件测试运算符可以是[]也可以[[]]

| 运算符          | 说明                           | 表达式              |
| --------------- | ------------------------------ | ------------------- |
| -b file         | 是否是块设备文件               | [ -b $file ]        |
| -c file         | 是否是字符设备文件             | [ -c $file ]        |
| -d file         | 是否是目录                     | [ -d $file ]        |
| -f file         | 是否是普通文件                 | [ -f $file ]        |
| -g file         | 是否设置了SGID位               | [ -g $file ]        |
| -k file         | 是否设置了粘着位（sticky bit） | [ -k $file ]        |
| -p file         | 是否是管道文件                 | [ -p $file ]        |
| -u file         | 是否设置了SUID位               | [ -u $file ]        |
| -r file         | 文件是否可读                   | [ -r $file ]        |
| -w file         | 文件是否可写                   | [ -w $file ]        |
| -x file         | 文件是否可执行                 | [ -x $file ]        |
| -s file         | 文件是否为空                   | [ -s $file ]        |
| -e file         | 文件\目录是否存在              | [ -e $file ]        |
| file1 -nt file2 | new than：file1是否比file2新   | [ file1 -nt file2 ] |
| file1 -nt file2 | older than：file1是否比file2旧 | [ file1 -ot file2 ] |

-S：检查文件是否是socket

-L：检测文件是否存在且是符号链接



### 运算命令

#### expr命令

expr语法：

```shell
expr 算术表达式          
result = `算术表达式`              #赋值变量时注意是反引号
expr length "str"                #计算字符串长度
expr substr "str" start end      #注意该语法下标从1开始（非常反人类），end为结束的下标，并包含end位置的字符
expr index "str" char            #查找char在字符串中首次出现的下标（依然从1开始）
expr match "str" 正则表达式        #匹配正则表达式，正则表达式默认带^，返回值为符合匹配的子串的长度，否则返回0
```



#### (())命令

用于算术表达式的计算，可以用$获取计算的结果。

语法

```shell
((a=b+1))                        #使用双括号可以不用$取变量值,token之间有无空格都可以
result=$((a+b))                  #赋值可以用$
((a>7 && b==c))                  #可以进行逻辑运算，配合if等命令使用
((a=b+1, c=b+2))                 #允许多个表达式，用","隔开，其中的a,c变量都是左值
```



#### let命令

let命令主要用于算数运算，可以看做(())的弱化版，只能用于赋值，因为计算结果是右值，也无法作为条件判断

语法：

```shell
let a+b                          #结果是右值，无法捕获
let c=a+b                        #只能将结果找一个左值变量存放
let a=b+1 c=b+2                  #允许多个表达式，用" "隔开
```



#### $[]命令

$[]命令也可以用于整数运算，但是功能更弱，只能对单表达式进行计算与输出，无法赋值

语法：

```shell
$[表达式]                         #token之间可以不需要有空格
```



#### bc命令

对浮点数进行运算

语法：

```shell
bc [options] [参数]
echo "表达式" | bc [options]              #非交互式，利用管道执行单行表达式
result=`echo "表达式" | bc [options]`     #使用反引号，兼容性好
result=$(echo "表达式" | bc [options])    #使用$()，只有bash shell支持
result=`bc [options] << EOF              #多行表达式，使用反引号
exp1...                                  #如果多行表达式有多个结果，则返回一个结果数组
exp2...
exp3...
...
EOF
`
exp=$(bc [options] << EOF                #多行表达式，使用$()
exp1...
exp2...
exp3...
...
EOF
)

```

##### options

- -h：help
- -v：版本
- -l：mathlib，标准数学库
- -i：interactive，强制交互
- -w：warn，现实posix警告
- -s：standard，使用posix标准
- -q：quiet，静默模式

##### 参数：

包含计算指令的文件，有点类似matlab语法，如果不填则进行交互式的运算。

##### 内置变量：

一些有用的全局配置变量

| 变量名    | 作用                           |
| --------- | ------------------------------ |
| scale     | 设置全局精度，小数点后位数     |
| ibase     | 设置输入的数字进制，默认十进制 |
| obase     | 设置输出的数字进制，默认十进制 |
| last或者. | 存储了最近一次的计算结果       |



##### 内置函数

| 函数    | 作用                       |
| ------- | -------------------------- |
| s(x)    | 计算x的正弦值，x是弧度     |
| c(x)    | 计算x的余弦值，x是弧度     |
| a(x)    | 计算x的反正切值，x是弧度   |
| l(x)    | 计算x的自然对数            |
| e(x)    | 求e的x次幂                 |
| j(n, x) | 贝塞尔函数，计算n到x的阶数 |



## （五）流程控制语句

### if语句

语法：

```shell
if conditon
then
    ...
elif(optional)
    ...
else(optional)
    ...
fi
```



### case语句

类似switch但是语法并不相同

语法：

```shell
case $val in
case1)
    code...
    ...
    ;;                            #类似break
case2)
    code...
    ...
    ;;
*)                                #正则表达式，代表任意字符匹配，和default相同的效果
    code...
    ...
    ;;
esac
```



##### case类型

- 数字
- 字符串
- 正则表达式：
  - *：任意字符串
  - [abc]：[]内任意一个字符
  - [a-z]：任意一个小写字母
  - 正则1 | 正则2：或匹配



### while语句

循环，不赘述了

语法：

```shell
while condition
do
    ...
    ...
    continue;       #optional
    break;          #optional
done
```



### until语句

和while相反，条件为false就循环，条件为true时则停止

语法：

```shell
until condition
do
    ...
    ...
done
```



###  for语句

##### 范围for循环语法

```shell
for var in list
do
    ...
    ...
done
```

##### 关于list

- 可以是一个array
- 可以自己写一个序列，用空格隔开
  - for var in 1 2 3 4 5
- 可以自己写一个范围
  - for var in {1..5}



##### 不使用范围for循环的语法

```shell
for((i=start;i<end;i++))
do
    ...
done
```



### select语句

提供给用户一个select菜单并选择，后台根据选项做处理，主要用于交互，基本不会用于脚本。

语法：

```shell
select var in menu1 menu2 ...   #这是死循环，只有break语句或者ctrl+D退出循环，一般利用一个menu配合break来退出。
do
    ...
done
```



## （六）Shell函数

### 系统函数

其实学了常用命令，和系统函数也没差了



### 自定义函数

语法

```shell
function(optional) fun_name ()       #无参数
{
    ...
    return code                      #(optional)
}

function(optional) fun_name ()       #有参数也是空括号，通过特殊符号读取（具体用法在（二）shell的变量中已经提过）
{
    echo $#                          #获取参数个数
    echo $*                          #获取所有参数（字符串）
    echo $$                          #当前进程号
    for item in "$@"                 #加引号获取参数数组
    do
        echo $item
    done                             
    return code                      #(optional)
}
```



## （七）重定向

Linux的每个进程都会赋予三个标准输入输出流，stdin，stdout和stderr。

由于linux一切接文件，这三个流实际上与三个文件一一对应

在内核中，会给进程维护一个表记录打开文件的文件描述符，其中stdin，stdout，和stderr的文件描述符分别是0，1，2

shell的重定向语法无非就是改变了输入流和输出流，不再使用标准输入输出

语法：

| 命令                  | 作用                                                         |
| --------------------- | ------------------------------------------------------------ |
| 命令 > file           | 将命令结果转向文件中，覆盖式                                 |
| 命令 > file           | 命令的输入参数是从文件中读取                                 |
| 命令 > >file          | 将命令结果转向文件中，追加式                                 |
| 命令 \< file1 > file2 | 从file1读，并写入file2                                       |
| 命令 fd> file         | 根据文件描述符将命令结果转向文件中，覆盖式，主要是用于将stderr的消息重定向 |
| 命令 fd>> file        | 根据文件描述符将命令结果转向文件中，追加式                   |
| 命令 > file fd1>& fd2 | 将fd1和fd2文件描述符合并输出到file中(比如stdout和stderr)     |
| fd1<& fd2             | 将fd1和fd2文件描述符合并，从file中读取输入                   |
| <<tag                 | 读取终端输入数据，将开始标记tag和结束标记tag之间的内容作为输入(在交互式可以多行) |

注意文件描述符的语法，fd>，fd>>，fd>&，fd<&中间不能有空格

举个例子，输入重定向

从文件读取每行并打印

```shell
while read str; do echo $str; done < xxx.txt
```



## （八）Shell工具

### sed

可以对文本数据的每一行匹配查询后进行增删改查,支持按行，按字段，按正则匹配文本内容，有点类似sql的表操作

##### 语法

```shell
sed [参数] [模式匹配/命令] [文件名] 

```



##### 参数：

- -e：用于执行多条sed指令
- -i：直接对内容进行修改，否则默认只是预览
- -f：读取的文件
- -n：取消默认输出（sed默认输出所有文本），只显示处理过的行
- -r：使用拓展正则表达式

##### 模式

- lineno：填写行号
- /pattern/：完全匹配pattern字符串
- $：最后一行
- 1~2：操作奇数行（从第1行开始，每隔两行即匹配）
- 1,3：匹配1~3行数据（范围）
- 1,3!：匹配除了1~3行的所有数据（!取反）

##### 命令

- a：在匹配行的下一行增加内容
- c：更改匹配行内容
- d：删除匹配的内容
- i：向匹配行之前插入内容
- p：打印匹配的内容，通常与-n参数配合
- s：替换匹配的内容（注意和c的区别，一个是替换整行，一个是替换行内的部分匹配内容）
- =：打印被匹配的行号
- n：读取下一行

##### 特殊符号

- !：取反
- {sed命令1;sed命令2}：多命令



一些示例：

```shell
sed '/pattern/,$d' xxx.txt                          #删除匹配行到最后一行
sed '/pattern/,+nd' xxx.txt                         #删除匹配行及后面n行
sed 's/pattern/replace_str/' xxx.txt                #匹配有pattern的行并将第一个pattern替换为replace_str
sed 's/pattern/replace_str/2' xxx.txt               #匹配有pattern的行并将第二个pattern替换为replace_str
sed 's/pattern/replace_str/g' xxx.txt               #匹配有pattern的行并将所有pattern替换为replace_str
sed 's/pattern/replace_str/gw xxx2.txt' xxx.txt     #将替换后的行写入xxx2.txt
sed -e 'sed1' -e 'sed2' xxx.txt                     #执行多条指令
```



##### 暂存空间

sed提供一个缓冲区，从而提供更丰富的操作

比如将文件的某一行赋值到暂存空间，后从暂存空间粘贴到文件中指定的某行



### awk

##### 语法

```shell
awk [options] 'pattern{action}' [file]
```

##### options

- -F：指定分隔符
- -v：赋值一个用户自定义变量



##### 内置变量

- ARGC：命令行参数个数
- ARGV：命令行参数排列
- ENVIRON：支持队列中系统环境变量的使用
- FILENAME：awk浏览的文件名
- FNR：浏览文件的记录数
- FS：设置分隔符，等同于-F
- NF：分割后的列数
- NR：已读的记录数
- OFS：输出域分割符
- ORS：输出记录分隔符
- RS：控制记录分隔符
- $n：\$0整条记录 \$1:第一列
- $NF：最后一列的信息
