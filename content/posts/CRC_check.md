+++
title = 'CRC校验'
date = 2023-11-08T21:22:11+08:00
draft = false
categories = ['文档']
tags = ['计算机网络', 'Internet']
+++

### CRC 循环冗余校验

```c++
1. 选择一个生成多项式 G(x)
2. 根据 G(x) 得到除数
3. 计算除数的位数 n
4. 被除数为原始数据的比特流左移 n - 1 位
5. 被除数与除数相除得到余数(异或运算)
6. 实际发送数据为原始数据的比特流后面附加上余数的比特流
7. 接收方使用收到的数据的比特流与 G(x) 得到的除数相除
8. 判断余数，为 0 则传输数据无误，否则表示数据传输过程出现错误
```
