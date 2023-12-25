# 椭圆曲线(Elliptic Curve)的神奇性质
## 椭圆曲线的应用
### 签名
利用椭圆曲线的签名算法叫做 ECDSA (Elliptic Curve Digital Signature Algorithm)。
#### 给任何消息签名
https://app.mycrypto.com/sign-message
![[Pasted image 20231225103113.png]]
```
{
	"address": "0x32197ff73E5A309f51Bd94F1728DbFE2BED452b0",
	"msg": "Hello EC!",
	"sig": "0xc79ad7b9a57fdd9b0f1e3c7dd7910f3042a57848d51f399caa8c755c80bd03267d9d9cdf2d10361a0d5cc2096c74837bc33c7dea3369982b405f963bbb1806921c",
	"version": "2"
}
```

![[Pasted image 20231225103220.png]]
参考 https://medium.com/mycrypto/the-magic-of-digital-signatures-on-ethereum-98fe184dc9c7

#### 给以太坊交易签名
![[Pasted image 20231225103617.png]]
给交易签名本质上也是签署一个特定的信息。只不过可以把签署后的交易发到区块链上，链上签名验证通过后，可以执行相关的代码，比如转账等。

### 秘钥交换
参考 https://andrea.corbellini.name/2015/05/30/elliptic-curve-cryptography-ecdh-and-ecdsa/

利用椭圆曲线的秘钥交换叫做 ECDH (Elliptic curve Diffie-Hellman)。

过程：
1. 首先，Alice 和 Bob 生成他们自己的私钥和公钥
2. Alice 和 Bob 通过不安全的通道交换他们的公钥
3. Alice 计算 $S=d_AH_B$ ，Bob 计算  $S=d_BH_A$
![[Pasted image 20231225104601.png]]

S 就是获得的共享秘钥，只有他们两人知道，之后他们就可以用 S 进行加密和解密。

Code:

```python
# Alice generates her own keypair.

alice_private_key, alice_public_key = make_keypair()

print("Alice's private key:", hex(alice_private_key))

print("Alice's public key: (0x{:x}, 0x{:x})".format(*alice_public_key))

  

# Bob generates his own key pair.

bob_private_key, bob_public_key = make_keypair()

print("Bob's private key:", hex(bob_private_key))

print("Bob's public key: (0x{:x}, 0x{:x})".format(*bob_public_key))

  

# Alice and Bob exchange their public keys and calculate the shared secret.

s1 = scalar_mult(alice_private_key, bob_public_key)

s2 = scalar_mult(bob_private_key, alice_public_key)

assert s1 == s2

  

print('Shared secret: (0x{:x}, 0x{:x})'.format(*s1))
```

Result:
```shell
Alice's private key: 0x4f637def8cd2bffd769fedf79f65656780e1701f9b64d26ea8a5c6729e02ea16

Alice's public key: (0x4a1ae99a3451ba1fadfdf7b4705c3568cd5ecc5c4c21e9f86ce3ae5166fc2d25, 0xabb6fdaaadc42a83ee3bd761af7950bbe81624da7153af2e538a1538ad8aff55)

Bob's private key: 0xc64fda9723d8a7380d0a78becf2f161e41cecc61bdd39795bd94dc5894f81382

Bob's public key: (0xfb47290bd33fdbd133b40bd4df078cb2231e9a7854cb18dc10fd183f7cedac55, 0xf20e82a1b020f4c2ca712d11c1ed9b64fc7e60df657c5e0223a48c79dd6809fa)

Shared secret: (0x6e7b7773764d62ed36000f09c7db1b66a5341d29c2c10c395ded5d5ec97eb9cf, 0x507aea85dee2ba199692b0a5ff3fb017e5f4179b2012065a9912a8f7851056d1)
```

