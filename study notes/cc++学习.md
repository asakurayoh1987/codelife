# c/c++学习

## 编译过程

一个C/C++文件要经过预处理(preprocessing)、编译(compilation)、汇编(assembly)、和连接(linking)才能变成可执行文件。

### 1. 预处理

```bash
# -E的作用是让gcc在预处理结束后停止编译
gcc -E main.c -o main.i
```

预处理阶段主要处理include和define等。它把#include包含进来的.h 文件插入到#include所在的位置，把源程序中使用到的用#define定义的宏用实际的字符串代替

### 2. 编译阶段

```bash
# -S的作用是编译后结束，编译生成了汇编文件
gcc -S main.i -o main.s
```

在这个阶段中，gcc首先要检查代码的规范性、是否有语法错误等，以确定代码的实际要做的工作，在检查无误后，gcc把代码翻译成汇编语言。

### 3. 汇编阶段

```bash
gcc -c main.s -o main.o
```

汇编阶段把 .s文件翻译成二进制机器指令文件.o,这个阶段接收.c, .i, .s的文件都没有问题。

### 4. 链接阶段

```bash
gcc -o main main.s
```

链接阶段，链接的是函数库。在main.c中并没有定义”printf”的函数实现，且在预编译中包含进的”stdio.h”中也只有该函数的声明。系统把这些函数实现都被做到名为libc.so的动态库。

函数库一般分为静态库和动态库两种

静态库是指编译链接时，把库文件的代码全部加入到可执行文件中，因此生成的文件比较大，但在运行时也就不再需要库文件了。Linux中后缀名为”.a”。

动态库与之相反，在编译链接时并没有把库文件的代码加入到可执行文件中，而是在程序执行时由运行时链接文件加载库。Linux中后缀名为”.so”，如前面所述的libc.so就是动态库。gcc在编译时默认使用动态库。

> 静态库节省时间：不需要再进行动态链接，需要调用的代码直接就在代码内部
> 动态库节省空间：如果一个动态库被两个程序调用,那么这个动态库只需要在内存中
> Java中在不经过封装的情况下只能直接使用动态库。

## clang安装

```bash
sudo apt-get install clang
```

也可指定版本安装：

```bash
sudo apt-get install clang-10
```

