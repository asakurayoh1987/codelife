# Linux常用命令

## cheat list

* curl cht.sh

## netstat

```bash
netstat -atpn
```

* n - 以数字的形式展示地址，比如localhost会转成127.0.0.1
* a - 显示所有的链接
* t - tcp协议
* p - 进程号
* l - 显示监听中的socket

## lsof

```bash
lsof -n -P -i:22
```

- n - 同netstat，不将ip转换为hostname
- P - 不将port number转换为service name
- i - 查看指定端口号的进程



## tcpdump

```
tcpdump -nn -i eth0 port 80
tcpdump -i any port 8080 -nn -w capture.pkt
```
* nn - 以数字形式展示
* i - 指定抓取的网卡
* port - 指定端口
* w - 抓包结果写入文件

## man

帮助文档

## strace

查询系统调用

```bash
strace -ff -o ./trace.txt	需要执行的命令
# 指定参数“-T”则以微秒级的精度去采集各种系统调用所消耗的实际时间
# 指定参数“-tt”能以微秒为单位来显示处理发生的时刻
```

## sar

获取进程分别在用户模式和内核模式下运行的时间比例

```bash
# 第三个参数1用来指定采集信息的周期，第四个参数用来采集信息的次数
sar -P ALL 1 1

# 每秒采集一次内存使用情况
sar -r 1

# 每秒采集一次中断情况，比如fault/s表示缺页中断发生
sar -B 1

# 查看系统设备上的I/O处理情况，比如读写速度（单位是扇区，512字节）
sar -d -p 1
```

## ldd

查看程序所依赖的库

```bash
ldd /bin/echo
# 查看glibc版本
ldd --version
```



## ulimit

查看或设置系统的一些限制，比如可打开的文件描述数的个数

## nc

监听或建立链接的工具

```bash
# 监听
nc -l 5556
# 连接
nc 10.0.2.107 5556
```

## 查看系统信息

```bash
# 显示 Linux 系统架构
uname -a
uname -r
# 查看操作系统是32位还是64位
dpkg --print-architecture
# 显示操作系统架构类型
arch
# 查看系统发行版本信息
lsb_release -a
cat /etc/redhat-release
```

## scp

```bash
scp -r ~/myproject/mock/caserepo root@172.31.4.36:/home/ycyu/web/mock/
```

## iptables

```bash
# 查询raw表的所有链上的规则信息，raw还可替换为mangle、filter、nat
iptables -t raw -L

# 查询指定表指定链上的规则信息
iptables -t raw -L OUTPUT
```

## ip rule

* Usage: ip rule [ list | add | del ] SELECTOR ACTION （add 添加；del 删除； llist 列表）
* SELECTOR := [ from PREFIX 数据包源地址，支持CIDR格式] [ to PREFIX 数据包目的地址，支持CIDR格式] [ tos TOS 服务类型][ dev STRING 物理接口] [ pref NUMBER ] [fwmark MARK iptables 标签]
* ACTION := [ table TABLE_ID 指定所使用的路由表] [ nat ADDRESS 网络地址转换][ prohibit 丢弃该表| reject 拒绝该包| unreachable 丢弃该包]
* [ flowid CLASSID ]
* TABLE_ID := [ local | main | default | new | NUMBER ]

> CIDR是英文“Classless Inter-Domain Routing”的简写，中文名称“无类别域间路由”，是网络中分配IP地址的一种方法。

* example

```bash
# 通过路由表 inr.ruhep 路由来自源地址为192.203.80/24的数据包
ip rule add from 192.203.80/24 table inr.ruhep prio 220

# 把源地址为193.233.7.83的数据报的源地址转换为192.203.80.144，并通过表1进行路由
ip rule add from 193.233.7.83 nat 192.203.80.144 table 1 prio 320
```

* linux启动时会默认添加三条缺省的规则，可以通过`ip rule list`来查看：

```bash
# 匹配任何条件 查询路由表local(ID 255) 路由表local是一个特殊的路由表，包含对于本地和广播地址的高优先级控制路由。rule 0非常特殊，不能被删除或者覆盖
0:      from all lookup local
# 匹配任何条件 查询路由表main(ID 254) 路由表main(ID 254)是一个通常的表，包含所有的无策略路由。系统管理员可以删除或者使用另外的规则覆盖这条规则
32766:  from all lookup main
# 匹配任何条件 查询路由表default(ID 253) 路由表default(ID 253)是一个空表，它是为一些后续处理保留的。对于前面的缺省策略没有匹配到的数据包，系统使用这个策略进行处理。这个规则也可以删除
32767:  from all lookup default
```

## ip route

linux 系统中，可以自定义从 1－252个路由表，其中，linux系统维护了4个路由表：

* 0#表： 系统保留表
* 253#表： defulte table 没特别指定的默认路由都放在改表
* 254#表： main table 没指明路由表的所有路由放在该表
* 255#表： locale table 保存本地接口地址，广播地址、NAT地址 由系统维护，用户不得更改

查看路由表信息：

* ip route list table <table_number>
* ip route list table <table_name>

table_name与table_number的对应关系在/etc/iproute2/rt_tables中

## du -sh

查看文件或目录大小

参数解释：

* a：列出所有的文件与目录容量，因为默认仅统计目录的容量而已
* h：以人们较易读的容量格式呈现(G/M/K)显示，自动选择显示的单位大小
* s：列出总量而已，而不列出每个个别的目录占用容量
* k：以KB为单位进行显示
* m：以MB为单位进行显示常用命令参考 查看当前目录大小[plain] du -sh ./

## id 用户名

查看用户所在的组

```bash
id ycyu
# 配合whoami可以查看当前的用户名
id `whoami`
```
## grep

