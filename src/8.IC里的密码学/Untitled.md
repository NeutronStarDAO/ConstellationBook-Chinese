https://juejin.cn/post/7118221301986115591





# **解释一下：**

# **Basics of Polynomials for Cryptography**

* **polynomials**
* Mar 16, 2020

A **polynomial** ϕ of **degree** d is a vector of d+1 **coefficients**:

ϕ=[ϕ0,ϕ1,ϕ2,…,ϕd](1)

For example, ϕ=[1,10,9] is a degree 2 polynomial. Also, ϕ′=[1,10,9,0,0,0] is also a degree 2 polynomial, since the zero coefficients at the end do not count. But ϕ′′=[1,10,9,0,0,0,1] is a degree 6 polynomial, since the last non-zero coefficient is ϕ6=3.

*“A list of numbers? That makes no sense!”* Don’t panic! You are probably more familiar to polynomials expressed as function of a variable X:

ϕ(X)=ϕ0+ϕ1⋅X+ϕ2⋅X2+⋯+ϕd⋅Xd]=∑i=0d+1ϕiXi(2)(3)

For example, ϕ=[1,10,9] and ϕ(X)=9X2+10X+1 are one and the same thing.

**Note:** The degree is defined as the index i of the last non-zero coefficient: deg(ϕ)=i s.t. ∀j>i,ϕj=0.

## **The basics of polynomials**

### **Roots of polynomials**

We say z is a *root* of ϕ(X) if ϕ(z)=0. In this case, ∃q(X) such that ϕ(X)=q(X)(X−z).

But what if z is also a root q(X)? We can capture this notion as follows: we say z has a *multiplicity* k if ∃q′(X) such that ϕ(X)=q′(X)(X−z)k.

### **The polynomial remainder theorem**

This theorem says that:

ϕ(a)=y⇔∃q(X),ϕ(X)=q(X)(X−a)+ϕ(a)(4)

This property is leveraged by certain cryptosystems**1**.

### **Dividing polynomials**

Division of polynomials conceptually resembles division of integers.

Specifically, dividing a polynomial a(X) by b(X) gives a **quotient** q(X) and a **remainder** r(X) such that:

a(X)=q(X)b(X)+r(X)(5)

Importantly, degr<degb and, if dega≥degb, then degq=dega−degb. Otherwise, q(X)=0.

NZ



Here is a brief explanation of some basics of polynomials that are relevant for cryptography:

* A polynomial is a mathematical expression involving a sum of terms, each term consisting of a constant (called a coefficient) multiplied by variables raised to whole number powers. For example, 1 + 2x + 3x^2 is a polynomial.
* The degree of a polynomial is the highest power of the variable(s) in the polynomial. In the above example, the degree is 2.
* Polynomials can have roots - values of the variable(s) that make the polynomial equal to zero. For example, x=1 is a root of the polynomial x^2 - 2x - 3.
* The polynomial remainder theorem relates evaluation of a polynomial at a value to polynomial division. It says that dividing a polynomial by (x-a) results in a remainder that is equal to evaluating the original polynomial at x=a.
* Polynomials can be divided similar to integer division, resulting in a quotient polynomial and a remainder polynomial of lower degree. This is useful for tasks like polynomial factorization.
* Polynomials are widely used in cryptography, such as in RSA which relies on properties of modular polynomial arithmetic. Factoring polynomials is also an important hard mathematical problem leveraged in cryptosystems.

Copy



请作为老师，用尽可能通俗易懂的、口语化的方式给不了解它的人详细讲解以下内容，可以用比喻、类比的手法解释难懂的概念，如果能达到让一个高中生听懂的地步就很好了：

NZ



好的,我来用通俗的语言为你解释一下多项式在密码学中的基本知识。

多项式就像是一个变量x的指数函数,它由不同次方项的系数相加而成。比如2 + 5x + 3x^2,这里2是零次项x^0的系数,5是一次项x^1的系数,3是二次项x^2的系数。整个函数就叫二次多项式,因为最高次方项是2。

多项式可以有根 - 使函数等于0的x的值。就像二次方程ax^2 + bx + c = 0有两个根,多项式也可以有一个或多个根。找到这些根就像是解方程一样。

多项式除法跟整数除法类似,用大多项式除以小多项式,可以得到商多项式和余数多项式。商多项式次数等于大多项式次数减去小多项式次数。这在分解多项式时很有用。

多项式在密码学中很重要,比如RSA算法用了模多项式运算的性质。多项式分解也是一个硬的数学问题,可以用来设计密码系统。

密码学里还用到了多项式余数定理。这个定理说,用一个数除多项式,余数就是用这个数代入多项式得出的值。就像小学学过的9除以2的余数是1,因为9/2=4余1,代入得到4*2+1=9。

总之,多项式是变量的指数函数,可以做除法产生商和余数,可以有根,也可以应用在密码系统中。希望这样通俗的解释可以让你对多项式的基本概念有所了解。如果例子不太明白可以再举一些生活中的类比,请你提出任何疑惑,我会继续解释。

Copy



请作为老师，用尽可能通俗易懂的、口语化的方式给不了解它的人详细讲解以下内容，可以用比喻、类比的手法解释难懂的概念，如果能达到让一个高中生听懂的地步就很好了：**Basics of Polynomials for Cryptography**

* **polynomials**
* Mar 16, 2020

A **polynomial** ϕ of **degree** d is a vector of d+1 **coefficients**:

ϕ=[ϕ0,ϕ1,ϕ2,…,ϕd](1)

For example, ϕ=[1,10,9] is a degree 2 polynomial. Also, ϕ′=[1,10,9,0,0,0] is also a degree 2 polynomial, since the zero coefficients at the end do not count. But ϕ′′=[1,10,9,0,0,0,1] is a degree 6 polynomial, since the last non-zero coefficient is ϕ6=3.

