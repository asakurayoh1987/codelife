# Linux学习

## Linux目录结构规范

- /bin - 基本命令二进制文件
- /sbin - 基本的系统二进制文件，通常是 root 运行的
- /dev - 设备文件，通常是硬件设备接口文件
- /etc - 主机特定的系统配置文件
- /home - 系统用户的家目录
- /lib - 系统软件通用库
- /opt - 可选的应用软件
- /sys - 包含系统的信息和配置(第一堂课介绍的)
- /tmp - 临时文件( /var/tmp ) 通常在重启之间删除
- /usr/ - 只读的用户数据
- /usr/bin - 非必须的命令二进制文件
- /usr/sbin - 非必须的系统二进制文件，通常是由 root 运行的
- /usr/local/bin - 用户编译程序的二进制文件
- /var -变量文件 像日志或缓存

## 登录远程主机

```bash
# 格式为：ssh 用户名@远程主机的公网ip
ssh ycyu@xxx.xxx.xxx.xxx

# 如果本地的用户名与远程主机的用户名相同，可以省略
ssh xxx.xxx.xxx.xxx
```

## 关于免密登录

- 本地生效密钥对，并将公钥拷贝至远程主机

```bash
# 以rsa的加密方式生成密钥对
ssh-keygen -t rsa -C "your_email@example.com"

# 访问~/.ssh文件夹查看密钥对，以.pub结尾的是公钥
cat ~/.ssh/id_rsa.pub

# 复制公钥，添加到远程主机的~/.ssh/authorized_keys中
```

- 使用ssh-copy-id

```bash
ssh-copy-id 用户名@IP地址
```

## 修改hosts

比如上一步中的远程服务器的公网ip不容易记住，那可以指定一个host映射，操作如下

```bash
sudo vim /etc/hosts
# 添加如下，格式为：ip地址 别名1 别名2
xxx.xxx.xxx.xxx aliyun-ycyu aliyun
```

## 修改登录超时

```bash
# 查看是否有超时限制，比如得到的数值为900，即为900s不操作时退出登录
echo $TMTIMOUT

# 修改/etc/profile中的变量TMOUT的值为空：TMOUT=
vim /etc/profile
```



## alias 添加别名

```bash
# 直接使用alias命令可以查看所有的别名
alias

# 创建别名
alias tails="tail -f"
```

直接使用alias只在当前会话生效，如果希望每次进入系统时都有效，需要将其添加到系统的环境配置中，以zsh为例则需要将上述的别名配置添加到"~/.zshrc"中

## 关于vim

### 进入插入模式

- i：光标所在位置进行插入
- I：光标所在行首进行插入
- o：光标所在位置的下一列进行插入
- O：光标所在位置的下一行进行插入
- a：光标所在位置的后一位置进行插入
- A：光标所在行尾进行插入

### 命令模式

- set paste：粘贴但忽略格式
- vsplit：屏幕左右拆分，此时使用ctrl+w来切换窗口，此时使用`q!`则会仅关闭当前窗口
- split：屏幕上下拆分

### 普通模式

- u：撤回
- ctrl+R：重做
- x：删除单个字母
- dd：删除单行
- yy：复制当前行
- p：粘贴

### 可视块模式

ctrl+v进行可视块模式

### 关于vim的使用

- vimtutor
- vimtutor zh
- ctags：ctrl+]，可以查看某个c/c++语言下函数的定义，查看完按住ctrl+T则可以返回
- 在man手册中查看某个函数的描述，按住K，即可，如果想在man手册的第3章中查找，则是按住3K
- ctrl+z：挂起，可以使用fg再将其切回前台
## 命令的学习

### man 手册

```bash
man pwd
```

man分为很多册，1-8册，可以通过`man -f 命令`查找是在哪一册中

```bash
# 查找属于哪个手册
man -f printf

# 查找后发现printf在第1册与第3册中，可以使用如下在指定册中查看
man 3 printf

# 按关键字查找
man -k printf
```

