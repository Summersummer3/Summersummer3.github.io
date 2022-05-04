---
author: "Techsum"
title: "ECDSA与SM2"
date: 2020-04-01T14:03:23
description: "椭圆曲线算法入门及ECDSA与SM2算法详解"
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: Techsum
categories:
- math
libraries:
- mathjax
tags: 
- 密码学
---

## 椭圆曲线基本学习

文章：https://andrea.corbellini.name/2015/05/17/elliptic-curve-cryptography-a-gentle-introduction/#elliptic-curves

书籍：密码学原理与实践 第6章 



### 椭圆曲线方程

$$
\left\lbrace (x, y) \in \mathbb{R}^2\ |\ y^2 = x^3 + ax + b,\ 4 a^3 + 27 b^2 \ne 0 \right\rbrace\ \cup\ \left\lbrace 0 \right\rbrace
$$



### 群与阿贝尔群

​	$\mathbb{G}$ 是一个 **群 (Group)** 如果该集合上定义了一种运算 $ + $:

1. **封闭性**: $a \in \mathbb{G}, b \in \mathbb{G}$ ,则 $a + b \in \mathbb{G}$ ;
2. **结合律:** $a \in \mathbb{G}, b \in \mathbb{G}, c \in \mathbb{G}$, $ (a + b) + c = a + (b +c)$ ;
3. 存在**单位元** $0 \in \mathbb{G}$, 使得 $a \in \mathbb{G}$，$a + 0 = 0 + a = a$;
4. 每一个元素存在**逆元**：对于集合内任意元素$a, \exists b \in \mathbb{G}$ 满足 $a + b = 0$，记做$a = -b$

如果该群还满足:

5. **交换律：** $a \in \mathbb{G}, b \in \mathbb{G}$， $a + b = b + a$ 

则该群被称为**阿贝尔群**.



### 有限域

$\mathbb{F}$ 是一个 **域(Field)** 如果该集合上定义了两种运算 $(\cdot\ ;+)$

1. **封闭性:** $a \in \mathbb{F}, b \in \mathbb{F}$，则 $a + b \in \mathbb{G}; a \cdot b \in \mathbb{G}$
2. **结合律:** $a \in \mathbb{F}, b \in \mathbb{F}, c \in \mathbb{F}$，$ (a + b) + c = a + (b +c);\ (a \cdot b) \cdot c = a \cdot (b \cdot c)$ 
3. 存在**加法单位元** $0 \in \mathbb{F}$，使得 $a \in \mathbb{F}$，$a + 0 = 0 + a = a$
4. 存在**乘法单位元** $e \in \mathbb{F}$，使得 $a \in \mathbb{F}$，$a \cdot e = e \cdot a = a$
5. **交换律：** $a \in \mathbb{F}, b \in \mathbb{F}$， $a + b = b + a$，$a \cdot b = b \cdot a$
6. **逆元:** 对于集合内任意元素$a, \exists b \in \mathbb{F}; \exists c \in \mathbb{F}$ 满足 $a + b = 0; a \cdot c = e$， 记做$a = -b;\ a = c^{-1}$，$0^{-1}$无意义
7. **分配律:**  $a \in \mathbb{F}, b \in \mathbb{F}, c \in \mathbb{F}$；$a \cdot (b + c) = a \cdot b + a \cdot c$

注意：加法逆元定义减法，乘法逆元定义除法



**有限域**指的是元素有限的域，属于计算机和密码学的基本数学原理之一

典型的有限域例子：$\mathbb{F}_p = \{0, 1, ..., p-1\}$,  $p$为质数，

定义 (+)：$a + b \mod p$

定义 ($\cdot$)：$a \cdot b \mod p$

计算 $a ^ {-1}$ : 拓展欧几里得算法



### 椭圆曲线上的群