*“A list of numbers? That makes no sense!”* Don’t panic! You are probably more familiar to polynomials expressed as function of a variable X:

ϕ(X)=ϕ0+ϕ1⋅X+ϕ2⋅X2+⋯+ϕd⋅Xd]=∑i=0d+1ϕiXi(2)(3)

For example, ϕ=[1,10,9] and ϕ(X)=9X2+10X+1 are one and the same thing.

**Note:** The degree is defined as the index i of the last non-zero coefficient: deg(ϕ)=i s.t. ∀j>i,ϕj=0.

## **The basics of polynomials**

### **Roots of polynomials**

We say z is a *root* of ϕ(X) if ϕ(z)=0. In this case, ∃q(X) such that ϕ(X)=q(X)(X−z).

But what if z is also a root q(X)? We can capture this notion as follows: we say z has a *multiplicity* k if ∃q′(X) such that ϕ(X)=q′(X)(X−z)k.

### **The polynomial remainder theorem**

This theorem says that:

ϕ(a)=y⇔∃q(X),ϕ(X)=q(X)(X−a)+ϕ(a)(4)

This property is leveraged by certain cryptosystems**1**.

### **Dividing polynomials**

Division of polynomials conceptually resembles division of integers.

Specifically, dividing a polynomial a(X) by b(X) gives a **quotient** q(X) and a **remainder** r(X) such that:

a(X)=q(X)b(X)+r(X)(5)

Importantly, degr<degb and, if dega≥degb, then degq=dega−degb. Otherwise, q(X)=0.

NZ



好的,多项式在密码学中的基础知识我来给你详细解释一下:

首先,多项式就是一个变量x的指数函数,由不同次方项的系数相加构成。比如2 + 5x + 3x^2,这里的2、5、3就是系数,x的0次方、1次方、2次方构成了这个二次多项式。多项式的最高次方项决定了它的次数,这也叫多项式的度。

多项式可以有根,就是使多项式等于0的x值。找到根就像是解方程找出变量的值一样。如果一个数是多项式的根,而且这个根在分解后的多项式中还是根,那么这个根的重数就是它在分解多项式中出现的次数。

多项式余数定理说明,用一个数去除多项式,最后得到的余数就是用这个数代入多项式计算的值。就像我们小学学习的余数定理一样。

多项式除法类似整数除法,被除数÷除数=商,还有一个余数。多项式除法中,余数的次数一定低于除数的次数。商的次数等于被除数的次数减去除数的次数。

多项式在密码学中很重要,比如RSA算法用了模多项式运算的特性。多项式分解也是数学难题,可以应用在密码系统中。

总之,多项式是Variable的指数函数,可以除法得到商和余数,有根的概念,在密码学中有广泛应用。我尽量用通俗易懂的语言和生活中的类比解释这些概念,如果你还有不明白的地方请随时提问,我会继续用更简单的方式解释。

Copy



现在请进一步解释，不用太通俗易懂。但是要专业、具体、详细：

# **Basics of Polynomials for Cryptography**

* **polynomials**
* Mar 16, 2020

A **polynomial** ϕ of **degree** d is a vector of d+1 **coefficients**:

ϕ=[ϕ0,ϕ1,ϕ2,…,ϕd](1)

For example, ϕ=[1,10,9] is a degree 2 polynomial. Also, ϕ′=[1,10,9,0,0,0] is also a degree 2 polynomial, since the zero coefficients at the end do not count. But ϕ′′=[1,10,9,0,0,0,1] is a degree 6 polynomial, since the last non-zero coefficient is ϕ6=3.

*“A list of numbers? That makes no sense!”* Don’t panic! You are probably more familiar to polynomials expressed as function of a variable X:

ϕ(X)=ϕ0+ϕ1⋅X+ϕ2⋅X2+⋯+ϕd⋅Xd]=∑i=0d+1ϕiXi(2)(3)

For example, ϕ=[1,10,9] and ϕ(X)=9X2+10X+1 are one and the same thing.

**Note:** The degree is defined as the index i of the last non-zero coefficient: deg(ϕ)=i s.t. ∀j>i,ϕj=0.

## **The basics of polynomials**

### **Roots of polynomials**

We say z is a *root* of ϕ(X) if ϕ(z)=0. In this case, ∃q(X) such that ϕ(X)=q(X)(X−z).

But what if z is also a root q(X)? We can capture this notion as follows: we say z has a *multiplicity* k if ∃q′(X) such that ϕ(X)=q′(X)(X−z)k.

### **The polynomial remainder theorem**

This theorem says that:

ϕ(a)=y⇔∃q(X),ϕ(X)=q(X)(X−a)+ϕ(a)(4)

This property is leveraged by certain cryptosystems**1**.

### **Dividing polynomials**

Division of polynomials conceptually resembles division of integers.

Specifically, dividing a polynomial a(X) by b(X) gives a **quotient** q(X) and a **remainder** r(X) such that:

a(X)=q(X)b(X)+r(X)(5)

Importantly, degr<degb and, if dega≥degb, then degq=dega−degb. Otherwise, q(X)=0.

NZ



好的,我来更专业和详细地解释一下多项式在密码学中的基础知识:

1. 多项式定义

多项式φ是一个由系数向量(φ0,φ1,...,φd)表示的d次多项式。其中,φi是x^i项的系数。一个多项式可以写成函子形式φ(x)=φ0+φ1x+...+φdx^d。多项式的最高次项决定了多项式的次数d,也叫度。

1. 多项式的根

如果对于某个复数z,代入多项式后使其等于0,即φ(z)=0,则称z是多项式φ的一个根。根据FACTOR定理,如果z是根,则φ(x)可以分解为φ(x)=(x-z)q(x)的形式。

