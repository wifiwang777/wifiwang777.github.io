+++
title = '门限签名方案2——TSS(threshold-signature-sheme)'
date = 2022-02-19 12:06:28
draft = false
+++

### 背景

目前市面上流行的门限椭圆曲线数字签名（Threshold ECDSA）方案是由Rosario Gennaro and Steven
Goldfeder在2018年发布的论文（简称gg18）为理论基础实现的，本文主要描述了gg18在ECDSA中的大致流程。

##### 链接

Multiparty Threshold ECDSA (GG18)

[http://aandds.com/blog/multiparty-threshold-ecdsa.html](http://aandds.com/blog/multiparty-threshold-ecdsa.html)

[gg18.pdf](/门限签名方案2——TSS(threshold-signature-sheme)/gg18.pdf)<br/>
[gg20.pdf](/门限签名方案2——TSS(threshold-signature-sheme)/gg20.pdf)

### TSS过程

- ### 密钥生成

```text
假设现在有个T-N等于2-3的tss方案，参与者为a,b,c;
1、a,b,c分别各选择一个随机数为自己的秘密值,如，Sa = 1 ,Sb = 2, Sc = 3，
整体的秘密（私钥d）=Sa + Sb +Sc = 1 + 2 + 3 =6;
2、用各自选择的秘密值，再随机各生成一个(T-1)次的多项式，如：
Fn_a = x + 1
Fn_b = 2x + 2
Fn_c = 3x + 3
3、将各自的秘密值分享给其他参与者
假设a的id为1，b的id为2，c的id为3，将各自的id分享给参与者
a，b，c使用自己的多项式方程将其他人的id带入其中
a:  
    Fn_a(1) = 2 #a自己留存
    Fn_a(2) = 3 #发送给b
    Fn_a(3) = 4 #发送给c
    
b:  
    Fn_b(1) = 4 #发送给a
    Fn_b(2) = 6 #b自己留存
    Fn_b(3) = 8 #发送给c
c:  
    Fn_c(1) = 6  #发送给a
    Fn_c(2) = 9  #发送给b
    Fn_c(3) = 12 #c自己留存
4、a，b，c分别计算他们的Xi
X_a = Fn_a(1) + Fn_b(1) + Fn_c(1) = 2 + 4 + 6 = 12  #a计算并保存
X_b = Fn_a(2) + Fn_b(2) + Fn_c(2) = 3 + 6 + 9 = 18  #b计算并保存
X_c = Fn_a(3) + Fn_b(3) + Fn_c(3) = 4 + 8 + 12 = 24 #c计算并保存

至此，私钥"6"已经通过2-3的方式分散到了a,b,c三个参与者中，也就是说，(1,12),(2,18),(3,24)这三个点，
其中任意两个点都可以通过拉格朗日插值算法，得到x=0处的值就是秘密值"6"。
此时a的碎片值就是"12"
此时b的碎片值就是"18"
此时c的碎片值就是"24"
```

![](/门限签名方案2——TSS(threshold-signature-sheme)/拉格朗日插值.png)

#### 那么公钥如何产生呢？

```text
  a,b,c分别计算Sa*G、Sb*G、Sc*G然后分享给其他人（不会泄漏各自的秘密值），
最终每个人都能得到Sa*G、Sb*G、Sc*G。
  然后计算Sa*G+Sb*G+Sc*G=(Sa+Sb+Sc)*G=d*G=Q,就能得到公钥了。

```

- ### 签名

#### 思路

##### 求r

```text
现在a，b要进行签名
R(x,y)=k^-1 * G (mod n),r=x。
注: 1、此处k^-1表示为k对于n（椭圆曲线的阶）的取模逆元，即找到一个数z，令k*z ≡ 1 (mod n) ,k^-1 = z
    2、此处的k的含义跟普通ECDSA签名算法里面的k并不是一个概念，但是能满足签名和验签公式，因为满足k*k^-1=1
引入一个变量γ，γ=∑γ_i
R = k^-1 * γ * γ^-1 * G = (kγ)^-1 * γG = (kγ)^-1 * (γ_a +γ_b)G = (kγ)^-1 * (γ_a*G + γ_b*G)
每个参与者单独计算γi*G,这个可以公开，因为γi*G反推不出γi。
然后就是想个办法计算σ=k*γ了
```

##### 求s

```text
s = k(m+dr)
  =(k_a + k_b)m + kdr
  =(k_a + k_b)m + ∂*r
  =(k_a + k_b)m + (∂_a + ∂_b)*r
  =(k_a*m + ∂_a*r) + (k_b*m + ∂_b*r)
  
引入变量∂，令k*d=∂=∑∂_i
```

#### 签名步骤

```text
由上面密钥生成的步骤可得，a的碎片x_a为12，b的碎片x_b为18。
它们是私钥"6"的(2,3)Feldman-VSS分享方案。
我们先它变为整体秘密"6"的2人additive sharing分享方案。
即要满足 d_a + d_b = 6  ，通过拉格朗日系数相乘方法可以求得：
d_a = µ_a * x_a = [(0-2)/(1-2)] * x_a = 2 * x_a = 24
d_b = µ_b * x_b = [(0-1)/(2-1)] * x_b = -1 * x_b = -18

显然满足d_a + d_b = 24 - 18 = 6

1、a，b随机选择k_i、γ_i,如：
k_a,k_b
γ_a,γ_b
Ps：k_i,γ_i自始自终都不会直接分享出去

定义k = k_a + k_b 
定义γ = γ_a + γ_b

2、每两个人都进行两次MtA(Multiplicative-to-Additive)协议
2.1、第一次MtA协议，Pi,Pj对k_i,γ_j使用MtA(i!=j)，
把Pi的结果记为α_ij，Pj得到的结果记为ß_ij,也就是满足：
k_i*γ_j = α_ij + ß_ij
例如：
k_a * γ_b = α_{ab} + ß_{ab}
k_b * γ_a = α_{ba} + ß_{ba}
此时a手中可知的数字：k_a、γ_a、α_{ab}、ß_{ba}
此时b手中可知的数字：k_b、γ_b、ß_{ab}、α_{ba}

定义 σi = k_i*γ_i + ∑α_ij + ∑ß_ji,则：
σ_a = k_a*Ya + α_{ab} + ß_{ba}
σ_b = k_b*Yb + α_{ba} + ß_{ab}

可以推出k*γ = (∑k_i) * (∑γ_i) = ∑σi = σ，推导过程：
k*γ = (k_a + k_b) * (Ya + Yb)
    = k_a*Ya + k_a*Yb + k_b*Ya + k_b*Yb  
    = k_a*Ya + α_{ab} + ß_{ab} + α_{ba} + ß_{ba} + k_b*Yb  
    = σ_a + σ_b
    = ∑σ_i
    
2.2、第二次MtA协议，Pi,Pj对k_i,d_j使用MtA(i!=j)，
把Pi的结果记为µ_ij，Pj得到的结果记为v_ij,也就是满足：
k_i*d_j = µ_ij + v_ij
例如：
k_a * d_b = µ_{ab} + v_{ab}
k_b * d_a = µ_{ba} + v_{ba}
此时a手中可知的数字：k_a、d_a、µ_{ab}、v_{ba}
此时b手中可知的数字：k_b、γ_b、v_{ab}、µ_{ba}

定义 ∂_i = k_i*d_i + ∑µ_ij + ∑v_ji,则：
∂_a = k_a*Wa + µ_{ab} + v_{ba}
∂_b = k_b*Wb + µ_{ba} + v_{ab}


可以推出k*d = (∑k_i) * (∑d_i) = ∑∂_i = ∂，推导过程：同2.1

3、a/b公开σ_i，这样所有参与者都能计算出k*γ=σ。

4、a/b公开γ_i*G,通过上面的思路，这样就求得r了。

5、在第二次MtA协议后，a/b各自计算出了∂_i,从而得到s_i=k_i*m + ∂_i*r,于是求得s=∑s_i了。


```

### 一些知识点

#### paillier同态加密

引用 [https://www.cnblogs.com/sssssaylf/p/12398133.html](https://www.cnblogs.com/sssssaylf/p/12398133.html)

##### 算法描述:

![](/门限签名方案2——TSS(threshold-signature-sheme)/paillier同态加密.png)

###### 重要作用(加法同态)

```text
Encrypt(m1+m2) = Encrypt(m1) * Encrypt(m2)
```

##### MtA(Multiplicative-to-Additive)协议

假设Alice有个秘密值a，Bob有个秘密值b，则可以在Alice不泄漏a、Bob不泄露b的情况下 ，Alice、Bob分别得到另一个秘密值α、ß，满足： ab = α + ß 具体过程如下

![](/门限签名方案2——TSS(threshold-signature-sheme)/MtA协议.png)