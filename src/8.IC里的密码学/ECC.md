https://www.bilibili.com/video/BV1v44y1b7Fd

https://www.bilibili.com/video/BV1CG4y1A7kD

https://tlu.tarilabs.com/cryptography/elliptic-curves

https://andrea.corbellini.name/2015/05/17/elliptic-curve-cryptography-a-gentle-introduction

https://andrea.corbellini.name/2023/01/02/ec-encryption

https://andrea.corbellini.name/2015/05/23/elliptic-curve-cryptography-finite-fields-and-discrete-logarithms



https://mp.weixin.qq.com/s?__biz=MzA5NzI4MzkyNA==&mid=2247483725&idx=1&sn=a2a26dc15833209a0bd3252d174a1738&scene=21#wechat_redirect

https://mp.weixin.qq.com/s?__biz=MzA5NzI4MzkyNA==&mid=2247483718&idx=1&sn=494dd987eea23a4165c57bb78bfd61ad&scene=21#wechat_redirect

https://mp.weixin.qq.com/s?__biz=MzA5NzI4MzkyNA==&mid=2247483730&idx=1&sn=a903cf4769c1f1ab7b8c9e71c6557b74&scene=21#wechat_redirect

https://mp.weixin.qq.com/s?__biz=MzA5NzI4MzkyNA==&mid=2247483735&idx=1&sn=a71d67f33e11c119f51ea4b4ae61e81e&scene=21#wechat_redirect

https://mp.weixin.qq.com/s?__biz=MzA5NzI4MzkyNA==&mid=2247483740&idx=1&sn=b0c565359af600ea97e736e689c14ce6&scene=21#wechat_redirect

https://mp.weixin.qq.com/s?__biz=MzU5MzMxNTk2Nw==&mid=2247486862&idx=1&sn=38b326ce8d694617252e58ea3f0c3a3c&chksm=fe131c9ec96495888fe990458b5f164440a4ca386db93904ce30ce5096344d17083daab9030e&scene=21#wechat_redirect



https://blog.csdn.net/mutourend/article/details/90643302

https://mp.weixin.qq.com/s?__biz=MzA5NzI4MzkyNA==&mid=2247483762&idx=1&sn=fc0088ab15a875d1d8e5a1d1136f8a0a&scene=21#wechat_redirect







Ed25519：https://learnblockchain.cn/article/1663

EdDSA签名机制：https://learnblockchain.cn/article/1697

https://mp.weixin.qq.com/s?__biz=MzA5NzI4MzkyNA==&mid=2247484218&idx=1&sn=0e50fc6a3278ca783c54f332dc52ac26&scene=21#wechat_redirect

https://datatracker.ietf.org/doc/html/rfc8032#section-2



 介绍

Edwards曲线数字签名算法(EdDSA)是Schnorr签名系统的一种变体,使用(可能扭曲的)Edwards曲线。
EdDSA需要用某些参数实例化,本文档描述了一些推荐的变体。

为了促进EdDSA在互联网社区的采用,本文档以面向实现的方式描述了该签名方案,并提供了示例代码和测试向量。

EdDSA的优点如下:

1. EdDSA在各种平台上都具有高性能;

2. 每个签名不需要使用唯一的随机数;

3. 对侧信道攻击的抵抗力更强;

4. Ed25519和Ed448分别只使用32个或57个字节的小公钥和64个或114个字节的签名;

5. 公式是“完整的”,即对曲线上的所有点都有效,没有例外。这消除了EdDSA对不可信公钥执行昂贵的点验证的需要; 

6. EdDSA提供碰撞抵抗能力,这意味着散列函数碰撞不会破坏该系统(仅对PureEdDSA有效)。

原始的EdDSA论文[EDDSA]和“用于更多曲线的EdDSA”[EDDSA2]中 generalized 版本提供了进一步的背景信息。 RFC 7748 [RFC7748] 讨论了特定的曲线,包括Curve25519 [CURVE25519] 和 Ed448-Goldilocks [ED448]。