### 费马大定理
悬而未决350多年的著名数学难题“费马猜想”(即“费马大定理”)，就是由英国数学家Andrew Wiles(现为美国普林斯顿大学教授)于1994年应用椭圆曲线的理论而彻底解决的。

## 椭圆曲线密码学(ECC) 的优势

|RSA key size (bits)|ECC key size (bits)|
|---|---|
|1024|160|
|2048|224|
|3072|256|
|7680|384|
|15360|521|

ECC 使用更少的内存，而且密钥生成和签名的速度要快得多。不过RSA 密钥大小和 ECC 密钥大小之间不存在线性关系。

## 椭圆曲线——“一根神线”
![[Pasted image 20231225110236.png]]
![[Pasted image 20231225110326.png]]

椭圆曲线的理论及其应用作为现代数论中的一个分支学科，可以说是集纯粹性、优美性、挑战 性、应用性、实用性为一体的一个“突出例子”，如果说“数论是数学的皇后”(高斯的名言)的话，那么椭圆曲线理论就是皇后的皇冠上的一颗闪亮的“明珠”.

## 实数域上的椭圆曲线
定义：
$y^2 = x^3 + ax + b$ 
其中 $4a^3 + 27b^2 \neq 0$
再加上一个无穷远点，也可以理解为 0 点
A + 0 = 0 + A = A

![[Pasted image 20231225111045.png]]

### 群(Group)
之所以介绍群，是因为群有很多优美的性质，可以把椭圆曲线定义成一个群，这样椭圆曲线就可以直接利用群的性质。

群的性质:
![[Pasted image 20231225111721.png]]

比如，在数的乘法下，所有正有理数和正实数都分别构成 一个乘法群，但所有正整数不能构成一个群，因为性质 (3)不成立.

再比如，在数的加法下，所有整数、所有有理数，所有实数，和所有复数都分别构成一个加法群(Additive Group).

如何在椭圆曲线上定义一个群：
- 该群的元素是椭圆曲线的点；
- 单位元是无穷远 0 处的点；
- 点 P 的倒数是关于 x 轴对称的点；
- 加法由以下规则给出：给定三个对齐的非零点 P、Q、R ，它们的总和为 P + Q + R = 0 。

![[Pasted image 20231225112226.png]]

### 计算
#### 加法的几何计算法

![[Pasted image 20231225112310.png]]


P + Q + R = 0 => P + Q = -R
通过 P 和 Q 绘制直线。该线与椭圆曲线第三个点 R 相交。与之对称的点 −R 是 
P+Q 的结果。

#### 加法的代数算法
![[Pasted image 20231225112620.png]]
![[Pasted image 20231225112636.png]]
例子：

曲线 $y^2 = x^3 - 7x + 10$ 

P = (1, 2), Q = (3, 4)
![[Pasted image 20231225112851.png]]

P + Q = -R = (-3, 2)

验证发现这个点也在椭圆曲线上。

Try https://andrea.corbellini.name/ecc/interactive/reals-add.html
#### 乘法：Scalar multiplication 标量乘法

椭圆曲线不支持点和点之间的乘法( P * Q)，只支持标量乘法：
![[Pasted image 20231225113316.png]]
其中 n 为自然数。

标量乘法可以转换为加法。

不过直接加的算法复杂度是 $O(2^k)$（如果 n 是 k 个二进制数）。

优化: **double and add** 算法

例如 n=151 。其二进制表示为 $10010111_2$ 。这种二进制表示可以转化为 2 的幂和
![[Pasted image 20231225113703.png]]
![[Pasted image 20231225113727.png]]

七次加倍和四次加法

## 有限域上的椭圆曲线
![[Pasted image 20231225114033.png]]

p 为素数。

有限域上的计算：
![[Pasted image 20231225114608.png]]

有限域上的椭圆曲线:

![[Pasted image 20231225114041.png]]

### 计算
#### 加法：几何上
![[Pasted image 20231225114121.png]]