安装多个版本时，可以通过`update-alternatives`来处理，具体参见[这里](https://blog.csdn.net/DumpDoctorWang/article/details/84567757)

```bash
sudo apt-get install clang-3.8 clang-4.0

# for clang-3.8
sudo update-alternatives --install /usr/bin/clang clang /usr/bin/clang-3.8 1 --slave /usr/bin/clang++ clang++ /usr/bin/clang++-3.8
# for clang-4.0
sudo update-alternatives --install /usr/bin/clang clang /usr/bin/clang-4.0 2 --slave /usr/bin/clang++ clang++ /usr/bin/clang++-4.0

# 此时如果执行`clang --version`得到的是4.0的打屏，此时可以通过如下命令，调整clang各版本的优先级
sudo update-alternatives --config clang
```

## lldb调试

参考[这里](https://www.developerfiles.com/debugging-c-with-clang-compiler-and-lldb/)及[官方文档](https://lldb.llvm.org/use/tutorial.html)以及[掘金上的这篇](https://juejin.cn/post/6844903805398548493#heading-9)

## 查看gcc当前默认的编译标准

```bash
gcc -E -dM - </dev/null | grep "STDC_VERSION"
```

关于编译标准的解释可以参照[这里](http://c.biancheng.net/view/8053.html)

## 数学库的使用

```c
#include <math.h>
```
```bash
# 使用数学库里，编译时要指定`-lm`
gcc app.c -lm -o agg.out
```

## 函数与数组

函数是压缩的数组，数组是展开的函数

## 整型的使用

```c
#include <inttypes.h>
// 声明64位的int型
int64_t c;
```

## 指针函数——函数作为参数传递

```c
其中`int (*fun)(int)`即为作为参数传递的函数声明，声明其为接受一个整型参数并返回一个整型数值的函数
int calc(int (*fun)(int), int n);
```

## 变参函数

```c
#include <inttypes.h>
#include <stdarg.h>

int max_int(int n, ...) {
	// 声明32位整型的最小值，此宏定义在inttypes.h中
	int ans = INT32_MIN;
	// 声明va_list类型的变参，通过它获取参数n之后的参数列表
	va_list arg;
	// 定位n后面第一个参数的位置
	va_start(arg, n);
	while (n--) {
		// 获取下一个可变参数列表中的参数
		int temp = va_arg(arg, int);
		if (temp > ans) ans = temp;
	}
	// 结束获取可变参数列表的动作，释放arg参数
	va_end(arg);
	return ans;
}
```

## 使用'#'前缀在宏中将一个变量字符化

```c
#include<stdio.h>

#define MAX(a, b) a > b ? a : b
#define P(func) {\
    printf("%s = %d\n", #func, func);\
}

int main() {
    P(MAX(2, 3));
    return 0;
}
```

## 宏中使用'##'进行变量的连接

```c
#define contact(a, b) a##b
int main() {
	contact(abc, def) = 112233;
	console.log("%d", abcdef);
	return 0;
}
```

`##`还有一个使用场景就是在定义变长参数的函数宏中，如下：

```c
#define log(format, ...) printf(format, __VA_ARGS__)
```

但实际使用时，如果这样写`debug("message")`，宏编译后会变成`printf("message",)`，此时变长参数的长度为0，导致宏编译后的函数调用多带了一个逗号，从而报错，此时就可以使用`##`来处理

```c
#define log(format, ...) printf(format, ##__VA_ARGS__)
```

这里，如果可变参数被忽略或为空，`##`操作将使预处理器去除掉它前面的那个逗号

更多关于[可变参数宏](https://www.jianshu.com/p/958162214e91)

## 预定义的宏

- __typeof：__typeof(a) 宏展开后，如果a是整型，则它会被替换为'int'

```c
#include<stdio.h>
#define MAX(a, b) ({\
	__typeof(a) _a = (a);\
	__typeof(b) _b = (b);\
	_a > _b ? _a : _b;\
})
#define P(func) {\
  printf("%s = %d\n", #func, func);\
}

int main() {
   P(MAX(2, 3));
   return 0;
}
```

## 查看宏替换后的代码

```bash
# 通过指定"-E"来进行宏展开
gcc -E xxx.c
```

## 全局区数组的定义

```c
#include <stdio.h>

#define MAX_N 200000

int prime[MAX_N + 5];

int main() {
	return 0;
}
```

上述代码关于prime数组是定义在main函数之外，表示此数组定义在全局区，全局区定义的数组不会受系统栈8MB的限制，对于长数组需要使用这种方式进行声明，同时在全局区的数组，系统会额外对数组做一次清空的操作，而如果在main函数中声明，则需要手动进行清楚，比如 `int prime[100] = {0}`

## 线性筛算法找指定范围内的素数

```c
#include <stdio.h>
#define MAX_N 200000

int prime[MAX_N + 5];
void init_prime() {
		// N = M * P，N为任意数，P为最小素因子，M要保持是当前最大值
    for (int i = 2; i <= MAX_N; i++) {
    		// 索引值为0表示是当前索引值为素数，其中第0个元素用来记录当前已经找到的素数个数，prime[i]代表第i个素数
        if (!prime[i]) prime[++prime[0]] = i;
        // 从当前已经找到素数开始遍历
        for (int j = 1; j <= prime[0]; j++) {
        		// 如果已经超过范围则路过
            if (prime[j] * i > MAX_N) break;
            // 如果满足则说明该索引值是合数，prime[j]代表P，i代表N
            prime[prime[j] * i] =1;
            // 从最小的素数开始遍历时，一旦i%当前的素数能除尽，就满足了线性筛中要求的M保持最大
            if (i % prime[j] == 0) break;
        }
    }
    return;
}

int main() {
    init_prime();
    printf("%d\n", prime[10001]);
    return 0;
}
```

## 约数和定理

## 结构体字节对齐

1. 结构体变量的首地址能够被其最宽基本类型成员的大小所整除；
1. 结构体每个成员相对结构体首地址的偏移量(offset)都是成员大小的整数倍，如有需要编译器会在成员之间加上填充字节(internal adding)；
1. 结构体的总大小为结构体最宽基本类型成员大小的整数倍，如有需要编译器会在最末一个成员之后加上填充字节{trailing padding}。

对于以上三条的解释如下：

- 第一条：编译器在给结构体开辟空间时，首先找到结构体中最宽的基本数据类型，然后寻找内存地址能被该基本数据类型所整除的位置，作为结构体的首地址。将这个最宽的基本数据类型的大小作为上面介绍的对齐模数。
- 第二条：为结构体的一个成员开辟空间之前，编译器首先检查预开辟空间的首地址相对于结构体首地址的偏移是否是本成员大小的整数倍，若是，则存放本成员，反之，则在本成员和上一个成员之间填充一定的字节，以达到整数倍的要求，也就是将预开辟空间的首地址后移几个字节。
- 第三条：结构体总大小是包括填充字节，最后一个成员满足上面两条以外，还必须满足第三条，否则就必须在最后填充几个字节以达到本条要求。

可通过宏#pragma pack(n) n=1，2，4等来指定对齐方式：

- 使用伪指令#pragma pack(n)：C编译器将按照n个字节对齐；
- 使用伪指令#pragma pack()： 取消自定义字节对齐方式。

## 头文件stdint.h

其中定义了一些固定位数的整型类型的声明，以及一些宏，比如32位整型最大值：`INT32_MAX`

## 条件编译

具体参见[这里](http://c.biancheng.net/view/289.html)

通过gcc的选项`-D`来进行条件编译，如下代码段：

```c
#ifdef DEBUG

#deinfe MAX_N 100000

#endif
```

以上代码在编译时使用`gcc -D DEBUG`，则会进行条件编译，声明宏：MAX_N

## 关于头文件的引用

```c
// 从系统路径下查找stdio.h头文件
#include <stdio.h>
// 从当前目录下查找head1.h头文件
#include "head1.h"
```

`#include`称为预处理命令，在编译时，会被替换成具体的文件内容，可以理解为是占位

## 多文件编译

- 将多个文件分别生成对象文件：`gcc -c main.c` 与 `gcc -c app.c`，分别生成对象文件`main.o`与`app.o`
- 链接对象文件：`gcc app.o main.o`


## 头文件只放声明，源文件只放定义

## 预编译、编译、链接

`g++ -c main.c`编译生成 main.o对象文件，再通过`g++ main.o`链接生成a.out可执行文件，举例：函数未声明，在编译期会报错，函数未定义则是在链接期报错 PS:`gcc -E main.c`执行预编操作

也就是说在编译期，只会进行语法错误的检查，比如函数未声明；而链接期，会将各个模块进行组合，将程序跑起来

## ld main.o

查看对象文件中的函数等定义

## 链接库的使用

基于linux可以使用如下方式：

```bash
# ar -a 创建链接库
# *.a 此后缀表示静态链接库
# 指定src下的所有对象文件
# 约定以lib为前缀
ar -r libhaizei.a ./src/*.o
# 生成libhaizei.a之后拷贝至lib目录下，其使用方式如下
# unite.cc中通过include包含了libhaizei中定义的函数
# unite.cpp中include了目录./include下的头文件
g++ -I./include -c unite.cpp
# 生成unite.o对象文件后，在接下的链接过程中使用之前生成的libhaizei.a
# 此命令会在./lib目录下查找libhaizei.a，-l的效果就是在haizei前增加lib，其后添加.a
g++ -L./lib unite.o -lhaizei
```

## c++使用c中的头文件

以`#include <stdio.h>`为例，在c++中使用时，将后缀名.h去掉，同时在前面添加c，即`#include <cstdio>`

## STL容器的使用

STL：标准模板库，但事实上它并不在标准中，所以可能有些杂七杂八的也塞到STL里

STL有以下几个组件：

- 容器
- 适配器
- 迭代器
- 算法
- 仿函数
- 分配器

### 字符串相关用法

> C/C++参考手册：[cppreference](https://en.cppreference.com/w/)

```c++
#include <string>
// 定义一个字符串对象，如下string是一个类，s是一个对象
string s;
```

#### 操作

- 获取字符串长度：`s.size()`或`s.length()`
- 赋值：`s = "abc"`，还可以进行`s = "a" + "b"`
- 查找：`s.find("ab", 4)`，返回值为子符在原串中的位置索引，如果第二个参数传递，则表示从位置4开始查找，如果未找到，返回`string:npos`
- 插入：`s.insert(0, "abc")`，在位置0插入“abc”
- 截取：`s.substr(0, 4)`，从位置0开始，截取长度为4的字符串
- 替换：`s.replace(0, 4, "abc")`，从位置0开始，替换长度4的子串，替换为“abc”

string声明的对象无法使用scanf来写入值

### vector 向量

```c++
#include <string>
// 声明一个元素为int类型的向量
vector<int> v
```
vector称为向量，可以认为是一个动态数组

#### 操作

- 获取vector中的元素数量：`v.size()`
- 在最后加入：`v.push_back(5)`
- 判断是否为空：`v.empty()`

#### 关于二维向量：

```c++
// 在c++11之前，`int> >`这里中间有个空格，不然报错，c++11及之后允许不加空格：g++ vector.cpp --std=c++11
vector<vector<int> > v(5, vector<int>(6, 8));
```

#### 关于存储空间——一片连续的区域

#### 关于扩容规则：
可以通过`v.capacity()`来查看当前向量的容量，关于扩容规则，取决于具体的编译器，比如有是2倍，也有1.5倍的，通过下面的代码可以看到具体的扩容情况，

```c++
int main() {
    vector<int> v;
    int cnt = 0;
    for (int i = 1; i <= 10000000; i++) {
        v.push_back(i);
        if (cnt != v.capacity()) {
            cnt = v.capacity();
            cout << i << " " << cnt << endl;
        }
    }
    return 0;
}
```
转出结果可能如下，说明在当前机器的编译器上，扩容规则是2倍：

```bash
1 1
2 2
3 4
5 8
9 16
17 32
33 64
65 128
129 256
257 512
513 1024
1025 2048
2049 4096
4097 8192
8193 16384
16385 32768
32769 65536
65537 131072
131073 262144
262145 524288
524289 1048576
1048577 2097152
2097153 4194304
4194305 8388608
8388609 16777216
```
### stack 栈

```c++
#include <stack>
// 声明一个元素为int类型的栈
stack<int> sta;
```

#### 操作

- 获取栈大小：`sta.size()`
- 入栈：`sta.push(1)`
- 出栈：`sta.pop()`
- 获取栈顶元素：`sta.top()`
- 判断栈是否为空：`sta.empty()`，返回bool

#### 关于存储

栈在内存中的存储形式是一个”双端队列“（两头都可以添加和删除，封住一端就形成了栈），双端队列也是一个容器，所以栈其实是基于另一个容器来实现的，栈准确说是一个容器适配器

### queue 队列

```c++
#include <queue>
// 声明一个元素为int类型的队列
queue<int> que
```

#### 操作

- 获取元素数量：`que.size()`
- 入队：`que.push(1)`
- 出队：`que.pop()`
- 获取队首元素：`que.front()`
- 获取队尾元素：`que.back()`
- 判断是否为空：`que.empty()`

#### 存储结构

队列在内存中也是使用双端队列进行封装的，所以队列其实也是一个容器适配器

### deque 双端队列

```c++
#include <deque>
// 声明一个元素为int类型的双端队列
deque<int> deq
```
#### 操作

- 获取元素个数：`deq.size()`
- 前端插入：`deq.push_front(x)`
- 后端插入：`deq.push_back(x)`
- 前端删除：`deq.pop_front()`
- 后端删除：`deq.pop_back()`
- 获取前端元素：`deq.front()`
- 获取后端元素：`deq.back()`
- 判断是否为空：`deq.empty()`

#### 关于存储

双端队列是顺序存储的，也就是说支持随机访问，同时无论是在哪端插入、删除都是常数组的，这个原因就涉及到deque的存储结构

deque维护了一个指针表的结构，指针表是连续的，指针表中的每个指针又指向一个连续存储的空间

### list 双向链表

### priority_queue 优先队列

优先队列，本质上是一棵树，也就是堆，会在插入元素或删除元素后，经过调整(O(logN))保持树根是优先级最大的数

```c++
#include <queue>
// 声明一个元素为int类型的优先队列，默认是一个大根堆
priority_queue<int> que
```

#### 操作

- que.size()
- que.push(x)
- que.pop()
- que.top()
- que.empty()

#### 示例

```c++
#include <iostream>
#include <queue>
#include <string>
using namespace std;

int main() {
		// 声明一个小顶堆，此处可以看出内部存储使用的是vector<int>（这也是默认值），第三个参数表示比较方法，这里传greater<int>，创建的是小顶堆 ，默认是传less<int>，表示大顶堆
    priority_queue<int, vector<int>, greater<int> > que;
    que.push(5);
    que.push(-2);
    que.push(99);
    que.push(66);
    que.push(11);
    while (!que.empty()) {
        cout << que.top() << endl;
        que.pop();
    }

    /* 字符串比较的示例，大顶堆
    priority_queue<string> que;
    que.push("12345");
    que.push("1234567890");
    que.push("[]--");
    que.push("abcdfjafkdfafdsaf");
    que.push("AGGGGG");
    while (!que.empty()) {
        cout << que.top() << endl;
        que.pop();
    }
    */
    /* 整型比较的示例
    priority_queue<int> que;

    que.push(5);
    que.push(-2);
    que.push(99);
    que.push(66);
    que.push(11);

    cout << que.size() << endl;
    while (!que.empty()) {
        cout << que.top() << endl;
        que.pop();
    }
    */
    return 0;
}
```

#### 关于自定义结构的排序

两种方案：

1. 运算符重载，重载"<"

```c++
#include <iostream>
#include <queue>
using namespace std;

struct node {
    int x, y;
    // 重载<，
    bool operator< (const node &b) const {
        // 因为默认是大顶堆，所以下面对比时，明明重载的是小于号，比较时却要用于大于号
        return this->x > b.x;
    }
};

int main() {
    priority_queue<node> que;
    que.push((node){5, 6});
    que.push((node){1, 99});
    que.push((node){-2, -7});

    cout << que.size() << endl;

    while (!que.empty()) {
        cout << que.top().x << " " << que.top().y << endl;
        que.pop();
    }

    return 0;
}
```

2. 仿函数

```c++
#include <iostream>
#include <queue>
using namespace std;

struct node {
    int x, y;
};

// 仿函数
struct cmp {
    bool operator() (const node &a, const node &b) {
        return a.x > b.x;
    }
};

int main() {
		// 将cmp作为第三个参数传入
    priority_queue<node, vector<node>, cmp> que;
    que.push((node){5, 6});
    que.push((node){1, 99});
    que.push((node){-2, -7});

    cout << que.size() << endl;

    while (!que.empty()) {
        cout << que.top().x << " " << que.top().y << endl;
        que.pop();
    }

    return 0;
}
```

### 图