如果一个根z在分解后的q(x)中仍是根,且重复了k次,则说这个根z的重数是k。

1. 多项式余数定理

多项式余数定理指出:对于多项式φ(x)和常数a,有φ(a)=y 当且仅当存在某个多项式q(x),使得 φ(x) = q(x)(x-a) + y成立。这里y就是多项式φ(x)除以x-a的余数。

1. 多项式除法

多项式之间可以进行除法运算,运算过程类似整数除法。设多项式a(x)除以b(x),可以得到商多项式q(x)和余数多项式r(x),且deg(r)<deg(b)。如果deg(a)>=deg(b),则deg(q)=deg(a)-deg(b)。

1. 密码学应用

多项式在 RSA 加密算法等多方面有重要应用。比如基于分解大整数和分解多项式难度的公钥密码体制。

以上是对多项式基础知识在密码学方面的详细和专业阐述。请您指出还需要补充解释的部分,我很乐意继续详细解释。

Copy



请作为老师，用尽可能通俗易懂的、口语化的方式给不了解它的人详细讲解以下内容，可以用比喻、类比的手法解释难懂的概念，如果能达到让一个高中生听懂的地步就很好了：

# **Lagrange interpolation**

* **polynomials**
* **lagrange**
* Jul 28, 2022

Recall from **our basics discussion** that a **polynomial** ϕ of **degree** d is a vector of d+1 **coefficients**:

ϕ=[ϕ0,ϕ1,ϕ2,…,ϕd](1)

## **How to compute a polynomial’s coefficients from a bunch of its evaluations**

Given n pairs (xi,yi)i∈[n], one can compute or *interpolate* a degree ≤n−1 polynomial ϕ(X) such that: ϕ(xi)=yi,∀i∈[n]

Specifically, the *Lagrange interpolation* formula says that:

ϕ(X)=∑i∈[n]yi⋅Li(X), where Li(X)=∏j∈[n],j≠iX−xjxi−xj(2)

This formula is intimidating at first, but there’s a very simple intuition behind it. The key idea is that Li(X) is defined so that it has two properties:

1. Li(xi)=1,∀i∈[n]
2. Li(xj)=0,∀j∈[n]∖{i}

You can actually convince yourself that Li(X) has these properties by plugging in xi and xj to see what happens.

**Important:** The Li(X) polynomials are dependent on the set of xi’s only (and thus on n)! Specifically each Li(X) has degree n−1 and has a root at each xj when j≠i! In this sense, a better notation for them would be L[xi,n]i(X) or L[n]i(X) to indicate this dependence.

## **Example: Interpolating a polynomial from three evaluations**

Consider the following example with n=3 pairs of points. Then, by the Lagrange formula, we have:

ϕ(X)=y1L1(X)+y2L2(X)+y3L3(X)(3)

Next, by applying the two key properties of Li(X) from above, you can easily check that ϕ(xi)=yi,∀i∈[3]:

ϕ(x1)ϕ(x2)ϕ(x3)=y1L1(x1)+y2L2(x1)+y3L3(x1)=y1⋅1+y2⋅0+y3⋅0=y1=y1L1(x2)+y2L2(x2)+y3L3(x2)=y1⋅0+y2⋅1+y3⋅0=y2=y1L1(x3)+y2L2(x3)+y3L3(x3)=y1⋅0+y2⋅0+y3⋅1=y3(4)(5)(6)

An **important detail** is that the degree of the interpolated ϕ(X) is ≤n−1 and not necessarily exactly equal to n−1. To see this, consider interpolating the polynomial ϕ(X) such that ϕ(i)=i for all i∈[n]. In other words, xi=yi=i.

The inspired reader might notice that the polynomial ϕ(X)=X could satisfy our constraints. But is this what the Lagrange interpolation will return? After all, the interpolated ϕ(X) is a sum of degree n−1 polynomials Li(X), so could it have degree 1? Well, it turns out, yes, because things cancel out. To see this, take a simple example, with n=3:

ϕ(X)=∑i∈[3]i⋅Li(X)=∑i∈[3]i⋅∏j∈[3]∖iX−ji−j=1⋅X−21−2X−31−3+2⋅X−12−1X−32−3+3⋅X−13−1X−23−2=X−2−1X−3−2+2⋅X−11X−3−1+3⋅X−12X−21=12(X−2)(X−3)−2(X−1)(X−3)+32(X−1)(X−2)=12[(X−2)(X−3)+3(X−1)(X−2)]−2(X−1)(X−3)=12[(X−2)(4X−6)]−2(X−1)(X−3)=(X−2)(2X−3)−2(X−1)(X−3)=(2X2−4X−3X+6)−2(X2−4X+3)=(2X2−7X+6)−2X2+8X−6=X(7)(8)(9)(10)(11)(12)(13)(14)(15)(16)

## **Computational overhead of Lagrange interpolation**

If done naively, interpolating ϕ(X) using the Lagrange formula in Equation **2** will take O(n2) time.

However, there are known techniques for computing ϕ(X) in O(nlog2n) time. We described **part of** these techniques in a **previous blog post**, but for the full techniques please refer to the *“Modern Computer Algebra”* book**1**.

NZ



好的,我来用通俗易懂的方式解释拉格朗日插值法:

拉格朗日插值法实际上就是从几个点求出一个多项式的方法。举个简单的例子,假设我们有三个点:(1,2),(2,5),(3,7),现在我想求出一个二次多项式,使其过这三个点。

根据拉格朗日公式,这个多项式可以写成:

f(x) = 2L1(x) + 5L2(x) + 7L3(x)

这里L1(x)、L2(x)、L3(x)就是拉格朗日基函数,它有两个重要性质:

(1) Li(xi) = 1,即在己定点xi处的值为1 (2) Li(xj) = 0,对其它点xj处的值为0

