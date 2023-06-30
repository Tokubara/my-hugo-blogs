+++
title = "对 NTT 表示的疑惑"
author = ["Isaac Quebec"]
tags = ["discussion", "todo"]
draft = false
+++

<div class="ox-hugo-toc toc">

<div class="heading">&#30446;&#24405;</div>

- [full-rns 的 decrypt 核实](#full-rns-的-decrypt-核实)
- [NTT 在取模的情况下怎么保持 arithmetic](#ntt-在取模的情况下怎么保持-arithmetic)
- [RESCALE 为什么取了 INTT 后立即取了 NTT?](#rescale-为什么取了-intt-后立即取了-ntt)
- [哪些情况下需要用 NTT, INTT? 即 NTT 不能保持的运算是什么?](#哪些情况下需要用-ntt-intt-即-ntt-不能保持的运算是什么)

</div>
<!--endtoc-->


## full-rns 的 decrypt 核实 {#full-rns-的-decrypt-核实}

full-rns 中:
\\(\operatorname{Dec}\_{\mathrm{sk}}(\mathrm{ct})\\). For ct \\(=\left(\mathrm{ct}^{(j)}\right)\_{0 \leq j \leq \ell}\\), output \\(\left\langle\mathrm{ct}^{(0)}, \mathrm{sk}\right\rangle\left(\bmod q\_0\right)\\).

所以 descrypt 竟然只用到了第一项?


## NTT 在取模的情况下怎么保持 arithmetic {#ntt-在取模的情况下怎么保持-arithmetic}

如果没有取模操作操作, 算术中的加减法是保持的. 但是如果取模, 事情就不同了.

这里我们假设 NTT 的模是 P.

举个例子, 这里 P=769.

```mathematica
With[{v1={27,23,17,12}, v2={12,17,22,23}, ql=31,qj=29}, Mod[(v1-v2), P]==dftINTT[ dftNTT[v1]-dftNTT[v2]]] (* True *)
With[{v1={27,23,17,12}, v2={12,17,22,23}, qj=29}, Mod[v1-v2,qj]==Mod[dftINTT[dftNTT[v1]-dftNTT[v2]],qj] ] (* False *)
With[{v1={27,23,17,12}, v2={12,17,22,23}, qj=29}, Mod[v1-v2,qj]==Mod[dftINTT[ Mod[dftNTT[v1]-dftNTT[v2],qj]],qj] ] (* False *)
With[{v1={27,23,17,12}, v2={12,17,22,23}, qj=29}, (v1-v2)==dftINTT[dftNTT[v1]-dftNTT[v2]]] (* False *)
```

第一行没有限制取模, 很自然有 \\(\operatorname{INTT}(\operatorname{NTT}(v\_{1})-\operatorname{NTT}(v\_{2}))=v\_{1}-v\_{2}\\).
但从第二行起, 我们需要 \\(v\_{1}\\), \\(v\_{2}\\) 关于 29 取模 (代码中的 `qj`), `v1-v2` 是 `{15, 6, -5, -11}`. `Mod[v1-v2,qj]` 是 `{15, 6, 24, 18}`. 但如果用 NTT 和 INTT, `dftNTT[v1]-dftNTT[v2]` 这步没有损失, 但是 `dftINTT[dftNTT[v1]-dftNTT[v2]]` 会关于 q 取模, 得到的是 `{15, 6, 764, 758}`, 因为 -5 关于 769 取模是 764, -11 关于 769 取模是 758. 但再关于 29 取模就不对了, `Mod[dftINTT[dftNTT[v1]-dftNTT[v2]],qj]` 是 `{15, 6, 10, 4}`.

所以问题是, 在本就需要取模的情况下, NTT, INTT 自己也有自己的取模操作, 此时如何保持一致?


## RESCALE 为什么取了 INTT 后立即取了 NTT? {#rescale-为什么取了-intt-后立即取了-ntt}

{{< figure src="/ox-hugo/Pasted_image_20221014123301.png" >}}


## 哪些情况下需要用 NTT, INTT? 即 NTT 不能保持的运算是什么? {#哪些情况下需要用-ntt-intt-即-ntt-不能保持的运算是什么}

比如 fast base conversion:
\\[\operatorname{Conv}\_{\mathcal{C} \rightarrow \mathcal{B}}\left([a]\_{\mathcal{C}}\right)=\left(\sum\_{j=0}^{\ell-1}\left[a^{(j)} \cdot \hat{q}\_{j}^{-1}\right]\_{q\_{j}} \cdot \hat{q}\_{j} \quad\left(\bmod p\_{i}\right)\right)\_{0 \leq i<k} \\]

其中 \\({\hat{q}\_{j}=\prod\_{j^{\prime} \neq j} q\_{j^{\prime}} \in \mathbb{Z}}\\), \\(\hat{q\_{j}}^{-1}\\) 是 \\(\hat{q\_{j}}\\) 关于模 \\(q\_{j}\\) 的逆.

我想这里应该需要先用 INTT 得到多项式的系数表示 (区别于 NTT 表示)再进行 fast base conversion. 并且 ModUp 之后应该不需要做 NTT, 而是应该在 ModDown 完成后再做 NTT 得到多项式表示.