在查看man手册时，可以使用之前提到的方向键以及hjkl四个键来上下移动，也可以使用b与f来向后或向前翻页

### tldr

man手册太长了，所有有了tldr，不过它是需要额外安装的，安装后使用如下

```bash
tldr passwd
```

### which

查找命令的位置

```bash
which alias
```

### !!

双感叹号直接输出上一次执行的命令，比如你在操作vim file时，发现该文件是需要管理权限的，此时使用`sudo !!`，其效果与`sudo vim file`一样

### chmod、chown、chgrp

- chmod 更改文件权限

```bash
# 给file文件的ugo都赋予执行的权限，ugo=user group other
chmod a+x file
# 将file文件的other用户减去执行权限
chmod o-x file
# 设置file文件的权限为rwxr-xr-x
chmod 755 file
# 操作同上，只是换一种写法
chmod u=rwx,go=rx file
```

- chown	更改文件所属用户

```bash
# 修改file的所属用户为ycyu,所属组为root
chown ycyu:root file
# 同上,只不过递归修改此文件夹下的所有文件
chown -R ycyu:root file
# 修改文件file所属用户为ycyu
chown ycyu file 
```

- chgrp 更改文件所属组

```bash
# 修改文件file的所属用户组
chgrp root file
```

## zsh及常用命令

### 安装与配置

```bash
# 安装
sudo apt update
sudo apt install zsh
# 修改默认shell解释器
chsh -s /bin/zsh
```

### d命令

查看最近访问的文件夹

### 操作

- 向后移动一个字符：ctrl+b
- 向前移动一个字符：ctrl+f
- 移动至行首：ctrl+a
- 移动至行尾：ctrl+e
- 向后删除光标所在字符：ctrl+d
- 删除至行尾：ctrl+k
- 向前删除一个单词：ctrl+w
- 向后删除一个单词：alt+d
- 清空整行：ctrl+u
- 粘贴：ctrl+y
- 撤回：ctrl+shift+_
- 交互当前光标处单词与前一个单词：ctrl+xt
- 将当前命令移入缓存并清空当前行：ctrl+q
- 将当前命令通过默认编辑器来进行编辑：ctrl+xe （编辑后通过wq来保存则会在命令行生效）
- 文件递归查找：`ls **/a`
- 文件类型别名：alias -s，比如：alias -s c=vim，表示默认使用vim打开c文件，设置之后可以直接输入`a.c`，就会使用vim打开该文件了