1. 曲线上的点的集合组成群
2. $x$无穷远点为单位元$0$ 
3. 点$P$与它的逆$Q$关于直线$x = 0$对称
4. **加法**定义：$P + Q + R = 0$，如果这三点是非0点，且在同一条直线上(即一条直线与该曲线相交于三点，无穷远点为0)  $=> P + Q = -R$

加法同样需要满足结合律.


### 几何意义上的加法

最重要的情况: 

1. 如果 $P=Q, P + Q$，物理意义是切线, 与曲线交于另一点$R$, 满足：$2P = -R$
2. 如果$P, Q$直线的第三点刚好为$P\ or\ Q$，则也将包含一条切线，计算相同: $P + Q + P = 0\ => P + Q = -P$



### 代数意义上的加法

#### 不同两点相加

$P(P_x, P_y), Q(Q_x, Q_y)，P\ \ne Q$, 求  $T(T_x, T_y) = P + Q$

曲线方程：$y^2 = x^3 +ax + b$

1. 计算斜率 $k = \frac{P_y - Q_y}{P_x - Q_x}$

2. 曲线方程连立上直线方程 $y = kx + c$

   => $ 0 = x^3 - k^2x^2 + (a - 2kc)x + b - c^2$

3.  铭记三次求根公式之三根之和是二次项系数的相反数: $T_x = k^2 - P_x - Q_x$

4.  由于斜率 $k = \frac{T_y - P_y}{T_x - P_x}$，$T_y = k(T_x - P_x) + P_y$



#### 相同两点相加

$P(P_x, P_y)$，求 $T(T_x,T_y) = P + P = 2P$

和上面基本相同，但计算直线斜率需要根据切线计算

对曲线方程两边求隐微分:

$\mathrm{d}(y^2) = \mathrm{d}(x^3 +ax + b)$ => $2y\mathrm{d}y = (3x^2 + a)\mathrm{d}x$

将$P_x, P_y$带入，获得斜率:

$k = \frac{\mathrm{d}y}{\mathrm{d}x} = \frac{3P_x^2 + a}{2P_y}$

所以 $T_x = k^2 - 2P_x$，$T_y = k(T_x - P_x) + P_y$



#### 标量积

$P(P_x, P_y)$，求 $nP = \underbrace{P + P + P + ... + P}_{\text{n times}}$，$n > 2$

1. 将 $n$ 用二进制表示；以151为例子，$151_{10} = 10010111_2 = 2^0 + 2^1 + 2^2 + 2^4 + 2^7$

2. $nP = P + 2P + 2^2P + 2^4P + 2^7P$

3. 根据上面两项计算规则，分别计算$P, 2P，P + 2P$

4. 计算$2^2P = 4P = 2 \cdot 2P$，对$2P$做相同点相加即可

5. 同理计算$2^3P = 8P = 2 \cdot 4P$，$2^4P = 16P = 2 \cdot 8P$, 依次类推，每计算到一个二进制中为$1$的阶数， 完成一次两点相加即可

   

### 曲线上的有限域

取几何曲线上的坐标$(x, y)$，$x, y \in \mathbb{F}_p$, $p$ 是一个质数，形成一条离散曲线：
$$
\begin{array}{rcl}
  \left\lbrace(x, y) \in (\mathbb{F}_p)^2 \right. & \left. | \right. & \left. y^2 \equiv x^3 + ax + b \pmod{p}, \right.
  \left. 4a^3 + 27b^2 \not\equiv 0 \pmod{p}\right\rbrace\ \cup\ \left\lbrace0\right\rbrace
\end{array}
$$

从连续曲线上的加法可以推出**有限域上的加法公式**：

$P(P_x, P_y), Q(Q_x, Q_y)，P\ \ne Q$, 求  $T(T_x, T_y) = P + Q$
$$
\begin{array}{rcl}
  k & = &(P_y - Q_y)(P_x - Q_x)^{-1} \bmod{p} \\
  T_x & = & (k^2 - P_x - Q_x) \bmod{p} \\
  T_y & = & [P_y + k(T_x - P_x)] \bmod{p} \\
