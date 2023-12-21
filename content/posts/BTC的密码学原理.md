+++
title = 'BTC的密码学原理'
date = 2021-08-28 18:00:00
draft = false
+++

# 1 非对称加密

## 1.1 简介

      非对称密码体制也叫公钥加密技术，该技术是针对私钥密码体制(对称加密算法)的缺陷被提出来的。
    在公钥加密系统中，加密和解密是相对独立的，加密和解密会使用两把不同的密钥，加密密钥(公开密钥)向公众公开，
    谁都可以使用，解密密钥(秘密密钥)只有解密人自己知道，非法使用者根据公开的加密密钥无法推算出解密密钥。
      btc正是使用非对称加密解决资产的所有权。

## 1.2 RSA加密

#### ps：不是重点，只是解释非对称加密是什么原理。

    比较典型的非对称加密，原理是大质数的分解。
    具体描述：
    任意选取两个不同的大素数p和q。为方便计算，这里取p=17,q=19。
    1、得到p和q的乘积N=17*19=323。
    2、得到(p-1)=16和(q-1)=18的最小公倍数为L=144。
    3、任意选取一个大整数E，满足E与L的最大公约数为1，此时E=5时满足条件，此时E即为公钥。
    4、找一个数D，使D满足1<D<L,且(E*D) mod L=1,如D=29时满足,(5*29)%144 = 145%144=1，此时D即为私钥。


    加解密过程：
    公钥E=5，私钥D=29, N(公开整数) =323。
    设明文m为：123
    加密过程：密文c=m^E mod N = 123^5 % 323 = 28153056843 % 323 = 225
    解密过程：明文m=c^D mod N = 225^29 % 323 = 123
    
    程序验证过程：

![](/BTC的密码学原理/rsa验证.png)

## 1.3 椭圆曲线加密

      随着计算机性能的提升，市面上的加密技术越来越不安全，1024位的RSA私钥加密已经可以破解，目前有效的手段只是
    将1024位换成2048位，但随着技术的进步，RSA算法的破解难度会越来越低，因此需要用更安全的加密算法来代替。
      椭圆曲线加密算法(Elliptic Curve Cryptography)，简称ECC，是基于椭圆曲线数学理论实现的一种非对称加密算法。
    相比RSA，ECC优势是可以使用更短的密钥，来实现与RSA相当或更高的安全。

### 椭圆曲线定义：

