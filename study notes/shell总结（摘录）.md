# shell知识总结

摘录自[头条](https://www.toutiao.com/article/7189129910215115316/?app=news_article&timestamp=1673912159&use_new_style=1&req_id=20230117073559D1582511A4854630995C&group_id=7189129910215115316&wxshare_count=1&tt_from=weixin&utm_source=weixin&utm_medium=toutiao_android&utm_campaign=client_share&share_token=178ac750-af52-4ffc-a6a7-7c6463548fc7&source=m_redirect)

# 一、什么是shell？

shell是一个用 C 语言编写的程序，它是用户使用 Linux 的桥梁。Shell 既是一种命令语言，又是一种程序设计语言。

Shell 是指一种应用程序，这个应用程序提供了一个界面，用户通过这个界面访问操作系统内核的服务。

Shell属于内置的脚本，程序开发的效率非常高，依赖于功能强大的命令可以迅速地完成开发任务（批处理）语法简单，代码写起来比较轻松，简单易学。

# 二、shell的分类

# cat /etc/shells

在linux中有很多类型的shell，不同的shell具备不同的功能，shell还决定了脚本中函数的语法，Linux中默认的shell是 /bash/bash（重点），流行的shell有ash、bash、ksh、csh、zsh等，不同的shell都有自己的特点以及用途。

# 编写规范：

\#!/bin/bash [指定告知系统当前这个脚本要使用的shell解释器]

Shell相关指令

# 文件命名规范：

*.sh

.sh是linux下bash shell 的默认后缀

# Bash 常用快捷键

| 快捷键 | 作用                                                         |
| ------ | ------------------------------------------------------------ |
| ctrl+A | 把光标移动到命令行开头。如果我们输入的命令过长，想要把光标移动到命令行开头时使用。 |
| ctrl+E | 把光标移动到命令行结尾。                                     |
| ctrl+C | 强制终止当前的命令。                                         |
| ctrl+L | 清屏，相当于clear命令。                                      |
| ctrl+U | 删除或剪切光标之前的命令。我输入了一行很长的命令，不用使用退格键一个一个字符的删除，使用这个快捷键会更加方便 |
| ctrl+K | 删除或剪切光标之后的内容。                                   |
| ctrl+Y | 粘贴ctrl+U或ctul+K剪切的内容。                               |
| ctrl+R | 在历史命令中搜索，按下ctrl+R之后，就会出现搜索界面，只要输入搜索内容，就会从历史命令中搜索。 |
| ctrl+D | 退出当前终端。                                               |
| ctrl+Z | 暂停，并放入后台。这个快捷键牵扯工作管理的内容，我们在系统管理章节详细介绍。 |
| ctrl+S | 暂停屏幕输出。                                               |
| ctrl+Q | 恢复屏幕输出。                                               |

# 输入输出重定向

linux 的标准输入与输出

| 设备   | 设备名      | 文件描述符 | 类型         |
| ------ | ----------- | ---------- | ------------ |
| 键盘   | /dev/stdin  | 0          | 标准输入     |
| 显示器 | /dev/stdout | 1          | 标准输出     |
| 显示器 | /dev/stderr | 2          | 标准错误输出 |

输入重定向

**输入重定向**：是指不使用系统提供的标准输入端口，而进行重新的指定。换言之，输入重定向就是不使用标准输入端口输入文件，而是使用指定的文件作为标准输入设备。（重定向简单理解就是使用 “<”符来修改标准输入设备）

| 类型                       | 符号（语法）       | 功能                                                 |
| -------------------------- | ------------------ | ---------------------------------------------------- |
| 标准输入                   | 命令<文件1         | 命令把文件1的内容作为标准输入设备                    |
| 标识符限定输入             | 命令<<标识符       | 命令把标准输入中读入内容，直到遇到“标识符”分解符为止 |
| 输入输出重定向（同时使用） | 命令< 文件1 >文件2 | 命令把文件1的内容作为标准输入，把文件2作为标准输出。 |

输出重定向

**输出重定向**：（通俗的讲，重定向输出就是把要输出的文件信息写入到一个文件中去，而不是将要输出的文件信息输出到控制台（显示屏），在linux中，默认的标准输出设备是控制台（或称为显示器）,用户输出的信息默认情况下都会显示到控制台.

&表示全部文件，文件不管对错，1表示标准输出文件，2表示标准错误输出。

| 类型                       | 符号（语法）           | 功能                                                       |
| -------------------------- | ---------------------- | ---------------------------------------------------------- |
| 标住输出重定向             | 命令 > 文件            | 以覆盖方式，把命令的正确输出内容输出到指定的文件或设备当中 |
| 标住输出重定向             | 命令 >> 文件           | 以追加方式，把命令的正确输出内容输出到指定的文件或设备当中 |
| 标准错误输出重定向         | 错误命令 2> 文件       | 以覆盖方式，把命令的错误输出输出到指定的文件或设备当中     |
| 标准错误输出重定向         | 错误命令 2>> 文件      | 以追加方式，把命令的错误输出输出到指定的文件或设备当中     |
| 正确输出和错误输出同时保存 | 命令 > 文件 2>&1       | 以覆盖的方式，把正确输出和错误输出都保存到同一个文件当中。 |
| 正确输出和错误输出同时保存 | 命令 >> 文件 2>&1      | 以追加的方式，把正确输出和错误输出都保存到同一个文件当中。 |
| 正确输出和错误输出同时保存 | 命令 &> 文件           | 以覆盖的方式，把正确输出和错误输出都保存到同一个文件当中。 |
| 正确输出和错误输出同时保存 | 命令 &>> 文件          | 以追加的方式，把正确输出和错误输出都保存到同一个文件当中。 |
| 正确输出和错误输出同时保存 | 命令 >> 文件1 2>>文件2 | 把正确的输出追加到文件1中，把错误的输出追加到文件2中。     |

/dev/null 文件

如果希望执行某个命令，但又不希望在屏幕上显示输出结果，那么可以将输出重定向到 **/dev/null**中.

[root@localhost ~]$ command > dev/null

# 多命令顺序执行

| 多命令执行符 | 作用             | 格式                                                         |
| ------------ | ---------------- | ------------------------------------------------------------ |
| ；           | 命令1 ；命令2    | 多个命令顺序执行，命令之间没有任何逻辑联系                   |
| &&           | 命令1 && 命令2   | 当命令1正确执行（? = 0 ） ， 则命令2才会执行；当命令 1执行不正确（?=0），则命令2不会执行 |
| \|\|         | 命令1 \|\| 命令2 | 当命令1执行不正确（?≠0），则命令2才会执行;当命令1正确执行（?≠0），则命令2不会执行. |

# shell脚本的执行

[root@localhost ~]$ vim test.sh

\#!/bin/bash

echo “hello world”

两种方式执行shell脚本

第一种：给文件增加执行权限

[root@localhost ~]$ chmod u+x test.sh

[root@localhost ~]$ ./test.sh #绝对路径或相对路径执行

第二种（了解）：通过Bash调用执行脚本

[root@localhost ~]$ bash test.sh

# 三、shell变量

# 什么是变量？

在一个脚本周期内,其值可以发生改变的量就是变量。

# 1. 变量的命名规则：

在定义变量时，有一些规则需要遵守:

l 命名只能使用英文字母，数字和下划线，首个字符不能以数字开头。

l 等号左右两侧不能有空格，可以使用下划线“_”，变量的值如果有空格，需要使用单引号或双引号包括。如:“test=“hello world!””。其中双引号括起来的内容“$”，“(”和反引号都拥有特殊含义，而单引号括起来的内容都是普通字符。

l 不能使用标点符号，不能使用bash里的关键字（可用help命令查看保留关键字）。

l 环境变量建议大写，便于区分

l 如果需要增加变量的值，那么可以进行变量值的叠加。不过变量需要用双引号包含"$变量名"或用${变量名}包含变量名。

<!---->

[root@localhost ~]$ test=123

[root@localhost ~]$ test="$test"456

[root@localhost ~]$ echo $test

123456

\#叠加变量test，变量值变成了123456

[root@localhost ~]$ test=${test}789

[root@localhost ~]$ echo $test

123456789

\#再叠加变量test，变量值编程了123456789

**关于单双引号的问题：**

双引号能够识别变量，双引号能够实现转义（类似于“*”）

单引号是不能识别变量，只会原样输出，单引号是不能转义的

# shell中特殊符号

| 符号 | 作用                                                         |
| ---- | ------------------------------------------------------------ |
| ''   | 单引号。在单引号中所有的特殊符号，如“$”和”(反引号)都没有特殊含义。单引号括起来的都是普通字符，会原样输出 |
| ""   | 双引号。在双引号中特殊符号都没有特殊含义.“$”表示“调用变量的值”，“`”（esc键下面）表示“引用命令”，“\”表示“转义符”。 |
| ``   | 反引号。反引号括起来的内容是系统命令，在Bash中会先执行它。和()作用一 样， 不过推荐使用()，因为反引号非常容易看错。 |
| $()  | 和反引号作用一样，用来引用系统命令。(推荐使用)               |
| ()   | 用于一串命令执行时，()中的命令会在子Shell中运行              |
| {}   | 用于一串命令执行时，{ }中的命令会在当前Shell中执行。也可以用于变量变形与替换。 |
| [ ]  | 用于变量的测试。                                             |
| #    | 在Shell脚本中，#开头的行代表注释。                           |
| $    | 用于调用变量的值，如需要调用变量name的值时，需要用$name的方式得到变量的值。 |
| \    | 转义符，跟在\之后的特殊符号将失去特殊含义，变为普通字符。如\$将输出“$”符号，而不当做是变量引用。 |

**单引号和双引号**

[root@localhost ~]$ name=sc

\#定义变量name 的值是sc（就是最正直的人，超哥我了!）

[root@localhost ~]$ echo '$name'

$name

\#如果输出时使用单引号，则$name原封不动的输出

[root@localhost ~]$ echo "$name"

sc

\#如果输出时使用双引号，则会输出变量name的值 sc



[root@localhost ~]$ echo `date`

2018年10月21日星期一18:16:33 CST

\#反引号括起来的命令会正常执行

[root@localhost ~]$ echo '`date`'

```
date
```

\#但是如果反引号命令被单引号括起来，那么这个命令不会执行，―date`会被当成普通字符输出

[root@localhost ~]$ echo "`date'"

2018年10月21日星期一18:14:21 CST

\#如果是双引号括起来，那么这个命令又会正常执行

**反引号**

[root@localhost ~]$ echo ls

ls

\#如果命令不用反引号包含，命令不会执行，而是直接输出

[root@localhost ~]$ echo `ls`

anaconda-ks.cfginstall.loginstall.log.syslog sh test testfile

\#只有用反引号包括命令，这个命令才会执行

[root@localhost ~]$ echo $(date)

2018年10月21日星期一18:25:09 CST

\#使用$(命令)的方式也是可以的

# 2. 变量的分类

l 用户自定义变量: 这种变量是最常见的变量，由用户自由定义变量名和变量的值。

l 环境变量: 这种变量中主要保存的是和系统操作环境相关的数据，比如当前登录用户，用户的家目录，命令的提示符等。不是太好理解吧，那么大家还记得在Windows中，同一台电脑可以有多个用户登录，而且每个用户都可以定义自己的桌面样式和分辨率，这些其实就是Windows的操作环境，可以当做是Windows的环境变量来理解。环境变量的变量名可以自由定义，但是一般对系统起作用的环境变量的变量名是系统预先设定好的。

l 位置参数变量: 这种变量主要是用来向脚本当中传递参数或数据的，变量名不能自定义，变量作用是固定的。

l 预定义变量: 是Bash中已经定义好的变量，变量名不能自定义，变量作用也是固定的。

| 变量分类                       | 名称   | 作用   | 内容   |
| ------------------------------ | ------ | ------ | ------ |
| 用户自定义变量                 | 自定义 | 自定义 | 自定义 |
| 用户自定义环境变量             | 自定义 | 自定义 | 自定义 |
| 系统自带环境变量(/etc/profile) | 确定   | 确定   | 自定义 |
| 位置参数变量                   | 确定   | 自定义 | 自定义 |
| 预定义变量                     | 确定   | 自定义 | 自定义 |

# 2.1 用户自定义变量

# 2.1.1 变量定义

[root@localhost ~]$ 2name="shen chao"

-bash: 2name=shen chao: command not found

\#变量名不能用数字开头

[root@localhost ~]$ name = "shenchao"

-bash: name: command not found

\#等号左右两侧不能有空格

[root@localhost ~]$ name=shen chao

-bash: chao: command not found

\#变量的值如果有空格，必须用引号包含

# 2.1.2 变量调用

[root@localhost ~]$ name="shen chao"

\#定义变量name

[root@localhost ~]$ echo $name #调用变量使用 $变量名

shen chao

\#输出变量name的值

# 2.1.3 变量查看

[root@localhost ~]$ set [选项]

选项:

-u:如果设定此选项，调用未声明变量时会报错（默认无任何提示）

-x:如果设定此选项，在命令执行之前，会把命令先输出一次

+<参数> :取消某个set曾启动的参数。



[root@localhost ~]$ set

BASH=/bin/bash

…省略部分输出…

name='shen chao'

\#直接使用set 命令，会查询系统中所有的变量，包含用户自定义变量和环境变量

[root@localhost ~]$ set -u

[root@localhost ~]$ echo $file

-bash: file: unbound variable

\#当设置了-u选项后，如果调用没有设定的变量会有报错。默认是没有任何输出的。

[root@localhost ~]$ set -x

[root@localhost ~]$ ls

+ls --color=auto

anaconda-ks.cfginstall.loginstall.log.syslog sh tdir testtestfile

\#如果设定了-x选项，会在每个命令执行之前，先把命令输出一次



[root@localhost ~]$ set +x

\#取消启动的x参数

# 2.1.4 变量删除

[root@localhost ~]$ unset 变量名

# 2.2 环境变量

# 2.2.1 环境变量设置

[root@localhost ~]$ export age="18"

\#使用export声明的变量即是环境变量

# 2.2.2 环境变量查询和删除

**env命令和set命令的区别**：

set命令可以查看所有变量，而env命令只能查看环境变量。

[root@localhost ~]$ unset gender #删除环境变量gender

[root@localhost ~]$ env | grep gender

# 2.2.3 系统默认环境变量

[root@localhost ~]$ env

HOSTNAME=localhost.localdomain #主机名

SHELL=/bin/bash #当前的shell

TERM=linux #终端环境

HISTSIZE=1000 #历史命令条数

SSH_CLIENT=192.168.4.1594824 22 #当前操作环境是用ssh连接的，这里记录客户端ip

SSH_TTY=/dev/pts/1 #ssh连接的终端时pts/1

USER=root #当前登录的用户

..........更多参数可以使用set和env命令查看.............

# 2.3 位置参数变量

| 位置参数变量 | 作用                                                         |
| ------------ | ------------------------------------------------------------ |
| $n           | n为数字，$0表示当前 Shell脚本程序的名称，$1-9代表 第一到第九个参数 , 十以上的参数需要用大括号包含，如9代表第一到第九个参数,十以上的参数需要用大括号包含，如9代表第一到第九个参数,十以上的参数需要用大括号包含，如${10} |
| $*           | 这个变量代表命令行中所有的参数，$把所有的参数看成一个整体    |
| $@           | 这个变量也代表命令行中所有的参数，不过$@把每个参数区分对待   |
| $#           | 这个变量代表命令行中所有参数的个数                           |

$1 是你给你写的shell脚本传的第一个参数，$2 是你给你写的shell脚本传的第二个参数…

[root@localhost sh]$ vim test.sh

\#!/bin/sh

echo "shell脚本本身的名字: $0"

echo "传给shell的第一个参数: $1"

echo "传给shell的第二个参数: $2"

保存退出后，你在Test.sh所在的目录下输入 bash Test.sh 1 2

结果输出：

shell脚本本身的名字: Test.sh

传给shell的第一个参数: 1

传给shell的第二个参数: 2

$*会把接收的所有参数当成一个整体对待，而$@则会区分对待接收到的所有参数。举个例子:

[root@localhost sh]$ vi parameter2.sh

\#!/bin/bash

for i in"$*"

\#定义for循环，in后面有几个值，for会循环多少次，注意“$*”要用双引号括起来

\#每次循环会把in后面的值赋予变量i

\#Shell把$*中的所有参数看成是一个整体，所以这个for循环只会循环一次

do

echo "The parameters is: $i"

\#打印变量$i的值

done

x=1

\#定义变量x的值为1

for y in"$@"

\#同样in后面的有几个值，for循环几次，每次都把值赋予变量y

\#可是Shel1中把“$@”中的每个参数都看成是独立的，所以“$@”中有几个参数，就会循环几次

do

echo "The parameter$x is: $y"

\#输出变量y的值

x=$(( $x +1 ))

\#然变量x每次循环都加1，为了输出时看的更清楚

done

# 2.4 预定义变量

| 预定义变量 | 作用                                                         |
| ---------- | ------------------------------------------------------------ |
| $?         | 最后一次执行的命令的返回状态。如果这个变量的值为0，证明上一个命令正确执行;如果这个变量的值为非О(具体是哪个数，由命令自己来决定），则证明上一个命令执行不正确了。 |
| $$         | 当前进程的进程号（PID)                                       |
| $!         | 后台运行的最后一个进程的进程号(PID)                          |

先来看看”$?”这个变量，举个例子说明

[root@localhost sh]$ ls

count.sh hello.sh parameter2.sh parameter.sh

\#ls命令正确执行

[root@localhost sh]$ echo $?

\#预定义变量“$?”的值是0，证明上一个命令执行正确

[root@localhost sh]$ ls install.log

ls:无法访问install.log:没有那个文件或目录

\#当前目录中没有install.log文件，所以ls命令报错了

[root@localhost sh]$ echo $?

2

\#变量“$?”返回一个非О的值，证明上一个命令没有正确执行

\#至于错误的返回值到底是多少，是在编写ls命令时定义好的，如果碰到文件不存在就返回数值2

再来说明下”$$”和”$!”这两个预定义变量

[root@localhost sh]$ vi variable.sh

\#!/bin/bash

echo "The current process is $$"

\#输出当前进程的PID.

\#这个PID就是variable.sh这个脚本执行时，生成的进程的PID

find /root -name hello.sh &

\#使用find命令在root目录下查找hello.sh文件

\#符号&的意思是把命令放入后台执行，工作管理我们在系统管理章节会详细介绍

echo "The last one Daemon process is $!"

\#输出这个后台执行命令的进程的PID，也就是输出find命令的PID号

# 3. 只读变量

[root@localhost sh]$ vi readonly.sh

\#!/bin/bash

a=10

\#语法：readonly 变量名

readonly a

a=20 #会报错readonly variable

echo $a

# 4. 接受键盘输入

[root@localhost ~]$ read [选项][变量名]

选项:

-a 后跟一个变量，该变量会被认为是个数组，然后给其赋值，默认是以空格为分割符。

-p： “提示信息”：在等待read输入时，输出提示信息

-t： 秒数：read命令会一直等待用户输入，使用此选项可以指定等待时间

-n： 数字：read命令只接受指定的字符数，就会执行

-s： 隐藏输入的数据，适用于机密信息的输入

-d： 后面跟一个标志符，其实只有其后的第一个字符有用，作为结束的标志。

-e： 在输入的时候可以使用命令补全功能。

变量名:

变量名可以自定义，如果不指定变量名，会把输入保存入默认变量REPLY.

如果只提供了一个变量名，则整个输入行赋予该变量.

如果提供了一个以上的变量名，则输入行分为若干字，一个接一个地赋予各个变量，而命令行上的最后一个变量取得剩余的所有字

写个例子来解释下read命令：

[root@localhost sh]$ vi read.sh

\#!/bin/bash



read -t 30 -p "Please input your name: " name

\#提示“请输入姓名”并等待30 秒，把用户的输入保存入变量name 中

echo "Name is $name"

\#看看变量“$name”中是否保存了你的输入



read -s -t 30 -p "Please enter your age: " age

\#提示“请输入年龄”并等待30秒，把用户的输入保存入变量age中

\#年龄是隐私，所以我们用“-s”选项隐藏输入

echo -e "\n"

\#调整输出格式，如果不输出换行，一会的年龄输出不会换行

echo "Age is $age"



read -n 1 -t 30 -p "Please select your gender[M/F]:" gender

\#提示“请选择性别”并等待30秒，把用户的输入保存入变量gender

\#使用“-n1”选项只接收一个输入字符就会执行（都不用输入回车）

echo -e "\n"

echo "Sex is $gender"

# 四、shell 运算符

在shell中，运算符和其他编程脚本语言一样，常见的有算数运算符、关系运算符、逻辑运算符、字符串运算符、文件测试运算符等

# 1. 算数运算符

原生bash不支持简单的数学运算，但是可以通过其他命令来实现，例如 awk 和 expr，expr 最常用。

expr 是一款表达式计算工具，使用它能完成表达式的求值操作。

例如，两个数相加(注意使用的是反引号 ` 而不是单引号 ')：

[root@localhost ~]$ vi computer.sh

\#!/bin/bash

val=`expr 2 + 2`

echo "两数之和为 : $val"

\#注意

\#表达式和运算符之间要有空格，例如 2+2 是不对的，必须写成 2 + 2，这与我们熟悉的大多数编程语言不一样。

\#完整的表达式要被 ` ` 包含，注意这个字符不是常用的单引号，在 Esc 键下边。

下表列出了常用的算术运算符，假定变量 a 为 10，变量 b 为 20

| 运算符 | 说明                                            | 举例                            |
| ------ | ----------------------------------------------- | ------------------------------- |
| +      | 加法                                            | expr $a + $b 结果为 30。        |
| -      | 减法                                            | expr $a - $b 结果为 -10。       |
| *      | 乘法                                            | expr $a * $b 结果为 200。       |
| /      | 除法                                            | expr $b / $a 结果为 2。         |
| %      | 取余                                            | expr $b % $a 结果为 0。         |
| =      | 赋值                                            | a=$b 将把变量 b 的值赋给 a。    |
| ==     | 相等。用于比较两个数字，相同则返回 true（真）。 | [ $a == $b ] 返回 false（假）。 |
| !=     | 不相等。用于比较两个数字，不相同则返回 true。   | [ $a != $b ] 返回 true。        |

注意：条件表达式要放在方括号之间，并且要有空格，必须写成 [ $a == $b ]。

[root@localhost ~]$ vi computers.sh

\#!/bin/bash

a=10

b=20

echo ' '

echo 'a+b= ' `expr $a + $b`

echo 'a-b= ' `expr $a - $b`

echo 'a*b= ' `expr $a \* $b`

echo 'a/b= ' `expr $a / $b`

echo 'a%b= ' `expr $a % $b`



\#判断是否相等

if [ $a == $b ]

then

echo 'a等于b'

else

echo 'a不等于b'

fi

# 2. 关系运算符

关系运算符只支持数字，不支持字符串，除非字符串的值是数字。

下表列出了常用的关系运算符，假定变量 a 为 10，变量 b 为 20：

| 运算符 | 单词                | 说明                                                  | 举例                       |
| ------ | ------------------- | ----------------------------------------------------- | -------------------------- |
| -eq    | equal               | 检测两个数是否相等，相等返回 true。                   | [ $a -eq $b ] 返回 false。 |
| -ne    | not equal           | 检测两个数是否相等，不相等返回 true。                 | [ $a -ne $b ] 返回 true。  |
| -gt    | great than          | 检测左边的数是否大于右边的，如果是，则返回 true。     | [ $a -gt $b ] 返回 false。 |
| -lt    | less than           | 检测左边的数是否小于右边的，如果是，则返回 true。     | [ $a -lt $b ] 返回 true。  |
| -ge    | great than or equal | 检测左边的数是否大于等于右边的，如果是，则返回 true。 | [ $a -ge $b ] 返回 false。 |
| -le    | less than or equal  | 检测左边的数是否小于等于右边的，如果是，则返回 true。 | [ $a -le $b ] 返回 true。  |

[root@localhost ~]$ [ 10 -gt 10 ]

[root@localhost ~]$ echo $?

1

[root@localhost ~]$ [ 10 -eq 10 ]

[root@localhost ~]$ echo $?

0

案例：判断当前输入的用户是否存在。如果存在则提示“用户存在”否则提示“用户不存在”。

如果要在shell脚本使用linux命令，可以使用$()包裹命令

例如：disk_size=$(df -h | awk ‘NR==2 {print $5}’)

[root@localhost ~]$ vim demo.sh

\#!/bin/bash

\#接受用户的输入

read -p '请输入需要查询的用户名:' username



\#获取指定用户名在passwd文件中出现的次数

count=$(cat /etc/passwd | grep $username | wc -l)

\#count=`cat /etc/passwd | grep $username | wc -l`



\#判断出现的次数，如果次数=0则用户不存在，反之存在

if [ $count == 0 ]

then

echo '用户不存在'

else

echo '用户存在'

fi

# 3. 逻辑运算符

下表列出了常用的布尔运算符，假定变量 a 为 10，变量 b 为 20：

| 运算符 | 说明                                                | 举例                                     |
| ------ | --------------------------------------------------- | ---------------------------------------- |
| !      | 非运算，表达式为 true 则返回 false，否则返回 true。 | [ ! false ] 返回 true。                  |
| -o     | 或（或者）运算，有一个表达式为 true 则返回 true。   | [ $a -lt 20 -o $b -gt 100 ] 返回 true。  |
| -a     | 与（并且）运算，两个表达式都为 true 才返回 true。   | [ $a -lt 20 -a $b -gt 100 ] 返回 false。 |

或运算：一个为真即为真，全部为假才是假

与运算：一个为假即为假，全部为真才是真

# 4. 字符串运算符

下表列出了常用的字符串运算符，假定变量 a 为 “abc”，变量 b 为 “efg”：

| 运算符 | 说明                                      | 举例                     |
| ------ | ----------------------------------------- | ------------------------ |
| =      | 检测两个字符串是否相等，相等返回 true。   | [ $a = $b ] 返回 false。 |
| !=     | 检测两个字符串是否相等，不相等返回 true。 | [ $a != $b ] 返回 true。 |
| -z     | 检测字符串长度是否为0，为0返回 true。     | [ -z $a ] 返回 false。   |
| -n     | 检测字符串长度是否为0，不为0返回 true。   | [ -n $a ] 返回 true。    |
| str    | 检测字符串是否为空，不为空返回 true。     | [ $a ] 返回 true。       |

# 5. 文件测试运算符（重点）

文件测试运算符用于检测 Unix/Linux 文件的各种属性。

| 操作符  | 说明                                                         | 举例                      |
| ------- | ------------------------------------------------------------ | ------------------------- |
| -b file | 检测文件是否是块设备文件，如果是，则返回 true。              | [ -b $file ] 返回 false。 |
| -c file | 检测文件是否是字符设备文件，如果是，则返回 true。            | [ -c $file ] 返回 false。 |
| -d file | 检测文件是否是目录，如果是，则返回 true。                    | [ -d $file ] 返回 false。 |
| -f file | 检测文件是否是普通文件（既不是目录，也不是设备文件），如果是，则返回 true。 | [ -f $file ] 返回 true。  |
| -g file | 检测文件是否设置了 SGID 位，如果是，则返回 true。            | [ -g $file ] 返回 false。 |
| -k file | 检测文件是否设置了粘着位(Sticky Bit)，如果是，则返回 true。  | [ -k $file ] 返回 false。 |
| -p file | 检测文件是否是有名管道，如果是，则返回 true。                | [ -p $file ] 返回 false。 |
| -u file | 检测文件是否设置了 SUID 位，如果是，则返回 true。            | [ -u $file ] 返回 false。 |
| -r file | 检测文件是否可读，如果是，则返回 true。                      | [ -r $file ] 返回 true。  |
| -w file | 检测文件是否可写，如果是，则返回 true。                      | [ -w $file ] 返回 true。  |
| -x file | 检测文件是否可执行，如果是，则返回 true。                    | [ -x $file ] 返回 true。  |
| -s file | 检测文件是否为空（文件大小是否大于0），不为空返回 true。     | [ -s $file ] 返回 true。  |
| -e file | 检测文件（包括目录）是否存在，如果是，则返回 true。          | [ -e $file ] 返回 true。  |

注意：权限几个判断，如果只有一个部分符合，则认为是有权限的。

# 五、流程控制

# 1. if条件判断

# 1.1 单分支if条件

语法：

if [ 条件判断式 ]

then

程序

fi

案例：统计根分区使用率

[root@localhost ~]$ vi sh/if1.sh

\#!/bin/bash

\#统计根分区使用率

rate=$(df -h | grep "/dev/sda2" | awk '{print $5}’| cut -d "%"-f1)

\#把根分区使用率作为变量值赋予变量rate

if [ $rate -ge 80 ]

\#判断rate的值如果大于等于80，则执行then程序

then

echo "Warning!/dev/sda3 is fu11!!"

\#打印警告信息。在实际工作中，也可以向管理员发送邮件。

fi

案例：创建目录

[root@localhost ~]$ vi sh/add_dir.sh

\#!/bin/bash

\#创建目录，判断是否存在，存在就结束，反之创建

echo "当前脚本名称为$0"

DIR="/media/cdrom"

if [ ! -e $DIR ]

then

mkdir -p $DIR

fi

echo "$DIR 创建成功"

# 1.2 双分支if条件语句

语法：

if [ 条件判断式 ]

then

条件成立时，执行的程序

else

条件不成立时，执行的另一个程序

fi

案例1：备份mysql数据库

[root@localhost ~]$ vi sh/bakmysql.sh

\#!/bin/bash

\#备份mysql数据库。



ntpdate asia.pool.ntp.org &>/dev/null

\#同步系统时间

date=$(date +%y%m%d)

\#把当前系统时间按照“年月日”格式赋子变量date

size=$(du -sh /var/lib/mysql)

\#统计mysql数据库的大小，并把大小赋予size变量



if [ -d /tmp/dbbak ]

\#判断备份目录是否存在，是否为目录

then

\#如果判断为真，执行以下脚本

echo "Date : $date!" > /tmp/dbbak/dbinfo.txt

\#把当前日期写入临时文件

echo "Data size : $size" >> /tmp/dbbak/dbinfo.txt

\#把数据库大小写入临时文件

cd /tmp/dbbak



\#进入备份目录

tar -zcf mysql-lib-$date.tar.gz /var/lib/mysql dbinfo.txt &> /dev/null

\#打包压缩数据库与临时文件，把所有输出丢入垃圾箱（不想看到任何输出）

rm -rf /tmp/dbbak/dbinfo.txt

\#删除临时文件

else

mkdir /tmp/dbbak

\#如果判断为假，则建立备份目录

echo "Date : $date!" > /tmp/dbbak/dbinfo.txt

echo "Data size : $size" >> /tmp/dbbak/dbinfo.txt

\#把日期和数据库大小保存如临时文件

cd /tmp/dbbak

tar -zcf mysql-lib-$date.tar. gz dbinfo.txt /var/lib/mysql &> /dev/null

\#压缩备份数据库与临时文件

rm -rf/tmp/dbbak/dbinfo.txt

\#删除临时文件

fi

案例2：判断apache是否启动，如果没有启动则自动启动

[root@localhost ~]$ vi sh/autostart.sh

\#!/bin/bash

\#判断apache是否启动，如果没有启动则自动启动



port=$(nmap -sT 192.168.4.210 | grep tcp | grep http | awk '{print $2}')

\#使用nmap命令扫描服务器，并截取 apache服务的状态，赋予变量port

\#只要状态是open，就证明正常启动

if [ "$port" == "open"]

\#如果变量port的值是“open”

then

echo "$(date) httpd is ok!” >> /tmp/autostart-acc.log

\#则证明apache 正常启动，在正常日志中写入一句话即可

else

/etc/rc.d/init.d/httpd start &>/dev/null

\#否则证明apache没有启动，自动启动apache

echo "$(date) restart httpd !!" >> /tmp/autostart-err.log

\#并在错误日志中记录自动启动apche 的时间

fi

nmap端口扫描命令，格式如下：

[root@localhost ~]$ nmap -sT 域名或IP

选项:

-s 扫描

-T 扫描所有开启的TCP端口



\#知道了nmap命令的用法，我们在脚本中使用的命令就是为了截取http的状态，只要状态是“or.



\#就证明apache启动正常，否则证明apache启动错误。来看看脚本中命令的结果:

[root@localhost ~]$ nmap -sT 192.168.4.210 | grep tcp | grep http | awk ' fprint $2}’

\#扫描指定计算机，提取包含tcp 的行，在提取包含httpd 的行，截取第二列open

\#把截取的值赋予变量port

# 1.3 多分支if条件语句

语法：

if [ 条件判断式1 ]

then

当条件判断式1成立时，执行程序1

elif [ 条件判断式2 ]

then

当条件判断式2成立时，执行程序2

…省略更多条件…

else

当所有条件都不成立时，最后执行此程序

fi

案例：判断用户输入的是什么文件

[root@localhost ~]$ vi sh/if-elif.sh

\#!/bin/bash

\#判断用户输入的是什么文件



read -p "Please input a filename: " file

\#接收键盘的输入，并赋予变量file

if [ -z "$file” ]

\#判断file变量是否为空

then

echo "Error, please input a filename"

\#如果为空，执行程序1，也就是输出报错信息

exit 1

\#退出程序，并返回值为Ⅰ(把返回值赋予变量$P）

elif [ ! -e "$file” ]

\#判断file的值是否存在

then

echo "Your input is not a file!"

\#如1果不存在，则执行程序2

exit 2

\#退出程序，把并定义返回值为2

elif [ -f "$file” ]

\#判断file的值是否为普通文件

then

echo "$file is a regulare file!”

\#如果是普通文件，则执行程序3

elif [ -d "$file” ]

\#到断file的值是否为目录文件

then

echo "$file is a directory!"

\#如果是目录文件，网执行程序4

else

echo "$file is an other file!”

\#如果以上判断都不是，则执行程序5

fi

# 2. 多分支case条件语句

case语句和if…elif…else语句一样都是多分支条件语句,不过和if多分支条件语句不同的是，case语句只能判断一种条件关系，而if语句可以判断多种条件关系。

语法:

case $变量名 in

"值1")

如果变量的值等于值1，则执行程序1

;;

"值2")

如果变量的值等于值2，则执行程序2

::

…省略其他分支…

*)

如果变量的值都不是以上的值，则执行此程序

;;

esac

这个语句需要注意以下内容:

l case语句，会取出变量中的值，然后与语句体中的值逐一比较。如果数值符合，则执行对应的程序，如果数值不符，则依次比较下一个值。如果所有的值都不符合，则执行 “*)” (*代表所有其他值）中的程序。

l case语句以“case”开头，以“esac”结尾。

每一个分支程序之后要通过“;;”双分号结尾，代表该程序段结束(千万不要忘记，每次写case语句，都不要忘记双分号）。

案例：

[root@localhost ~]$ vi sh/if-case.sh

\#!/bin/bash

read -p "请输入一个字符，并按Enter确认：" KEY

case "$KEY" in

[a-z]|[A-Z])

echo "您输入的是字母"

;;



[0-9])

echo "您输入的是数字"

;;



*)

echo "您输入的是其他字符"

;;

esac

# 3. for循环

for循环是固定循环，也就是在循环时已经知道需要进行几次的循环，有时也把for循环称为计数循环。for的语法有如下两种:

语法一:

for 变量 in 值1 值2 值3 …(可以是一个文件等)

do

程序

done



这种语法中for循环的次数，取决于in后面值的个数（空格分隔），有几个值就循环几次，并且每次循环都把值赋予变量。

也就是说，假设in后面有三个值，for会循环三次，第一次循环会把值1赋予变量，第二次循环会把值2赋予变量，以此类推。

语法二:

for (( 初始值;循环控制条件;变量变化 ))

do

程序

done



语法二中需要注意:

初始值:在循环开始时，需要给某个变量赋予初始值，如i=1;



循环控制条件:用于指定变量循环的次数，如i<=100，则只要i的值小于等于100，循环就会继续;



变量变化:每次循环之后，变量该如何变化，如i=i+1。代表每次循环之后，变量i的值都加1。

语法一举例：打印时间

[root@localhost ~]$ vi sh/for.sh

\#!/bin/bash

\#打印时间



for time in morning noon afternoon evening

do

echo "This time is $time!"

done

语法一举例：批量解压缩脚本

[root@localhost ~]$ vi sh/auto-tar. sh

\#!/bin/bash

\#批量解压缩脚本



cd/lamp

\#进入压缩包目录

ls *.tar.gz > ls.log

\#把所有.tar.gz结尾的文件的文件覆盖到ls.log 临时文件中

for i in $(cat ls.log) `

\#或者这样写for i in `cat ls.log`



\#读取ls.log文件的内容，文件中有多少个值，就会循环多少次，每次循环把文件名赋予变量i

do

tar -zxf $i &>/dev/null

\#加压缩，并把所有输出都丢弃

done

rm -rf /lamp/ls.log

\#删除临时文件ls.log

语法二举例：从1加到100

[root@localhost ~]$ vi sh/add. sh

\#!/bin/bash

\#从1加到100



s=0

for (( i=1;i<=100;i=i+1 ))

\#定义循环100 次

do

s=$(( $s+$i ))

\#每次循环给变量s赋值

done

echo "The sum of 1+2+...+100 is : $s"

\#输出1加到100的和

语法二举例：批量添加指定数量的用户

[root@localhost ~]$ vi useradd.sh

\#!/bin/bash

\#批量添加指定数量的用户



read -p "Please input user name: " -t 30 name

\#让用户输入用户名，把输入保存入变量name



read -p "Please input the number of users: " -t 30 num

\#让用户输入添加用户的数量，把输入保存入变量num



read -p "Please input the password of users: " -t 30 pass

\#让用户输入初始密码，把输入保存如变量pass



if [ ! -z "$name" -a ! -z "$num"-a ! -z "$pass"]

\#判断三个变量不为空

then

y=$(echo $num | sed 's/[0-9]//g')

\#定义变量的值为后续命令的结果

\#后续命令作用是，把变量num 的值替换为空。如果能替换为空，证明num 的值为数字

\#如果不能替换为空，证明num的值为非数字。我们使用这种方法判断变量num 的值为数字

if [ -z "$y"]

\#如果变量y的值为空，证明num变量是数字

then

for (( i=1 ; i<=$num; i=i+1 ))

\#循环num变量指定的次数

do

/usr/sbin/useradd $name$i &>/dev/null

\#添加用户，用户名为变量name 的值加变量i的数字

echo $pass | /usr/bin/passwd --stdin $name$i &>/dev/null

\#给用户设定初始密码为变量pass 的值

done

fi

fi

语法二举例：批量删除用户

[root@localhost ~]$ vi sh/userdel.sh

\#!/bin/bash

\#批量删除用户



user=$(cat /etc/passwd | grep " /bin/bash"|grep -v "root"Icut -d ":" -f 1)

\#读取用户信息文件，提取可以登录用户，取消root用户，截取第一列用户名



for i in $user

\#循环，有多少个普通用户，循环多少次

do

userdel -r $i

\#每次循环，删除指定普通用户

done

# 4. while循环

语法：

while [ 条件判断式 ]

do

程序

done

案例：1加到100

[root@localhost ~]$ vi sh/addnum.sh

\#!/bin/bash

\#从1加到100



i=1

s=0

\#给变量i和变量s赋值



while [ $i -le 100 ]

\#如果变量i的值小于等于100，则执行循环

do

s=$(( $s+$i ))

i=$(( $i+1 ))

done

echo "The sum is: $s"

案例：输入的数值进行比较判断

[root@localhost ~]$ vi sh/addnum.sh

\#!/bin/bash

PRICE=$(expr $RANDOM % 1000)

TIMES=0



echo "商品的价格为0-999之间，猜猜看是多少？"

while true

do

read -p "请输入您猜的价格：" INT

let TIMES++



if [ $INT -eq $PRICE ] ; then

echo "恭喜您猜对了，实际价格是 $PRICE"

echo "您总共猜了 $TIMES 次"

exit 0

elif [ $INT -gt $PRICE ] ; then

echo "太高了"

else

echo "太低了"

fi

done

# 5. until循环

和while循环相反，until循环时只要条件判断式不成立则进行循环，并执行循环程序。一旦循环条件成立，则终止循环。

语法：

until [ 条件判断式 ]

do

程序

done

案例一：1加到100

[root@localhost ~]$ vi sh/until.sh

\#!/bin/bash

\#从1加到100



i=1

s=0

\#t给变量i和变量s赋值



until [ $i -gt 100 ]

\#循环直到变量i的值大于100，就停止循环

do

s=$(( $s+$i ))

i=$(( $i+1 ))

done

echo "The sum is: $s"

# 6. 函数

语法：

function 函数名 () {

程序

}

案例：接收用户输入的数字，然后从1加到这个数字

[root@localhost ~]$ vi sh/function.sh

\#!/bin/bash

\#接收用户输入的数字，然后从1加到这个数字



function sum () {

\#定义函数sum

s=0

for (( i=0; i<=$num;i=i+1 ))

\#循环直到i大于$1为止。$1是函数sum 的第一个参数

\#在函数中也可以使用位置参数变量，不过这里的$1指的是函数的第一个参数

do

s=$(( $i+$s ))

done

echo "The sum of 1+2+3...+$1 is :$s"

\#输出1加到$1的和

}



read -p "Please input a number: " -t 30 num

\#接收用户输入的数字，并把值赋予变量num

y=$(echo $num | sed 's/[0-9]//g')

\#把变量num的值替换为空，并赋予变量y



if [ -z "$y"]

\#判断变量y是否为空，以确定变量num中是否为数字

then

sum $num

\#调用sum函数，并把变量num的值作为第一个参数传递给sum函数

else

echo "Error!! Please input a number!"

\#如果变量num 的值不是数字，则输出报错信息

fi

# 7. 特殊流程控制语句

# 7.1 exit语句

系统是有exit命令的，用于退出当前用户的登录状态。可是在Shell脚本中，exit语句是用来退出当前脚本的。也就是说，在Shell脚本中，只要碰到了exit语句，后续的程序就不再执行，而直接退出脚本。

exit的语法如下:

exit [返回值]

如果exit命令之后定义了返回值，那么这个脚本执行之后的返回值就是我们自己定义的返回值。可以通过查询$?这个变量，来查看返回值。如果exit之后没有定义返回值，脚本执行之后的返回值是执行exit 语句之前，最后执行的一条命令的返回值。写一个exit 的例子:

[root@localhost ~]$ vi sh/exit.sh

\#!/bin/bash

\#演示exit的作用



read -p "Please input a number: " -t 30 num

\#接收用户的输入，并把输入赋予变量num

y=$ (echo $num | sed 's/[0-9]//g')

\#如果变量num 的值是数字，则把num的值替换为空，否则不替换

\#把替换之后的值赋予变量y

[ -n "$y" ] && echo "Error! Please input a number!" && exit 18

\#判断变量y的值如果不为空，输出报错信息，退出脚本，退出返回值为18

echo "The number is: $num"

\#如果没有退出加班，则打印变量num中的数字

# 7.2 break语句

当程序执行到break语句时，会结束整个当前循环。而continue 语句也是结束循环的语句，不过continue 语句单次当前循环，而下次循环会继续。

案例：

[root@localhost ~]$ vi sh/break.sh

\#!/bin/bash

\#演示break 跳出循环



for (( i=1;i<=10; i=i+1 ))

\#循环十次

do

if ["$i" -eq 4 ]

\#如果变量i的值等于4

then

break

\#退出整个循环

fi

echo $i

\#输出变量i的值

done

执行下这个脚本，因为一旦变量i的值等于4，整个循环都会跳出，所以应该只能循环三次:

[root@localhost ~]$ chmod 755 sh/break.sh

[root@localhost ~]#sh/break.sh

1

2

3

# 7.3 continue语句

continue也是结束流程控制的语句。如果在循环中，continue语句只会结束单次当前循环。

案例：

[root@localhost ~]$ vi sh/break.sh

\#!/bin/bash

\#演示continue



for (( i=1;i<=10;i=i+1 ))

\#循环十次

do

if ["$i" -eq 4 ]

\#如果变量i的值等于4

then

continue

\#退出换成continue

fi

echo $i

\#输出变量i的值

done

执行下这个脚本:

[root@localhost ~]$ chmod 755 sh/continue.sh

[root@localhost ~]#sh/break.sh

1

2

3

5

6

7

8

9

10

\#少了4这个输出

# 六、字符截取、替换和处理命令

**正则表达式**

| 元字符 | 描述                                         | 示例                                                    |
| ------ | -------------------------------------------- | ------------------------------------------------------- |
| \      | 转义符，将特殊字符进行转义，忽略其特殊意义   | a.b匹配a.b，但不能匹配ajb，.被转义为特殊意义            |
| ^      | 匹配行首，awk中，^则是匹配字符串的开始       | ^tux匹配以tux开头的行                                   |
| $      | 匹配行尾，awk中，$则是匹配字符串的结尾       | tux$匹配以tux结尾的行                                   |
| .      | 匹配除换行符\n之外的任意单个字符             | ab.匹配abc或abd，不可匹配abcd或abde，只能匹配单字符     |
| [ ]    | 匹配包含在[字符]之中的任意一个字符           | coo[kl]可以匹配cook或cool                               |
| [^]    | 匹配[^字符]之外的任意一个字符                | 123[^45]不可以匹配1234或1235，1236、1237都可以          |
| [-]    | 匹配[]中指定范围内的任意一个字符，要写成递增 | [0-9]可以匹配1、2或3等其中任意一个数字                  |
| ?      | 匹配之前的项1次或者0次                       | colou?r可以匹配color或者colour，不能匹配colouur         |
| +      | 匹配之前的项1次或者多次                      | sa-6+匹配sa-6、sa-666，不能匹配sa-                      |
| *      | 匹配之前的项0次或者多次                      | co*l匹配cl、col、cool、coool等                          |
| ()     | 匹配表达式，创建一个用于匹配的子串           | ma(tri)?匹配max或maxtrix                                |
| {n}    | 匹配之前的项n次，n是可以为0的正整数          | [0-9]{3}匹配任意一个三位数，可以扩展为[0-9][0-9][0-9]   |
| {n,}   | 之前的项至少需要匹配n次                      | [0-9]{2,}匹配任意一个两位数或更多位数不支持{n,}{n,}{n,} |
| {n,m}  | 指定之前的项至少匹配n次，最多匹配m次，n<=m   | [0-9]{2,5}匹配从两位数到五位数之间的任意一个数字        |
| \|     | 交替匹配\|两边的任意一项                     | ab(c\|d)匹配abc或abd                                    |

# 1 字符截取、替换命令

# 1.1 cut 列提取命令

[root@localhost ~]$ cut [选项] 文件名

选项:

-f 列号: 提取第几列

-d 分隔符: 按照指定分隔符分割列

-n取消分割多字节字符

-c 字符范围: 不依赖分隔符来区分列，而是通过字符范围（行首为0）来进行字段提取。“n-”表示从第n个字符到行尾;“n-m”从第n个字符到第m个字符;“一m”表示从第1个字符到第m个字符。

--complement补足被选择的字节、字符或字段

--out-delimiter指定输出内容是的字段分割符

cut命令的默认分隔符是制表符，也就是“tab”键，不过对空格符可是支持的不怎么好啊。我们先建立一个测试文件，然后看看cut命令的作用吧:

[root@localhost ~]$ vi student.txt

idnamegendermark

1limingm86

2scm67

3tgn90

[root@localhost ~]$ cut -f 2 student.txt

\#提取第二列内容

那如果想要提取多列呢?只要列号直接用“，”分开，命令如下:

[root@localhost ~]$ cut -f 2,3 student.txt

如果我想用cut命令截取df命令的第一列和第三列，就会出现这样的情况:

[root@localhost~]$ df -h | cut -d " " -f 1,3

Filesystem

/dev/sda2

tmpfs

/dev/sda1

# 1.2 awk 编程

参考：

linux文本处理三剑客：grep，sed及awk

# 1.2.1 awk 概述

AWK 是一种处理文本文件的语言，是一个强大的文本分析工具。

# 1.2.2 printf 格式化输出

[root@localhost ~]$ printf ‘输出类型输出格式’ 输出内容



输出类型:

%c: ASCII字符.显示相对应参数的第一个字符

%-ns: 输出字符串，减号“-”表示左对齐(默认右对齐)，n是数字指代输出几个字符,几个参数就写几个%-ns

%-ni: 输出整数，n是数字指代输出几个数字

%f： 输出小数点右边的位数

%m.nf: 输出浮点数，m和n是数字，指代输出的整数位数和小数位数。如%8.2f代表共输出8位数，其中2位是小数，6位是整数。



输出格式:

\a: 输出警告声音

\b: 输出退格键，也就是Backspace键

\f: 清除屏幕

\n: 换行

\r: 回车，也就是Enter键

\t: 水平输出退格键，也就是Tab 键

\v: 垂直输出退格键，也就是Tab 键

为了演示printf命令，我们需要修改下刚刚cut命令使用的student.txt文件，文件内容如下:

[root@localhost ~]$ vi student.txt

ID Name php Linux MySQL Average

1 AAA 66 66 66 66

2 BBB 77 77 77 77

3 CCC 88 88 88 88



\#printf格式输出文件

[root@localhost ~]$ printf '%s\t %s\t %s\t %s\t %s\t %s\t \n’ $(cat student.txt)

\#%s分别对应后面的参数,6列就写6个

ID Name php Linux MySQL Average

1 AAA 66 66 66 66

2 BBB 77 77 77 77

3 CCC 88 88 88 88

如果不想把成绩当成字符串输出，而是按照整型和浮点型输出，则要这样:

[root@localhost ~]$ printf '%i\t %s\t %i\t %i\t %i\t %8.2f\t \n’ \ $(cat student.txt | grep -v Name)

# 1.2.3 awk 基本使用

[root@localhost ~]$ awk‘条件1{动作1} 条件2{动作2}…’ 文件名

条件（Pattern）:

一般使用关系表达式作为条件。这些关系表达式非常多，例如:

x > 10 判断变量x是否大于10

x == y 判断变量x是否等于变量y

A ~ B 判断字符串A中是否包含能匹配B表达式的子字符串

A !~ B 判断字符串A中是否不包含能匹配B表达式的子字符串



动作（Action） :

格式化输出

流程控制语句



常用参数：



-F指定输入时用到的字段分隔符

-v自定义变量

-f从脚本中读取awk命令

-m对val值设置内在限制

我们这里先来学习awk基本用法，也就是只看看格式化输出动作是干什么的。

[root@localhost ~]$ awk '{printf $2 "\t" $6 "\n"}’ student.txt

\#输出第二列和第六列

比如刚刚截取df命令的结果时，cut命令已经力不从心了，我们来看看awk命令:

[root@localhost ~]$ df -h | awk '{print $1 "\t" $3}'

\#截取df命令的第一列和第三列

# 1.2.4 awk 的条件

| 条件的类型 | 条件   | 说明                                                         |
| ---------- | ------ | ------------------------------------------------------------ |
| awk保留字  | BEGIN  | 在awk程序一开始时，尚未读取任何数据之前执行。BEGIN后的动作只在程序开始时执行一次 |
| awk保留字  | END    | 在awk程序处理完所有数据，即将结束时执行。END后的动作只在程序结束时执行一次 |
| 关系运算符 | >      | 大于                                                         |
| 关系运算符 | <      | 小于                                                         |
| 关系运算符 | >=     | 大于等于                                                     |
| 关系运算符 | <=     | 小于等于                                                     |
| 关系运算符 | ==     | 等于。用于判断两个值是否相等，如果是给变量赋值，请使用“”号   |
| 关系运算符 | !=     | 不等于                                                       |
| 关系运算符 | A~B    | 判断字符串A中是否包含能匹配B表达式的子字符串                 |
| 关系运算符 | A!~B   | 判断字符串A中是否不包含能匹配B表达式的子字符串               |
| 正则表达式 | /正则/ | 如果在"//"中可以写入字符，也可以支持正则表达式               |

**BEGIN**

BEGIN是awk的保留字，是一种特殊的条件类型。BEGIN的执行时机是“在 awk程序一开始时，尚未读取任何数据之前执行”。一旦BEGIN后的动作执行一次，当awk开始从文件中读入数据，BEGIN的条件就不再成立，所以BEGIN定义的动作只能被执行一次。

例如:

[root@localhost ~]$ awk 'BEGIN{printf "This is a transcript \n" } {printf $2 "\t" $6 "\n"}’ student.txt

\#awk命令只要检测不到完整的单引号不会执行，所以这个命令的换行不用加入“|”,就是一行命令

\#这里定义了两个动作

\#第一个动作使用BEGIN条件，所以会在读入文件数据前打印“这是一张成绩单”(只会执行一次)

\#第二个动作会打印文件的第二字段和第六字段

**END**

END也是awk保留字，不过刚好和BEGIN相反。END是在awk程序处理完所有数据，即将结束时执行。END后的动作只在程序结束时执行一次。例如:

[root@localhost ~]$ awk 'END{printf "The End \n"} {printf $2 "\t" $6 "\n"}’ student.txt

\#在输出结尾输入“The End”，这并不是文档本身的内容，而且只会执行一次

**关系运算符**

举几个例子看看关系运算符。假设我想看看平均成绩大于等于87分的学员是谁，就可以这样输入命令:

例子1:

[root@localhost ~]$ cat student.txt | grep -v Name | awk '$6 >= 87 {printf $2 "\n"}'

\#使用cat输出文件内容，用grep取反包含“Name”的行

\#判断第六字段（平均成绩）大于等于87分的行，如果判断式成立，则打第六列（学员名$2）

加入了条件之后，只有条件成立动作才会执行，如果条件不满足，则动作则不运行。通过这个实验，大家可以发现，虽然awk是列提取命令，但是也要按行来读入的。这个命令的执行过程是这样的:

1）如果有BEGIN条件，则先执行BEGIN定义的动作。

2）如果没有BEGIN条件，则读入第一行，把第一行的数据依次赋予$0、$1、$2等变量。其中$0代表此行的整体数据，$1代表第一字段，$2代表第二字段。

3）依据条件类型判断动作是否执行。如果条件符合，则执行动作，否则读入下一行数据。如果没有条件，则每行都执行动作。

4）读入下一行数据，重复执行以上步骤。

再举个例子，如果我想看看Sc用户的平均成绩呢:

例子2:

[root@localhost ~]$ awk '$2 ~ /AAA/ {printf $6 "\n"}' student.txt

\#如果第二字段中输入包含有“Sc”字符，则打印第六字段数据

85.66

这里要注意在awk中，使用“//”包含的字符串，awk命令才会查找。也就是说字符串必须用“//”包含，awk命令才能正确识别。

**正则表达式**

如果要想让awk 识别字符串，必须使用“//”包含，例如:

例子1:

[root@localhost ~]$ awk '/Liming/ {print}’student.txt

\#打印Liming的成绩

当使用df命令查看分区使用情况是，如果我只想查看真正的系统分区的使用状况，而不想查看光盘和临时分区的使用状况，则可以:

例子2:

[root@localhost ~]$ df -h | awk '/sda[O-9]/ {printf $1 "\t" $5 "\n"}’

\#查询包含有sda数字的行，并打印第一字段和第五字段

# 1.2.5 awk 内置变量

| awk内置变量 | 作用                                                         |
| ----------- | ------------------------------------------------------------ |
| $0          | 代表目前awk所读入的整行数据。我们已知awk是一行一行读入数据的，$0就代表当前读入行的整行数据。 |
| $n          | 代表目前读入行的第n个字段。比如，$1表示第1个字段(列)，$2表示第2个字段(列)，如此类推 |
| NF          | 当前行拥有的字段（列）总数。                                 |
| NR          | 当前awk所处理的行，是总数据的第几行。                        |
| FS          | 用户定义分隔符。awk的默认分隔符是任何空格，如果想要使用其他分隔符（如“:”），就需要FS变量定义。 |
| ARGC        | 命令行参数个数。                                             |
| ARGV        | 命令行参数数组。                                             |
| FNR         | 当前文件中的当前记录数（对输入文件起始为1）。                |
| OFMT        | 数值的输出格式（默认为%.6g）。                               |
| OFS         | 输出字段的分隔符（默认为空格）。                             |
| ORS         | 输出记录分隔符（默认为换行符）。                             |
| RS          | 输入记录分隔符（默认为换行符）。                             |

awk常用统计实例

1、打印文件的第一列(域) ：

awk '{print $1}' filename



2、打印文件的前两列(域) ：

awk '{print $1,$2}' filename



3、打印完第一列，然后打印第二列 ：

awk '{print $1 $2}' filename



4、打印文本文件的总行数 ：

awk 'END{print NR}' filename



5、打印文本第一行 ：

awk 'NR==1{print}' filename



6、打印文本第二行第一列 ：

sed -n "2, 1p" filename | awk 'print $1'



\1. 获取第一列

ps -aux | grep watchdog | awk '{print $1}'



\2. 获取第一列，第二列，第三列

ps -aux | grep watchdog | awk '{print $1, $2, $3}'



\3. 获取第一行的第一列，第二列，第三列

ps -aux | grep watchdog | awk 'NR==1{print $1, $2, $3}'



\4. 获取行数NR

df -h | awk 'END{print NR}'



\5. 获取列数NF（这里是获取最后一行的列数，注意每行的列数可能是不同的）

ps -aux | grep watchdog | awk 'END{print NF}'



\6. 获取最后一列

ps -aux | grep watchdog | awk '{print $NF}'



\7. 对文件进行操作

awk '{print $1}' fileName



\8. 指定分隔符（这里以:分割）

ps -aux | grep watchdog |awk -F':' '{print $1}'



\9. 超出范围不报错

ps -aux | grep watchdog | awk '{print $100}'



[root@localhost ~]$ cat /etc/passwd | grep "/bin/bash" | awk '{FS=":"} {printf $1 "\t" $3 "\n"}’

\#查询可以登录的用户的用户名和UID

这里“:”分隔符生效了，可是第一行却没有起作用，原来我们忘记了“BEGIN”条件，那么再来试试;

[root@localhost ~]$ cat /etc/passwd | grep "/bin/bash" | awk 'BEGIN {FS=":"} {printf $1 "\t" $3 "\n"}’



[root@localhost ~]$ cat /etc/passwd | grep "/bin/bash" | awk 'BEGIN {FS=":"} {printf $1 "\t" $3 "\t 行号:” NR "\t 字段数:" NF "\n"}’

\#解释下awk命令

\#开始执行{分隔符是“:”}{输出第一字段和第三字段输出行号(NR值）字段数(NF值）}

root 0 行号:1 字段数:7

user1 501 行号:2 字段数:7

如果我只想看看sshd这个伪用户的相关信息，则可以这样使用:

[root@localhost ~]$ cat /etc/passwd | awk 'BEGIN {FS=":"} $1=="sshd" {printf $1 "\t" $3 "\t 行号:" NR "\t 字段数:" NF "\n"}’

\#可以看到sshd 伪用户的UID是74，是/etc/passwd_文件的第28行，此行有7个字段

# 1.2.6 awk 流程控制

我们再来利用下student.txt文件做个练习，后面的使用比较复杂，我们再看看这个文件的内容:

[root@localhost ~]$ cat student.txt

ID Name php Linux MySQL Average

1 AAA 66 66 66 66

2 BBB 77 77 77 77

3 CCC 88 88 88 88

我们先来看看该如何在awk中定义变量与调用变量的值。假设我想统计PHP成绩的总分，那么就应该这样:

[root@localhost ~]$ awk 'NR==2 {php1=$3}

NR==3 {php2=$3}

NR==4 {php3=$3;totle=phpl+php2+php3;print "totle php is " totle}’ student.txt

\#统计PHIP成绩的总分

我们解释下这个命令。

"NR==2 {php1=$3}" (条件是NR==2，动作是php1=$3）这句话是指如果输入数据是第二行（第一行是标题行），就把第二行的第三字段的值赋予变量"php1"。

"NR==3 {php2=$3}" 这句话是指如果输入数据是第三行,就把第三行的第三字段的值赋予变量"php2"。

"NR==4 {php3=$3;totle=phpl+php2+php3;print "totle php is " totle}"（"NR==4"是条件，后面{}中的都是动作)这句话是指如果输入数据是第四行﹐就把第四行的第三字段的值赋予变量"php3";然后定义变量totle的值是"php1+php2+php3";然后输出"totle php is"关键字，后面加变量totle的值。

在awk编程中，因为命令语句非常长，在输入格式时需要注意以下内容:

l 多个条件 {动作} 可以用空格分割，也可以用回车分割。

l 在一个动作中，如果需要执行多个命令，需要用 “;” 分割，或用回车分割。

l 在awk中，变量的赋值与调用都不需要加入“$”符。

l 条件中判断两个值是否相同，请使用 “==”，以便和变量赋值进行区分。

在看看该如何实现流程控制，假设如果Linux成绩大于90，就是一个好男人(学PHP的表示压力很大!) :

[root@localhost ~]$ awk '{if (NR>=2) {if ($4>60) printf $2 "is a good man!\n"}}’ student.txt

\#程序中有两个if判断，第一个判断行号大于2，第二个判断Linux成绩大于90分

Liming is a good man !

Sc is a good man !

# 1.2.7 awk 函数

awk编程也允许在编程时使用函数，我们讲讲awk的自定义函数。awk函数的定义方法如下:

function 函数名（参数列表）{

函数体

}

我们定义一个简单的函数，使用函数来打印student.txt的学员姓名和平均成绩，应该这样来写函数：

[root@localhost ~]$ awk 'function test(a,b) { printf a "\t" b "\n"}

\#定义函数test，包含两个参数，函数体的内容是输出这两个参数的值

{ test($2,$6) } ' student.txt

\#调用函数test，并向两个参数传递值。

Name Average

AAA 87.66

BBB 85.66

CCC 91.66

# 1.2.8 awk 中调用脚本

对于小的单行程序来说，将脚本作为命令行自变量传递给awk是非常简单的，而对于多行程序就比较难处理。当程序是多行的时候，使用外部脚本是很适合的。首先在外部文件中写好脚本，然后可以使用awk的-f选项，使其读入脚本并且执行。

例如，我们可以先编写一个awk脚本:

[root@localhost ~]$ vi pass.awk

BEGIN {FS=":"}

{ print $1 "\t" $3}

然后可以使用“一f”选项来调用这个脚本:

[root@localhost ~]$ awk -f pass.awk /etc/passwd

rooto

bin1

daemon2

…省略部分输出…

# 1.3 sed 文本选取、替换、删除、新增的命令

sed主要是用来将数据进行选取、替换、删除、新增的命令。

语法:

[root@localhost ~]$ sed [选项] ‘[动作]’ 文件名



选项:

-n: 一般sed命令会把所有数据都输出到屏幕，如果加入此选择，则只会把经过sed命令处理的行输出到屏幕。

-e: 允许对输入数据应用多条sed命令编辑。

-f 脚本文件名: 从sed脚本中读入sed操作。和awk命令的-f非常类似。

-r: 在sed中支持扩展正则表达式。

-i: 用sed的修改结果直接修改读取数据的文件，而不是由屏幕输出



动作:

num a \: 追加，在当前行后添加一行或多行。添加多行时，除最后一行外，每行末尾需要用“\”代表数据未完结。num表示第几行

c \: 行替换，用c后面的字符串替换原数据行，替换多行时，除最后一行外，每行末尾需用“”代表数据未完结。

num i \: 插入，在当期行前插入一行或多行。插入多行时，除最后一行外，每行末尾需要用“”代表数据未完结。num表示第几行

d ; 删除，删除指定的行。

p : 打印，输出指定的行。

s : 字串替换，用一个字符串替换另外一个字符串。格式为“行范围s/"旧字串/新字串/g”（和vim中的替换格式类似）。

对sed命令大家要注意，sed所做的修改并不会直接改变文件的内容（如果是用管道符接收的命令的输出，这种情况连文件都没有），而是把修改结果只显示到屏幕上，除非使用“-i”选项才会直接修改文件。

# 2 字符处理命令

# 2.1 sort 排序命令

[root@localhost~]$ sort [选项] 文件名

选项:



-f: 忽略大小写

-b: 忽略每行前面的空白部分

-n: 以数值型进行排序，默认使用字符串型排序

-r: 反向排序

-u: 删除重复行。就是uniq命令

-t: 指定分隔符，默认是分隔符是制表符

-k n[,m]: ―按照指定的字段范围排序。从第n字段开始，m字段结束（默认到行尾）

案例：

sort命令默认是用每行开头第一个字符来进行排序的，比如:

[root@localhost~]$ sort /etc/passwd

\#排序用户信息文件

如果想要反向排序，请使用“-r”选项:

[root@localhost~]$ sort -r/etc/passwd

\#反向排序

如果想要指定排序的字段，需要使用“-t”选项指定分隔符，并使用“-k”选项指定字段号。加入我想要按照UID字段排序/etc/passwd文件:

[root@localhost~]$ sort -t ":" -k 3,3 /etc/passwd

\#指定分隔符是“:”，用第三字段开头，第三字段结尾排序，就是只用第三字段排序

因为sort默认是按照字符排序，前面用户的UID的第一个字符都是1，所以这么排序。要想按照数字排序，请使用“-n”选项:

[root@localhost~]$ sort -n -t ":" -k 3,3 /etc/passwd

当然“-k”选项可以直接使用“-k 3”，代表从第三字段到行尾都排序（第一个字符先排序，如果一致，第二个字符再排序，知道行尾）。

# 2.2 uniq 取消重复行

uniq [-c/d/D/u/i] [-f Fields] [-s N] [-w N] [InFile] [OutFile]

参数解释：

-c: 在每列旁边显示该行重复出现的次数。

-d: 仅显示重复出现的行列，显示一行。

-D: 显示所有重复出现的行列，有几行显示几行。

-u: 仅显示出一次的行列

-i: 忽略大小写字符的不同

-f Fields: 忽略比较指定的列数。

-s N: 忽略比较前面的N个字符。

-w N: 对每行第N个字符以后的内容不作比较。

\# uniq.txt



My name is Delav

My name is Delav

My name is Delav

I'm learning Java

I'm learning Java

I'm learning Java

who am i

Who am i

Python is so simple

My name is Delav

That's good

That's good

And studying Golang

直接去重

uniq uniq.txt

结果为：

My name is Delav

I'm learning Java

who am i

Who am i

Python is so simple

My name is Delav

That's good

And studying Golang

显示重复出现的次数

uniq -c uniq.txt 、

结果为：

3 My name is Delav

3 I'm learning Java

1 who am i

1 Who am i

1 Python is so simple

1 My name is Delav

2 That's good

1 And studying Golang

你会发现，上面有两行 ”My name is Delav ” 是相同的。也就是说，当重复的行不相邻时，uniq 命令是不起作用的。所以，经常需要跟

sort 命令一起使用。

sort uniq.txt | uniq -c

结果为：

1 And studying Golang

3 I'm learning Java

4 My name is Delav

1 Python is so simple

2 That's good

1 who am i

1 Who am i

只显示重复的行，并显示重复次数

uniq -cd uniq.txt

结果为：

3 My name is Delav

3 I'm learning Java

2 That's good

显示所有重复的行，不能与 -c 一起使用

uniq -D uniq.txt

My name is Delav

My name is Delav

My name is Delav

I'm learning Java

I'm learning Java

I'm learning Java

That's good

That's good

忽略第几列字符

下面这里 -f 1 忽略了第一列字符，所以"who am i" 和 "Who am i" 判定为重复

uniq -c -f 1 uniq.txt

结果为：

3 My name is Delav

3 I'm learning Java

2 who am i

1 Python is so simple

1 My name is Delav

2 That's good

1 And studying Golang

忽略大小写

下面这里 -i 忽略了大小写，所以"who am i" 和 "Who am i" 判定为重复

uniq -c -i uniq.txt

结果为：

3 My name is Delav

3 I'm learning Java

2 who am i

1 Python is so simple

1 My name is Delav

2 That's good

1 And studying Golang

忽略前面N个字符

下面这里 -s 4 表示忽略前面四个字符，所以"who am i" 和 "Who am i" 判定为重复

uniq -c -s 4 uniq.txt

结果为：

3 My name is Delav

3 I'm learning Java

2 who am i

1 Python is so simple

1 My name is Delav

2 That's good

1 And studying Golang

忽略第N个字符后的内容

uniq -c -w 2 uniq.txt

# 2.3 wc 统计命令

[root@localhost~]$ wc [选项] 文件名

选项：

-l：只统计行数

-w：只统计单词数

-m：只统计字符数