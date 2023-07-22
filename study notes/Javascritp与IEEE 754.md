# Javascript与IEEE 754

[知乎原文](https://zhuanlan.zhihu.com/p/30703042)

[JavaScript 浮点数运算的精度问题 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/191395766)

Javascript只有一种数字类型：Number，它包括了整型与小数，遵循的是IEEE 754的一个子集，使用64位双精度来表示

其中64位包括：

- 符号位S：第1位，0表示正数，1表示负数
- 指数位E：中间11位，用来表示次方
- 尾数位M：最后52位，超出的部分自动进一舍零

![IEEE754](https://oss.kuyinyun.com/11W2MYCO/rescloud1/72b9cb357e3344ffb542d54eb47f244d.png)

实际的数字用以下公式计算：
$$
V=(-1)^S * 2^E * M
$$

> 注意以上的公式遵循科学计数法的规范，在十进制中 `0<M<10`，到二进制就是 `0<M<2`。也就是说整数部分只能是1，所以可以被舍去，只保留后面的小数部分。如 `4.5` 转成二进制就是 `100.1`，科学计数法表示是 `1.001*2^2`，舍去1后 `M = 001`。E是一个无符号整数，因为长度是11位，取值范围是 0~2047。但是科学计数法中的指数是可以为负数的，所以约定减去一个中间数 1023，`[0,1022]` 表示为负，`[1024,2047]` 表示为正。如 4.5 的指数 `E = 1025`，尾数 `M = 001`。
>
> 上述这段话是有问题的，参见[IEEE 754 | Wikiwand](http://link.zhihu.com/?target=https%3A//www.wikiwand.com/zh-hans/IEEE_754)64位双精度部分：
>
> 双精度的指数部分是−1022～+1023加上1023，指数值的大小从1～2046（0（2进位全为0）和2047（2进位全为1）是特殊值）。浮点小数计算时，指数值减去偏正值将是实际的指数大小。
>
> 因此[1, 1022]表示负，[1024, 2046]表示正。
>
> 附三种情况（1~2046、 0、 2047）
>
> **（1）E不全为0或不全为1（1~2046）。**这时，浮点数就采用上面的规则表示，即指数E的计算值减去1023，得到真实值，再将有效数字M前加上第一位的1。
>
> **（2）E全为0（0）。**这时，浮点数的指数E等于1-1023，有效数字M不再加上第一位的1，而是还原为0.xxxxxx的小数。这样做是为了表示±0，以及接近于0的很小的数字。
>
> **（3）E全为1（2047）。**这时，如果有效数字M全为0，表示±无穷大（正负取决于符号位s）；如果有效数字M不全为0，表示这个数不是一个数（NaN）。
>
> 

最终公式变成：
$$
V=(-1)^S*2^{E-1023}*(M+1)
$$
所以 `4.5` 最终表示为（M=001、E=1025）：

![IEEE754_demo](https://oss.kuyinyun.com/11W2MYCO/rescloud1/f45a4e9960fb410d8ffe31cb21831abe.png)

这里有一个[转换工具](https://www.binaryconvert.com/result_double.html)

下面再以 `0.1` 为例解释浮点误差的原因，`0.1` 转成二进制表示为 `0.0001100110011001100`(1100循环)，`1.100110011001100x2^-4`，所以 `E=-4+1023=1019`；M 舍去首位的1，得到 `100110011...`。最终就是：

![IEEE754_demo](https://oss.kuyinyun.com/11W2MYCO/rescloud1/3f8f03403fbf4ae5960b6ffc90d22935.png)

举例（这里直接贴了gtp4生成的答案）

十进制数3.14转换成单精度浮点数表示的具体步骤：

To convert the decimal number 3.14 to a single-precision floating-point number, we need to follow the IEEE 754 standard for floating-point arithmetic. Single-precision floating-point numbers use 32 bits, with 1 bit for the sign, 8 bits for the exponent, and 23 bits for the mantissa.

Here are the steps to convert 3.14 to a single-precision floating-point number:

1. Determine the sign bit: Since 3.14 is a positive number, the sign bit is **0**.

2. Convert the integer part (3) to binary:

```
3 (decimal) = 11 (binary)
```



3. Convert the fractional part (0.14) to binary:

Multiply the fractional part by 2 repeatedly and take the integer part at each step:

```
  0.14 * 2 = 0.28 -> 0

  0.28 * 2 = 0.56 -> 0

  0.56 * 2 = 1.12 -> 1

  0.12 * 2 = 0.24 -> 0

  ...
```



The binary representation of the fractional part is 0.0010....

4. Combine the integer and fractional parts in binary:

 ```
 3.14 (decimal) = 11.0010... (binary)
 ```



5. Normalize the binary number:

Move the binary point to the right of the most significant bit (MSB) and count the number of places moved. This will be the exponent value.

```
11.0010... (binary) = 1.10010... x 2^1 (normalized)
```



6. Calculate the biased exponent:

Add the bias value (127 for single-precision) to the exponent value.

```
1 (exponent) + 127 (bias) = 128
```



7. Convert the biased exponent to binary:

```
128 (decimal) = 10000000 (binary)
```



8. Determine the mantissa:

Remove the leading 1 from the normalized binary number and take the first 23 bits.

```
1.10010... (normalized) -> 10010... (mantissa, 23 bits)
```



9. Combine the sign bit, biased exponent, and mantissa:

```
0 (sign) | 10000000 (biased exponent) | 10010001111010111000011 (mantissa)
```



The single-precision floating-point representation of 3.14 is 01000000010010001111010111000011 in binary.

 这个答案与工具处理的结果一致：

![image-20230614130653429](../media/image-20230614130653429.png)