这样就保证了f(1)=2,f(2)=5,f(3)=7。

为什么有这样的性质呢,这就涉及到拉格朗日基函数的构造方法了。L1(x) 是这样构造的:

L1(x) = (x-x2)/(x1-x2) * (x-x3)/(x1-x3)

你可以验证一下,这样构造出来的L1(x)确实满足上述两个性质。L2(x)、L3(x)也是类似构造。

所以拉格朗日插值法实际上就是构造出具有特定性质的多项式,从而达到通过给定点进行曲线拟合的目的。通俗地说,就是找出一个多项式,使其曲线过给定的所有点。这在很多实际问题中都有应用。

我用通俗的语言和简单例子解释了拉格朗日插值法的基本思想。如果例子理解起来还有困难,请你提出来,我会想办法进一步解释,确保这个概念对你来说足够清楚。

Copy



现在请进一步解释，不用太通俗易懂。但是要专业、具体、详细：

# **Lagrange interpolation**

* **polynomials**
* **lagrange**
* Jul 28, 2022

Recall from **our basics discussion** that a **polynomial** ϕ of **degree** d is a vector of d+1 **coefficients**:

ϕ=[ϕ0,ϕ1,ϕ2,…,ϕd](1)

## **How to compute a polynomial’s coefficients from a bunch of its evaluations**

Given n pairs (xi,yi)i∈[n], one can compute or *interpolate* a degree ≤n−1 polynomial ϕ(X) such that: ϕ(xi)=yi,∀i∈[n]

Specifically, the *Lagrange interpolation* formula says that:

ϕ(X)=∑i∈[n]yi⋅Li(X), where Li(X)=∏j∈[n],j≠iX−xjxi−xj(2)

This formula is intimidating at first, but there’s a very simple intuition behind it. The key idea is that Li(X) is defined so that it has two properties:

1. Li(xi)=1,∀i∈[n]
2. Li(xj)=0,∀j∈[n]∖{i}

You can actually convince yourself that Li(X) has these properties by plugging in xi and xj to see what happens.

**Important:** The Li(X) polynomials are dependent on the set of xi’s only (and thus on n)! Specifically each Li(X) has degree n−1 and has a root at each xj when j≠i! In this sense, a better notation for them would be L[xi,n]i(X) or L[n]i(X) to indicate this dependence.

## **Example: Interpolating a polynomial from three evaluations**

Consider the following example with n=3 pairs of points. Then, by the Lagrange formula, we have:

ϕ(X)=y1L1(X)+y2L2(X)+y3L3(X)(3)

Next, by applying the two key properties of Li(X) from above, you can easily check that ϕ(xi)=yi,∀i∈[3]:

ϕ(x1)ϕ(x2)ϕ(x3)=y1L1(x1)+y2L2(x1)+y3L3(x1)=y1⋅1+y2⋅0+y3⋅0=y1=y1L1(x2)+y2L2(x2)+y3L3(x2)=y1⋅0+y2⋅1+y3⋅0=y2=y1L1(x3)+y2L2(x3)+y3L3(x3)=y1⋅0+y2⋅0+y3⋅1=y3(4)(5)(6)

An **important detail** is that the degree of the interpolated ϕ(X) is ≤n−1 and not necessarily exactly equal to n−1. To see this, consider interpolating the polynomial ϕ(X) such that ϕ(i)=i for all i∈[n]. In other words, xi=yi=i.

The inspired reader might notice that the polynomial ϕ(X)=X could satisfy our constraints. But is this what the Lagrange interpolation will return? After all, the interpolated ϕ(X) is a sum of degree n−1 polynomials Li(X), so could it have degree 1? Well, it turns out, yes, because things cancel out. To see this, take a simple example, with n=3:

ϕ(X)=∑i∈[3]i⋅Li(X)=∑i∈[3]i⋅∏j∈[3]∖iX−ji−j=1⋅X−21−2X−31−3+2⋅X−12−1X−32−3+3⋅X−13−1X−23−2=X−2−1X−3−2+2⋅X−11X−3−1+3⋅X−12X−21=12(X−2)(X−3)−2(X−1)(X−3)+32(X−1)(X−2)=12[(X−2)(X−3)+3(X−1)(X−2)]−2(X−1)(X−3)=12[(X−2)(4X−6)]−2(X−1)(X−3)=(X−2)(2X−3)−2(X−1)(X−3)=(2X2−4X−3X+6)−2(X2−4X+3)=(2X2−7X+6)−2X2+8X−6=X(7)(8)(9)(10)(11)(12)(13)(14)(15)(16)

## **Computational overhead of Lagrange interpolation**

If done naively, interpolating ϕ(X) using the Lagrange formula in Equation **2** will take O(n2) time.

However, there are known techniques for computing ϕ(X) in O(nlog2n) time. We described **part of** these techniques in a **previous blog post**, but for the full techniques please refer to the *“Modern Computer Algebra”* book**1**.

NZ



拉格朗日插值法允许从n个点(xi,yi)中插值出一个度<=n-1的多项式φ(x),使得φ(xi)=yi, i∈[n]成立。

其插值公式为:

φ(x) = ∑yi⋅Li(x) ,  i∈[n]

其中Li(x)为拉格朗日基函数:

Li(x) = ∏j∈[n],j≠i (x - xj)/(xi - xj)

拉格朗日基函数的两个重要性质:

1. Li(xi) = 1, 对每个i∈[n]
2. Li(xj) = 0, 对每个j∈[n], j≠i

这保证了φ(x)通过每个点(xi,yi)。

例如,给定3个点(x1,y1),(x2,y2),(x3,y3),则有:

φ(x) = y1L1(x) + y2L2(x) + y3L3(x)

可以验证φ(x1)=y1,φ(x2)=y2,φ(x3)=y3。

