+++
title = "fast base conversion 的向量化计算"
author = ["Isaac Quebec"]
tags = ["discussion"]
draft = false
+++

\\(\operatorname{Conv}\_{\mathcal{C} \rightarrow \mathcal{B}}\left([a]\_{\mathcal{C}}\right)=\left(\sum\_{j=0}^{\ell-1}\left[a^{(j)} \cdot \hat{q}\_{j}^{-1}\right]\_{q\_{j}} \cdot \hat{q}\_{j} \quad\left(\bmod p\_{i}\right)\right)\_{0 \leq i<k}\\)

\\({\hat{q}\_{j}=\prod\_{j^{\prime} \neq j} q\_{j^{\prime}} \in \mathbb{Z}}\\), \\(\hat{q\_{j}}^{-1}\\) 是 \\(\hat{q\_{j}}\\) 关于模 \\(q\_{j}\\) 的逆.

如果不考虑 \\(\left(\sum\_{j=0}^{\ell-1}\left[a^{(j)} \cdot \hat{q}\_{j}^{-1}\right]\_{q\_{j}} \cdot \hat{q}\_{j} \right)\\) 可能积累到很长的宽度了, 可能需要提前取模, 按照这个式子的做法是:
长度为 l 的位乘, 长度为 l 的取模(取模的数一样), 长度为 l 的点乘. 然后 k 次数量取模, 每次用于取模的数都不同. 这个性能猜测会很糟糕, 主要问题在于点乘和最后的 k 次取模, 这 k 次取模难以向量化, 因为模数不同.