\end{array}
$$
若$P\ = Q$
$$
k = (3 P_x^2 + a)(2 P_y)^{-1} \bmod{p}
$$


### 曲线上的循环子群

#### 循环子群的阶

对于离散曲线上的任意点$P$, 存在最小的 $n$ 使得 $nP = 0$, 此时 $n$ 称作以 $P$ 为基点的循环子群的阶



#### 找基点的方法

1. 计算椭圆曲线的阶$N$ (Schoof's algorithm: [https://en.wikipedia.org/wiki/Schoof%27s_algorithm](https://en.wikipedia.org/wiki/Schoof's_algorithm))
2. 选择一个阶为$n$的子群。n必须是素数且必须是$N$的因子
3. 计算辅因子 $h = N/n$
4. 在曲线上选择一个随机的点 $T$
5. 计算$G = hT$，点乘
6. 如果$G = 0$, 返回4， 否则找到基点 $G$, 子群的阶为 $n$, $h$ 被称为辅因子



**原理**:  根据拉格朗日定理，$n$ 整除 $N$ 且 $n$ 为质因子，且任意点 $T$  满足$NT = 0$， 则：$n(hT) = 0$ 恒成立, 那么若$hT\ \ne 0$，则 $hT$ 作为基点的阶一定为$n$. ($n$ 一定是素数, 否则不成立)



### 曲线上的离散对数问题

对于曲线上的基点 $G$， 已知 $n$ 计算 $P = nG$ 是容易的

但是已知$P, G$, 计算 $n$ 是很困难的 



## ECDH & ECDSA

### ECDH

1. CA选用共同曲线，并下发相同基点$G$，其阶数为 $n$, 则私钥的取值范围为$d \in \{1, ..., n - 1\}$
2. Alice随机选择私钥$d_A$，计算 Pubkey: $P_A = d_AG$, 通过非安全信道传递给Bob
3. Bob随机选择私钥$d_B$，计算 Pubkey: $P_B = d_BG$，通过非安全信道传递给Alice
4. Alice和Bob分别计算$S = d_AP_b = d_BP_A = d_Ad_BG$， 共享秘密成功



秘密共享成功后可以每次通信时明文传递salt, 每次通过 $key = KDF(salt + S)$，得到具体通信对称秘钥，加密通讯(TLS/SSL)

通过服务器动态生成的ECDH一般称作**ECDHE**



### ECDSA

定义依然继承上文，$n$ 为 $G$ 作为基点的子群阶数

定义 $bit(x)$ 为表示 $x$ 需要的比特数；注意计算DSA时，若摘要值的比特数 $bits(digest(plain\_test)) > bits(n)$，则需要截取摘要值的低 $bits(n)$ 进行签名.

#### 符号标记

截取前n-bits函数 ： $trun_{bit(n)}(digest)$

截取后的摘要值：$z = trun_{bit(n)}(digest(plain\_test))$，$digest$ 需要选择安全摘要算法：内部要求**SHA-256**以上

私钥：$d$

公钥：$P = dG$



#### 签名算法

1. 随机选择 $k \in \{1, ..., n -1 \}$
2. 计算$T = kG = (T_x, T_y)$
3. 计算数字 $r = T_x \mod n$， 若$r = 0$则返回1
4. 计算$s = k^{-1}(z + rd) \mod n$, 如果$s = 0$，返回1

5. 最后签名：$(r, s)$



#### 验证算法

1. 计算 $u_1 = s^{-1}z \mod n$
2. 计算 $u_2 = s^{-1}r \mod n$
3. 计算 $T' = u_1G\ +\ u_2P$

若$T'_x = r \mod n$，验签成功，否则失败



#### 正确性

我们尝试计算的其实还是$T = kG$，若此时 $z$ 是正确摘要值，则有:

$k = s^{-1}(z + rd)\ mod\ n\ =>\ k = s^{-1}z + s^{-1}rd \mod n$

带入上式  $T = s^{-1}zG + s^{-1}rdG = u_1G + u_2(dG) = u1G + u_2P$