```bash
 grep -REn "免费|限时领|立即领取|0元" GitProject/  --exclude-dir="h5-pocket" --exclude-dir="node_modules" --exclude-dir="dist" --exclude-dir="public" --exclude-dir=".history"  --exclude-dir="test"  --exclude="*.md"
```

## find

```bash
# 当前目录下查找文件夹名称中包括"Ember"，-name指定要查找的内容，-i指定不区分大小写，-type指定为d表示查找的是文件夹而不是文件
find ./ -iname "*Ember*" -type d
```

## time

输出后续操作执行耗时

```bash
# a.out是一个c编译后的可执行文件
time ./a.out
```

## curl

```bash
# 以指定ip商品解析指定地址
curl 'https://hw.diyring.cc' -I --resolve hw.diyring.cc:443:183.131.124.79
```

## 关于IFS

用于控制输出时的换行处理，默认空格就会导致换行，比如coder.txt中的内容如下：
Twinkle, twinkle, little star,
How I wonder what you are.
直接使用for语句和echo打印时，遇到空格就会换行，需要像下面这样：

```bash
IFS=$'\n'
for i in `cat coder.txt`; do echo "$i"; done
unset IFS
```

## 查看系统内核版本

```bash
# 查看内核版本
cat /proc/version
uname -a
uname -r
# 查看发行版本
lsb_release -a
cat /etc/issue
# 查看是64位还是32位
getconf LONG_BIT
# 查看系统架构
arch
# 查看比较全面的信息
hostnamectl
# centos下
cat /etc/redhat-release
```

## 查看系统内核的glibc版本

```bash
ldd --version
```

## CentOS7 相关

```bash
# 解决yum install时提示没有可用的包，比如默认安装autojump-zsh提示没有包，通过以下命令使用epel并更新repo后就可以安装了
yum -y install epel-release
yum repolist

# 安装包
yum -y install devtoolset-11-gcc-plugin-devel
# 查看可用的包
yum list available devtoolset-11-\*
# 删除包
yum remove devtoolset-8-\*
# 查看可用的开发
yum grouplist | grep development
```

## 时间及时区设置

```bash
# 通过ntp同步网络时间
yum install ntp
ntpdate pool.ntp.org
```

之后再通过`date`来查看系统时间即为当前时区的时间了

单独修改时区：

```bash
# 查看时区
timedatectl list-timesones
# 设置时区
timedatectl set-timezone Asia/Shanghai
```

## 查看域名解析信息

```bash
nslookup depend.iflytek.com
# 还可以查看本地配置的域名解析服务
cat /etc/resolv.conf
```



## ss 查看全连接队列大小和当前等待accept的连接个数

```bash
# 对于 LISTEN 状态的套接字，Recv-Q 表示 accept 队列排队的连接个数，Send-Q 表示全连接队列（也就是 accept 队列）的总大小
ss -lnt
```

![image-20221019090332904](../media/image-20221019090332904.png)

## 查看或修改主机名

```bash
# 查看
hostname
hostnamectl
# 永久修改 centos
hostnamectl set-hostname c2
# 临时修改，重启后失效
hostname c2
```

##  crontab

- -e：编辑该用户的计时器设置
- -l：列出该用户的计时器设置
- -r：删除该用户的计时器设置
- -u<用户名称>：指定要设定计时器的用户名称

格式：

minute   hour   day   month   week   command     顺序：分 时 日 月 周

添加计时器：

```bash
crontab -e
# 接下来添加一条，每小时执行一次syncwork
* */1 * * * syncwork
```

[更多内容](https://wangchujiang.com/linux-command/c/crontab.html)

## strings

用于在包中查找字符串

```bash
# 查看glibc版本
ldd --version
strings /lib64/libc.so.6 | grep GLIBC

```

## ldconfig

动态链接库相关

```bash
# 更新动态链接库
ldconfig
# 查看动态链接库版本信息
ldconfig -v
```

## taskset

```bash
# 指定程序(这里的./xxx)仅运行在指定的逻辑CPU上
taskset -c 0 ./xxx
```

## ps

查看进程信息

```bash
# 打印进程相关的信息，这里列的字段表示：进程di、组件名、虚拟内存占用、物理内存占用
# min_flt(minor fault)表示程序的页请求错误次数.内存中已经缓存有进程所需要的页面数据了。只要把该页面数据与进程的虚拟地址空间建立映射关系就可以了。
# maj_flt(major fault)表示程序的页请求错误次数。内存中没有缓存有进程所需要的页面数据。内核必需要通知CPU从磁盘中把页面数据加载到内存中来。（缺页请求）
ps -o pid,comm,vsz,rss,min_flt,maj_flt
```

## 查看交互分区信息

```bash
swapon --show
# free命令可以看到
free

# sar也可以查看系统中是否发生了交换处理，可以看到每秒有多少换出和多少换入，如果系统性能突然下降，且此两个值非零，则说明发生了系统抖动
sar -W 1

# 在上一条基础上通过下方命令可以确认该交换处理到底是暂时性（马上会结束）的还是毁灭性（引发系统抖动）的
# 通过kbswpused字段的值了解交换分区使用量的变化趋势
sar -S
```

## dd 

复制文件并对原文件的内容进行转换和格式化处理

```bash
dd if=/dev/zero of=sun.txt bs=1M count=1
```

- **if** 代表输入文件。如果不指定if，默认就会从stdin中读取输入。
- **of** 代表输出文件。如果不指定of，默认就会将stdout作为默认输出。
- **bs** 代表字节为单位的块大小。
- **count** 代表被复制的块数。
- **/dev/zero** 是一个字符设备，会不断返回0值字节（\0）。

## nmap

```bash
# -s 扫描
# -T 扫描所有开启的TCP端口
nmap -sT 域名或IP
```

