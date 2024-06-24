+++
title = '归集分析——erc20'
date = 2024-06-24T17:27:06+08:00
draft = false
+++

# ERC20 归集分析

### 热钱包

[0x646d15ccc9157ee02a51747a6fd5d8b914f655f0](https://etherscan.io/address/0x646d15ccc9157ee02a51747a6fd5d8b914f655f0)

![](/归集分析——erc20/hotAddress.png)

### 用户钱包

[0xedcc356c5c5854f5dc2225ee2b090bddb26e4142](https://etherscan.io/address/0xedcc356c5c5854f5dc2225ee2b090bddb26e4142)
![](/归集分析——erc20/userAddress.png)

## 归集步骤

### 1. 给用户钱包添加归集手续费（用于用户执行approve）

tx: [0xb88551971f58180e2f1a21bfd66fd7b44707a8e2b53a3429a0820b186b48c961](https://etherscan.io/tx/0xb88551971f58180e2f1a21bfd66fd7b44707a8e2b53a3429a0820b186b48c961)

![](/归集分析——erc20/approveFee.png)

fee: 0.000223939148118 ETH (0.76U)
### 2. 用户地址执行approve

tx: [0xe917d167535d462636fa705c7b7382bf729656987ba8660f6936855ec7a2b1b4](https://etherscan.io/tx/0xe917d167535d462636fa705c7b7382bf729656987ba8660f6936855ec7a2b1b4)

![](/归集分析——erc20/approve.png)

fee: 0.000490599366525675 ETH (1.66U)

### 3. 热钱包将用户钱包approve的钱转到热钱包

tx: [0x81d7ba9ff5523ec2adc04ee83d3d63f35e69701cb7413226937e6c187c9cb580](https://etherscan.io/tx/0x81d7ba9ff5523ec2adc04ee83d3d63f35e69701cb7413226937e6c187c9cb580)

![](/归集分析——erc20/transferFrom.png)

fee: 0.000715370376723908 ETH (2.42U)