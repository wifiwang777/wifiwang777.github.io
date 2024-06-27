+++
title = '归集分析——trc20'
date = 2024-06-27T10:21:02+08:00
draft = false
+++

# 使用代理来免费/减少手续费的使用

## 观察别人的归集

#### transactions

[用户钱包](https://tronscan.org/#/address/TAgsEbu8JHXSYLKu3JTtL5SVHBp56Q4Lic)

![img.png](/归集分析——trc20/transactions.png)

#### transfers

[用户钱包](https://tronscan.org/#/address/TAgsEbu8JHXSYLKu3JTtL5SVHBp56Q4Lic/transfers)

![img.png](/归集分析——trc20/transfers.png)

### 归集步骤

#### 1.active account

[f7fed942932b16841280ecb47909d2d81bac76aab6212df721486df8a7fdc4a1](https://tronscan.org/#/transaction/f7fed942932b16841280ecb47909d2d81bac76aab6212df721486df8a7fdc4a1)
![img.png](/归集分析——trc20/active.png)

#### 2.delegate energy

[99bbd1f01d05e708652fd1d13284eb435f745cd411eec9fb6444eee41c0a6f46](https://tronscan.org/#/transaction/99bbd1f01d05e708652fd1d13284eb435f745cd411eec9fb6444eee41c0a6f46)
![img.png](/归集分析——trc20/delegate.png)

#### 3.approve

[4b68295762d286b77a615a23a0f87bf16374213f68b4f710f49c1d0864b0d0d9](https://tronscan.org/#/transaction/4b68295762d286b77a615a23a0f87bf16374213f68b4f710f49c1d0864b0d0d9)
![img.png](/归集分析——trc20/approve.png)

#### 4.undelegate

[9eb7f2e7657c0b85feeb997a1c615454388ee2daed8306ac284daa4721c362d2](https://tronscan.org/#/transaction/9eb7f2e7657c0b85feeb997a1c615454388ee2daed8306ac284daa4721c362d2)
![img.png](/归集分析——trc20/undelegate.png)

#### 5.transferFrom

[c181f63b403c0e3dd3bf94c879681f6b551ae749ac9416c9efb12e11964461c9](https://tronscan.org/#/transaction/c181f63b403c0e3dd3bf94c879681f6b551ae749ac9416c9efb12e11964461c9)
![img.png](/归集分析——trc20/transferFrom.png)

### 交易分析

#### 热钱包1:TJfEgAsE4zMHHeT62vYGL4md2GBcvX5E3X

用于激活账号、接受approve、执行transferFrom

#### 热钱包2:TBu8ATfngYLgGucPjT3URoU5iFKXsoYRCH

用于质押TRX获得免费能量，并将能量代理给用户钱包

#### 交易分析会发现总经过5次交易，完成归集

1. 由于是外部直接转usdt到该用户钱包，该钱包没有激活（trc20链特性，需要激活/转trx，该钱包才能交易），所以**热钱包1**
   先转了0.000001trx给该钱包，用于激活（其实可以连这个0.000001trx都可以不转）
2. **热钱包2**代理了资源（energy）给该钱包，使其可以直接使用**热钱包2**的资源，来进行交易
3. 该钱包执行了USDT合约的approve方法，将资产approve给**热钱包1**，此操作消耗了**热钱包2**代理给该钱包的资源，
4. **热钱包2**取消了资源代理
5. **热钱包1**执行了USDT合约的transfer方法，将该钱包的USDT直接转到其他钱包。

#### 热钱包2质押情况

![img.png](/归集分析——trc20/stack.png)

### 能量分析

整个归集流程中只有approve和transferFrom会消耗能量

1. approve = 49799（**用户钱包**, 基本固定这个数量）
2. transferFrom = 32305（**热钱包1**, 浮动）
