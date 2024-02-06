https://mp.weixin.qq.com/s?__biz=MzU5MzMxNTk2Nw==&mid=2247486055&idx=1&sn=a0bd7d530ceeb74a3cbb5fa10466bd9d&chksm=fe131b77c964926173afd7459f4a7d6512ebe0cfb58e9a90b731ac8d6d8df9cc139260d95a72&scene=21#wechat_redirect

https://mp.weixin.qq.com/s?__biz=MzU5MzMxNTk2Nw==&mid=2247486744&idx=1&sn=26425829ffedf25e9cf2652eb2dd24cd&chksm=fe131c08c964951ee9ce7204425b0f38a15a87e09da9fa44219ca3b53f2572dffe34904f8b60&scene=178&cur_album_id=1458661849167511555#rd



# Groth16算法介绍

看 zk-SNARK 的文章或者资料的时候，经常会碰到一些算法名称，比如 Groth16 ，GGPR13 等等。这些名称是由算法提出的作者名称或者名称首字母以及相应的年份组成。Groth16，是由 Jens **Groth** 在 20**16** 年提出的算法。GGPR13 ，是由Rosario **G**ennaro，Craig **G**entry，Bryan **P**arno，Mariana **R**aykova在20**13**年提出的算法。

零知识证明（zk-SNARK )，从 QSP/QAP 到 Groth16，期间也有很多学者专家，提出各种优化（优化计算时间，优化证明的大小，优化电路的尺寸等等）。Groth16 提出的算法，具有非常少的证明数据（ 2/3 个证明数据）以及一个表达式验证。

Groth16 论文（On the Size of Pairing-based Non-interactive Arguments）的下载地址：https://eprint.iacr.org/2016/260.pdf

本文主要从工程应用理解的角度介绍Groth16算法的证明和验证过程。文章中所用的中文字眼可能和行业中不一样，欢迎批评指出。



**术语介绍**

**Proofs** - 在零知识证明的场景下，Proofs指具有完美的完备性（Completeness）以及完美的可靠性（Soundness）。也就是，具有无限计算资源也无法攻破。

**Arguments** - 在零知识证明的场景下，Arguments是指具有完美的完备性以及多项式计算的可靠性。也就是，在多项式计算能力下，是可靠的。

**Schwartz-Zippel 定理** - 假设![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYclHzPwWtRzfKvPsAias3CwaMOJqMdxNzCibtpVLKpd8WwScSwUJMYuoVA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)是个n元多项式，多项式总的阶为d。如果![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcUY0icNfYEcOVPDEHBEGLs4CbC2uBe21GvqtlKiaWkFibK7BoboQ3XmhQg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)是从有限集合S中随机选取，则![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcznrq94rLVaX7uAR1Xw0ssOXb2Wd4RffIm5knhkXfVMNeYwLIdMJgug/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)的概率是小于等于![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcuGIZTwIwZDFvajtpaYH6Vgb8dSyoJI4ESic1ibwqEzqpeqdv8VLLJplw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)。简单的说，如果多元多项式，在很大的集合中随机选取参数，恰好函数f等于0的概率几乎为0。

https://brilliant.org/wiki/schwartz-zippel-lemma/

**线性（Linear）函数** - 假设函数f满足两个条件：**1.** ![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcCicQVdayicgrKEjjQa0uUP6LwBiaXRdrlWWDcJicic8Ufy6xQC3IKPN1Z1A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1) **2.**![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcpZrsWUg3lylRpRAzwUx9WfXdgcVaMut4Y7dWIdFgql972oUqCthb4g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)，则称函数f为线性函数。

**Affine 函数** - 假设函数g，能找到一个线性函数f，满足![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYctTuSqtU2RaoEOd9LibTpGPqOBI3icuuvbwtabHXmve3qtbAw7icJx66nQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)，则称函数g为Affine函数。也就是，Affine函数是由一个线性函数和偏移构成。

**Trapdoor函数** - 假设一个Trapdoor函数f，![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYc2Tjpd2oicWn4DG5B08mAcMQCBc37W9khwGib2YrIam3UbMibbWBLgL99A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)很容易，但是![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcrX0Q5VSjAWFokUg0bWHV4XYILvw4zPxrgOWI8ennsOxC02rvXnpMoA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)非常难。但是，如果提供一个secret，![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcrX0Q5VSjAWFokUg0bWHV4XYILvw4zPxrgOWI8ennsOxC02rvXnpMoA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)也非常容易。