注意插值多项式的度不一定等于n-1,取决于点(xi,yi)的分布。当xi与yi对应时,φ(x)=x的情况。

直接计算φ(x)的时间复杂度为O(n^2),可用快速傅里叶变换降至O(nlogn)。

以上是对拉格朗日插值法的详细数学描述。请指出需要补充解释的部分,我会进一步阐述具体过程与证明。

Copy





TXT

paste.txt

22.56 KB

请作为老师，用尽可能通俗易懂的、口语化的方式给不了解它的人详细讲解以下内容，可以用比喻、类比的手法解释难懂的概念，如果能达到让一个高中生听懂的地步就很好了：

NZ



好的,我来用简单的语言为你解释配对(pairing)在密码学中的作用和应用。

首先,配对就是一个数学函数,它把两个不同的椭圆曲线群映射到一个目标群。举个简单比喻,配对就像一个翻译官,可以把两国人民的语言翻译到一种“通用语言”。

配对有两个很酷的性质:

1. 双线性 - 如果我们把群元素进行某种运算,配对的结果也相应地运算。就像翻译官可以准确翻译语言中的加减乘除运算。
2. 非退化 - 配对不会把所有内容都翻译成一样,不同元素的配对结果也不同。

有了配对,我们可以设计出很多高效的密码算法:

1. 三方迪菲-赫尔曼密钥交换,让三方都可以商定一个密钥,之前只有两方可以。
2. BLS签名,使签名非常短小,也可以进行多签名聚合。
3. 基于身份的加密,使用邮箱作为公钥,不需要记忆或传递公钥。
4. 零知识证明,可以证明知道某事,但不泄露事情详情。

配对的计算需要两步,第一步类似翻译过程,第二步类似提升翻译结果的准确度。具体数学细节比较复杂,但使用配对可以像调用API函数一样,不需要了解内部细节。

总之,配对是一种超强大的密码学工具,使许多运算变得容易和高效。我尽量用通俗的语言和日常生活的类比解释清楚它的主要思想,具体数学证明等细节此处不展开。如果例子还不太明白,请提出来,我会继续细致解释。

Copy





TXT

paste.txt

22.56 KB

现在请进一步解释，不用太通俗易懂。但是要专业、具体、详细：

NZ



好的,我来尽可能专业和详细地解释配对在密码学中的概念和应用:

1. 配对的定义

配对是定义在三个具有相同阶p的环G1,G2和GT上的双线性映射e: G1 x G2 → GT。其中G1,G2的生成元分别是g1和g2,GT的生成元是gT。当G1=G2时,称为对称配对;否则为非对称配对。

1. 配对的两个关键性质

(1) 双线性:对所有u∈G1,v∈G2,a,b∈Zp,有e(ua,vb)=e(u,v)ab。这保证了运算符可以从G1和G2移入GT中。

(2) 非退化:e(g1,g2)≠1GT。这排除了恒等映射的无意义情况。

1. 配对的计算

配对运算包括两步:米勒循环计算基底值fp,u(v),最后一次求幂运算(fp,u(v))^(qk-1)/p。米勒循环复杂度较低。

1. 配对在密码学中的应用

(1) 三方Diffie-Hellman密钥交换协议

(2) BLS短签名方案,支持签名聚合

(3) 基于身份的加密方案

(4) 零知识证明方案

1. 实现配对加速的方法

使用非对称配对;利用多重指数运算进行批处理;优化G1和G2的使用等。

综上所述,配对作为一种特殊的双线性映射,其双线性和非退化性使其在构建高效密码方案中发挥巨大作用。我尽量用专业的方式全面系统地概述了配对的相关知识,请您指出还需要补充解释的部分。

Copy





TXT

paste.txt

22.56 KB

请按照原意翻译

NZ



配对或双线性映射 2022年12月31日

摘要:配对,或双线性映射,是密码学中一个非常强大的数学工具。配对为我们提供了最简洁的零知识证明1,2,3,最高效的阈值签名4,第一个可用的基于身份的加密(IBE)方案5,以及许多其他高效的密码系统6。在这篇文章中,我将教你一点关于配对性质、密码应用和迷人历史的知识。事实上,读完这篇文章后,你们中的一些人可能会想待一两年的监狱。

推特更正:最初宣布这篇博文的推文声称“SNARKs不可能没有[配对]”,其中高亮显示的S意味着这种SNARKs的“简洁性”。然而,感谢推特上的几位朋友,我意识到这不完全正确,取决于人们对“简洁”的理解。具体来说,在Gentry和Wichs7定义的对数意义上的“简洁”SNARKs,可以从各种假设得出,包括离散对数8或随机谕示9。此外,在O(1)组元素证明大小意义上的“简洁”SNARKs,也可以从RSA假设得出10。目前,配对为我们提供的是具有最小具体证明大小(即字节数)的SNARKs。

初步知识 你熟悉具有素阶的循环群(例如椭圆曲线) 设1GT为群GT的幺元 设x←RS表示从集合S中随机采样元素x 回忆⟨g⟩=G表示g是群G的生成元 配对的定义 配对,也称为双线性映射,是一个函数e:G1×G2→GT,定义在三个具有素阶p的群G1,G2和GT上,其生成元分别是g1=⟨G1⟩,g2=⟨G2⟩和gT=⟨GT⟩。

当G1=G2时,称配对为对称的。否则,它是非对称的。

最重要的是,配对对密码学有两个有用的性质:双线性和非退化。

双线性 双线性要求对所有u∈G1,v∈G2,和a,b∈Zp:

e(ua,vb)=e(u,v)ab(2)

对密码学目的来说,这是最酷的性质。例如,这使得三方迪菲-赫尔曼密钥交换等有用的应用成为可能。

非退化 非退化要求:

e(g1,g2)≠1GT(3)

