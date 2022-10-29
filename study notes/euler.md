# Project Euler

## Euler-1 指定范围内3或5的总数之和

等差数列求和公式： (1 + n)/2

## 常见时间复杂度

- O(1)：常数级
- O(logn)：对数，比如二分查找
- O(n)：线性，比如遍历查找
- O(nlogn)：线性对数，快速排序
- O(n^2)：二次方，冒泡排序

## Euler-2 偶斐波那契数

方法一：使用大数组来保存斐波那契列
方法二：使用两个变量，保存前一项与后一项

引入空间复杂度的概念

## Euler-4 最大回文乘积

```c++
#include<iostream>
using namespace std;

int func(int x) {
    int raw = x, t = 0;
    while (x) {
        t = t * 10 + x % 10;
        x /= 10;
    }
    return raw == t;
}

int main() {
    int ans = 0;
    for (int i = 100; i < 1000; i++) {
        for (int j = i; j < 1000; j++) {
            if (func(i * j)) {
                ans = max(ans, i * j);
            }
        }
    }
    cout << ans << endl;
    return 0;
}
```

## Euler-36 双进制回文数

回文数

```c++
#include<iostream>
using namespace std;

int func (int x, int n) {
    int raw = x, t = 0;
    while(x) {
        t = t * n + x % n;
        x /= n;
    }
    return raw == t;
}

int main() {
    int ans = 0;
    for (int i = 1; i < 1000000; i++) {
        if (func(i, 10) && func(i, 2)) {
            ans += i;
        }
    }
    cout << ans << endl;
    return 0;
}
```

## Eurler-6 平方的和与和的平方之差

```c++
#include<iostream>
using namespace std;

int main() {
    int psum = 0, sum = 0;
    for (int i = 1; i <= 100; i++) {
        sum += i;
        psum += i * i;
    }

    cout << sum * sum - psum << endl;
    return 0;
}
```

## Eurler-8 连续数字最大乘积

滑动窗口法

```c++
#include<iostream>
using namespace std;

char str[1005];
long long ans, now = 1, zero_cnt;

int main() {
    cin >> str;
    for (int i = 0; i < 1000; i++) {
        if (i < 13) {
            now *= str[i] - '0';
            ans = now;
        } else {
            if (str[i] == '0') {
                zero_cnt++;
            } else {
                now *= str[i] - '0';
            }
            if (str[i - 13] == '0') {
                zero_cnt--;
            } else {
                now /= str[i - 13] - '0';
            }
            if (zero_cnt == 0) {
                ans = max(ans, now);
            }
        }
    }

    cout << ans << endl;

    return 0;
}
```

## Eurler-11 方阵中的最大乘积

方向数组

```c++
#include<iostream>
using namespace std;

int num[30][30], ans;
int dirx[4] = {-1, -1, 0, 1};
int diry[4] = {0, 1, 1, 1};

int main() {
    for (int i = 5; i < 25; i++) {
        for (int j = 5; j < 25; j++) {
            cin >> num[i][j];
        }
    }

    for (int i = 5; i < 25; i++) {
        for (int j = 5; j < 25; j++) {
            for (int k = 0; k < 4; k++) {
                int t = num[i][j];
                for (int l = 1; l < 4; l++) {
                    int x = i + dirx[k] * l;
                    int y = j + diry[k] * l;
                    t *= num[x][y];
                }
                ans = max(ans, t);
            }
        }
    }

    cout << ans << endl;

    return 0;
}
```

## Eurler-14 最长考拉兹序列

结合递推的思想，利用记忆数组来通过空间换时间，减少递归的深度

递归+记忆数组 约等于 递推

```c++
#include<iostream>
using namespace std;

// 数据暂存数组
int num[10000005];

int func(long long x) {
    if (x == 1) return 1;
    // 记忆化相关
    if (x < 10000000 && num[x] != 0) return num[x];
    int t;
    if (x & 1) t = func(3 * x + 1) + 1;
    else t = func(x / 2) + 1;
    // 记忆化相关
    if (x < 10000000) num[x] = t;
    return t;
}

int main() {
    int ans = 1, cnt = 1;
    for (int i = 2; i < 1000000; i++) {
        int t = func(i);
        if (t > cnt) {
            ans = i ;
            cnt = t;
        }
    }

    cout << ans << " " << cnt << endl;
}
```

## Euler-13 大和

int类型数据范围 -2^31~2^31-1 (减1是因为0的存在)

大整数加法的处理过程：