Ed25519的目的是在128位安全级别左右运行,而Ed448的目标是在224位安全级别左右运行。足够大的量子计算机将能够破解两者。经典计算机能力的合理预测得出结论,Ed25519是完全安全的。 Ed448是为那些对性能要求较低并希望对椭圆曲线分析攻击进行避险的应用程序提供的。

符号和约定

全文使用以下符号:

p 表示定义基础字段的素数

GF(p) 具有p个元素的有限字段

x^y x乘以自身y次

B 感兴趣的群或子群的生成元

[n]X 将X加到自己n次

h[i] 八位字节串h的第i个八位字节

h_i h的第i位

a || b (比特)字符串a连接(比特)字符串b 

a <= b a小于或等于b

a >= b a大于或等于b

i+j i和j的和

i*j i和j的乘积

i-j 从i中减去j

i/j i除以j

i x j i和j的笛卡尔积

(u,v) 椭圆曲线点,x坐标为u,y坐标为v

SHAKE256(x, y) 输入x的SHAKE256 [FIPS202]输出的前y个八位字节

OCTET(x) 值为x的八位字节

OLEN(x) 字符串x中的八位字节数

dom2(x, y) 签名或验证Ed25519时为空八位字节串。否则,八位字节串:“SigEd25519 no Ed25519 collisions” || octet(x) || octet(OLEN(y)) || y,其中x的范围是0-255,y是一个最多255个八位字节的八位字节串。 “SigEd25519无Ed25519冲突”为ASCII(32个八位字节)。

dom4(x, y) 八位字节串“SigEd448” || octet(x) || octet(OLEN(y)) || y,其中x的范围是0-255,y是一个最多255个八位字节的八位字节串。 “SigEd448”为ASCII(8个八位字节)。

括号(即“(”和“)”)用于分组表达式,以避免描述依赖于运算符之间的绑定顺序。

比特字符串通过从左到右获取比特,将每个八位字节中从最低有效位到最高有效位打包,并在每个八位字节填满时移动到下一个八位字节,转换为八位字节字符串。 从八位字节字符串到比特字符串的转换是这一过程的逆过程; 例如,16位比特字符串 

b0 b1 b2 b3 b4 b5 b6 b7 b8 b9 b10 b11 b12 b13 b14 b15

以此顺序转换为两个八位字节x0和x1:

x0 = b7*128+b6*64+b5*32+b4*16+b3*8+b2*4+b1*2+b0

x1 = b15*128+b14*64+b13*32+b12*16+b11*8+b10*4+b9*2+b8

将位打包到左侧的位中的小端编码。如果与上面定义的比特字符串到八位字节字符串的转换组合,则会导致小端编码到八位字节(如果长度不是8的倍数,最后一个八位字节的最高有效位仍未使用)。

关键字“MUST”、“MUST NOT”、“REQUIRED”、“SHALL”、“SHALL NOT”、“SHOULD”、“SHOULD NOT”、“RECOMMENDED”、“MAY”和“OPTIONAL”在本文档中按[RFC2119]中描述的方式解释。

3. EdDSA算法

EdDSA是一个有11个参数的数字签名系统。

不打算直接实现带有11个输入参数的通用EdDSA数字签名系统。 参数选择对于安全和高效操作至关重要。 相反,您将实现EdDSA的特定参数选择(如Ed25519或Ed448),有时略微推广以涵盖Ed25519和Ed448,从而实现代码重用。

因此,对通用EdDSA的精确解释对实现者实际上不是特别有用。 为了背景和完整性,这里简要描述了通用EdDSA算法。

某些参数的定义,例如n和c,可以帮助解释算法中一些不直观的步骤。

此描述紧跟[EDDSA2]。

EdDSA有11个参数:

1. 一个奇素数幂p。 EdDSA使用定义在有限字段GF(p)上的椭圆曲线。

2. 一个整数b,其中2^(b-1)> p。 EdDSA公钥正好有b位,EdDSA签名正好有2*b位。 建议b是8的倍数,以便公钥和签名长度是一个整数个八位字节。

3. GF(p)元素的(b-1)位编码。

4. 一个输出2*b位的密码哈希函数H。建议使用保守的哈希函数(即无法创建碰撞的哈希函数),它对EdDSA的总成本影响不大。