为什么需要这个性质?我们需要非退化,因为如果没有非退化,很容易(也很无用)定义一个(退化的)双线性映射,对所有输入都返回1GT。这样的映射将满足双线性,但完全无用。

效率 效率要求存在一个多项式时间的算法,以组元素的大小(即λ=log2p)为参数,可以用于计算配对e的任何输入。

为什么需要这个要求?它排除了平淡无奇但计算上不可行的配对。(点击展开。)

历史 这是我对配对的有限历史理解,主要来自Dan Boneh在这个视频中的叙述,以及我对相关文献的研究。如果你知道更多历史,请给我发邮件,我会试着加入。

一个数学家在监狱中 (密码)配对的历史始于一位名叫André Weil11的数学家,二战期间他因拒绝服兵役被送入法国军队的监狱。在那里,Weil“设法说服那位思想开明的监狱主任,允许[他]拥有一个单人牢房,可以保留[..]一支笔,墨水和纸。”

Weil利用新获得的工具,在两个椭圆曲线群之间定义了一个配对。然而,在那个时候非常奇怪的是,Weil下了很大功夫来确保他定义的配对是可计算的。正是这额外的努力使今天基于配对的密码学成为可能。12

去监狱,不去大学? 有趣的是,Weil在监狱中的时间是如此富有成效,以至于他开始怀疑是否应该每年在那里呆几个月。更妙的是,Weil考虑是否应该向有关部门建议每个数学家都应该在监狱中呆一段时间。Weil写道:

我开始认为,对抽象科学来说,没什么比监狱更有益的了。

[...]

我的数学工作进展远远超出了我最狂野的想象,我甚至有点担心——如果只有在监狱里我才能工作得这么好,那我是不是每年都得安排呆两三个月的牢房?

与此同时,我正考虑给相关部门写一份报告,大致如下:“致科学研究主任:最近通过个人经历,我有机会发现监狱系统机构的待遇对纯粹的、无私的研究有相当大的好处,特此报告,等等。”

你可以在他迷人的自传中读到这一切,这本自传是从他作为一位数学家的视角写的。13

从破译密码学到建立密码学 Weil的工作奠定了基础。但是还需要三个发展,配对密码学才能兴起。

第一步发展:Miller算法 1985年,Victor Miller写了一篇手稿,展示了Weil配对(实际上涉及求幂指数级多项式)可以高效地在多项式时间内计算。14

1984年12月,Miller在IBM关于椭圆曲线密码学的演讲中声称,椭圆曲线离散对数难以计算,难于有限域上的普通离散对数。15 Miller受到听众Manuel Blum的挑战,用还原论证支持这个说法:即展示一个求解椭圆曲线离散对数的算法B可以高效地转化成另一个算法A,用于求解有限域中的离散对数。这样的还原将意味着B解决的问题(即计算椭圆曲线离散对数)至少与A的问题(即计算有限域离散对数)一样困难,如果不是更困难的话。

Miller试图通过考虑椭圆曲线群和有限域之间唯一的关联——Weil配对,来找到一个还原。有趣的是,他很快意识到,虽然Weil配对给出了一个还原,但它是朝相反方向的:即一个求解有限域离散对数的算法A可以在Weil配对的帮助下,高效地转化成一个求解椭圆曲线离散对数的算法B。这个“不需要的”还原很容易看出来。由于e(ga,g)=e(g,g)a,求解椭圆曲线元素ga∈G的离散对数,只需要求e(g,g)a∈GT的离散对数,后者实际上是一个有限域的乘法子群(参见配对的内部细节)。

这几乎表明Miller想证明的相反结论,可能会破坏椭圆曲线密码学。但幸运的是,Weil配对映射到的扩域的度数太大,使这个“不需要的”还原变得低效,因此并不能构成还原。

这整个事件使Miller对研究Weil配对是否可以有效计算起了兴趣,这导致了他的算法发现。有趣的是,他将这篇手稿提交给了FOCS,一个顶级理论计算机科学会议,但论文被拒绝,要到很久后才在密码学杂志(根据Miller的说法)发表。16

第二步发展:MOV攻击 1991年,Menezes、Vanstone和Okamoto17利用Miller计算Weil配对的有效算法,打破某些椭圆曲线上的离散对数假设,时间复杂度为亚指数级。这相当惊人,因为在那时对椭圆曲线还没有已知亚指数级时间的算法。

他们的攻击,称为MOV攻击,使用配对将椭圆曲线离散对数挑战ga∈G映射到目标群体为e(ga,g)=e(g,g)a∈GT。由于目标群是有限域Fkq的子群,这允许使用更快的亚指数时间算法来计算e(g,g)a上的离散对数。

第三步发展:Joux的三方Diffie-Hellman 到目前为止,配对似乎只对密码分析有用。没有人知道如何使用它们来建立(而不是攻击)密码学。

这在2000年发生了改变,当时Joux18使用配对实现了一种在三方之间的一轮密钥交换协议,或三方Diffie-Hellman。在此之前,这种一轮协议只在两方之间已知,而三方需要两轮。

从那时起,大量新的高效密码学涌现:

BLS(短)签名4 基于身份的加密5 可加性同态加密,支持一次乘法运算19
 简洁的零知识证明1

这里可以注意到一个有趣的模式,配对如何从一个用于攻击密码系统的密码分析工具,发展成一个用于建立密码系统的构造工具。有趣的是,在格密码学的发展中也出现了相同的模式。

配对运算的算数技巧 在密码学家证明密码系统的正确性或安全性时,他们经常使用一些与配对相关的技巧。

最明显的技巧“指数相乘”来自双线性特性。

e(ua,vb)=e(u,v)ab(9)

双线性也意味着以下技巧: e(u,vb)=e(u,v)b(10)

或者,另一种表达: e(ua,v)=e(u,v)a(11)

