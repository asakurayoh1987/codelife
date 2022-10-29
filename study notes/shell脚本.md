# Shell脚本学习

## 1. 遍历文件夹下的指定文件并执行相应的命令

```bash
#!/bin/bash
# $1表示命令行执行时传递的第一个参数，比如当前脚本名为webp.sh，则对于"./webp.sh src/welfare/img"，$1则为"src/welfare/img"
for file in `ls $1/*.png`
do
# 这里是对输入文件的后缀名进行替换，替换后的文件名作为输出文件的名称
# sed命令在这里的作用为进行替换操作，对于's/.png/.webp/'，其中的s表示进行replace操作
  output=`echo $file | sed 's/.png/.webp/'`
  `cwebp -q 80 $file -o $output`
done
```

** 这里有一个需要注意的地方，在遍历文件夹获取文件并通过echo输出文件名时，如果文件名里有空格，则会作为两行输出，因为默认情况下空格也是作为分隔符使用的，此时可以在遍历之前添加`IFS=$(echo -en '\n\b')`**

>IFS: The Internal Field Separator that is used for word splitting after expansion and to split lines into words with the read built-in command. The default value is "\<space>\<tab>\<new-line>”.

关于 `echo -en '\n\b'`说明如下：

* echo -n 不换行输出
* echo -e 处理特殊字符

```bash
#!/bin/bash
IFS=$(echo -en '\n\b')
for file in `ls *.htm`
do
  output=`echo $file | sed 's/.htm/.pdf/'`
  echo $file
done
```

## 2. shell变量的用法

### 2.1 linux中shell变量$#,$@,$0,$1,$2的含义解释: 

- $$ : Shell本身的PID（ProcessID） 
- $!  : Shell最后运行的后台Process的PID 
- $?  : 最后运行的命令的结束代码（返回值） 
- $-  : 使用Set命令设定的Flag一览 
- $* : 所有参数列表。如"$*"用「"」括起来的情况、以"$1 $2 … $n"的形式输出所有参数。 
- $@ : 所有参数列表。如"$@"用「"」括起来的情况、以"$1" "$2" … "$n" 的形式输出所有参数。 
- $# : 添加到Shell的参数个数 
- $0 : Shell本身的文件名 
- $1～$n : 添加到Shell的各参数值。$1是第1参数、$2是第2参数…。
	
### 2.2 介绍下Shell中的${}、##和%%使用范例

假设定义了一个变量为：

```bash
file=/dir1/dir2/dir3/my.file.txt
```

可以用${ }分别替换得到不同的值：

- ${file#\*/}：删掉第一个 / 及其左边的字符串：dir1/dir2/dir3/my.file.txt
- ${file##\*/}：删掉最后一个 /  及其左边的字符串：my.file.txt
- ${file#\*.}：删掉第一个 .  及其左边的字符串：file.txt
- ${file##\*.}：删掉最后一个 .  及其左边的字符串：txt
- ${file%/\*}：删掉最后一个  /  及其右边的字符串：/dir1/dir2/dir3
- ${file%%/\*}：删掉第一个 /  及其右边的字符串：(空值)
- ${file%.\*}：删掉最后一个  .  及其右边的字符串：/dir1/dir2/dir3/my.file
- ${file%%.\*}：删掉第一个  .   及其右边的字符串：/dir1/dir2/dir3/my

记忆的方法为：

- \# 是 去掉左边（键盘上#在 $ 的左边）
- %是去掉右边（键盘上% 在$ 的右边）
- 单一符号是最小匹配；两个符号是最大匹配

- ${file:0:5}：提取最左边的 5 个字节：/dir1
- ${file:5:5}：提取第 5 个字节右边的连续5个字节：/dir2

也可以对变量值里的字符串作替换：

- ${file/dir/path}：将第一个dir 替换为path：/path1/dir2/dir3/my.file.txt
- ${file//dir/path}：将全部dir 替换为 path：/path1/path2/path3/my.file.txt

```bash
for image in *.png;
do
  convert "$image" "${image%.*}.jpg" -quality 80;
done
```

## 3. 最佳实践

### 3.1 set -euo pipefail

如果不去 `set -euo pipefail`，脚本中可能有指令失败了，然而脚本运行完毕之后仍然显示成功。

### 3.2 set -x

调试bash脚本用`set -x`，这样每个指令开跑之前都会print出来再跑。

### 3.3 set指令能设置所使用shell的执行方式，可依照不同的需求来做设置

　-a 　标示已修改的变量，以供输出至环境变量。
　-b 　使被中止的后台程序立刻回报执行状态。
　-C 　转向所产生的文件无法覆盖已存在的文件。
　-d 　Shell预设会用杂凑表记忆使用过的指令，以加速指令的执行。使用-d参数可取消。
　-e 　若指令传回值不等于0，则立即退出shell。　　
　-f　 　取消使用通配符。
　-h 　自动记录函数的所在位置。
　-H Shell 　可利用"!"加<指令编号>的方式来执行history中记录的指令。
　-k 　指令所给的参数都会被视为此指令的环境变量。
　-l 　记录for循环的变量名称。
　-m 　使用监视模式。
　-n 　只读取指令，而不实际执行。
　-p 　启动优先顺序模式。
　-P 　启动-P参数后，执行指令时，会以实际的文件或目录来取代符号连接。
　-t 　执行完随后的指令，即退出shell。
　-u 　当执行时使用到未定义过的变量，则显示错误信息。
　-v 　显示shell所读取的输入值。
　-x 　执行指令后，会先显示该指令及所下的参数。
　+<参数> 　取消某个set曾启动的参数。

[set命令解析](https://www.cnblogs.com/robinunix/p/11635560.html)

