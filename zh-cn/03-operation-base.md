# 运算基础

![访问统计](https://visitor-badge.glitch.me/badge?page_id=senlypan.qa.03-operation-base&left_color=blue&right_color=red)

> 作者: 潘深练
>
> 创建: 2022-10-22

## 一、异或

### 1.1、简介

众所周知，编程语言一般都内置了3种位运算符 `&(AND)、|(OR)、~(NOT)`，用来实现位运算，但其实还有一种非常常用的位运算，即异或 `^(XOR)`，数学中常用⊕表示。

异或的运算逻辑如下：

1 ⊕ 1 = 0

1 ⊕ 0 = 1

0 ⊕ 1 = 1

0 ⊕ 0 = 0

简单来说，异或的特性是，两个值相同，结果为0，不同则结果为1。

由于异或特殊的运算特性，使其可以实现一些神奇的操作，如下：

1. 实现加减法
2. 加解密
3. 密钥交换
4. 数据备份

**运算定律**

1. 任何值与自身异或，结果为0

```shell
x ^ x = 0
```

2. 任何值与0异或，结果为其自身

```shell
x ^ 0 = x
```

3. 交换律

```shell
x ^ y ^ z = x ^ z ^ y
```

4. 结合律

```shell
x ^ y ^ z = x ^ (y ^ z)
```

### 1.2、异或巧妙应用场景

- 👉 [segmentfault - 《异或的4种堪称神奇的运用场景》](https://segmentfault.com/a/1190000042456870)

### 1.3、异或测验

```java
public class Xor {

    public static void main(String[] args) {

        int a = 111;
        int b = 15;
        int c = 88;

        String aBit = Integer.toBinaryString( a );
        String bBit = Integer.toBinaryString( b );
        String cBit = Integer.toBinaryString( c );
        String abBit = Integer.toBinaryString((a^b));
        String acBit = Integer.toBinaryString((a^c));
        String bcBit = Integer.toBinaryString((b^c));

        String abcBit = Integer.toBinaryString(  a ^ (b^c)  );
        String bacBit = Integer.toBinaryString(  b ^ (a^c)  );
        String cabBit = Integer.toBinaryString(  c ^ (a^b)  );

        // 标题
        System.out.println( " ============================================ ");
        System.out.println( " 值 \t\t " + "十进制 \t\t " + "二进制");

        // a^b
        System.out.println( " ============================================ ");
        System.out.println( " a \t\t\t " + a      + " \t\t " + fill(aBit));
        System.out.println( " b \t\t\t " + b      + " \t\t " + fill(bBit));
        System.out.println( " a^b \t\t " + (a^b)  + " \t\t " + fill(abBit));

        // a^c
        System.out.println( " ============================================ ");
        System.out.println( " a \t\t\t " + a      + " \t\t " + fill(aBit));
        System.out.println( " c \t\t\t " + c      + " \t\t " + fill(cBit));
        System.out.println( " a^c \t\t " + (a^c)  + " \t\t " + fill(acBit));

        // b^c
        System.out.println( " ============================================ ");
        System.out.println( " b \t\t\t " + b      + " \t\t " + fill(bBit));
        System.out.println( " c \t\t\t " + c      + " \t\t " + fill(cBit));
        System.out.println( " b^c \t\t " + (b^c)  + " \t\t " + fill(bcBit));

        // a^b^c (交换律)
        System.out.println( " ============================================ ");
        System.out.println( " a ^ (b^c) \t " + (a ^ (b^c))  + " \t\t " + fill(abcBit));
        System.out.println( " b ^ (a^c) \t " + (b ^ (a^c))  + " \t\t " + fill(bacBit));
        System.out.println( " c ^ (a^b) \t " + (c ^ (a^b))  + " \t\t " + fill(cabBit));

        // 变量交换 ==> d ^ d = 0 , d ^ 0 = d
        System.out.println( " ============================================ ");
        int d = 10;
        int e = 987;
        String dBit = Integer.toBinaryString( d );
        String eBit = Integer.toBinaryString( e );
        System.out.println( " d \t\t\t " + d      + " \t\t " + fill(dBit));
        System.out.println( " e \t\t\t " + e      + " \t\t " + fill(eBit));
        d ^= e;
        e ^= d;
        d ^= e;
        System.out.println( " d \t\t\t " + d      + " \t\t " + fill(dBit));
        System.out.println( " e \t\t\t " + e      + " \t\t " + fill(eBit));


    }

    /**
     * 填充对齐
     * @param binaryStr
     * @return
     */
    public static String fill(String binaryStr){
        while( binaryStr.length() < 10 ){
            binaryStr = "0" + binaryStr;
        }
        return binaryStr;
    }
}

```

输出结果：

```shell
 ============================================ 
 值 		 十进制 		 二进制
 ============================================ 
 a 			 111 		 0001101111
 b 			  15 		 0000001111
 a^b 		    96 		 0001100000
 ============================================ 
 a 			 111 		 0001101111
 c 			  88 		 0001011000
 a^c 		    55 		 0000110111
 ============================================ 
 b 			  15 		 0000001111
 c 			  88 		 0001011000
 b^c 		    87 		 0001010111
 ============================================ 
 a ^ (b^c) 	  56 		 0000111000
 b ^ (a^c) 	  56 		 0000111000
 c ^ (a^b) 	  56 		 0000111000
 ============================================ 
 d 			  10 		 0000001010
 e 			 987 		 1111011011
 d 			 987 		 0000001010
 e 			  10 		 1111011011
```