另一个技巧,也是定义双线性的等价方式: e(u,v⋅w)=e(u,v)⋅e(u,w)(12)

为什么行得通?设v和w相对于g2的离散对数分别为y,z。则: e(u,v⋅w)=e(u,gy2⋅gz2)=e(u,gy+z2)=e(u,g2)y+z=e(u,g2)y⋅e(u,g2)z=e(u,gy2)⋅e(u,gz2)=e(u,v)⋅e(u,w)(13)(14)(15)(16)(17)(18)

或者,另一种表达: e(u,v/w)=e(u,v)e(u,w)(19)

配对的应用 三方Diffie-Hellman 这个协议由Joux在2000年提出18,假设配对是对称的:即G1=G2=⟨g⟩=defG。

有三方Alice、Bob和Charles,密钥分别为a,b,c。他们互相发送公钥ga,gb,gc,并约定共享密钥k=e(g,g)abc。20

如何做到?

考虑Alice的视角。她从Bob和Charles获得gb和gc。首先,她可以用自己的密钥a计算gab。其次,她可以用配对计算e(gab,gc)= e(g,g)abc=k。

由对称性,其他参与方也可以做同样的计算,并认同相同的k。

这个协议也可以推广到非对称配对,其中G1≠G2。

BLS签名 Boneh、Lynn和Shacham基于配对提出了一种非常短的签名方案4,具体如下:

假设G2=⟨g2⟩且存在随机谕示函数H:{0,1}∗→G1。 密钥s∈Zp,公钥pk=gs2∈G2。 签名消息m时,计算σ=H(m)s∈G1。
 验证签名σ与消息m匹配公钥pk,检查e(σ,g2)=?e(H(m),pk) 可以看出,正确计算的签名通过验证,因为: e(σ,g2)=?e(H(m),pk)⇔e(H(m)s,g2)=?e(H(m),gs2)⇔e(H(m),g2)s=?e(H(m),g2)s⇔e(H(m),g2)=e(H(m),g2)(20)(21)(22)(23)

参见BLS论文4关于安全性证明。

BLS签名的酷属性 BLS签名非常惊人:

给椭圆曲线库,实现很简单 可以聚合在同一消息m上的不同公钥的多个签名为一个多签名,只需2次配对验证 甚至可以聚合不同消息的签名为单个聚合签名 但这需要n+1次配对验证 可以轻松有效构建BLS门限签名方案,t出n个签名者可协作签名,但少于t不能生成有效签名 更好的是,BLS门限签名确定性,对应门限可验证随机函数,可用于链上生成随机性 可以定义BLS签名的高效盲化变体,签名方可以在不知晓消息m的情况下对其签名 BLS签名在实践中非常高效 就我记得,它们是最高效的(1)多签名,(2)聚合签名和(3)门限签名方案 对单签名者BLS,比非配对友好曲线上的Schnorr签名慢 如果你对这些签名概念感到困惑,可以看我的幻灯片。

基于身份的加密(IBE) 在IBE方案中,可以直接对友好的电子邮件地址(或电话号码)加密,而不需要记忆或正确输入笨拙的公钥。

Boneh和Franklin基于配对给出一个非常高效的IBE方案。5

对于IBE来说,需要引入一个可信任的第三方(TTP)称为私钥生成中心(PKG),根据用户的邮件地址为他们生成密钥。这个PKG拥有一个主密钥(MSK)msk∈Zp,对应一个主公钥(MPK)mpk=gs2,其中⟨g2⟩=G2。

mpk对外公开,可以用来针对给定邮件地址的用户加密消息。关键是,PKG必须保密msk。否则,获取它的攻击者可以导出任意用户的密钥并解密所有消息。

可以看出,PKG是一个单点故障:msk被盗就危及所有用户的保密性。为降低风险,可以将PKG分散到多个机构,需要达到门限数目的机构被入侵才能盗取msk。

设H1:{0,1}∗→G∗1和HT:GT→{0,1}n为两个模型化为随机预言机的哈希函数。要向ID为id的用户加密一个n比特消息m,可以计算:

gidrc=e(H1(id),mpk)∈GT←RZp=(gr2,m⊕HT((gid)r))∈G2×{0,1}n

为了解密,ID为id的用户必须先从PKG获取其解密密钥dskid。我们假设PKG有方法认证用户身份,然后再发给他们密钥。例如可以通过邮件实现。

PKG计算用户的解密密钥:

dskid=H1(id)s∈G1

用户拿到解密密钥后,可以对密文c=(u,v)进行解密:

m=v⊕HT(e(dskid,u))

你可以看出,正确加密的密文可以被成功解密,因为:

m=v⊕HT(e(dskid,u)) =(m⊕HT((gid)r))⊕HT(e(H1(id)s,gr2)) =(m⊕HT(e(H1(id),mpk)r))⊕HT(e(H1(id),g2)rs) =m⊕(HT(e(H1(id),gs2)r)⊕HT(e(H1(id),g2)rs)) =m⊕(HT(e(H1(id),g2)rs)⊕HT(e(H1(id),g2)rs)) =m

关于这个方案在选择明文攻击下的安全性证明,请参考原论文。

配对是如何工作的?我大多数情况下并不清楚。为什么?因为我从未真正需要知道这一点。这也正是配对的美妙之处:可以将其作为一个黑盒使用,对内部一无所知。

尽管如此,让我们还是稍微描绘一下这个黑盒的内部。我们考虑流行的BLS12-381配对友好曲线,这是Barreto、Lynn和Scott提出的BLS曲线族的一员。

关于BLS12-381,参与配对的三个群组G1,G2,GT如下:

G1是椭圆曲线E(Fq)={(x,y)∈(Fq)2 | y2=x3+4}的一个子群,其中|G1|=p