#### 加法：代数上
和实数域上的公式是一样的，只不过需要 $\mod p$
try https://andrea.corbellini.name/ecc/interactive/modk-add.html
### 椭圆曲线的阶(order)
椭圆曲线的阶：点的数量

### 标量乘法和循环子群
标量乘法和实数域一样：
![[Pasted image 20231225113316.png]]
其中 n 为自然数。

  
有限域中椭圆曲线的点相乘（标量乘法）有一个有趣的特性。选取曲线 $y^2≡x^3+2x+3$(mod 97) 和点 P=(3,6) 。现在计算 P 的所有倍数：
![[Pasted image 20231225115038.png]]
![[Pasted image 20231225115059.png]]

这构成了只包含 5 个元素的循环子群，P 叫做 generator 或者 base point。

循环子群是 ECC 和其他密码系统的基础。

循环子群的阶（order）n，是使得 nP = 0 的最小的正整数 n。

椭圆曲线算法就是在循环子群上工作的，相关的参数:
![[Pasted image 20231225115926.png]]
**domain parameters** 一共包含上面的 6 个 tuple $(p, a, b, G, n, h)$

Code:
https://github.com/andreacorbellini/ecc/blob/master/scripts/ecdhe.py
```python
import collections
import random

EllipticCurve = collections.namedtuple('EllipticCurve', 'name p a b g n h')

curve = EllipticCurve(
    'secp256k1',
    # Field characteristic.
	p=0xfffffffffffffffffffffffffffffffffffffffffffffffffffffffefffffc2f,
    # Curve coefficients.
    a=0,
    b=7,
    # Base point.
    g=(0x79be667ef9dcbbac55a06295ce870b07029bfcdb2dce28d959f2815b16f81798,
       0x483ada7726a3c4655da4fbfc0e1108a8fd17b448a68554199c47d08ffb10d4b8),
    # Subgroup order.
    n=0xfffffffffffffffffffffffffffffffebaaedce6af48a03bbfd25e8cd0364141,
    # Subgroup cofactor.
    h=1,
)
```

### 如何寻找循环子群的阶
  
 P 的阶数通过拉格朗日定理与椭圆曲线的阶数联系起来，该定理指出子群的阶数是父群阶数的除数。换句话说，如果椭圆曲线包含 N 个点，并且其子群之一包含 n 点，则 n 是 N 的除数。

找到 n 的方法：
1. 使用 Schoof 算法计算椭圆曲线的阶数 N
2. 找出 N 的所有除数
3. 对于每个除数 n，计算 nP
4. 使得 nP = 0 最小的 n 是子群的阶

### 如何寻找 generator
对于我们的 ECC 算法，我们需要高阶子组。所以一般我们会选择一条椭圆曲线，计算它的阶数（ N ），选择一个高除数作为子群阶数（ n ）并最终找到合适的基点。也就是说：我们不会选择一个基点，然后计算它的阶数，而是相反：我们会先选择一个看起来足够好的阶数，然后再寻找合适的基点。

拉格朗日定理意味着数字 $h = N/n$ 始终是整数（因为 n 是 N 的除数）。数字 h 有一个名称：它是子群的辅因子(**cofactor of the subgroup**)。

对于椭圆曲线上的每个点 P，都有 NP = 0

则：n(hP) = 0

G = hP 就是我们要找的 generator（如果 G 不为 0）

寻找 generator方法：
1. 计算椭圆曲线的阶 N
2. 选择子群的 order n。为了使算法正常工作，该数字必须是素数并且必须是 N 的除数。
3. 计算  $h = N/n$ 
4. 在曲线上选择一个随机点 P
5. 计算 G = hP 
6. 如果 G 为 0，返回步骤 4，如果 G 不为 0，则 G 就是 generator

注意，此算法仅在 n 为素数时才有效，因为如果不是素数，就表明 n 有除数，就无法保证 n 是使得 nP = 0 的最小的正整数 n。