**Jens Groth是谁？**

Groth 是英国伦敦 UCL 大学的计算机系的教授。伦敦大学学院 (University College London），简称 UCL ，建校于 1826 年，位于英国伦敦，是一所世界著名的顶尖高等学府，为享有顶级声誉的综合研究型大学，伦敦大学联盟创始院校，英国金三角名校，与剑桥大学、牛津大学、帝国理工、伦敦政经学院并称 G5 超级精英大学。

http://www0.cs.ucl.ac.uk/staff/j.groth

Groth 从 2009 年开始，每年发表一篇或者多篇密码学或者零知识证明的文章，所以你经常会听到 Groth09 ，Groth10 等等算法。

简言之，牛人～。





**NILP**

Groth16 的论文先引出 NILP（non-interactive linear proofs）的定义：

1. ![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcZicV437GgPicp9Kj6H3pXzELte7BGFy311byrDosibusFupMQ3ia5nTsaw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1) ：设置过程，生成![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcgNjVqIeBNNtr3GCQhYuwiclIZwuS2SPeeuRloGM6o6ZNEP0EMLZgaQw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1), ![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYc3oWfF72fwGayjQx3FiafNxOYfojELnlCQmM2pkiauNjkjxTDzVyG76zQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)。
2. ![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcQ7RjbXCwuU9ntpS6b6BVtnKzwVf3soZ7NB4fCLmYLgAUQISmOIsd5Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1) ：证明过程，证明过程又分成两步：a. 生成线性关系![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcnpFczGhVB66iaj0tVZW7K4doH1eVumbwwGsRQ258IhOIiaBQbkHKZbpQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)，其中ProofMatrix是个多项式算法。b. 生成证明：![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcpm5gcxpYkuH3Q9PrzFmpYK9FcdZ5L68qTvxQdwzcytaEABiblUYoicwA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)。
3. ![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcCrmAaAL6xDdooQoqnrzjTQCwmU0VBkOZ1MNXuTOhWxFV1aOe9iaY7PA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1) ：验证过程，验证者使用![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYc4X7BlHcmicHv3KEzaR0up4SdeMVz8ZIQCdAz1GUxt78ZUlaDRfxMzAg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)生成电路t，并验证![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcM9r5EiaLH0qm2Kub7ia05Pf7DnuzIoYJqj3WANmwrqLEemFicNDbaQTRA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)是否成立。

在 NILP 定义的基础上，Groth16 进一步定义了 split NILP ，也就是说，CRS 分成两部分![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcIW4Xofiaaiap6SKM6MZhXdre32yCQfAe1HonrTPUNu68iaxU61ae9dL3g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)，证明者提交的证明也分成两部分![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcZZEuVezcagjOFXZCaCYmVUNLNYQ8GiaURM9PxYQSRmIguw1zzG6mcLA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)。

总的来说，核心在 “ Linear ” 上，证明者生成的证明和 CRS 成线性关系。



***\*QAP的NILP\****



QAP的定义为"**R**elation"：![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcyttKcTkWbCKrHHdXxumGfX6lq2dhB5juFxibYcJkWZmOTQZPz9icLbEw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)。也就是说，statements为![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcRB7qGXgUFGmPfbicJXbGpSSMicmwylcuVWVm8hiadoKbraFPyKCpWmYjw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)， witness为![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcic3lOU91LP5ALY4g45o2ankibXoEZRqRxlGf6zKcpoCfAxEnJxVnBocQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)，并且![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcvuKwfD769uyMib3hOyzjib5w0gVjsssOKWeTpm9NthXhbKeic6Xpdwdow/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)的情况下，满足如下的等式：

![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcMHUibqiaqaFcBtvTjibDNAAVicYomFhPhk9dzmSrPIfJjq5lrlFZ1ib6r3w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcsBvbucRpQzzMfBBic6FqOur1Ru5WicPHtQBEr4bmoQD85fzd1wLElCicg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)的阶为n。

1. 设置过程：随机选取![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcC132nVYicibdKqWwuJiaoOib5l0oOf5CIlFtex6SPQUHysODS6dYbkeYUQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)，生成![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYc5icqwFicVlicFb8UrRPGwJHrUUicK09YicAKibPF2XD0jnpg0yEaIEN6btfg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)。

    ![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcS6G7ydUwM59O5GbEOCGBfQ1S1agews6kU7pqkG2BLQRjWzRiagnJhnQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

    ![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYc4j8kQXgPS3CcicgoXGa85JyA7UGMUiaDSCccYM5zpCQt8ia4KzsURBAUw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

2. 证明过程：随机选择两个参数![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYc52Fcic6bCsS7yXaJqcibAsvpwGuDDp6wq0Btkaib2PwkxQkXb56XUI8OA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)，计算![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcqUtyLNfOs733R81xu2dXMMiaBI08YuUQHDs3X4sGlplBUWKTBmrmNqA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

    ![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcNDIffFjkX39qj3ea3PFYQHkibPVdq6QKGic3NvacnDbAewoOM1nRwkSQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

    ![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYciaNKNEgKkkpZ70L9ctOmfQEuR8PvHviaicOHheBxicS9zx3JF2hGuxxbiag/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

    ![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcQUNq7H1cHyOVCyF1f8x6Qdz1Ck22wBQLH9ydfwZy29oAuM9ogyfNMw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

3. 验证过程：

    验证过程，计算如下的等式是否成立：

    ![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcQWpFHa85KV35aEJcQk6C338zTcicrZMjnbIObR1awxv7Fq28wGTRr4Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

注意，设置过程中的x是一个值，不是代表多项式。在理解证明/验证过程的时候，必须要明确，A/B/C的计算是和CRS中的参数成线性关系（NILP的定义）。在明确这一点的基础上，可以看出![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcPUKGlwLBVJP0dVAhcDHsERcSXy3Xiauv77F9w1DKF4bWes1XGwp4xXA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)的参数能保证A/B/C的计算采用统一的![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcoAsEeqh3xs5NJuX6x1ccbN1q821wEfcfNvHZQwTKAxxg03WicRxSpfw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)参数。因为![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYc7ncm8uDTHIEt1icCSBNrdWNL1M1tiaJ6cNRXTd6w1s1l0o8qQiaBibHnaA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)会包含![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYctk9FOz6vwGUyJYuqZCvLWKSF54KC9r9PlVECOv7Xrc8mNL5iaqXjlPw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)子项，要保证![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYc7ncm8uDTHIEt1icCSBNrdWNL1M1tiaJ6cNRXTd6w1s1l0o8qQiaBibHnaA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)和C相等，必须采用统一的![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcoAsEeqh3xs5NJuX6x1ccbN1q821wEfcfNvHZQwTKAxxg03WicRxSpfw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)参数。参数![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYc52Fcic6bCsS7yXaJqcibAsvpwGuDDp6wq0Btkaib2PwkxQkXb56XUI8OA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)增加随机因子，保证零知识（验证者无法从证明中获取有用信息）。参数![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcbKmsibjdOxyJFPOcT3DqVs4NSphs5MbEDgM3bxtd59YqGQHbzRibAjgg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)保证了验证等式的最后两个乘积独立于![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcPUKGlwLBVJP0dVAhcDHsERcSXy3Xiauv77F9w1DKF4bWes1XGwp4xXA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)的参数。

**完备性证明（Completeness)：**
完备性证明，也就是验证等式成立。

![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcqeQibbTa3y1w6V8nLDEfsW7Gr2bwJFpNlRB7iamhhr9NOibYUKzZlTnjQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcwLf9icluAAJcLiaZgX3qhf9lPDdDyogahyKdgcvNNXO0elraSuxWd8aQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)
![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYckg3podcneWNhytmPefkNbh0EudULqibwHZiagB813gIeVxcrM6hsNh9A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)
![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcFicictmee4d7kUBRkACaDaJt5iacAzcJib2mPibfCIpyibicd4TZXsUYMuJiaQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcsx5ZvNGYOGdsZE2pWSVsaEdqbqX3RkQl71krJzmrgsGJXLGjJZYy8g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYczYJoZhpnIVb5k4zruKn8BbX29bOvWiaI5HjdO6HicibXPhyat5iaLMRicsQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcFicictmee4d7kUBRkACaDaJt5iacAzcJib2mPibfCIpyibicd4TZXsUYMuJiaQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcCQfazjMs5icsyKjXiaY1mbuPRpNt7zbzLcRMFButtw1XiaicJttgFsTYKw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcsx5ZvNGYOGdsZE2pWSVsaEdqbqX3RkQl71krJzmrgsGJXLGjJZYy8g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYctV7iaxr1YMMDFRmM8kleZIQmUu1RgwAuMkV5e4XJ2icm6oZ9icBQKGMaA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYceMXZOsHRCIpkh3Jwyr2Ab2Bkdyce96BOS8j6JFu7wEOs7wYq0Q2YjQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**可靠性证明 (Soundness):**

Groth16算法证明的是**statistical knowledge soundness**，假设证明者提供的证明和CRS成**线性**关系。也就是说，证明A可以用如下的表达式表达(A和CRS的各个参数成线性关系）：

![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYczw0XPqqSxRmKFxic5IBsVWQrcKqSWbNmBI7UFuic8E03xpl52Qb390UQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

同理，B/C都可以写成类似的表达：

![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcUd6dK04xqxpqgjRgYnj62W5pbmhtVRYKIsG9ZqiaPdp7UyyulSRlYeA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcakuKYllfZ1mD7TOrzUulsf3oTy2Pd2FcoTG1CpfnJ9RfmCNlCQEoGw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

从Schwartz-Zippel 定理，我们可以把A/B/C看作是![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcaNJybAStn4Bu41pFRCoZHXDVKOiaIl9HNficlLP3xRqxoEBjAUE7GpicQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)的多项式。观察![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcQWpFHa85KV35aEJcQk6C338zTcicrZMjnbIObR1awxv7Fq28wGTRr4Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)这个验证等式，发现一些变量的限制条件：

1）![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYc1Gh82obD6a8soibf4wHN6xURAGO1wqVaRgqgjoFoGd85k5YCaPxb6Qw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1) (等式的右边没有![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcRlYQGicfEwibEOOWkk8X7y8ia6x9XShrDoHW2xR1B1Ml7LmicYZPCXFjmg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1))

不失一般性，可以假设![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcXaS2LXfr0bBydRnUiajOy9EXdaCib7n748uCsz3QnVZt5k4Zklq9bOUg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)。

2）![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcbgy8CbFgnacn5lE19CoScauNSIGvfHd5JPtNMBxRTWeoPqP1pwnjeA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1) （等式右边![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcusHwDQp3DpQEAv91aVLsHRBaZUMicicLBSSXbiaUqjTFG00CAkl6epZKQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)）

不失一般性，可以假设![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcp0QOxvalPvuyF6VtTNDT6HibwHgVqbNdNbicUaiaU7qm7cNTslQsMlcRQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)。

3）![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcfkGYMs1RVicOm1yW1ty95ZZBoe58tsrtpxSx3g9cjXA42qFsj6NfgRQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)(等式的右边没有![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcYVmxC6qO19h7L1KPOves9SoLnNKdNYQwxNwDtlc0b29Bw37h6TWicKg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)因子)

也就是![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYc354ptT7CIS8Ov8icFicNcmMcNPFibb3hfXb7QOUe6U1u9xU9yBia5txY2w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)。

在上述三个约束下，A/B的表达式变成：

![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYclVSSwV6S8UCtByuou4VMPicSIr5dB3YVuuvdkLms9ynlUHiaibuInhN4w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcemGVwRkj1jJZUI2P3iamZk4aqwrOFTGZ2icuNOxribT5A5gxEibXuutF2Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

4）等式的右边没有![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcYuibCtuMT8zaiaQrhHf2RcV71lmDiaQicOe0ET2V0bjYHXX53HOoibqOyVA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcrrHV0S3gGPDGb17p4p3HVv0v6cHZ3wD78GfoSmWYDVdj3AuAFyHuvQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

不失一般性，![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYc5pN35hbfAMyNHk2l8QXOwunrZ8zWic6ZEfWQlLibqySud3Krhm96vjeQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

5）等式的右边没有![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcr8EbCfRP6u0s2thaiapwH4yEOzZhGkpd7uhhIcfsyicT3Re9r2HFDCxA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcV1z7UONzTaO8LwyH0dZnzVNibuEjoHmfQSzBVLramBpPJQ3TcR6libeg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

不失一般性，![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYco2icD8Nu8eCa3oQL58YzMMtb9SEtSGsyGurnjGVE1rPWwZZ2Y7EVC7Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)。

6）等式的右边没有![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcZ6sKfbYxIRBNUvsjusoJbBacZDAFHnicQxxq9ibxYkCeJqjZPaeMbbfg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)， ![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYccVvqTxNZecYaOhoBd8jrp1KbhwzxSd6Biaibr9Ql6tZ4UibhGfUMyawqQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcrXGbFfok6RmiayaSicBZlZPWkVHlY6s9IyWDMSxMYJQuyRk9jHGvjX6g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYckxGgxpmbZo9g1xTfa6NE63QOWjbzFTvQ55ibXhIzIbxOPYYM9H0VP3Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

所以 

![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYc5jQiaUNn5CIQgclOF5Ix3FdibujboZdyK0tvxSyXfwicuZpTpsoePNaIw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

7）等式的右边没有![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYc4NTn08fY6vJbzxeib97pm7g0ichhBml7Qvfo4icAliatNtxNdia7WqOpjSA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)和![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcYoIU7qlznM3yBFx7CtKmibfGHraibCVEE8QWyVlZqlRlaicq7DMianiccLA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYceic5y9HF3nCL9TcSt9wgLr9evnIn6ibk6kU3qB51Krr8WfTZRpYQqqQA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

所以，![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcbElorIuqxtiblAia7ENEEwiaDZkXYCBlaE0BJdQZib5mPrpvfh72KteVqA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)。

在上述七个约束下，A/B的表达式变成：

![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcoJRm6mRJWfHSJbibic3fG7NzurfIq4uZyuiaEWWovO85DxzC3o0w8e6Hw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcAaVZIgcpGPnsRVY6NSMmC8jIQiauE3iaEicC4jYP0N6Apdg6DY1giaFiaDw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

再看验证的等式：

![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcQWpFHa85KV35aEJcQk6C338zTcicrZMjnbIObR1awxv7Fq28wGTRr4Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcNppy2E4waYVhdeGG0KUCXI9AibNcJLh6L4xWD16HnknWNuAzTKXiaeXg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

观察![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcicCQeHMAocbeiakOhTKe5Tsw1EibibrscwhiaVcezs8vOic1hHS3RKCNaDew/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)，因为不存在 ![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcpzuW6KUMounCcbMEiaXAFn0vBg5BTJ4M7A7rG4y9TAflDk7s330xiaPw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)，所以，![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcYu9rKiaXWVMY5UTiaXTygJUKzLvibjM6GBQDq4yVdxgbWhvSlko4ILXZA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)。

也就是说

![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcX7St3xbZbrgpyPdHAYPAM8B1vRew2olH0YVboFJaA32V3RjD0Zkgrg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

代入验证等式，所以可以推导出：

![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYc2r7dmfE0QNOGdR75KCooQMLtpl0wqYm2b4UsuJicr0Tkf7nqdwvUebQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)，

![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYctwjSE3KLjpspToKgmB6oW6YIaWO8LnzIrT7Mj5flDwiaFlqyoibC8HqQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

如果，假设，对于![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYca9RvEPiaEM5grPa3xVIcs8JjWLHJaOxQNyT2BEKwr2zxZju1SP4vFEw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)，则

![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYc1HG1WI977Kia93DUrSmKDjE4XWA3BNvFt3AcNdUaSK43ibQAGDFubl6Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcxJp8oAlB9CLoibetf5zyXFz5HFpLT9Y3WuYJ1Dd98BNIEQkAClLfdBQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

代入A/B，可以获取以下等式：

![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcEuAhjn3KnXoaxcyl2FicRmlvahibAsbKfEAkuibevb81YaZo4cDK8VcSA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

在证明和CRS线性关系下，所有能使验证等式成立的情况下，![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcmMgMOfPZKibUKGicE7hDkOqiaXc8bC0TStdROeaHn0rtjibmG9NqcYlIag/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)等式必须成立。也就说，能提供正确证明的，肯定知道witness。





**QAP的NIZK Arguments**

从QAP的NILP可以演化为QAP的NIZK Arguments。也就是说Groth16算法并不是完美的可靠，而是多项式计算情况下可靠。QAP的定义为"**R**elation"

![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcTLWicJGdGicWaxMF8BpeXhJ2csjZ7dXm0ZxafPzqT6IzthkcmI99FDzA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

也就是说，在一个域![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcqJna9HWCCrnUeKkw2VDYExn4F8zhtkZupQNXcV8UodQy3DUS5tSMnQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)中，statements为![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcicyqDvSib1ux9ic4fc5LAp4gtYG2KqfGxFedIADdeSiazDsyibuIzyWkgpw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)， witness为![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcvcUtAc7DjicDbnoC4Zicv6RM3Icl0xpjWwxx3EfibuIBTURia3mDSS0Nhg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)，并且![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcvuKwfD769uyMib3hOyzjib5w0gVjsssOKWeTpm9NthXhbKeic6Xpdwdow/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)的情况下，满足如下的等式（![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcsBvbucRpQzzMfBBic6FqOur1Ru5WicPHtQBEr4bmoQD85fzd1wLElCicg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)的阶为n）：

![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcMHUibqiaqaFcBtvTjibDNAAVicYomFhPhk9dzmSrPIfJjq5lrlFZ1ib6r3w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

也就是说，三个有限群![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcYNy6acTwVTmfeAEmQicHiaB4ne5Baf3QXp23FQEw1boicXSpT9B5GgJrA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1), 对应的生成元分别是![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcPCzJSyA0Qvbll2rickcXCViagL51fXPALR9pymTDXnlnTEn7AHI94aRg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)。为了方便起见，也为了和论文的表达方式一致，![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcwtHvAkUBvibhChAmMIs3qibBryMicChhJA76ubIDgVEM7bQOAbxA5rktw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)的计算用![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcL2nRPULcMUUqhU6x8ibztsjaD0X9pNoLiaw5rhCmvL8MNXSvVtYfszibw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)表示，![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcIiaM78Ifx8Fe6S1LJibLYAWjRgW6YNdbHhf0qdoHr5fjiauiciaFCaBibe3A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)的计算用![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcib8pHicQKzhjJP0valtHgWtsTxSPcibH4QXMwL4n2YUkEHS5A1EAFNQgQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)表示。

1. 设置过程：随机选取![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcyansvQfK9lLIKweWKf1ZNVs0vrtvM4xZC1IeIRCy4SUGGnNt7w4FibA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)，生成![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYc5icqwFicVlicFb8UrRPGwJHrUUicK09YicAKibPF2XD0jnpg0yEaIEN6btfg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)。

    ![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcS6G7ydUwM59O5GbEOCGBfQ1S1agews6kU7pqkG2BLQRjWzRiagnJhnQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

    ![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcibLAmiaIGicAqIo8venxQZ2Q0b55n8gZvR3pLaGcj7TaO0vqY1V2bI6GQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

    ![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcbicCVYXW24ssD3VPxjlGD6vAZIaZxQF3Vfm5OnucLXsAVmuA7w91UVw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

    ![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYccw1Kh9L58fDiawxeAwibjqFmj8ql8eAqDu57kObjdPKIYlvIP4KUkz2A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

2. 证明过程：随机选择两个参数![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYc52Fcic6bCsS7yXaJqcibAsvpwGuDDp6wq0Btkaib2PwkxQkXb56XUI8OA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)，计算![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcZIemSLWCNnkeyibus38qQgz16w4zttZ2tI3gMdibzanxM09zS1r0J1xw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

    ![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcNDIffFjkX39qj3ea3PFYQHkibPVdq6QKGic3NvacnDbAewoOM1nRwkSQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

    ![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYciaNKNEgKkkpZ70L9ctOmfQEuR8PvHviaicOHheBxicS9zx3JF2hGuxxbiag/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

    ![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcQUNq7H1cHyOVCyF1f8x6Qdz1Ck22wBQLH9ydfwZy29oAuM9ogyfNMw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

3. 验证过程：验证如下的等式是否成立。

    ![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYc3Xvib6LyCkj2VCnMaMCHu5IDHKpR1DUavQUrIib1b8oSWr2plElIEq1Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

很容易发现，验证过程的等式也可以用4个配对函数表示:

![Image](https://mmbiz.qpic.cn/mmbiz_png/IXicdJl0t7b7xky9DE9xpSyzZUNpYMwYcibF4rqkFIxsiaRFz5RRCESvBxrrdrUClM4gtzHgrtTXmpvhK5N3X4Rpw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

证明过程和QAP的NILP的证明过程类似，不再详细展开。





证明元素的最小个数

论文指出zk-SNARK的最少的证明元素是**2个**。上述的证明方式是需要提供3个证明元素（A/B/C）。论文进一步说明，如果将电路进行一定方式的改造，使用同样的理论，可以降低证明元素为2个，但是，电路的大小会变的很大。

**总结**：Groth16算法是Jens Groth在2016年发表的算法。该算法的优点是提供的证明元素个数少（只需要3个），验证等式简单，保证完整性和多项式计算能力下的可靠性。Groth16算法论文同时指出，zk-SNARK算法需要的最少的证明元素为2个。目前Groth16算法已经被ZCash，Filecoin等项目使用。