G2是另一个椭圆曲线E′(Fq2)={(x,y)∈(Fq2)2 | y2=x3+4(1+i)} 的一个子群,其中i是-1的平方根,|G2|=p。

GT是所有Fqk中的p次根,其中k=12称为嵌入度。

配对映射e(⋅,⋅)如何跨这三个群工作?其展开形式类似:

e(u,v)=fp,u(v)(qk−1)/p

这里要知道计算配对包含两步:

1. 计算基fp,u(v),也称为米勒循环
2. 将这个基提升到固定指数(qk−1)/p,也称为最终指数法

这一步比第一步耗时更多

想进一步了解内部工作原理,可参考其他资料。

实现基于配对的密码学

这一节讨论实践者可以利用的各种实现层面的细节,以加速他们的配对实现。

使用不对称配对!

BLS12-381配对是不对称的:即G1≠G2是两个不同的群(同阶)。但是也存在对称配对,其中G1=G2是同一个群。

不幸的是,这样的对称配对只存在于特殊的超奇异曲线上,这大大限制了基础效率或安全性。也就是说,在相同安全级别下,这种超奇异曲线不如不对称配对使用的曲线高效。

因此,现今密码实践者,就我所知,完全依赖不对称配对,因为在相同安全级别下它们更高效。

BLS12-381性能

下面给出Filecoin实现中BLS12-381曲线的一些关键性能数据(使用Rust blstrs库封装了流行的blst库)。这些微基准测试在10核2021 Apple M1 Max上用cargo bench运行获得。

配对计算时间:

米勒循环计算需210微秒

最终指数法需276微秒

所以整个配对大约需486微秒。

指数运算时间:

G1指数运算最快,需72微秒

G2指数运算约为其2倍,需136微秒

GT指数运算比G2慢约3.5倍,需500微秒

注意:这些基准测试中,指数运算的底是随机选取的,没有对其进行预计算,否则可以使这些时间提速2-4倍。

多指数运算:这是一种众所周知的优化,我在此简要提及。具体来说,许多库可以比逐一计算k个指数运算并聚合其积快很多地计算积∏0<i<k(gi)xi。

例如,blstrs在这方面似乎非常快:

在G1中的大小为256的多指数运算总共只需760微秒,即每个指数运算只需3微秒!如果普通计算需要总共18.5毫秒,慢了24倍。

在G2中的大小为256的多指数运算总共只需1.88毫秒,即每个指数运算只需7.33微秒!如果普通计算需要总共35.3毫秒,慢了18.8倍。

群元素大小:

G1群元素最小,例如BLS12-381上为48字节,BN254曲线上为32字节

G2群元素为其2倍大,例如BLS12-381上为96字节

GT元素为其12倍大。一般来说,对于嵌入度为k的配对友好曲线,元素大小为k倍。

其他操作时间:

G1乘法(注意我们使用乘法表示法而不是加法表示法):

普通乘法:565纳秒(两点均在射影(X,Y)坐标下)

混合乘法:438纳秒(第一点在射影坐标,第二点在仿射(X,Y,Z)坐标下)

更快,因为节省了一个射影到仿射转换

G2乘法:

普通乘法:1,484纳秒

混合乘法:1,095纳秒

散列到G1大约需50微秒(不考虑额外的通过SHA2-256散列更大消息的时间)

在G1和G2间转换:

设计基于配对的密码方案时,要仔细考虑使用G1还是G2。

例如,在BLS签名中,如果要获得较小签名,可以计算签名σ=H(m)s∈G1,并接受稍大一些的公钥be在G2中。另一方面,如果要最小化公钥大小,可以让公钥在G1中,同时承受稍长的G2签名计算时间。

G1和G2的使用还会受到其他因素影响,例如存在同构φ:G2→G1,或散列到这些群的能力等。事实上,这种同构的存在将配对分为两种类型:2型和3型(更多信息见Galbraith等人的文献)。

与非配对友好椭圆曲线比较:

与不可配对的椭圆曲线群相比,配对友好椭圆曲线大约慢2倍。

例如,流行的素阶椭圆曲线群Ristretto255提供:

≈2倍快的40微秒指数运算

使用预计算可以使其加速到10微秒,当指数运算的底固定时

32字节群元素大小

多重配对:

回忆计算一个配对的实际工作原理(见公式34),你会注意到以下优化:

当需要计算n个配对的积时,可以先计算n个米勒循环,然后进行1次最终指数法,而不是进行n次。这大大减少了许多应用中的配对计算时间。

即:∏ie(ui,vi)=∏i(fp,ui(vi)(qk−1)/p)=(∏ifp,ui(vi))(qk−1)/p

总结:

本博客文章本打算只对配对的三个性质:双线性、非退化和高效计算做一个简短概述。

不幸的是,我无法抑制对它们迷人历史的阐述。同样,我也无法让你在没有见识几个配对强大密码应用的情况下就离开。

之后,我意识到实现配对密码系统的从业者可能会从略知一二它们的内部工作原理中受益,因为这些细节可以用来优化实现。

致谢:

我要感谢Dan Boneh帮助我澄清和添加配对历史部分关于Weil的内容,以及感谢他2015年的Simons演讲,启发我进行更多研究并写下这篇历史记录。

非常感谢:

Lúcás Meier、Pratyush Mishra、Ariel Gabizon和Dario Fiore对“简洁”(Succinct)在SNARK中代表的含义,以及提醒我从RSA假设出发可以构建O(1)大小的SNARK证明的见解。

Sergey Vasilyev指出BLS12-381椭圆曲线定义中的错字。

@BlakeMScurr指出关于Joux工作的错误引用。

Conrado Guovea指出我Victor Miller关于其如何开发Weil配对计算算法的叙述(此处有讨论)。

Chris Peikert指出还有大量不依赖配对的高效IBE方案。