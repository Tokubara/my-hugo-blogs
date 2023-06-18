+++
title = "对 TensorFHE Hada-Mult kernel 意义的疑惑"
author = ["Isaac Quebec"]
draft = false
+++

<div class="ox-hugo-toc toc">

<div class="heading">&#30446;&#24405;</div>

- [TensorFHE 认为 Hada-Mult 是向量 (多项式系数) 位乘取模](#tensorfhe-认为-hada-mult-是向量--多项式系数--位乘取模)
- [FHETensor 不认为 Hada-Mult 与 NTT 是一回事](#fhetensor-不认为-hada-mult-与-ntt-是一回事)
- [Hada-Mult 在 CKKS 中的对应操作](#hada-mult-在-ckks-中的对应操作)
- [HMULT 对应 py-fhe ckks_evaluator.py 中的 multiply 函数, Hada-Mult 对应 polynomial.py 中的 multiply 函数](#hmult-对应-py-fhe-ckks-evaluator-dot-py-中的-multiply-函数-hada-mult-对应-polynomial-dot-py-中的-multiply-函数)
- [Polynomial.multiply 的实现表明 Hada-Mult 应是多项式乘法, 而且用 NTT 实现](#polynomial-dot-multiply-的实现表明-hada-mult-应是多项式乘法-而且用-ntt-实现)
- [小结](#小结)

</div>
<!--endtoc-->


## TensorFHE 认为 Hada-Mult 是向量 (多项式系数) 位乘取模 {#tensorfhe-认为-hada-mult-是向量--多项式系数--位乘取模}

TensorFHE 中是这样介绍 Hada-Mult 的:

> Hadamard Multiplication (Hada-Mult) can be formulated as c = (a◦b) mod q [14]. The a◦b indicates the **element-wise product** of two polynomials represented as vectors (also known as Hadamard product) [16].

要点在于认为它是 element-wise product, 也就是认为是两个多项式系数的位乘. 比如 2x^2+3x+1 与 3x^2+4x+1, 其 Hada-Mult (假设 q 很大, 不考虑取模) 是 (2,3,1) 与 (3,4,1) 的位乘即 (6,12,1).


## FHETensor 不认为 Hada-Mult 与 NTT 是一回事 {#fhetensor-不认为-hada-mult-与-ntt-是一回事}

<a id="table--TABLE II: HIERARCHICAL RECONSTRUCTION MODEL OF CKKS."></a>

| Operation | Description                              | Composing Kernels                           |
|-----------|------------------------------------------|---------------------------------------------|
| HMULT     | Multiply two ciphertexts.                | NTT, Hada-Mult, Conv, Ele-Add               |
| CMULT     | Multiply ciphertext with plaintext.      | Hada-Mult, Ele-Add                          |
| HROTATE   | Roatate ciphertext.                      | NTT, Hada-Mult, Ele-Add, Conv, ForbeniusMap |
| RECALE    | Reduce the security level of ciphertext. | NTT, Ele-Sub                                |
| HADD      | Add two ciphertexts.                     | Ele-Add                                     |

注意看, Hada-Mult 与 NTT 是两个不同的 kernel.


## Hada-Mult 在 CKKS 中的对应操作 {#hada-mult-在-ckks-中的对应操作}

HMULT (两个 ciphertext 乘法) 用到了 Hada-Mult. 如下, 第一行 d2=Hada-Mult(a0,a1).

<a id="figure--Algorithm 2: HMULT"></a>

{{< figure src="/ox-hugo/Pasted_image_20221014121019.png" >}}

我们来看看 CKKS 原文对于密文乘法是怎么做的:

{{< figure src="/ox-hugo/Pasted_image_20220928115254.png" >}}

d2=a1a2, 正对应上面的 d2=Hada-Mult(a0,a1) 的描述. 从符号来看应该是多项式乘法.

为了验证这里的确是多项式乘法, 我们看 py-fhe HMULT 对应的代码.


## HMULT 对应 py-fhe ckks_evaluator.py 中的 multiply 函数, Hada-Mult 对应 polynomial.py 中的 multiply 函数 {#hmult-对应-py-fhe-ckks-evaluator-dot-py-中的-multiply-函数-hada-mult-对应-polynomial-dot-py-中的-multiply-函数}

-   run_test_multiply(test_ckks_arithmetic.py): 这个函数就是测例中对两个 message 进行加密的入口
    ciph_prod = self.evaluator.multiply(ciph1, ciph2, self.relin_key) 这个函数中的这一行就是对两个 ciphertext 进行加密, 对应的是:
    -   multiply(self, ciph1, ciph2, relin_key) (ckks_evaluator.py)
        其中对应 d2=Hada-Mult(a0,a1) 的是这一行: c2 = ciph1.c1.multiply(ciph2.c1, modulus, crt=self.crt_context), 这里的 multiply 函数也就对应着 Hada-Mult 操作. 这个函数对应的是:
        -   multiply(self, poly, coeff_modulus, ntt=None, crt=None) (polynomial.py)

因此我们可以确定, polynomial.py 中的 Polynomial.multiply 就对应 Hada-Mult 操作.


## Polynomial.multiply 的实现表明 Hada-Mult 应是多项式乘法, 而且用 NTT 实现 {#polynomial-dot-multiply-的实现表明-hada-mult-应是多项式乘法-而且用-ntt-实现}

Polynomial.multiply 的文档是:

> Multiplies two polynomials in the ring using NTT.
>
> Multiplies the current polynomial to poly inside the ring R_a using the Number Theoretic Transform (NTT) in O(nlogn).

它说这是多项式乘法, 并且用了 NTT.

它的实现与它的文档一致.

```python
def multiply(self, poly, coeff_modulus, ntt=None, crt=None):
    if crt:
        return self.multiply_crt(poly, crt)

    if ntt:
        a = ntt.ftt_fwd(self.coeffs)
        b = ntt.ftt_fwd(poly.coeffs)
        ab = [a[i] * b[i] for i in range(self.ring_degree)]
        prod = ntt.ftt_inv(ab)
        return Polynomial(self.ring_degree, prod)

    return self.multiply_naive(poly, coeff_modulus)
```

因此我们可以确定, Hada-Mult 就对应多项式乘法.


## 小结 {#小结}

在 TensorFHE 中认为 Hada-Mult 是多项式位乘, 并且认为 Hada-Mult 与 NTT 是两个不同的 kernel. 对比 CKKS 原文以及 py-fhe 对 HMULT 的实现之后, 我认为 Hada-Mult 应该是多项式乘法, 并且与 NTT 是一回事.