[参考](https://zhuanlan.zhihu.com/p/42629724)

    一条椭圆曲线就是一组被 y^2 = x^3 + ax + b 定义的且满足 4a^3 + 27b^2 ≠ 0 的点集。
    4a^3 + 27b^2 ≠ 0 这个限定条件是为了保证曲线不包含奇点(在数学中是指曲线上任意一点都存在切线) 。
    BTC使用了secp256k1这条椭圆曲线：
    y^2 = x^3 + 7
    ps：椭圆曲线的形状，并不是椭圆的。只是因为椭圆曲线的描述方程，类似于计算一个椭圆周长的方程故得名。

![](/BTC的密码学原理/secp256k1.jpeg )

    椭圆曲线加法：
      过曲线上的两点A、B画一条直线，找到直线与椭圆曲线的交点，交点关于x轴对称位置的点，定义为A+B，
    即为加法。如下图所示：A + B = C

![](/BTC的密码学原理/ecc加法.png)

    椭圆曲线的二倍运算：
    在A点做切线，与椭圆曲线的交点，交点关于x轴对称位置的点，定义为A + A，即2A，即为二倍运算。

![](/BTC的密码学原理/ecc乘法.png)

    补充：
      以上y^2 = x^3 + 7 这个方程是而为平面中的一条光滑的曲线，x,y的取值为实数域。但是在密码学中并非使用
    实数域，而是整数，一般是取一质数p为模，所有操作都以模p的形式执行，p即为质数域的阶，最后实际上用到的方程为：
        y^2 mod n = (x^3 + 7) mod n
    此时，椭圆曲线不再是一条光滑曲线，而是一些不连续的点(依然满足椭圆曲线的运算规则) 。

# 2 私钥和公钥：

      私钥是一个随机的大数字，取值范围为1 ~ 2^256，一般是通过生成随机字节，再对其使用SHA256算法产生。
    假设私钥为d，G为基点，则公钥Q=dG。椭圆曲线上的已知G和dG，求d是非常困难的，也就是说，已知公钥和基点，
    想要算出私钥是非常困难的。
    
    公钥加密：选择随机数r，将明文消息编码到曲线上得到点M，然后生成密文C，该密文是一个点对，C = {rG, M+rQ}，其中Q为公钥。
    私钥解密：M + rQ - d(rG) = M + r(dG) - d(rG) = M，其中d、Q分别为私钥、公钥。
    
    签名算法原理：
    私钥签名：
      选择随机数k，使用倍加算法计算kG得到点(x, y)。其中签名的r就是x。
      得到要签名的消息M进行hash得到m
      s =  (m + dr) * k^-1 (mod n)
      此处的k^-1表示k对于n的模逆元（以下同理），如(k*y) ≡ 1 (mod n) ,此时y就称为k的模逆元，记作k^-1,可知y = (i*n + 1)/k(i为任意正整数)
      即得到签名的{r,s}
    公钥验证签名：
      接收方得到消息M进行hash得到m、以及签名{r, s}。
      使用发送方公钥Q计算：mG*s^-1 + rQ*s^-1，并与kG比较，如相等即验签成功。
      原理：mG*s^-1 + rQ*s^-1 (mod n)
        = mG*s^-1 + r(dG)*s^-1 
        = (m+rd)G*s^-1 
        = (m+rd)G * ((i*n + 1)/s) 
        = (m+rd)G * ((i*n + 1) / (m + dr) * k^-1) 
        = G * ((i*n + 1)/k^-1)
        = kG
      kG为一个点(x,y)，如果x = r，表示验证通过了。
      注：
      ((i*n + 1)/k^-1)=k，相当于(k^-1)^-1=k

    特别注意：每笔交易使用的随机数必须不一样，不然私钥就会被泄漏了，以下为推导过程。
      s1 = (m1 + dr) * k^-1 （1）
      s2 = (m2 + dr) * k^-1 （2）

     （1）-（2）得：
      s1 - s2 = (m1 - m2) * k^-1
      k^-1 = (m1 - m2) / (s1 - s2)

      将k^-1的值带入（1）
      d = (s1/k^-1 - m1)/r (s1、k^-1、m1，都可以得出) 

# 3 根据签名算公钥

      假设签名为{x,s}，m为交易的摘要，点R(x,y)为签名时的随机数k*G得到的点，由于点R在曲线上，所以我们可以根据R的
    横坐标x来计算y的值（但是就像压缩公钥的情况一下，此时，y会有两个值，所以有些币（如eth）在签名时还需要带上一个标识v，
    代表y的值为正或负，因为eth的txhex里面是没有包含from地址的，通过这样可以得到公钥，然后算出from地址）
    根据签名的公式，s = (m+dx)*k^-1 (mod n)
    (s*R)*x^-1 - (m*G)*x^-1 = [((m+dx)*k^-1)*R - mG]*x^-1
                      =  [((m+dx)*k^-1)*(kG) - mG]*x^-1
                      =  [((m+dx)*G) - mG]*x^-1
                      =  (mG+dxG-mG)*x^-1
                      =  (dGx)*x^-1
                      =  dG
                      =  Q

## 倍加运算

    疑惑：既然私钥为一个很大的数(1 ~ 2^256)，那从基点通过私钥生成公钥不也要计算很多次吗？
    解答：倍加法，我们可以将一个大数转换成二进制格式,如3600转为二进制格式为：
    111000010000，又可以写为：2^11 + 2^10 + 2^9 + 2^4，此时，可以从基点开始一直进行二倍运算，
    并记录其中2^4, 2^9, 2^10, 2^11的点，将这几个点进行加法运算就得到公钥的点了，算法复杂度为私钥d的比特数
    的多项式长度。
    
    G点在椭圆曲线上k次乘积示意图：

![](/BTC的密码学原理/ecc乘积.png)

    感悟：之前对时间复杂度为O(logN)的算法没什么体会，看到上面的这个对比，才知道这种算法竟然能快这么多。

## 公钥的压缩与非压缩

      由于公钥是在曲线上点，那么，通过曲线方程"y^2 mod n = (x^3 + 7) mod n"是可以通过x推出y的值的，
    所以，在构建交易的时候，没有必要在把y值也附带进来增加体积，就只需提供x值作为压缩公钥；非压缩则是把x，y值都附带进来(标识为0x4) 。
      但是由于y值的解是一个平方根，有两个解，因此需要增加一个标识，0x2(偶数) 代表y值为正数，0x3(奇数) 代表y值
    为负数，以此确认了公钥的点。(关于奇偶性与正负性的关联，还没有找到答案) 

[参考](https://wumansgy.github.io/2018/10/30/%E6%A4%AD%E5%9C%86%E6%9B%B2%E7%BA%BF%E5%8A%A0%E5%AF%86%E7%AE%97%E6%B3%95%E8%AF%A6%E8%A7%A3/)

## 公钥生成地址

    流程：设公钥的坐标为:(x,y)，将x，y分别转为16进制格式(原先是2^256) ，得到2个32位的字节数组
    非压缩：前缀0x4+x+y拼接成一个65位的字节数组
    压缩：前缀0x2或0x3+x拼接成一个33位的字节数组
    
    设拼接后的字节数组为hexstr
    1. 计算`hash20` = ripemd160(sha256(publicKey))
    2. 定义`netID` （即`PubKeyHashAddrID`，正式网`0x00`，测试网是`0x6f`）
    3. 计算`checkSum` = sha256(sha256(`netID`+`hash20`))[:4]
    4. 计算`address` = base58(`netID`+`hash20`+`checkSum`)    


    注：这是传统形式的地址，被称为p2pkh(pay-to-public-key-hash)地址

### 私钥的wif格式

    非压缩：
    checksum = SHA256(SHA256(PrivateKeyID+data))[0:4]
    base58.Encode(PrivateKeyID+私钥+checksum)
    压缩：
    checksum = SHA256(SHA256(PrivateKeyID+data+0x01))[0:4]
    base58.Encode(PrivateKeyID+私钥+checksum)
    
    注意：这里的压缩并非真正的压缩，只是一个标识，表示推荐被用来生成压缩的公钥。

![](/BTC的密码学原理/btc参数.png)

# 3 地址

[参考](https://www.notion.so/BTC-a7b675c4e98d47ed960109bbe94372e4)

## 地址的本质

      传统的比特币地址都是以数字1开头，谁能提供公钥和对应私钥的签名，就能花费这笔BTC。
      实际上比特币的地址是一个加锁脚本，任何人能够构建对应的解锁脚本(SigScript) ，就能指定其中BTC的去向
    (out指定下一个加锁脚本) 。1开头地址对应的加锁脚本为如下格式：

![](/BTC的密码学原理/p2pkh地址hex.png)

转化为脚本语言后：

![](/BTC的密码学原理/p2pkh地址asm.png)

[以上交易示例链接](https://www.blockchain.com/btc/tx/2d07df466e1a442a4636e8828e70fe4ce6a2ed80184f71d6f3d0df053131f593)

## 脚本

### 介绍

      比特币的脚本是一种基于栈执行的逆波兰表达式语言，放置在UTXO上的加锁脚本和解锁脚本都是用这种脚本语言编写的。
    它是一种非常简单的语言，只能做很少的操作，并不能完成许多现代编程语言能够做的事情，但其实选择这种语言是一种深思
    熟虑的安全设计。其包含了许多操作码，但是故意限制了一个重要方面————除了有条件的流控制外，没有循环或复杂流程控制
    能力。这使得脚本语言是非图灵完备的，这意味着脚本复杂性和执行次数都是有限的，避免了有人故意使用无限循环或其他形
    式的"逻辑炸弹"，造成比特币网络的拒绝服务攻击。

### 示例

    比如现在有个加锁脚本为:
    2 
    ADD 
    5
    EQUAL
    
    意思是：你需要提供一个解锁脚本，将脚本上的数据依次入栈，并执行加锁脚本里的操作符，如果最终返回了true，则证明
    该解锁脚本是正确的，且该笔交易是有效的
    
    上述加锁脚本翻译一下就是：x + 2 = 5,你现在需要提供一个x的值，才能解锁该脚本，并花费这笔钱。

### 交易示例的脚本分析

加锁脚本

```
OP_DUP
OP_HASH160
423c4feb3458a72efc6fc40815ce43d134e798d4
OP_EQUALVERIFY
OP_CHECKSIG
```

解锁脚本

```
(签名)3045022100f00ed74401029ec4b8166943f74bd0608ac23ccfd1353f02ed250f75f920606402207bdd1a249308b334c5b26404eac4ad084fc6588748b6ee751926ff64388a9b3801
(公钥)03ec1569bf452dcf9a05f538f1c979d11789864c4c3e4eace3c8eb3a5f559171ce
```

验证流程：

    1、拼接解锁脚本和加锁脚本，得到如下：
    3045022100f00ed74401029ec4b8166943f74bd0608ac23ccfd1353f02ed250f75f920606402207bdd1a249308b334c5b26404eac4ad084fc6588748b6ee751926ff64388a9b3801
    03ec1569bf452dcf9a05f538f1c979d11789864c4c3e4eace3c8eb3a5f559171ce
    OP_DUP
    OP_HASH160
    423c4feb3458a72efc6fc40815ce43d134e798d4
    OP_EQUALVERIFY
    OP_CHECKSIG
    
    2、执行栈过程
    2.1 
      开始将脚本入栈
    栈：
    (栈底) 3045022100f00ed74401029ec4b8166943f74bd0608ac23ccfd1353f02ed250f75f920606402207bdd1a249308b334c5b26404eac4ad084fc6588748b6ee751926ff64388a9b3801    
    脚本：
    03ec1569bf452dcf9a05f538f1c979d11789864c4c3e4eace3c8eb3a5f559171ce
    OP_DUP
    OP_HASH160
    423c4feb3458a72efc6fc40815ce43d134e798d4
    OP_EQUALVERIFY
    OP_CHECKSIG

    2.2
      由于没有操作符号，继续入栈

    栈：
    (栈顶) 03ec1569bf452dcf9a05f538f1c979d11789864c4c3e4eace3c8eb3a5f559171ce
    (栈底) 3045022100f00ed74401029ec4b8166943f74bd0608ac23ccfd1353f02ed250f75f920606402207bdd1a249308b334c5b26404eac4ad084fc6588748b6ee751926ff64388a9b3801    
    脚本：
    OP_DUP
    OP_HASH160
    423c4feb3458a72efc6fc40815ce43d134e798d4
    OP_EQUALVERIFY
    OP_CHECKSIG

    2.3
      遇到操作符"OP_DUP"，表示复制

    栈：
    (栈顶) 03ec1569bf452dcf9a05f538f1c979d11789864c4c3e4eace3c8eb3a5f559171ce
           03ec1569bf452dcf9a05f538f1c979d11789864c4c3e4eace3c8eb3a5f559171ce
    (栈底) 3045022100f00ed74401029ec4b8166943f74bd0608ac23ccfd1353f02ed250f75f920606402207bdd1a249308b334c5b26404eac4ad084fc6588748b6ee751926ff64388a9b3801    
    脚本：
    OP_HASH160
    423c4feb3458a72efc6fc40815ce43d134e798d4
    OP_EQUALVERIFY
    OP_CHECKSIG

    2.4
      遇到操作符"OP_HASH160"，表示对栈顶元素进行hash160操作，得到结果：423c4feb3458a72efc6fc40815ce43d134e798d4

    栈：
    (栈顶) 423c4feb3458a72efc6fc40815ce43d134e798d4
           03ec1569bf452dcf9a05f538f1c979d11789864c4c3e4eace3c8eb3a5f559171ce
    (栈底) 3045022100f00ed74401029ec4b8166943f74bd0608ac23ccfd1353f02ed250f75f920606402207bdd1a249308b334c5b26404eac4ad084fc6588748b6ee751926ff64388a9b3801    
    脚本：
    423c4feb3458a72efc6fc40815ce43d134e798d4
    OP_EQUALVERIFY
    OP_CHECKSIG

    2.5
      没有操作符，继续入栈

    栈：
    (栈顶) 423c4feb3458a72efc6fc40815ce43d134e798d4
           423c4feb3458a72efc6fc40815ce43d134e798d4
           03ec1569bf452dcf9a05f538f1c979d11789864c4c3e4eace3c8eb3a5f559171ce
    (栈底) 3045022100f00ed74401029ec4b8166943f74bd0608ac23ccfd1353f02ed250f75f920606402207bdd1a249308b334c5b26404eac4ad084fc6588748b6ee751926ff64388a9b3801    
    脚本：
    OP_EQUALVERIFY
    OP_CHECKSIG

    2.6
      遇到操作符"OP_EQUALVERIFY"，表示比较栈顶的两个元素是否相等，相等则丢弃栈顶的2个元素并继续后面的操作，
    不相等则直接返回false，表示验证失败

    栈：
    (栈顶--公钥） 03ec1569bf452dcf9a05f538f1c979d11789864c4c3e4eace3c8eb3a5f559171ce 
    (栈底--签名) 3045022100f00ed74401029ec4b8166943f74bd0608ac23ccfd1353f02ed250f75f920606402207bdd1a249308b334c5b26404eac4ad084fc6588748b6ee751926ff64388a9b3801    
    脚本：
    OP_CHECKSIG

    2.7
      遇到操作符"OP_CHECKSIG"，表示校验签名 ，应用上面的校验流程，如果校验通过，则返回true，否则返回false。

## P2SH地址（脚本地址）

      传统的比特币地址(P2PKH)都是从数字1开头并且跟公钥直接关联，且解锁脚本的格式都是固定的：提供公钥的hash+签名。
      然而一笔交易里的out地址总是接收方提供的，接收方应该可以自定义加锁脚本，于是出现了P2SH地址，该类地址通常是数字3开头，
    比如比较常见的多签地址，就是定义了需要多个地址提供签名才能花费这笔out。

      生成方式：
        1. 计算`hash20` = ripemd160(sha256(script))
        2. 定义`netID` （即`ScriptHashAddrID`，正式网`0x05`，测试网是`0xc4`）
        3. 计算`checkSum` = sha256(sha256(`netID`+`hash20`))[:4]
        4. 计算`address` = base58(`netID`+`hash20`+`checkSum`)   

    tip：之所以所有的P2PKH地址都以数字1开头，所有的P2SH地址都以数字3开头，就是因为每类地址netID都是固定了，执行base58的结果也是固定的。

    Signature script: OP_0 <A sig> <C sig> <redeemScript>
    多签地址生成方式：
    1、选择n-m的数值，如2-3
    2、选择参与签名的地址的公钥,
    3、开始构建：
    52                                                                   //op_2 => n
	210377c59d4aeac11c225aca60417b905be0d328645fe8c8bfad6088c70778d2abe5 //参与多签的地址的公钥
	2102e0ec0904b91391b9123665a56b642d8b7b1e80e94c8a3b93c81a8492762b970b //参与多签的地址的公钥
	2103135369a26f1cb6f1cc7dd2012099bdb7ca3176da721cf59973b07a6774c409cf //参与多签的地址的公钥
	53                                                                   //op_3 => m
	ae                                                                   //OP_CHECKMULTISIG
    4、按照上面的生成方式生成地址。

    解锁脚本格式：
      OP_0 //固定值
      30450221009e265edee2aff34e2c3da0637f750ef032cf86a4d1a3106703f7ef613d084087022051138e34c9067deb996a31191295856ceaba5a0457ddfeb7b0e16dafda5ae88001 //参与多签的地址的公钥的私钥的签名
      304402207b4749485e676d5ff2236f68a1db5e9c26ceb71d1343643a511eadecc2dce03802205195a1203c72e11c47fd672d2dc23374d0e866e019feef45a7c7915c4e5e083501   //参与多签的地址的公钥的私钥的签名

      53210377c59d4aeac11c225aca60417b905be0d328645fe8c8bfad6088c70778d2abe52102e0ec0904b91391b9123665a56b642d8b7b1e80e94c8a3b93c81a8492762b970b210313 //上面的地址的脚本
    5369a26f1cb6f1cc7dd2012099bdb7ca3176da721cf59973b07a6774c409cf53ae                                                                                 //

# 4、交易

## 交易的结构

```json
{
  "txid": "efea9b5db2e2829b0ca99fe0c2816b356505a44324668e283402dea5663fb957",
  "hash": "efea9b5db2e2829b0ca99fe0c2816b356505a44324668e283402dea5663fb957",
  "version": 1,
  "size": 374,
  "vsize": 374,
  "weight": 1496,
  "locktime": 0,
  "vin": [
    {
      "txid": "d48cb977532b162ed0d365b3718ece25c1bc5c8b974a66ae84a5cbad50d3a649",
      "vout": 0,
      "scriptSig": {
        "asm": "3045022100ad30b3cf28b1ae9b051de41ee36990b6e9cb55c14ffdfba91192616bbed7d7eb02204c72026f50c7a6dc26bce4c6415a777309cc9f6c82f8c3ec4a526846ae02de45[ALL] 02e78ac7ecee5a3a84d6d7d23c342e60c6408eddbd83ad12d3eefc50c0b7c6657f",
        "hex": "483045022100ad30b3cf28b1ae9b051de41ee36990b6e9cb55c14ffdfba91192616bbed7d7eb02204c72026f50c7a6dc26bce4c6415a777309cc9f6c82f8c3ec4a526846ae02de45012102e78ac7ecee5a3a84d6d7d23c342e60c6408eddbd83ad12d3eefc50c0b7c6657f"
      },
      "sequence": 4294967295
    },
    {
      "txid": "1bac194abf8163e91825a06e7b22edcc410ab61f1838ed8c9f43aa1c4e12b76b",
      "vout": 1,
      "scriptSig": {
        "asm": "3045022100b17a39b6790590c057033442928f0e332e7d78f03f27f130f945f703d14d0658022057bba7d73692ead696ffe9070f0ed920cac523ba6b3c8f1e3571c4f89c0d2d04[ALL] 02e78ac7ecee5a3a84d6d7d23c342e60c6408eddbd83ad12d3eefc50c0b7c6657f",
        "hex": "483045022100b17a39b6790590c057033442928f0e332e7d78f03f27f130f945f703d14d0658022057bba7d73692ead696ffe9070f0ed920cac523ba6b3c8f1e3571c4f89c0d2d04012102e78ac7ecee5a3a84d6d7d23c342e60c6408eddbd83ad12d3eefc50c0b7c6657f"
      },
      "sequence": 4294967295
    }
  ],
  "vout": [
    {
      "value": 0.10692296,
      "n": 0,
      "scriptPubKey": {
        "asm": "OP_DUP OP_HASH160 c713030723df76ab3b722191d580fa31a8073892 OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "76a914c713030723df76ab3b722191d580fa31a807389288ac",
        "reqSigs": 1,
        "type": "pubkeyhash",
        "addresses": [
          "1K9cF95Gczk3bMDrPxW9qnuXhGqGMRVFZ4"
        ]
      }
    },
    {
      "value": 0.00283658,
      "n": 1,
      "scriptPubKey": {
        "asm": "OP_DUP OP_HASH160 7c63f5e7069bb7024ed82e65a3828a8a0c7b1214 OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "76a9147c63f5e7069bb7024ed82e65a3828a8a0c7b121488ac",
        "reqSigs": 1,
        "type": "pubkeyhash",
        "addresses": [
          "1CLiYAkoVWewctwiXMsHuZWf8T6YdpNCo5"
        ]
      }
    }
  ],
  "blockhash": "0000000000000000000c61959dd2676e8cb21eee5a00b3b81a2a764d7bfd1de8",
  "hex": "010000000249a6d350adcba584ae664a978b5cbcc125ce8e71b365d3d02e162b5377b98cd4000000006b483045022100ad30b3cf28b1ae9b051de41ee36990b6e9cb55c14ffdfba91192616bbed7d7eb02204c72026f50c7a6dc26bce4c6415a777309cc9f6c82f8c3ec4a526846ae02de45012102e78ac7ecee5a3a84d6d7d23c342e60c6408eddbd83ad12d3eefc50c0b7c6657fffffffff6bb7124e1caa439f8ced38181fb60a41cced227b6ea02518e96381bf4a19ac1b010000006b483045022100b17a39b6790590c057033442928f0e332e7d78f03f27f130f945f703d14d0658022057bba7d73692ead696ffe9070f0ed920cac523ba6b3c8f1e3571c4f89c0d2d04012102e78ac7ecee5a3a84d6d7d23c342e60c6408eddbd83ad12d3eefc50c0b7c6657fffffffff02c826a300000000001976a914c713030723df76ab3b722191d580fa31a807389288ac0a540400000000001976a9147c63f5e7069bb7024ed82e65a3828a8a0c7b121488ac00000000"
}
```

## hex如何转化为结构化的tx

    010000000249a6d350adcba584ae664a978b5cbcc125ce8e71b365d3d02e162b5377b98cd4000000006b483045022100ad30b3cf28b1ae9b051de41ee36990b6e9cb55c14ffdfba91192616bbed7d7eb02204c72026f50c7a6dc26bce4c6415a777309cc9f6c82f8c3ec4a526846ae02de45012102e78ac7ecee5a3a84d6d7d23c342e60c6408eddbd83ad12d3eefc50c0b7c6657fffffffff6bb7124e1caa439f8ced38181fb60a41cced227b6ea02518e96381bf4a19ac1b010000006b483045022100b17a39b6790590c057033442928f0e332e7d78f03f27f130f945f703d14d0658022057bba7d73692ead696ffe9070f0ed920cac523ba6b3c8f1e3571c4f89c0d2d04012102e78ac7ecee5a3a84d6d7d23c342e60c6408eddbd83ad12d3eefc50c0b7c6657fffffffff02c826a300000000001976a914c713030723df76ab3b722191d580fa31a807389288ac0a540400000000001976a9147c63f5e7069bb7024ed82e65a3828a8a0c7b121488ac00000000

    01000000                                                                 //版本号(小端序),原来的值00000002

    02                                                                       //代表几个in

    49a6d350adcba584ae664a978b5cbcc125ce8e71b365d3d02e162b5377b98cd4         //上一笔交易的hashid(小端序)，原来的值：d48cb977532b162ed0d365b3718ece25c1bc5c8b974a66ae84a5cbad50d3a649
    00000000                                                                 //上一笔交易的vout(小端序)
    6b                                                                       //解锁脚本长度：107
    48                                                                       //签名脚本长度：72
    3045022100ad30b3cf28b1ae9b051de41ee36990b6e9cb55c14ffdfba91192616bbed7d7eb02204c72026f50c7a6dc26bce4c6415a777309cc9f6c82f8c3ec4a526846ae02de4501 //签名
    21                                                                       //公钥的hash的长度：33
    02e78ac7ecee5a3a84d6d7d23c342e60c6408eddbd83ad12d3eefc50c0b7c6657f       //公钥的hash
    ffffffff                                                                 //序列号    
    6bb7124e1caa439f8ced38181fb60a41cced227b6ea02518e96381bf4a19ac1b         //上一笔交易的hashid(小端序)，原来的值：1bac194abf8163e91825a06e7b22edcc410ab61f1838ed8c9f43aa1c4e12b76b
    01000000                                                                 //上一笔交易的vout(小端序)
    6b                                                                       //解锁脚本长度：107
    48                                                                       //签名脚本长度：72
    3045022100b17a39b6790590c057033442928f0e332e7d78f03f27f130f945f703d14d0658022057bba7d73692ead696ffe9070f0ed920cac523ba6b3c8f1e3571c4f89c0d2d0401 //签名
    21                                                                       //公钥的hash的长度：33
    02e78ac7ecee5a3a84d6d7d23c342e60c6408eddbd83ad12d3eefc50c0b7c6657f       //公钥的hash
    ffffffff                                                                 //序列号

    02                                                                       //vout的数量

    c826a30000000000                                                         //vout0的value(satoshi,小端序), 0000000000a326c8=>10692296
    19                                                                       //加锁脚本长度：25
    76a914c713030723df76ab3b722191d580fa31a807389288ac                       //加锁脚本
    0a54040000000000                                                         //vout0的value(satoshi,小端序), 000000000004540a=>283658
    19                                                                       //加锁脚本长度：25
    76a9147c63f5e7069bb7024ed82e65a3828a8a0c7b121488ac                       //加锁脚本
    00000000                                                                 //locktime(小端序)

[官网说明](https://en.bitcoin.it/wiki/Weight_units#Detailed_example)

## 摘要

    摘要的本质是将其他的vin的脚本设置为空，然后对这个（不完整的交易+签名类型小端序）进行两次hash256。
    如上(签名类型为SigHashAll)，对于vin0,它获得的摘要内容是这样的（未经过2次hash256）：
    {
        "vin": [
            {
              "txid": "d48cb977532b162ed0d365b3718ece25c1bc5c8b974a66ae84a5cbad50d3a649",
              "vout": 0,
              "scriptSig": {
                "asm": "3045022100ad30b3cf28b1ae9b051de41ee36990b6e9cb55c14ffdfba91192616bbed7d7eb02204c72026f50c7a6dc26bce4c6415a777309cc9f6c82f8c3ec4a526846ae02de45[ALL] 02e78ac7ecee5a3a84d6d7d23c342e60c6408eddbd83ad12d3eefc50c0b7c6657f",
                "hex": "483045022100ad30b3cf28b1ae9b051de41ee36990b6e9cb55c14ffdfba91192616bbed7d7eb02204c72026f50c7a6dc26bce4c6415a777309cc9f6c82f8c3ec4a526846ae02de45012102e78ac7ecee5a3a84d6d7d23c342e60c6408eddbd83ad12d3eefc50c0b7c6657f"
              },
              "sequence": 4294967295
            },
            {
              "txid": "1bac194abf8163e91825a06e7b22edcc410ab61f1838ed8c9f43aa1c4e12b76b",
              "vout": 1,
              "scriptSig": {
                "asm": "",
                "hex": ""
              },
              "sequence": 4294967295
            }
          ],
          "vout": [
            {
              "value": 0.10692296,
              "n": 0,
              "scriptPubKey": {
                "asm": "OP_DUP OP_HASH160 c713030723df76ab3b722191d580fa31a8073892 OP_EQUALVERIFY OP_CHECKSIG",
                "hex": "76a914c713030723df76ab3b722191d580fa31a807389288ac",
                "reqSigs": 1,
                "type": "pubkeyhash",
                "addresses": [
                  "1K9cF95Gczk3bMDrPxW9qnuXhGqGMRVFZ4"
                ]
              }
            },
            {
              "value": 0.00283658,
              "n": 1,
              "scriptPubKey": {
                "asm": "OP_DUP OP_HASH160 7c63f5e7069bb7024ed82e65a3828a8a0c7b1214 OP_EQUALVERIFY OP_CHECKSIG",
                "hex": "76a9147c63f5e7069bb7024ed82e65a3828a8a0c7b121488ac",
                "reqSigs": 1,
                "type": "pubkeyhash",
                "addresses": [
                  "1CLiYAkoVWewctwiXMsHuZWf8T6YdpNCo5"
                ]
              }
            }
          ]
    }
    
    对于vin1，它的摘要是这样的（未经过2次hash256）：
    {
        "vin": [
            {
              "txid": "d48cb977532b162ed0d365b3718ece25c1bc5c8b974a66ae84a5cbad50d3a649",
              "vout": 0,
              "scriptSig": {
                "asm": "",
                "hex": ""
              },
              "sequence": 4294967295
            },
            {
              "txid": "1bac194abf8163e91825a06e7b22edcc410ab61f1838ed8c9f43aa1c4e12b76b",
              "vout": 1,
              "scriptSig": {
                "asm": "3045022100b17a39b6790590c057033442928f0e332e7d78f03f27f130f945f703d14d0658022057bba7d73692ead696ffe9070f0ed920cac523ba6b3c8f1e3571c4f89c0d2d04[ALL] 02e78ac7ecee5a3a84d6d7d23c342e60c6408eddbd83ad12d3eefc50c0b7c6657f",
                "hex": "483045022100b17a39b6790590c057033442928f0e332e7d78f03f27f130f945f703d14d0658022057bba7d73692ead696ffe9070f0ed920cac523ba6b3c8f1e3571c4f89c0d2d04012102e78ac7ecee5a3a84d6d7d23c342e60c6408eddbd83ad12d3eefc50c0b7c6657f"
              },
              "sequence": 4294967295
            }
          ],
          "vout": [
            {
              "value": 0.10692296,
              "n": 0,
              "scriptPubKey": {
                "asm": "OP_DUP OP_HASH160 c713030723df76ab3b722191d580fa31a8073892 OP_EQUALVERIFY OP_CHECKSIG",
                "hex": "76a914c713030723df76ab3b722191d580fa31a807389288ac",
                "reqSigs": 1,
                "type": "pubkeyhash",
                "addresses": [
                  "1K9cF95Gczk3bMDrPxW9qnuXhGqGMRVFZ4"
                ]
              }
            },
            {
              "value": 0.00283658,
              "n": 1,
              "scriptPubKey": {
                "asm": "OP_DUP OP_HASH160 7c63f5e7069bb7024ed82e65a3828a8a0c7b1214 OP_EQUALVERIFY OP_CHECKSIG",
                "hex": "76a9147c63f5e7069bb7024ed82e65a3828a8a0c7b121488ac",
                "reqSigs": 1,
                "type": "pubkeyhash",
                "addresses": [
                  "1CLiYAkoVWewctwiXMsHuZWf8T6YdpNCo5"
                ]
              }
            }
          ]
    }
    
    由此可以看出，签名对于各个in之间是独立的，无须按照顺序来签名。因此可以使用并发签名的方式，加快签名速度（小强发现的）。

### 签名类型

    签名的类型主要是影响摘要的生成方式，具体实现为清除其他in或out的值，或者整体都去掉，大部分用的是ALl类型。

| SigHash标志    | 数值   | 说明                                                             |
|--------------|------|----------------------------------------------------------------|
| All          | 0x01 | 每个in都保留所有的out，并将其他in的scriptSig内容置为空                            |
| None         | 0x02 | 每个in都去掉所有的out，并将其他in的scriptSig内容置为空                            |
| Single       | 0x03 | 每个in按照序号抽取一个out，没有对应则直接返回0x01，并将其他in的scriptSig内容置为空            |
| AnyOneCanPay | 0x80 | 不能单独使用，与上面组合使用。原本其他的in的结构还保留，只是scriptSig内容为空，加上后直接去掉其他所有的in的结构 |

### 类BTC的币如何实现代币交易

通过在vout里面增加额外数据

#### BCH

```json
{
  "value": 0.00000000,
  "n": 0,
  "scriptPubKey": {
    "asm": "OP_RETURN 5262419 1 1145980243 959a6818cba5af8aba391d3f7649f5f6a5ceb6cdcd2c2a3dcb5d2fbfc4b08e98 000000004190ab00 0000d78df6bde280",
    "hex": "6a04534c500001010453454e4420959a6818cba5af8aba391d3f7649f5f6a5ceb6cdcd2c2a3dcb5d2fbfc4b08e9808000000004190ab00080000d78df6bde280",
    "type": "nulldata"
  }
}
```

    说明：
    "6a"                                                                //OP_RETURN，标记该输出是不可花费的，无解锁脚本。
    "04"                                                                //Length of lokad_id field (4 bytes)
    "534c5000"                                                          //hex2string:SLP
    "01"                                                                //length of token_type (1 byte)
    "01"                                                                //token_type (1)
    "04"                                                                //length of transaction_type field (4 bytes)
    "53454e44"                                                          //hex2string :SEND
    "20"                                                                //length of token_id (32 bytes)
    "959a6818cba5af8aba391d3f7649f5f6a5ceb6cdcd2c2a3dcb5d2fbfc4b08e98"  //token_id
    "08"                                                                //amount length
    "000000004190ab00"                                                  //to地址的amount
    "08"                                                                //amount length
    "0000d78df6bde280"                                                  //找零地址的amount

#### AOK

```json
{
  "value": 0.00000000,
  "n": 1,
  "scriptPubKey": {
    "asm": "OP_DUP OP_HASH160 54e8d1b954d13d22991045c3686e168f174e1d9a OP_EQUALVERIFY OP_CHECKSIG OP_TOKEN_SCRIPT 616c707403434341809698000000000000000000 OP_DROP",
    "hex": "76a91454e8d1b954d13d22991045c3686e168f174e1d9a88acc014616c70740343434180969800000000000000000075",
    "reqSigs": 1,
    "type": "transfer_token",
    "token": {
      "name": "CCA",
      "amount": 0.10000000,
      "token_lock_time": 0
    },
    "addresses": [
      "KeHhxg2Awmy1Takv9Sb8EjwX1dTfPNyNPA"
    ]
  },
  "valueSat": 0
}
```

    说明：
    76                                         //OP_DUP
	a9                                         //OP_HASH160
	14                                         //长度（20）
	54e8d1b954d13d22991045c3686e168f174e1d9a   //hash160(公钥)
	88                                         //OP_EQUALVERIFY
	ac                                         //OP_CHECKSIG
	c0                                         //OP_TOKEN_SCRIPT(标识后面的是脚本)
	14                                         //脚本长度（20）
	616c7074                                   //固定值(猜测是交易类型：transfer_token)
	03434341                                   //hextostring:CCA
	809698000000000000000000                   //token金额（小端序）：989680：10000000=>0.1*10^8
	75                                         //OP_DROP(代表清空当前堆栈的内容)