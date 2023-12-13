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

> $$
> 解: 原式 = \lim_{x \to \infty} \left[(1+\frac{1}{x})^x \right]^x \cdot e^{-x}
> $$
> 
> $$
> \because x \to \infty, (1 + \frac{1}{x})^x \to e
> $$
> 
> $$
> \therefore x \to \infty, \left[(1+\frac{1}{x})^x \right]^x \to e^x
> $$
> 
> $$
> \therefore 原式 = \lim_{x \to \infty} \left[(1+\frac{1}{x})^x\right]^x \cdot e^{-x} = \lim_{x \to \infty} e^x \cdot e^{-x}
> $$
> 
> $$
> \because e^x \cdot e^{-x} = e^{x-x} = e^0 = 1
> $$
> 
> $$
> \therefore 当x \to \infty时, 原式 = \lim_{x \to \infty} \frac{[(1+\frac{1}{x})^x]^x}{e^x} = 1
> $$

### 第二题
$$
求 \int_{0}^{1} \frac{\ln(1+x)}{1+x^2}\mathrm{d}x 的积分
$$

> $$
> 令 x = \tan t 得 {d}x = \sec ^2t{d}t
> $$
>
> $$
> \because I = \int_{0}^{\frac{\pi}{4}} \ln(1+\tan t) {d}t
> $$
>
> $$
>      = \int_{0}^{\frac{\pi}{4}} \ln(1+\tan (\frac{\pi}{4} - u)) {d}u
> $$
>
> $$
>      = \int_{0}^{\frac{\pi}{4}} \ln(1 + \frac{1 - \tan u}{1 + \tan u}) {d}u
> $$
>
> $$
>      = \int_{0}^{\frac{\pi}{4}} \ln(\frac{2}{1 + \tan u}) {d}u
> $$
>
> $$
>      = \int_{0}^{\frac{\pi}{4}} \ln 2 {d}u - \int_{0}^{\frac{\pi}{4}} \ln (1 + \tan u) {d}u
> $$
> 
> $$
> \therefore  2I = \int_{0}^{\frac{\pi}{4}} \ln 2 {d}u
> $$
>
> $$
> \therefore I = \frac{\pi}{8} \ln 2
> $$