- 数字的存储：字符串形式输入，将数字倒序存储，一方面便于实现数位的对齐，比如'123'存到数组中时，3存储在下标为1的位置（下标为0的位置用来存储字符串的长度），另一方面在处理进位时方便
- 结果的最短长度：两个数字中较长者，
- 处理加法：创建答案数组，长度为两个数字中较长者，两数组中各位相加，结果存入答案数组的对应位中
- 处理进位

## Euler-25 1000位斐波那契数

## 大整数乘法

处理过程：

- 对于`a * b = c`，c的最短长度为：a的长度 + b的长度 - 1
- 存储时与大整数求和一样，利用数组倒序存储，两数对应位相乘的结果存储在**两数下标之和减1**的位置
- 处理进度，但要注意此时进位可能不止进1位

## 大整数减法

处理过程：

- 判断两者的大小，如果是小数减大数，则设置符号位，然后交换两数，首先根据字符串长度，其次按字典序比较
- 存储时与大整数求和一样，利用数组倒序存储，进行减法，不够减时不断从高位借1

## 大整数除法

处理过程：

- 同样利用数组，只不过此时不进行倒序存储
- 设置结果数组，长度与被除数相同
- 设置一个数组，作为**待除数**，然后将被除数的最高位加入到待除数中，与除数对比大小（首先根据字符串长度，其次按字典序比较），若小于除数，则对应的结果数组中对应位为0，若大于除数，则由数字1开始不断尝试比较（这里会用到大整数减法），将待除数减去除数的结果再与除数比较大小，若大于则继续加1后进行比较，若小于则继续计算结果数组中的下一位

例：被除数为135297 除数为35

1. 待除数：1，小于除数，结果数组为：0
2. 待除数：13，小于除数，结果数组为：00
3. 待除数：135，大于除数，结果数组为：001
4. 待除数：100（由135 - 35得到），大于除数，结果数组为：002
5. 待除数：65，大于除数，结果数组为：003
6. 待除数：30，小于除数，结果数组为：003
7. 待除数：302，大于除数，结果数组为：0031
8. 待除数：267，大于除数，结果数组为：0032
9. ...
10. 待除数：22，小于除数，结果数组为：0038
11. 待除数：229，大于除数，结果数组为：00381
12. ...
13. 待除数：19，小于除数，结果数组为：00386
14. 待除数：197，大于除数，结果数组为：003861
15. ...
16. 待除数：22，小于除数，结果数组为：003865

此时被除数用尽，得到结果：3865，余数22

## Eular-15 网格路径

关键知识：方向数组、递推

```c++
#include <iostream>
using namespace std;

long long ans[25][25];

int main() {
    for (int i = 1; i <= 21; i++) {
        for (int j = 1; j <= 21; j++) {
            if (i == 1 && j == 1) {
                ans[i][j] = 1;
            } else {
                ans[i][j] = ans[i - 1][j] + ans[i][j - 1];
            }
        }
    }

    cout << ans[21][21];

    return 0;
}
```

当然此题也可用数学方法，比如题目中的2*2的方格，其实就是4步中选2步向右，剩余的2步向下走，即C4-2，那对于所求20*20的方格，答案就是C40-20：

```c++
#include <iostream>
using namespace std;

int main() {
    long long num = 1;
    for (int i = 40, j = 1; i > 20; i--, j++) {
        num *= i;
        num /= j;
    }
    cout << num << endl;
    return 0;
}
```

## Eular-18 最大路径和I

关键知识：方向数组、递推

```c++
#include <iostream>
using namespace std;

int num[20][20], ans[20][20], fin;

int main() {
    for (int i = 1; i <= 15; i++) {
        for (int j = 1; j <= i; j++) {
            cin >> num[i][j];
        }
    }
    // 自顶向下
    /*
    for (int i = 1; i <= 15; i++) {
        for (int j = 1; j <= i; j++) {
            ans[i][j] = num[i][j] + max(ans[i - 1][j], ans[i - 1][j - 1]);
            if (fin < ans[i][j]) {
                fin = ans[i][j];
            }
            // fin = max(fin, ans[i][j]);
        }
    }
    cout << fin << endl;
    */
    
    // 自底向上
    for (int i = 15; i > 0; i--) {
        for (int j = 1; j <= i; j++) {
            ans[i][j] = num[i][j] + max(ans[i + 1][j], ans[i + 1][j + 1]);
        }
    }

    cout << ans[1][1] << endl;
    
    return 0;
}
```
