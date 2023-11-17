+++
title = '数学题'
date = 2023-11-17T11:52:58+08:00
draft = false
categories = ['math']
tags = ['math']
+++

### 第一题
$$
求 \lim_{x \to \infty} \frac{[(1+\frac{1}{x})^x]^x}{e^x} 的极限
$$

$$
解: 原式 = \lim_{x \to \infty} \left[(1+\frac{1}{x})^x \right]^x \cdot e^{-x}
$$

$$
因为: x \to \infty, (1 + \frac{1}{x})^x \to e
$$

$$
所以: x \to \infty, \left[(1+\frac{1}{x})^x \right]^x \to e^x
$$

$$
所以: 原式 = \lim_{x \to \infty} \left[(1+\frac{1}{x})^x\right]^x \cdot e^{-x} = \lim_{x \to \infty} e^x \cdot e^{-x}
$$

$$
因为: e^x \cdot e^{-x} = e^{x-x} = e^0 = 1
$$

$$
所以，当x \to \infty时, 原式 = \lim_{x \to \infty} \frac{[(1+\frac{1}{x})^x]^x}{e^x} = 1
$$