所以若 $z$ 发生改变，则此时计算出来的 $T'_x\ \ne\ r \mod n$



#### 随机数相等下的私钥复原

若每次取出的随机数 $k$ 都相等：

获取两份签名与摘要：$z_1, (r_1, s_1)$ 和 $z_2, (r_2, s_2)$



容易得到: $r_1 = r_2 = (kG)_x \mod n$

之后通过 $s_1 - s_2$ 计算 $k$：

$s_1 - s_2 = k^{-1}(z_1 + rd - z_2 - rd) \mod n$

$=> k = (z_1 - z_2)(s_1 - s_2)^{-1} \mod n$

之后计算 $d$ 就很简单了:

$ d = r^{-1}(s_1k - z_1) \mod n$



#### 通过签名恢复公钥

若已知曲线上 $x = r$ 对应的两点 $R, R'$，则可以从签名$(s, r)$中恢复公钥$P$:

$s = k^{-1}(z + rd) \mod n$

$=> skG = (z + rd)G$

注意 $kG = R$ 或者 $kG = R'$, 分别作为备选带入上式，同时$P = dG$:

$=>\ sR - zG = r(dG)\ => P = r^{-1}(sR - zG)$

or $=>\ P = r^{-1}(sR' - zG)$



##### 实现方法

**点压缩**：增加2bit来标识，一个用来标识 $R_x = r\mod n$ 或者 $R_x = r$，另一个标识$R_y$是基数还是偶数：

因为$R, R'$关于 $x$ 轴对称，$R_y + R'_y = 0 \mod p$， 所以$R_y ,P'_y$为一基一偶，可用一个bit标识

这样可以达到多用增加一个byte(04标记等)，来达成无需传递公钥即可验签



相关算法与代码：https://busy.org/@oflyhigh/397bw1



#### 伪造签名

##### 构造e方法

1. 随机选择 $a, b \in \{1, ... n\}$, 计算$T = aG + bP, r = T_x$
2. 计算 $s = rb^{-1}, e = arb^{-1}$
3. 若$e$ 为伪造摘要值, 可伪造合法签名 $(r, s)$



**正确性**:

$u_1 = s^{-1}e \mod \ n$

$u_2 = s^{-1}r \mod\ n$

将$s, e$带入

$u_1G + u_2P = (rb^{-1})^{-1}(arb^{-1})G + (rb^{-1})^{-1}rP = (rr^{-1})(bb^{-1})aG + (rr^{-1})bP = aG + bP = T$

由于$r = T_x \mod n$，校验通过



相关算法与代码：https://github.com/GoldSaintEagle/ECDSA-SM2-Signing-Attack



## SM2签名

标记不变，$z = SM3(message)$ 是消息的摘要值（国密要求摘要使用SM3），$d$ 是私钥， $P = dG$是公钥

### 签名算法

1. 随机选择 $k \in \{1, ..., n -1 \}$
2. 计算$T = kG = (T_x, T_y)$
3. 计算数字 $r = T_x + z \mod n$， 若$r = 0$则返回1
4. 计算$s = (1 + d)^{-1}(k - rd) \mod n$, 如果$s = 0$，返回1

5. 最后签名：$(r, s)$



### 验证算法

1. 计算消息值摘要$z' = SM3(message)$
2. 计算$T' = sG + (r + s)P$
3. 判断$r\ ?= T'_x + z \mod n$



### 正确性

首先计算 $k$ :

$s = (1 + d)^{-1}(k - rd) \mod n$ 

=> $s(1 + d) + rd = k \mod n$ => $s + (s + r)d = k \mod n$

所以 ：

$T = kG = sG + (s + r)(dG) = sG + (s + r)P = T'$

因此可以推导：

$r = (T_x + z) = (T'_x + z) \mod n$

所以如果$r = T'_x + z' \mod n$， 则 $z'$ 验签通过，否则 $z'$ 摘要有误