5. 一个整数c,值为2或3。 密钥EdDSA标量是2^c的倍数。 整数c是所谓共因子的以2为底的对数。

6. 一个整数n,其中c <= n < b。 密钥EdDSA标量正好有n + 1位,最高位(2^n位置)始终设置,最低c位始终清零。

7. GF(p)的一个非平方元素d。 常见的建议是取最接近零的使曲线可接受的值。

8. GF(p)的一个非零平方元素a。 如果p mod 4 = 1,则获得最佳性能的常见建议是a = -1; 如果p mod 4 = 3,则a = 1。

9. 一元素B != (0,1),属于集合E = { (x,y) 是GF(p) x GF(p)的成员,使得a * x^2 + y^2 = 1 + d * x^2 * y^2}。

10. 一个奇素数L,使得[L]B = 0和2^c * L = #E。 数字#E(曲线上的点数)是椭圆曲线E的标准数据的一部分,也可以计算为共因子*阶。

11. 一个“预哈希”函数PH。 PureEdDSA是EdDSA,其中PH是恒等函数,即PH(M)= M。 HashEdDSA是EdDSA,其中PH生成短输出,无论消息的长度如何; 例如,PH(M)= SHA-512(M)。

曲线上的点在加法下形成一个群,(x3,y3)=(x1,y1)+(x2,y2),其公式为

x1 * y2 + x2 * y1                y1 * y2 - a * x1 * x2
x3 = --------------------------,   y3 = ---------------------------
1 + d * x1 * x2 * y1 * y2          1 - d * x1 * x2 * y1 * y2

群中的中性元素是(0,1)。

与用于密码应用的许多其他曲线不同,这些公式是“完整的”; 对曲线上的所有点都有效,没有例外。 特别是,对所有输入点,分母都不为零。

存在更高效的公式,仍然是完整的,使用齐次坐标来避免对昂贵的模p求逆。 参见[Faster-ECC]和[Edwards-revisited]。

3.1. 编码

整数0 < S < L - 1以小端形式编码为b位字符串ENC(S)。

E的元素(x,y)编码为称为ENC(x,y)的b位字符串,这是y的(b-1)位编码与一个位串联,如果x为负则该位为1,如果x非负则为0。

使用GF(p)的编码来定义GF(p)的“负”元素:具体地,当x的(b-1)位编码在字典顺序上大于-x的(b-1)位编码时,x为负。

3.2. 密钥

EdDSA私钥是一个b位字符串k。 令散列H(k)=(h_0,h_1,...,h_(2b-1))确定一个整数s,s是2^n加上对所有整数i,c<= i <n,m = 2^i * h_i的总和。 令s确定倍数A = [s]B。 EdDSA公钥是ENC(A)。 下面签名时使用了位h_b,...h_(2b-1)。

3.3. 签名

使用私钥k对消息M的EdDSA签名定义为消息PH(M)的PureEdDSA签名。 描述得简单一点,EdDSA只是使用PureEdDSA对PH(M)进行签名。

使用私钥k对消息M的PureEdDSA签名是2*b位字符串ENC(R) || ENC(S)。 R和S的导出如下。 首先,将2*b位字符串按小端形式解释为整数{0,1,...,2^(2*b)-1},定义r = H(h_b || ... || h_(2b-1) || M)。 令R = [r]B和S = (r + H(ENC(R) || ENC(A) || PH(M)) * s) mod L。这里使用的是前一节中的s。

3.4. 验证

要在公钥ENC(A)下验证消息M上的PureEdDSA签名ENC(R) || ENC(S),请执行以下操作。 解析输入,以便A和R是E的元素,S是{0,1,...,L-1}的成员。 计算h = H(ENC(R) || ENC(A) || M),并在E中检查群方程[2^c * S]B = 2^c * R + [2^c * h]A。 如果解析失败(包括S超出范围)或群方程不成立,则拒绝签名。

消息M的EdDSA验证定义为PH(M)的PureEdDSA验证。

https://segmentfault.com/a/1190000019172260

https://blog.unclezs.com/pages/9760d3/#%E4%BB%8B%E7%BB%8D

https://javaguide.cn/system-design/security/encryption-algorithms.html#rsa