[更多](https://itnext.io/the-zsh-shell-tricks-i-wish-id-known-earlier-ae99e91c53c2)

### 任务管理

- &：在命令之后添加此符号，表示在后台执行该命令
- fg：将上述方法切至后台的命令切换至前台
- 分号：加在命令中间表示顺序执行
- &&：前一条命令执行成功，再执行后一条命令
- ||：前一条命令执行失败，再执行后一条命令
- ``：括起来来的命令，表示此条命令先执
- ctrl+z：命令挂起
- bg：将挂起的命令后台执行
- fg：将挂起的命令或后台执行的命令变为前台执行
- jobs：查看后台执行或挂起的任务及编号，列表中的“+”表示使用fg命令时会将哪个任务切到前台

### 重定向

- \>：命令到文件的重定向
- \>>：命令到文件的追加
- <：文件到命令的重定向
- <<： 结束符号，比如`./a.out << EOF`，此时会一直接受输入，直到输入EOF，也可将EOF改为别的字符，比如“end”，这样当输入为”end“时，则结束输入

### 管道相关

- \｜：将管道符号左边命令的标准输出作为管道右边命令的标准输入
- xargs：参数替换
- last：查看最近登录的用户记录信息

### 转义

- \\：单个字符的转义
- ''：其内的字符串进行完整的转义，比如echo '${a}'，可以直接打印出：${a}，而不是打印出变量a的值
- ""：软转义，允许出现特定的shell元字符
- $：变量值替换
- ``：命令替换

### 系统信息

- uptime：打印系统运行时长和平均负载
- w：当前用户列表以及正在执行的任务
- who：显示当前登录系统的用户信息
- whoami：打印当前有效的用户名称
- last：显示用户最近登录信息
- uname：打印当前系统信息
- date：显示或设置系统时间与日期
- cal：显示日历

### 文件与目录

- basename：取文件名
- dirname：取目录名
- ls：查看目录
	- A 去除列表下的如"."及".."，这种在shell脚本中来遍历文件列表时很有用，不然会无限递归
	- a 显示所有文件名
	- l 长列表格式
	- h 人类可读的方式
	- i 显示文件的inode
	- n 显示uid、gid
- cp 拷贝文件
	- i 若文件存在，则询问用户
	- r 递归复制
	- a pdr的集合
	- p 连同文件属性一起拷贝
	- d 若源文件为连接文件的属性，则复制连接文件属性
	- s 拷贝为软连接
	- l 拷贝为硬连接
	- u 源文件比目的文件新才拷贝

### 文件内容查阅

- nl 输出行号显示文件内容
	- -b 编号方式，比如 -b a表示所有行编号，空行也编号
	- -w 指定编号的宽度，不足长左边补0
- od 以二进制方式查看文件内容
- cat 正向连续读
	- A 相当于-vET 
	- v 列出看不出的字符
	- E 显示换行符为$
	- T 显示TAB为^I
	- b 列出行号
	- n 列出行号，边同空行的行号
- tac 反向连续读
- less 与more相似，但比more厉害
	- /string 向下查找string关键字
	- ?string 向上查找string关键字
	- n 继续向下查找
	- N 继续反向查找
	- q 退出
- more 一页一页的显示
	- /string 向下查找string关键字
	- :f 显示文件名称和当前显示的行数
	- q 离开
	- ? 查看其他命令
- head 只看头几行
	- n num 显示前num行
	- -n num 除了后num行，其他的都显示
- tailf 只看尾几行
	- n num 显示文件的后num行
	- -n num 显示从num行开始到结尾

### 查看文件与目录

- stat file.c 查看文件信息，包括访问时间（a-查看文件内容的时间）、修改时间（m-修改文件内容的时间）、变更时间（c-修改文件属性，比如所属用户与组）
- dt -T 查看文件系统
- mount 挂载
- touch 如果文件不存在 ，则创建，如果存在，则会同时修改acm
	- -a 仅修改访问时间
	- -c 仅修改文件的时间，若文件不存在，不新建
	- -d 修改文件日期
	- -m 仅修改文件的修改时间
	- -r 将第一个文件的时间应用到第二个文件上
	- -t 修改文件时间[yymmddhhmm]
- lsattr 查看文件隐藏属性
- chattr 修改文件属性

### 文件的特殊权限

通常使用`chmod 777`是修改文件的读、写、执行的权限，其中数字从左到右表示修改的是当前用户（u）、当前组（g）、其他用户（o）的权限，但还存在一类特殊权限，可以在这三个数字之前再加一位数字表示，数字的含义如下：

- set_uid(S=4)：作用于二进制程序文件，非脚本，表示用户在执行该程序时获得程序所有者的权限
- set_gid(S=2)：作用于目录和二进制程序文件，表示用户在该目录里，有效级变为目录所属组
- sticky bit(1)：作用于目录，表示在该目录下，用户只能删除自己创建的文件

除了数字也可使用字母s和t，表示时，是占用在原'x'的那一位，如果原来具体执行权限，则新的字母表示为小写，否则为大写

### 文件的定位与查找

- which 寻找可执行文件
- whereis 寻找特定文件，如果是命令，还可以查找到man手册的位置
- locate 搜索文件（可部分查找），依赖于索引，首次使用时会创建索引，updatedb会更新索引
- find 多样化高级查找

例子：统计指定类型的脚本文本的行数

```bash
find . -name "*.c" -o -name "*.cpp" -o -name "*.h" -o -name "*.sh" | xargs cat | wc -l
```

### 用户管理

#### 用户管理相关的文件

- /etc/passwd 用户名：密码占位：用户编号：归属组编号：用户信息：家目录：shell路径
- /etc/shadow 用户名：密码占位：最后修改密码的日期：密码不可改动日期：密码需要重新修改的日期：密码变更期限前警告日期：密码过期宽限时间：账号失效日期：保留
- /etc/group 组名：密码占位：组编号：组内成员
- /etc/gshadow 组名：密码占位：组管理员：组内成员
- /etc/sudoers 用户名：权限定义：权限（sudo）

- date -d “1970-01-01 18733 day” 计算1970年之后的18733天是哪个日期

#### 用户管理相关的命令

- su 切换用户
	su [-lmpfc] `<username>`
	- -|-l 重新登录，比如修改home目录
	- -m| -p 不更改环境变量
	- -c command：切换后执行命令，并退出
- sudo 临时切换为root用户
- passwd 设定用户密码
- gpasswd 设定组密码
- chsh 修改用户shell
- usermod 修改用户账号
- useradd 新建用户
- userdel 删除用户
- id 显示用户信息

#### 进程管理相关的命令

- free 打印系统情况和内存情况
	- free [-bkmgotsh]
	- -b|k|m|g 以字节、kb、m、g单位显示
	- -o 忽略缓冲区调节列
	- -s seconds 每隔seconds秒刷新一次
	- -t 求和（total）
	- -h 以可读形式显示
- top 显示当前系统进程情况，内存，cpu等信息
	- top [-bcdsSupnq]
	- -b 以批处理模式操作
	- -c 显示完整的命令
	- -d seconds 屏幕刷新间隔时间
	- -s 以安全模式运行
	- -S 以累积模式运行
	- -u uname 指定username
	- -p pid 指定pid
	- n nums 循环显示次数
	- -q root时，给尽可能高的优先级 
- htop （需要安装）top的增强版
- dstat 系统资源信息统计工具，实时监控磁盘、cpu、网络
- ps 报告当前进程状态
	- -aux 进程的具体状态
	- -ef
- pstree 树状显示进程派生关系
	- p 显示进程号
	- a 显示每个程序的完整指令
	- u 显示用户名
	- n 按pid排序
	- l 使用长列格式显示树状
- pgrep 查找进程ID
	- -o 起始进程号
	- -n 结束进程号
	- -l 显示进程名
	- -P pid 指定父进程
	- -g gid 指定进程组
	- -t tty 指定开启的进程终端
	- -u uid 指定uid
- kill 删除执行中的进程和工作
	- kill [-alpsu] `<pid>`
	- -a 处理当前进程时，不限制命令名和进程号的对应关系
	- -l 信息号id 不加信号id，则列出全部信息号
	- -p pid 给pid的进程只打印相关的进程号，而不发送任何信息
	- -s 信号id｜信号name 指定要发出的信号
	- u 指定用户
- pkill 批量按照进程名杀死进程

### 数据处理相关

- cut 切分
- grep 检索
- sort 排序
- wc 统计字符、字数、行数
- uniq 去重
- tee 双向重定向
- split 文件切分
- xargs 参数代换
- tr 替换、压缩、删除

#### cut

- -d c：以字符c进行分割
- -f num：分割后显示num指定段的内容，格式为[n-; n-m; -m; m,n]
- -b num：按字节进行分割，一个字符就是一个字节
- -c num：按字符进行分割

```bash
echo ${PATH} | cur -d : -f 1
echo ${PATH} | cur -c 1
```

#### grep

- -c：统计搜寻到的条目数（行数）
- -i：忽略大小写
- -n：顺序输出行号
- -v：反向输出（输出没找到的
- -w：匹配整个单词，而不是单词的一部分

```bash
last | grep -w ycyu
last | grep -w ycyu -c
last | grep -v ycyu
```

#### sort

- -f：忽略大小写
- -M：以月份名称排序，月份的三字母缩写的形式
- -n：根据数值进行排序
- -r：反向排序
- -u：uniq
- -c：检查文件是否有序
- -t：分隔字符，指定排序时用的栏位分隔字符
- -k：以分隔后的哪个区间排序

```bash
cat /etc/passwd | sort
# 检查ordered.txt中的内容是否有序
sort -c ordered.txt
# 对文件sort.txt进行排序，排序的字段为以冒号分隔后的第3列
cat sort.txt | sort -t : -k 3
# 对字段排序
cat sort.txt | sort -t : -k 3 
```

#### wc

- -l：仅列出行号
- -w：仅列出多少字
- -m：列出多少字符
- -c：列出多少字节
- -L：列出最长一行的字符长度

#### uniq

- -i：忽略大小写字符的不同
- -c：进行计数
- -u：只输出无重复的行

#### tee

- -a：append追加

```bash
ping www.baidu.com | tee output.txt
```

#### split

- -b SIZE：切分为SIZE bytes大小的文件
- -C SIZE：切分为SIZE bytes大小的文件，不断开一行
- -l num：以num行为大小进行切分

```bash
# 按每个文件5行进行切分，文件默认名为xaa,xab,xac...
split -l 5 output.txt
# 自定义生成文件的前缀，生成的文件就是_123aa,_123ab,_123ac
split -l 5 output.txt _123
# 按字节进行切分，这种切分会将原来完整的一行切分，影响可读
split -b 2000 output.txt
# 同上，但不会破坏行
split -C 2000 output.txt
```

#### xargs

- -p：执行前进行询问
- -n num：每次读入参数的个数
- -e xxx：读入xxx参数为止

```bash
# 原来的目的是用cat来查看ls.txt，但实际输出只是ls.txt字符串，而不是对应文件中的内容
echo "ls.txt" | cat
# 应使用这种方式
echo "ls.txt" | xargs cat
# 询问模式
echo "ls.txt" | xargs -p cat
# 指定每次传入的参数的个数，否则id会一次拿到很多的参数，从而报错
cat /etc/passwd | cut -d : -f 1 | xargs -n 1 id
# 每次读入1个参数，直到读到参数为games时截止
cat /etc/passwd | cut -d : -f 1 | xargs -n 1 -e games id
```

#### tr

- -c 取代所有不属于第一个字符集的字符
- -d 删除所有不属于第一字符集的字符
- -s 将连续重复的字符以单独一个字符表示

```bash
a="1122abab"
# 122abab
echo ${a} | tr -s 1
# 112abab
echo ${a} | tr -s 2
# 12abab
echo ${a} | tr -s 12
# 1122bb
echo ${a} | tr -d a
# 0000aa00
echo ${a} | tr -c a 0
# 将小写字母都替换成大写
echo ${a} | tr "[:lower:]" "[:upper:]"
```

#### bc

交互式的数学操作界面

```bash
# 求下列字符中数字之和
a="1 2 3 4 5 6 7 8 9 a v fdaf . /8"
echo ${a} | tr -c "[:digit:]" " " | tr -s " " | tr " " + | cut -c 1-19 | bc
```

```bash
echo ${PATH} | tr : "\n" | tail -n 1
```

### Linux三剑客

#### 正则表达式

- 转义字符要用单引号括起来
- [[:upper:]]，这种形式表示描述一类的词，比如所有的小写字母，这种形式称为POSIX字符集
	- [:alnum:]：所有字母和数字，等价于[0-9a-zA-Z]
	- [:digit:]：所有数字，等价于[0-9]
	- [:alpha:]：所有字母，等价于[a-zA-Z]
	- [:upper:]和[:lower:]，所有的大写字母，与小写字符，分别等价于[a-z]和[A-Z]
	- [:blank:]：空白
	- [:graph:]：非空字符
	- [:punct:]：标点符号
- [^[:upper:]]，除了之种形式的词以外
- ^$匹配空行
- \b 边界，比如grep "\b1"以1开头的，或 grep "he\b"，he后是一个边界的

#### grep

- -E 扩展正则表达式
	- +：匹配一个或多个
	- \*：匹配零个或多个
	- ？：匹配零个或1个
	- {2,4}：重复匹配的次数

- -P Perl正则表达式
	- 匹配空白字符：\f \v \n \r \t
	- 匹配特定字符（大写表示取反）：
		- 数字元字符：\d \D
		- 包括下划线在内的单字符：\w \W
		- 空白字符：\s \S
		- 匹配16进制或8进制：\x \0

#### sed

- 定义：非交互式文本编辑器
- 适用场合：
	- 非常大的文本文件，使用交互式文本编辑器操作非常慢
	- 编辑命令比较复杂，在普通编辑器中难以完成
	- 扫描一个大文件，并且需要经过一系列的操作

- 语法：sed [-nefri] command file
	- -n：安静模式，只输出处理的部分
	- -e：不编辑源文件，默认选项
	- -f：执行一个sh文件，批量操作
	- -r：支持正则
	- -i：直接修改文件

- command
	- a：追加
	- c：替换行
	- d：删除
	- i：插入
	- p：打印，这里加上`-n`，表示只打印处理的行
		~：表示步长，比如`sed -n '0~2p'`，打印偶数行
		+：表示连续几行，比如`sed -n '1,+3p'`打印第1行开始以及接下来的3行
	- s：替换字符，比如`sed 's/xxx/yyy/g'`全局将xxx替换成yyy

示例：

```bash
# 在第一行后追加内容“hello”，也就是增加了第二行，内容为“hello”
sed '1a hello'
# 将第一行与第二行替换为“hello”
sed '1,2c hello'
# 删除第1行
sed '1d'
# 删除最后一行
sed '$d'
# 删除从第2行到最后一行
sed '2,$d',
```

#### awk

- 语法：awk [-Ffv] 'BEGIN {commands} pattern {commands} END {commands}' file
- 选项：
	- -F fs：指定输入分隔符，fs可以是字符串或正则表达式
	- v var=value：赋值一个用户定义的变量，将外部变量传递给awk
	- -f scriptfile：从脚本文件中读取awk命令

第一步：执行BEGIN{commands}语句块中的语句
第二步：从文件或stdin中读取一行，执行pattern{commands}，逐行扫描文件，从第一行到最后一行重复这个过程，直到读完
第三步：当读至输入流末尾时，执行END{commands}语句块

### Shell编程

#### 注意事项

- 变量赋值时"="前后不得有空格
- 变量默认是全局变量，可以使用local声明局部变量
- 将一个字符串赋值给变量时，如果字符串中有空格，将字符串用双引号或单引号括起来

#### 特殊变量

- $0：获取当前执行shell脚本的文件名，包括路径
- $n：获取当前执行脚本的第n个参数，n=1...9，如果n大于9则需要将n使用大括号括起来
- $\*：获取当前执行的shell的所有参数，将所有命令行参数视为单个字符串，相当于"$1$2$3"
- $#：获得当前执行shell的参数个数
- $@：获取当前执行shell的所有参数，并保留参数之间的任何空白，相当于"$1" "$2" "$3"，这是将参数传给其他程序的最好办法
- $?：判断上一条指令是否执行成功，0表示成功，非零表示不成功
- $$：获取当前进程的PID
- $!：获取上一个指令的PID

#### 数组

- 下标从0开始，zsh默认从1开始
- 从某种意义上，变量都是数组
- 未指定下标从最大下标开始新增

##### 索引数组

grammer：declare -a arr
name[num]=value
name=(value1 value2 ...)

```bash
declare -a arr
arr[0]=10
arr[1]=20
# 输出10
echo ${arr[0]}
# 批量赋值
arr=([2]=g [3]=h)
# 输出h
echo ${arr[3]}
# 输出10 20
echo ${arr[@]}
```

##### 关联数组

grammer：declare -A arr
name[descriptor]=value

```bash
declare -A arr
arr[apple]=10
arr[banana]=20
# 输出10
echo ${arr[apple]}
```

##### 数组的操作

- 输出数组的内容：${arr[\*]} ${arr[@]}
- 确定数组元素的个数：${#arr[@]}
- 找到数组的下标：${!arr[@]}
- 数组追加：arr+=(a b c)
- 数组排序：sort
- 删除数组元素：unset，比如unset arr[a]

#### 输入输出

##### read

grammer：read \[-options\] \[variable...\]

- -a array：把输入赋值到数组array中，从索引零开始
- -d delimiter：字符串delimiter中的第一个字符指示输入结束，而不是一个换行符
- -e：使用readline来处理输入。这使得与命令行相同的方式编辑输入
- -n num：读取num个输入字符，而不是整行
- -p prompt：为输入显示提示信息，使用字符串prompt
- -r：Raw mode，不把反斜杠字符解释为转义字符
- -s：Silent mode，类似于登录时输入密码时，不将输入展示出来
- -t seconds：超时
- u fd：使用文件描述符fd中的输入，而不是标准输入

##### echo

grammar：echo string

```bash
echo -e "Hello KKB\n"
echo "Hello $name, This is KKB"
echo "\"Hello KKB\""
```

##### printf

grammar：printf format string [arguments...]

```bash
printf "hello world %s\\n" "KKB"
```

##### 函数

声明：

1. 方式一

```bash
function _printf_ {
	echo $1
	return
}
```

2. 方式二

```bash
_printf_() {
	echo $1
	return
}
```

3. 方式三

```bash
function _printf_() {
	echo $1
	return
}
```

调用：

```bash
_printf_ "Hello Kaikeba"
```

**bash -x可以进行脚本的调试**

#### 流程控制

##### IF

if [[ condition ]]; then
	#statements
elif [[ condition ]]; then
	#statements
else
	#statements
fi

关于这里的条件，使用的是test表达式，可以通过man test来查看相关的说明，可以通过test命令来检查条件表达式是否成立

```bash
if test "abc" = "abc" ; then
	echo "equal"
else
	echo "not equal"
fi
```

##### FOR

for ((i=0;i<$a;i++)); do
	#statments
done

for i in words; do
	#statments
done

```bash
# 也可以是`for i in {1..100}`
for i in `seq 1 100`; do
	echo ${i}
done
```

```bash
function sum(){
	ans=0
	for ((i=1; i <= $1; i++)); do
		ans=$[${ans}+${i}]
	done
	echo ans=${ans}
}
```

##### WHILE

while [[ condition ]]; do
	#statments
done

##### UNTIL

until [[ condition ]]; do
	#statments
done

##### CASE

case word in
	pattern1 )
		;;
	pattern2 )
		;;
esac

```bash
function test() {
	case $1 in
		1 )
			echo "Jan"
			;;
		2 )
			echo "Feb"
			;;
		3 )
			echo "March"
			;;
	esca
}
```

```bash
#!/bin/bash
read year month

# 计算某年某月有多少天
function run_year(){
	if [[ ($[$year%4] = 0 && $[$year%100] != 0) || $[$year%400] = 0 ]];
	then
		return 1
	else
		return 0
	fi
}
declare -a days
days=(31 28 31 30 31 30 31 31 30 31 30 31)

run_year

if [[ $? = 1 && $month = 2 ]];
    then
	    days[1]=29
fi

echo ${days[$month-1]}
```

